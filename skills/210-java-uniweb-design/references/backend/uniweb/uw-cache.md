### uw-cache — 缓存管理

**Maven 坐标**: `com.umtone:uw-cache`

基于 Caffeine 和 Redis 的融合缓存类库。

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

| 组件 | 类名 | 说明 |
|---|---|---|
| 融合缓存 | `FusionCache` | 本地(Caffeine) + 全局(Redis)，性能高 |
| 全局缓存 | `GlobalCache` | 直接操作 Redis，不占 JVM 内存 |
| 分布式锁 | `GlobalLocker` | 基于 Redis 的全局锁 |
| 融合计数器 | `FusionCounter` | 本地 + 全局计数器 |
| 全局计数器 | `GlobalCounter` | 基于 Redis 的全局计数器 |
| 全局集合 | `GlobalHashSet` | 基于 Redis Set |
| 有序集合 | `GlobalSortedSet` | 基于 Redis ZSet，支持延迟任务 |

**FusionCache API 详解**（本地 Caffeine + 全局 Redis）：

```java
// ========== 配置缓存 ==========
// 基础配置
static void config(Config config)

// 配置带数据加载器
static void config(Config config, CacheDataLoader<?, ?> cacheDataLoader)

// 配置带变更监听器
static void config(Config config, CacheChangeNotifyListener<?, ?> cacheChangeNotifyListener)

// 完整配置
static void config(Config config, CacheDataLoader<?, ?> cacheDataLoader, CacheChangeNotifyListener<?, ?> cacheChangeNotifyListener)

// ========== 存取数据 ==========
// 存入数据
static void put(Class<?> entityClass, Object key, Object value)
static void put(String cacheName, Object key, Object value)
static void put(Class<?> entityClass, Object key, Object value, long expireMillis)
static void put(String cacheName, Object key, Object value, long expireMillis)
static void put(Class<?> entityClass, Object key, Object value, boolean onlyLocal)
static void put(String cacheName, Object key, Object value, boolean onlyLocal)

// 批量存入
static void putAll(Class<?> entityClass, Map<Object, Object> map)
static void putAll(String cacheName, Map<Object, Object> map)

// 获取数据（带自动加载）
static <K, V> V get(Class<?> entityClass, K key)
static <K, V> V get(String cacheName, K key)

// 获取数据（不带自动加载）
static <K, V> V getIfPresent(Class<?> entityClass, K key)
static <K, V> V getIfPresent(String cacheName, K key)

// 移除数据
static void invalidate(Class<?> entityClass, Object key)
static void invalidate(String cacheName, Object key)
static void invalidateAll(Class<?> entityClass)
static void invalidateAll(String cacheName)

// ========== Config 配置类 ==========
Config(Class<?> entityClass, long localCacheMaxNum, long cacheExpireMillis)
Config(String cacheName, long localCacheMaxNum, long cacheExpireMillis)

// 配置属性：
- cacheName: 缓存名称
- localCacheMaxNum: 本地缓存最大数量（Caffeine）
- cacheExpireMillis: 缓存过期时间（毫秒）
- globalCache: 是否启用全局缓存（Redis）
- nullProtectMillis: 空值保护时间（防止缓存穿透）

// ========== 使用示例 ==========
// 示例1：基础配置（推荐）
public class OrderHelper {
    static{
        // 配置用户缓存
        FusionCache.config(new FusionCache.Config(
            User.class,           // 缓存名称
            1000,                 // 本地缓存最大1000条
            3600_000L             // 过期时间1小时
        ), new CacheDataLoader<Long, User>() {
            @Override
            public User load(Long userId) {
                // 缓存未命中时从数据库加载
                return daoManager.load(User.class, userId).getData();
            }
        });
    }
}

// 示例2：使用缓存
@Service
public class UserService {
    public User getUser(Long userId) {
        // 自动处理缓存命中/未命中
        return FusionCache.get(User.class, userId);
    }
    
    public void updateUser(User user) {
        // 更新数据库
        daoManager.update(user);
        // 清除缓存
        FusionCache.invalidate(User.class, user.getId());
    }
}
```

**GlobalCache API 详解**（直接操作 Redis）：

```java
// ========== 存入数据 ==========
static <K, V> CacheValueWrapper<V> put(Class<?> entityClass, K key, V value, long expireMillis)
static <K, V> CacheValueWrapper<V> put(String cacheName, K key, V value, long expireMillis)

// ========== 获取数据 ==========
static <K, V> CacheValueWrapper<V> get(Class<?> entityClass, K key, Class<V> valueClass)
static <K, V> CacheValueWrapper<V> get(String cacheName, K key, Class<V> valueClass)

// 带加载器获取（加 JVM 锁防止缓存击穿）
static <K, V> V get(Class<?> entityClass, K key, CacheDataLoader<K, V> cacheDataLoader, long expireMillis)
static <K, V> V get(String cacheName, K key, CacheDataLoader<K, V> cacheDataLoader, long expireMillis)

// ========== 判断与删除 ==========
static boolean containsKey(Class<?> entityClass, Object key)
static boolean containsKey(String cacheName, Object key)

static boolean delete(Class<?> entityClass, Object key)
static boolean delete(String cacheName, Object key)

// ========== 使用示例 ==========
// 直接操作 Redis 缓存
GlobalCache.put(User.class, 1L, user, 3600_000L);

CacheValueWrapper<User> wrapper = GlobalCache.get(User.class, 1L, User.class);
if (wrapper != null) {
    User user = wrapper.getValue();
}

// 带自动加载的获取
User user = GlobalCache.get(User.class, 1L, id -> {
    return daoManager.load(User.class, id).getData();
}, 3600_000L);
```

**GlobalLocker API 详解**（分布式锁）：

```java
// ========== 加锁 ==========
// 尝试加锁
static long tryLock(Class<?> entityType, Object lockerId, long lockTimeMillis)
static long tryLock(String lockerType, Object lockerId, long lockTimeMillis)

// 保持锁定（续期）
static boolean keepLock(Class<?> entityType, Object lockerId, long stamp, long lockTimeMillis)
static boolean keepLock(String lockerType, Object lockerId, long stamp, long lockTimeMillis)

// ========== 解锁 ==========
static boolean unlock(Class<?> entityType, Object lockerId, long stamp)
static boolean unlock(String lockerType, Object lockerId, long stamp)
static boolean unlock(Class<?> entityType, Object lockerId, long stamp, boolean force)
static boolean unlock(String lockerType, Object lockerId, long stamp, boolean force)

// ========== 使用示例 ==========
@Service
public class OrderService {
    
    public void processOrder(Long orderId) {
        // 尝试获取锁（30秒过期）
        long stamp = GlobalLocker.tryLock(Order.class, orderId, 30000L);
        
        if (stamp > 0) {
            try {
                // 执行业务逻辑
                process(orderId);
                
                // 如果需要续期
                GlobalLocker.keepLock(Order.class, orderId, stamp, 30000L);
                
            } finally {
                // 释放锁
                GlobalLocker.unlock(Order.class, orderId, stamp);
            }
        } else {
            throw new RuntimeException("获取锁失败，订单正在处理中");
        }
    }
}
```

**其他缓存组件速查**：

| 组件 | 核心方法 | 用途 |
|------|---------|------|
| `FusionCounter` | `increment()` / `decrement()` / `get()` | 本地+全局计数器，高性能 |
| `GlobalCounter` | `increment()` / `decrement()` / `get()` / `reset()` | 纯 Redis 计数器 |
| `GlobalHashSet` | `add()` / `remove()` / `contains()` / `members()` | Redis Set 操作 |
| `GlobalSortedSet` | `add()` / `range()` / `remove()` / `score()` | Redis ZSet，支持延迟任务 |

**重要注意事项**：
- Caffeine 设定过期时间后性能劣化 200 倍，建议仅设 Redis 过期时间
- Kryo 序列化**必须使用具体实现类**，不能使用接口类型（如 List/Map/Set），必须传 ArrayList/LinkedHashMap/HashSet 等

