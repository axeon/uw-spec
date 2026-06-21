# uw-cache — 缓存管理

**Maven 坐标**: `com.umtone:uw-cache`

基于 Caffeine 和 Redis 的融合缓存类库，提供 cache / locker / counter / hashSet / sortedSet 五大组件。

**配置前缀**: `uw.cache.redis`

```yaml
uw:
  cache:
    redis:
      database: 9
      host: 192.168.88.21
      port: 6380
      password: redispasswd
      lettuce:
        pool:
          max-active: 100
          max-idle: 8
          max-wait: 5000ms
          min-idle: 1
      timeout: 30s
```

> 启动自动配置：`uw.cache.conf.UwCacheAutoConfiguration` 已声明 `@ConditionalOnClass(RedisTemplate)`，引入 starter-data-redis 即生效。它注册两个 RedisTemplate：`dataCacheRedisTemplate`（byte[] 值，用于 Cache/HashSet/SortedSet）与 `longCacheRedisTemplate`（Long 值，用于 Counter/Locker）。两个模板共享 `uw.cache.redis` 配置。**同一 JVM 仅支持一套 Redis 配置**（Global* 组件通过 static 字段持有 template，后注入覆盖先注入）。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 高频实体详情缓存 | `FusionCache`（本地+Redis融合） | 必须在 `static {}` 块中 config + CacheDataLoader |
| 临时数据/列表缓存 | `GlobalCache`（纯Redis） | 不需要 static 初始化，行内 CacheDataLoader |
| 分布式锁 | `GlobalLocker.tryLock/keepLock/unlock` | stamp > 0 才持有锁，finally 中 unlock |
| 高性能计数器 | `FusionCounter`（本地+Redis融合） | 必须 static config，支持回写数据库 |
| 纯Redis计数器 | `GlobalCounter` | increment/decrement/get/set/delete |
| 去重/随机抽取 | `GlobalHashSet` | add/remove/contains/random/pop |
| 延迟任务/定时触发 | `GlobalSortedSet` | 按score范围查询和删除 |
| 缓存失效（单key） | `FusionCache.invalidate(Class, key)` | 默认通知集群；传 `key=null` 可全量清空 |
| 缓存失效（GlobalCache） | `GlobalCache.invalidate(cacheName, key)` | 方法名是 invalidate 不是 delete |
| 按前缀批量失效 | `GlobalCache.invalidatePrefix(cacheName, keyPrefix)` / `invalidateKeys(Class, prefix)` | 内部走 SCAN，注意大 keyset 阻塞 |
| 缓存列表数据 | 缓存 `ArrayList<Entity>` | 禁止缓存 PageList / 接口类型（Kryo序列化异常） |
| 判断缓存是否存在 | FusionCache: `containsKey(Class, key)`；GlobalCache: `containsKey(cacheName, key)` | 不要找 `getIfPresent`/`exists`，本库没有 |

**CacheDataLoader 约束**：是抽象类不是函数式接口，**不能用 lambda**，必须 `new CacheDataLoader<K,V>(){...}`，且泛型 V 必须是**具体实现类**（Kryo 反序列化要求）。

**选型决策**：单条实体详情用 FusionCache；列表、临时数据用 GlobalCache。

**性能注意**：Caffeine 设过期时间后性能劣化 200 倍，建议 `cacheExpireMillis` 保持 -1（永久），仅靠 Redis TTL 兜底。Kryo 序列化必须使用具体实现类（ArrayList/LinkedHashMap/HashSet），不能用接口类型。

> 完整陷阱列表见 [dev-standards.md](dev-standards.md)「AI Coding 禁忌清单」。

## FusionCache（本地 Caffeine + 全局 Redis）

> **包路径**：`uw.cache.FusionCache`

二级复合缓存：本地 Caffeine 兜性能，Redis 兜全局一致性，通过 Redis Pub/Sub（通道 `UW_CACHE_NOTIFY_CHANNEL`）做多实例失效同步。

**初始化**：必须在 Helper 的 `static {}` 块中完成 `FusionCache.config()`。**重复 config 会覆盖旧实例并丢弃旧本地缓存**（仅打 warn 日志）。

| 方法 | 说明 |
|------|------|
| `config(Config, CacheDataLoader)` | 配置缓存+数据加载器（static块中调用，仅一次） |
| `put(Class, key, value)` | 存入数据 |
| `put(Class, key, value, expireMillis)` | 存入+过期时间 |
| `put(Class, key, value, expireMillis, onlyLocal)` | `onlyLocal=true` 仅写本地不写 Redis |
| `putAll(Class, Map)` | 批量存入 |
| `get(Class, key)` | 获取（带自动加载，过期自动重试重载） |
| `getValueWrapper(Class, key)` | 获取包装对象（含过期时间） |
| `containsKey(Class, key)` | 是否存在且未过期 |
| `invalidate(Class, key)` | 按 key 失效（默认通知集群） |
| `invalidate(Class, key, notify)` | `notify=false` 仅本地失效不发广播 |
| `invalidate(Class, null)` | 传 null key 表示**全量清空本地缓存** |
| `refresh(Class, key)` | 刷新（删除后立即重新加载） |
| `notifyInvalidate(Class, key)` / `notifyRefresh(Class, key)` | 仅发布集群通知 |
| `localCacheSize(Class)` | 本地缓存条目数（注意是 `localCacheSize` 不是 `size`） |
| `localStats(Class)` | Caffeine 命中统计 |
| `keys(Class, keyPrefix)` | 列出 key（全局模式走 Redis SCAN） |

> ⚠️ **没有 `getIfPresent`、没有 `size` 方法**。判断存在用 `containsKey`，查大小用 `localCacheSize`。

**FusionCache.Config 构造**：位置参数 `new FusionCache.Config(Class, localCacheMaxNum, cacheExpireMillis)`，或 `FusionCache.Config.builder().cacheName(Class).localCacheMaxNum(n).cacheExpireMillis(ms).build()`。

| 属性 | 类型 | 默认 | 说明 |
|------|------|------|------|
| cacheName / entityClass | Class 或 String | — | 缓存名称 |
| localCacheMaxNum | int | 10000 | 本地缓存最大数量（Caffeine），0 表示不限制 |
| cacheExpireMillis | long | **-1** | 缓存过期毫秒数；**-1/0 都视为永久**，>0 设 Redis TTL（见下方注意） |
| isGlobalCache | boolean | true | 是否启用全局（Redis），false 表示纯本地节点缓存 |
| nullProtectMillis | long | 60000 | 空值保护（防穿透），loader 返回 null 时短时缓存空值 |
| failProtectMillis | long | 60000 | 失败保护，loader 抛异常时短时缓存空值 |
| reloadIntervalMillis | long | 100 | 重载间隔（不建议 <50ms） |
| reloadMaxTimes | int | 10 | 重载最大次数 |
| autoNotifyInvalidate | boolean | false | 加载成功后是否自动通知集群失效 |

> **cacheExpireMillis 取值**：代码默认 -1（视为永久）。>0 时会同时作为本地 wrapper 的过期时间与 Redis TTL。由于本地设过期会拖累 Caffeine 性能，**高频缓存建议保持 -1，仅靠 Redis TTL 兜底**。

## GlobalCache（纯 Redis）

> **包路径**：`uw.cache.GlobalCache`

不占用 JVM 内存，适合大对象、列表、低频访问。`loadValueWrapper` 用**锁条带化（1024 stripes）+ 双重检查**防缓存击穿，loader 失败走 nullProtect/failProtect。

构造：`GlobalCache.get(cacheName, key, CacheDataLoader, expireMillis)` — 不需要 static 初始化。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `put(Class, key, value, expireMillis)` | `CacheValueWrapper<V>` | 存入+过期时间 |
| `put(String cacheName, key, value, expireMillis)` | `CacheValueWrapper<V>` | 存入（自定义缓存名） |
| `get(Class, key, Class<V>)` | `CacheValueWrapper<V>` | 获取原始 wrapper（不触发加载） |
| `get(Class, key, CacheDataLoader, expireMillis)` | V | 获取+自动加载（加 JVM 锁防击穿） |
| `get(String cacheName, key, CacheDataLoader, expireMillis)` | V | 获取+自动加载（自定义缓存名） |
| `loadValueWrapper(...)` | `CacheValueWrapper<V>` | 获取 wrapper（带加载） |
| `containsKey(Class, key)` | boolean | 判断是否存在 |
| `invalidate(Class, key)` | boolean | 失效单个 key；**key=null 全量清空**该 cacheName |
| `invalidatePrefix(cacheName, keyPrefix)` | boolean | 按前缀批量失效（SCAN） |
| `invalidateKeys(Class, keyPrefix)` | boolean | `invalidatePrefix` 的 entityClass 重载 |
| `keys(Class, keyPrefix)` | Set\<String\> | 列出匹配前缀的 key |
| `notifyMsg(channel, message)` | Long | 发布 Pub/Sub 消息（内部用） |

> ⚠️ **没有 `getIfPresent`**。判断存在用 `containsKey`，读原始值用 `get(Class, key, Class<V>)`。

## GlobalLocker（分布式锁）

> **包路径**：`uw.cache.GlobalLocker`

基于 Redis setnx 的全局锁。**stamp 为 SnowflakeId（跨 JVM 全局唯一）**，解锁/续期用 **Lua CAS 脚本**保证原子性，避免锁过期被抢占后误删他人锁。

构造：`GlobalLocker.tryLock(Class, lockerId, lockTimeMillis)` — 返回 stamp（>0 表示获锁成功）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `tryLock(Class, Object, long lockTimeMillis)` | long | 尝试加锁，返回 stamp（>0 表示获锁成功，0 表示失败） |
| `keepLock(Class, Object, stamp, long lockTimeMillis)` | boolean | 原子续期（stamp 不匹配则失败） |
| `unlock(Class, Object, stamp)` | boolean | CAS 解锁（仅 stamp 匹配时删除） |
| `unlock(Class, Object, stamp, boolean force)` | boolean | `force=true` 强制删除（忽略 stamp） |
| `keys(Class, keyPrefix)` | Set\<String\> | 列出锁 key |

**推荐用法**：

```java
long stamp = GlobalLocker.tryLock(Order.class, orderId, 30_000L);
if (stamp > 0) {
    try {
        // 业务逻辑（执行较久时周期性 keepLock 续期）
        GlobalLocker.keepLock(Order.class, orderId, stamp, 30_000L);
    } finally {
        GlobalLocker.unlock(Order.class, orderId, stamp);
    }
}
```

> ⚠️ **执行时间不可超过 lockTimeMillis**，否则锁自动过期、其他实例可抢占，此时原 holder 的 unlock 因 stamp 不匹配不会误删，但业务已无锁保护。长任务必须用 `keepLock` 续期。

## FusionCounter（本地+全局融合计数器）

> **包路径**：`uw.cache.FusionCounter`

基于 AtomicLong（本地）和 Redis（全局）的复合计数器。本地高频累加，按 `syncGlobalMillis` 间隔原子化同步增量到 Redis。支持定期回写数据库（虚拟线程异步执行）。

**初始化**：在 Helper 的 `static {}` 块中 config；不 config 也可用（首次调用自动按默认 60s 同步配置）。

| 方法 | 说明 |
|------|------|
| `config(Class, syncGlobalMillis)` | 配置基本计数器（同步间隔） |
| `config(String counterType, syncGlobalMillis)` | 配置基本计数器（String 类型名） |
| `config(Class, syncGlobalMillis, writeBackMillis, BiConsumer)` | 配置带回写的计数器（定期写数据库） |
| `increment(Class, counterId)` / `increment(Class, counterId, long num)` | +1 / +N |
| `decrement(Class, counterId)` / `decrement(Class, counterId, long num)` | -1 / -N |
| `init(Class, counterId, long initNum)` | 设置初始值（setIfAbsent） |
| `get(Class, counterId)` / `get(Class, counterId, boolean forceSync)` | 获取数值，forceSync 强制同步 |
| `delete(Class, counterId)` | 删除计数器（先同步再删 Redis） |
| `getAndDelete(Class, counterId)` | 获取数值后删除 |

```java
static {
    // 基本计数器：每60秒同步到Redis
    FusionCounter.config(User.class, 60_000L);

    // 带回写的计数器：每60秒同步Redis，每300秒回写数据库
    FusionCounter.config(Order.class, 60_000L, 300_000L, (orderId, count) -> {
        dao.execute("UPDATE orders SET view_count=? WHERE id=?", new Object[]{count, orderId});
    });
}

// 使用
FusionCounter.increment(User.class, userId);
FusionCounter.increment(User.class, userId, 5);
```

> ⚠️ 本地计数与 Redis 同步存在 `syncGlobalMillis` 级别的短暂延迟（最终一致）。需要强一致读时用 `get(Class, counterId, true)` 强制同步。

## GlobalCounter（纯 Redis 计数器）

> **包路径**：`uw.cache.GlobalCounter`

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `increment(Class, counterId, long)` | long | 增加计数 |
| `decrement(Class, counterId, long)` | long | 减少计数 |
| `set(Class, counterId, long)` | void | 设置数值 |
| `setIfAbsent(Class, counterId, long)` | boolean | 仅在不存在时设置（初始化） |
| `get(Class, counterId)` | long | 获取数值（不存在返回 0） |
| `delete(Class, counterId)` | boolean | 删除计数器 |
| `getAndDelete(Class, counterId)` | long | 获取后删除（不存在返回 -1） |

## GlobalHashSet（Redis Set）

> **包路径**：`uw.cache.GlobalHashSet`

元素经 Kryo 序列化后存入 Redis Set。`add/remove/contains` 三者序列化协议一致，**contains 会正确序列化 item 后比对**。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `add(String setName, Object)` | boolean | 添加元素 |
| `remove(String setName, Object)` | long | 移除元素 |
| `size(String setName)` | long | 集合大小 |
| `contains(String setName, Object)` | boolean | 是否包含（内部序列化 item） |
| `random(String setName, Class<T>)` | T | 随机获取一个元素 |
| `random(String setName, long count, Class<T>)` | Set\<T\> | 随机获取多个元素（key 不存在返回空集） |
| `pop(String setName, Class<T>)` | T | 随机弹出（删除并返回） |
| `pop(String setName, long count, Class<T>)` | Set\<T\> | 随机弹出多个 |
| `list(String setName, Class<T>)` | Set\<T\> | 列出全部元素 |
| `delete(String setName)` | boolean | 删除整个 Set |

## GlobalSortedSet（Redis ZSet，延迟任务场景）

> **包路径**：`uw.cache.GlobalSortedSet`

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `add(String setName, Object data, double score)` | boolean | 添加元素（score 通常为时间戳） |
| `remove(String setName, Object)` | long | 移除元素 |
| `remove(String setName, Object...)` | long | 批量移除 |
| `removeRangeByScore(String, double min, double max)` | long | 按分数范围删除 |
| `size(String setName)` | long | 集合大小 |
| `listRangeByScore(String, Class<T>, double min, double max)` | Set\<T\> | 按分数范围查询 |
| `delete(String setName)` | boolean | 删除整个 ZSet |

```java
// 延迟任务示例：添加一个5分钟后执行的任务
GlobalSortedSet.add("delayedTasks", taskId, SystemClock.now() + 300_000L);

// 查询已到期的任务
Set<Long> readyTasks = GlobalSortedSet.listRangeByScore("delayedTasks", Long.class, 0, SystemClock.now());

// 删除已处理的到期任务
GlobalSortedSet.removeRangeByScore("delayedTasks", 0, SystemClock.now());
```

## Helper 使用示例

```java
public class UserHelper {
    private static final DaoManager dao = DaoManager.getInstance();

    // FusionCache 必须在 static 块中初始化
    static {
        FusionCache.config(new FusionCache.Config(
            User.class, 1000, 3600_000L
        ), new CacheDataLoader<Long, User>() {
            @Override
            public User load(Long userId) {
                return dao.load(User.class, userId).getData();
            }
        });
    }

    public static User getUser(long userId) {
        return FusionCache.get(User.class, userId);
    }

    public static void updateUser(User user) {
        dao.update(user);
        FusionCache.invalidate(User.class, user.getId());
    }

    // GlobalCache 使用（行内 CacheDataLoader，不需要 static 初始化）
    public static ArrayList<Product> listProducts(long saasId) {
        return GlobalCache.get("productList", saasId, new CacheDataLoader<Long, ArrayList<Product>>() {
            @Override
            public ArrayList<Product> load(Long sid) {
                return new ArrayList<>(dao.list(Product.class, "SELECT * FROM product WHERE saas_id=?", new Object[]{sid}).getData());
            }
        }, 1800_000L);
    }

    // 分布式锁使用
    public static void processOrder(long orderId) {
        long stamp = GlobalLocker.tryLock(Order.class, orderId, 30000L);
        if (stamp > 0) {
            try {
                process(orderId);
                GlobalLocker.keepLock(Order.class, orderId, stamp, 30000L);
            } finally {
                GlobalLocker.unlock(Order.class, orderId, stamp);
            }
        }
    }
}
```

**重要注意事项**：
- Caffeine 设定过期时间后性能劣化 200 倍，建议 `cacheExpireMillis` 保持 -1（永久），仅靠 Redis TTL 兜底
- Kryo 序列化**必须使用具体实现类**，不能使用接口类型（如 List/Map/Set），必须传 ArrayList/LinkedHashMap/HashSet 等；`CacheDataLoader<V>` 的 V 必须是具体类
- Global* 组件（Cache/Counter/Locker/HashSet/SortedSet）通过 static 字段持有 RedisTemplate，**同一 JVM 只支持一套 Redis 配置**
- GlobalLocker 的 stamp 是 SnowflakeId，跨 JVM 唯一；unlock/keepLock 已是 Lua CAS 原子操作，无需自行加锁

> 缓存使用的常见陷阱（CacheDataLoader 抽象类、GlobalCache 无 getIfPresent、FusionCache vs GlobalCache 选型）见 [dev-standards.md](dev-standards.md)「缓存使用常见陷阱」。
