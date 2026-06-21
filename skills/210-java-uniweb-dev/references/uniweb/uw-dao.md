# uw-dao — 数据访问层

**Maven 坐标**: `com.umtone:uw-dao`
**入口包**: `uw.dao`
**配置前缀**: `uw.dao`

> 本文档与 `uw-dao` 源码严格对齐，所有方法签名、字段名均为真实 API。

## 最小配置

```yaml
uw:
  dao:
    conn-pool:
      root:
        driver: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: secret
        min-conn: 1
        max-conn: 10
```

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 获取实例 | `DaoManager.getInstance()` | 每次返回新实例，勿作共享单例跨线程使用 |
| 保存 | `dao.save(entity)` | 先 `entity.setId(dao.getSequenceId(Class))`，再设 `createDate` |
| 批量保存 | `dao.save(List<Entity>)` | 单条 INSERT VALUES 多行 |
| 更新(主键) | `dao.update(entity)` | 差量更新：仅写 setter 触发过的字段；需先 `load` 再改 |
| 条件更新 | `dao.update(entity, QueryParam)` | entity 字段全量写入 SET，QueryParam 生成 WHERE |
| 按ID加载 | `dao.load(Class, id)` | 走**读库**（未配读库时回退写库）；加载后 `_IS_LOADED=true` |
| 列表查询(QueryParam) | `dao.list(Class, QueryParam)` 或 `dao.queryForList(Class, QueryParam)` | QueryParam 仅限 Controller 层 |
| 列表查询(SQL) | `dao.list(Class, "SELECT * FROM t WHERE ...", params)` | SQL 以 SELECT * FROM 开头 |
| 查单条 | `dao.queryForObject(Class, QueryParam)` 或 `dao.queryForObject(Class, sql, params)` | 禁止 list + get(0) |
| 查计数值 | `dao.queryForValue(Long.class, "SELECT COUNT(*) ...", params)` | 包装类型 NULL 返回 null；禁止 list+size() |
| 查值列表 | `dao.queryForValueList(Class, "SELECT col ...", params)` | 多行单列 |
| 查行列结果 | `dao.queryForRowSet("SELECT ...", params)` | 弱类型 `PageRowSet`，灵活但无类型安全 |
| 执行DML | `dao.execute(sql, params)` | 返回影响行数 |
| 删除(主键) | `dao.delete(entity)` | 实体须已设主键 |
| 条件删除 | `dao.delete(Class, QueryParam)` | — |
| 序列ID | `dao.getSequenceId(Class)` | 插入前必须调用 |
| 批量更新 | `dao.beginBatchUpdate()` + `dao.beginTransaction()` | **必须先开事务** |

**SaaS多租户安全规则**：仅当实体表含 `saas_id` 时适用。load/list 默认不自动带 saasId 过滤，需在 QueryParam 中显式声明 `@QueryMeta(expr="saas_id=?")` 或用 `AuthIdQueryParam`。

**load/queryForObject/queryForValue 返回值**：无数据（查不到行/值为 null）时返回 `DATA_NOT_FOUND_WARN`，`getData()` 为 null。更新/删除 0 行也返回 `DATA_NOT_FOUND_WARN`。判空用 `resp.isSuccess() && resp.getData() != null`，或直接 `resp.isOnWarn()`/`resp.getCode()` 判断是否无数据。

## 两个入口：DaoManager vs DaoFactory

> **推荐 `DaoManager`**。`DaoFactory` 为传统入口，仅供历史代码兼容，新代码统一用 `DaoManager`。

| 对比项 | `DaoManager`（推荐） | `DaoFactory`（传统，不推荐） |
|--------|--------------|--------------|
| 获取 | `DaoManager.getInstance()` | `DaoFactory.getInstance()` |
| 返回 | `ResponseData<T>` | 直接返回值 |
| 异常 | 封装在 ResponseData 中 | 抛 `TransactionException` |
| 更新0行 | `ResponseData.warn(DATA_NOT_FOUND_WARN)` | 返回 `0` |
| 生产错误 | 只返回错误码，不泄漏 SQL | 抛异常含完整信息 |
| 事务/批量 | `dao.beginTransaction()` / `dao.beginBatchUpdate()` | `dao.beginTransaction()` |
| 适用 | Controller/Service/Helper（推荐） | 历史代码、细粒度异常控制 |

两者均**每次 getInstance() 返回新实例**，持有独立的事务/批量状态，禁止作为共享 Bean 跨线程使用。

## DaoManager 核心方法

> **包路径**：`uw.dao.DaoManager`，构造：`DaoManager.getInstance()`

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `getSequenceId(Class<?>)` | long | 分布式序列（实体类名） |
| `getSequenceId(String seqName)` | long | 自定义序列名 |
| `save(T entity)` | `ResponseData<T>` | 单条保存 |
| `save(List<T>)` | `ResponseData<List<T>>` | 批量保存 |
| `update(T entity)` | `ResponseData<T>` | 主键差量更新 |
| `update(T entity, QueryParam)` | `ResponseData<Integer>` | 条件更新 |
| `delete(T entity)` | `ResponseData<Integer>` | 删除 |
| `delete(Class<T>, QueryParam)` | `ResponseData<Integer>` | 条件删除 |
| `load(Class<T>, Serializable id)` | `ResponseData<T>` | 按主键加载 |
| `list(Class<T>, String sql)` | `ResponseData<PageList<T>>` | SQL列表 |
| `list(Class<T>, String sql, Object[])` | `ResponseData<PageList<T>>` | 带参SQL列表 |
| `list(Class<T>, PageQueryParam)` | `ResponseData<PageList<T>>` | QueryParam列表 |
| `queryForObject(Class<T>, String sql, Object[])` | `ResponseData<T>` | 查单条 |
| `queryForObject(Class<T>, QueryParam)` | `ResponseData<T>` | QueryParam查单条 |
| `queryForValue(Class<T>, String sql, Object[])` | `ResponseData<T>` | 查标量值 |
| `queryForValueList(Class<T>, String sql, Object[])` | `ResponseData<List<T>>` | 查值列表 |
| `queryForRowSet(String sql, Object[])` | `ResponseData<PageRowSet>` | 查行列结果 |
| `queryForList(...)` | `ResponseData<PageList<T>>` | list() 的别名重载集 |
| `execute(String sql, Object[])` | `ResponseData<Integer>` | 执行DML |

> 所有方法均支持 `String connName` 首参重载（指定数据源）与 `String tableName` 重载（指定分表）。
> `queryForList` 是 `list` 的语义别名，16 个重载完全委托到对应 `list`。

## 链式调用设计（ResponseData）

DaoManager 所有方法返回 `ResponseData<T>`（来自 `uw-common`），支持链式后处理。Controller 可直接 `return dao.xxx()`。

**设计理念**：
- **零中间变量**：`return dao.list(Class, param)` 直接返回前端
- **链式后处理**：`onSuccess` / `onError` / `onWarn` / `onNotSuccess` 在返回前做额外操作
- **自动短路**：链中任一步非 success，后续 `onSuccess` 自动跳过，不会 NPE

### 直接返回模式

```java
@GetMapping("/list")
public ResponseData<PageList<User>> list(UserQueryParam param) {
    return dao.list(User.class, param);
}

@GetMapping("/load")
public ResponseData<User> load(long id) {
    return dao.load(User.class, id);
}

@PostMapping("/save")
public ResponseData<User> save(@RequestBody User user) {
    user.setId(dao.getSequenceId(User.class));
    user.setCreateDate(SystemClock.nowDate());
    return dao.save(user);
}
```

### 链式后处理模式

```java
// 保存成功后清缓存（Function 形式可转换返回类型）
@PostMapping("/save")
public ResponseData<User> save(@RequestBody User user) {
    user.setId(dao.getSequenceId(User.class));
    user.setCreateDate(SystemClock.nowDate());
    return dao.save(user).onSuccess(saved -> {
        FusionCache.invalidate(User.class, saved.getId());
    });
}

// 条件更新 + 日志（Consumer 形式）
@PostMapping("/enable")
public ResponseData<Integer> enable(long id) {
    return dao.load(User.class, id).onSuccess(user -> {
        user.setState(CommonState.ENABLED.getValue());
        user.setModifyDate(SystemClock.nowDate());
        dao.update(user);
    });
}
```

### 链式数据组装（父子表联查）

```java
@GetMapping("/listEx")
public ResponseData<PageList<OrderEx>> listEx(OrderQueryParam param) {
    return dao.list(OrderEx.class, param).onSuccess(orders -> {
        if (orders.isEmpty()) return;
        Object[] ids = orders.stream().map(OrderEx::getId).toArray();
        String inSql = "SELECT * FROM order_item WHERE order_id IN ("
            + String.join(",", Collections.nCopies(ids.length, "?")) + ")";
        dao.list(OrderItem.class, inSql, ids).onSuccess(items -> {
            Map<Long, List<OrderItem>> itemMap = items.stream()
                .collect(Collectors.groupingBy(OrderItem::getOrderId));
            orders.forEach(o -> o.setItemList(itemMap.getOrDefault(o.getId(), List.of())));
        });
    });
}
```

## DataEntity 接口与差量更新

> **包路径**：`uw.dao.DataEntity`

实体类须实现此接口 + `@TableMeta` + `@ColumnMeta`，并声明两个 `transient` 字段以支持差量更新。

| 接口方法 | 返回类型 | 说明 |
|---------|---------|------|
| `ENTITY_TABLE()` | String | 表名 |
| `ENTITY_NAME()` | String | 实体名称（日志用） |
| `ENTITY_ID()` | Serializable | 主键值 |
| `GET_UPDATED_INFO()` | DataUpdateInfo | 字段变更信息 |
| `CLEAR_UPDATED_INFO()` | void | 清除变更信息 |

**差量更新机制**：
1. 框架 `load` 后反射置 `_IS_LOADED=true`
2. 此后每次 setter 调用 `DataUpdateInfo.addUpdateInfo(info, "col", oldVal, newVal, _IS_LOADED)` 记录变更
3. `update(entity)` 仅对有变更的字段生成 `SET col=?`，未改字段不入 SQL
4. 非 load 来源（`_IS_LOADED=false`）的 entity，`addUpdateInfo` 不记录（初始赋值），update 时会写全部非主键字段

**实体类标准模板**：

```java
@TableMeta(tableName = "user", tableType = "table")
public class User implements DataEntity {

    // 框架 load 后置 true，触发差量更新；勿手动改
    private transient boolean _IS_LOADED;
    // 变更追踪，transient 防止序列化
    private transient DataUpdateInfo _UPDATED_INFO;

    @ColumnMeta(columnName = "id", primaryKey = true)
    private long id;

    @ColumnMeta(columnName = "user_name", dataSize = 50, nullable = false)
    private String userName;

    @ColumnMeta(columnName = "create_date")
    private Date createDate;

    @Override public String ENTITY_TABLE() { return "user"; }
    @Override public String ENTITY_NAME() { return "用户"; }
    @Override public Serializable ENTITY_ID() { return id; }
    @Override public DataUpdateInfo GET_UPDATED_INFO() { return _UPDATED_INFO; }
    @Override public void CLEAR_UPDATED_INFO() { _UPDATED_INFO = null; }

    public User setUserName(String userName) {
        // 第5个参数 _IS_LOADED 控制：load 后才追踪变更
        this._UPDATED_INFO = DataUpdateInfo.addUpdateInfo(_UPDATED_INFO, "user_name", this.userName, userName, _IS_LOADED);
        this.userName = userName;
        return this;
    }
    // 其他 getter/setter 同理
}
```

> `@ColumnMeta.columnName` 大小写不敏感：框架内部统一用小写做列名匹配，无论注解写大写小写都能正确映射。

## QueryParam / PageQueryParam

> **包路径**：`uw.common.dto.QueryParam` / `uw.common.dto.PageQueryParam`（位于 uw-common）

自定义查询参数类继承 `PageQueryParam<Self>`（带分页）或 `QueryParam<Self>`，用 `@QueryMeta` 标注查询字段。

**QueryParam 常量与链式方法**：

| 项 | 说明 |
|----|------|
| `SORT_NONE=0` / `SORT_ASC=1` / `SORT_DESC=2` | 排序常量 |
| `ADD_SORT(name, type)` | 链式追加排序（返回 self） |
| `SORT_NAME(String...)` / `SORT_TYPE(Integer...)` | 批量设排序 |
| `SELECT_SQL(String)` | 覆盖默认 `select * from table`（禁止网络传入） |
| `LIKE_QUERY_ENABLE(boolean)` | 开关 like 查询（默认 true） |
| `LIKE_QUERY_PARAM_MIN_LEN(int)` | like 最小长度（默认3，短于此转 = 查询） |
| `ADD_EXT_COND_SQL(String)` | 追加无参数 where 片段 |
| `ADD_EXT_COND(expr, value)` | 追加参数化 where（expr 含 `?`） |
| `ADD_EXT_COND_PARAM(col, value)` | 追加 `col=?` 等值条件 |
| `ALLOWED_SORT_PROPERTY()` | 覆盖：返回 `Java属性名 -> 列名` 映射（防排序注入） |

**PageQueryParam 分页字段**（HTTP 参数别名 `$pg/$rn/$si/$rt`）：

| 字段 | 别名 | 默认 | 说明 |
|------|------|------|------|
| `PAGE` | `$pg` | 1 | 页码（从1起） |
| `RESULT_NUM` | `$rn` | 10 | 每页条数 |
| `START_INDEX` | `$si` | 0 | 起始偏移（优先于 PAGE） |
| `REQUEST_TYPE` | `$rt` | 1 | 0=仅计数, 1=仅数据, 2=数据+计数 |

链式：`param.PAGE(2).RESULT_NUM(20).REQUEST_TYPE(PageQueryParam.REQUEST_ALL)`

**@QueryMeta 表达式**：

```java
@QueryMeta(expr = "user_name like ?")                 // LIKE（受最小长度保护）
@QueryMeta(expr = "status = ?")                       // 等值
@QueryMeta(expr = "create_date >= ?")                 // 范围
@QueryMeta(expr = "create_date >= ? and create_date <= ?")  // 双占位符，同一值绑定两次
@QueryMeta(expr = "id in (?)")                        // IN（List/数组自动展开占位符）
@QueryMeta(expr = "age between ? and ?")              // BETWEEN（长度2数组）
@QueryMeta(expr = "state >= 0")                       // 无占位符（字段非null即激活）
```

**QueryParam 定义示例**：

```java
public class UserQueryParam extends PageQueryParam<UserQueryParam> {

    @QueryMeta(expr = "user_name like ?")
    private String userName;

    @QueryMeta(expr = "status = ?")
    private Integer status;

    @QueryMeta(expr = "create_date >= ?")
    private Date createDateBegin;

    @QueryMeta(expr = "create_date <= ?")
    private Date createDateEnd;

    @Override
    public Map<String, String> ALLOWED_SORT_PROPERTY() {
        Map<String, String> map = new LinkedHashMap<>();
        map.put("createDate", "create_date");
        map.put("id", "id");
        return map;
    }
    // getter/setter ...
}
```

## 批量更新（BatchUpdateManager）

> **包路径**：`uw.dao.BatchUpdateManager`，获取：`dao.beginBatchUpdate()`

⚠️ **批量更新必须在事务中运行**：`beginBatchUpdate()` 前必须先 `beginTransaction()`，否则
`PreparedStatement` 处于 autoCommit，addBatch 数据被立即提交，框架抛 `TransactionException`。

| 方法 | 说明 |
|------|------|
| `setBatchSize(int)` | 每批条数（默认 100），满则自动 executeBatch |
| `getBatchList()` | 待执行 SQL 列表 |
| `submit()` | 提交剩余批次，返回 `Map<SQL, List<行数>>` |

```java
DaoManager dao = DaoManager.getInstance();
TransactionManager tx = dao.beginTransaction();      // 必须先开事务
BatchUpdateManager batch = dao.beginBatchUpdate();
batch.setBatchSize(500);
try {
    for (User u : userList) {
        dao.save(u);   // 框架自动判批量模式，等价 addBatch
    }
    batch.submit();
    tx.commit();
} catch (Exception e) {
    tx.rollback();
    throw e;
}
```

> MySQL 连接 URL 须加 `rewriteBatchedStatements=true` 才能真正合并批处理。
> `submit()` 后 batchSize 重置为默认值 100，连续批量提交需重新 setBatchSize。

## 序列发号（SequenceFactory）

> **包路径**：`uw.dao.SequenceFactory`（统一入口）

```java
long id = SequenceFactory.getSequenceId(User.class);      // 序列名 = 类名
long id2 = SequenceFactory.getSequenceId("order_seq");   // 自定义序列名
long cur = SequenceFactory.getCurrentId("order_seq");    // 查看当前号（不消耗）
```

**实现选择**（自动）：配置了 `uw.dao.redis` 且 Redis 可用时用 `FusionSequenceFactory`（Redis 段缓存，高吞吐），
否则用 `DaoSequenceFactory`（DB 乐观锁）。Redis 故障/未初始化时自动降级到 DB 实现，保证发号不中断。

**getCurrentId 语义**：返回"最近已发号"，非 DB 真值；未初始化返回 0。仅用于观测，不可作为下一待发号或业务判断。

**DB 表**（`sys_seq`）：`seq_name / seq_id / seq_desc / increment_num / create_date / last_update`。
`increment_num` 越大 DB 访问越少（tps ≈ increment_num × 100），但宕机会留更大空档。

## 多数据源路由

框架据 SQL 中**表名前缀**路由到连接池，SELECT→读池，INSERT/UPDATE/DELETE/REPLACE/MERGE/CREATE→写池：

```yaml
uw:
  dao:
    conn-route:
      root:
        write-pools: [root]
        read-pools: [root]
      list:
        order_:
          write-pools: [order-write]
          read-pools: [order-read]
```

- `order_` 前缀匹配 `order_2024`、`order_item` 等
- 未配 read-pools 时读操作自动回退 write-pools
- 手动指定：`dao.load("order-write", User.class, id)` 等所有方法均有 connName 首参重载

## 分表（Table Sharding）

```yaml
uw:
  dao:
    table-shard:
      log_access:
        shard-type: date       # date | id
        shard-rule: day        # date: day/month/year；id: 每片ID数（整数）
        auto-gen: true         # 后台每小时预建当天+次日分表
```

```java
String table = ShardingTableUtils.getTableNameByDate("log_access", new Date());  // log_access_20240101
String table2 = ShardingTableUtils.getTableNameById("user_profile", 1500000L);   // user_profile_1
dao.save(log, table);
```

## SQL 执行统计

```yaml
uw:
  dao:
    sql-stats:
      enable: true
      sql-cost-min: 100      # 仅记录 >=100ms 的慢SQL
      data-keep-days: 100    # 按天分表，自动清理超期表
```

写入 `dao_sql_stats_yyyyMMdd`，后台每30s批量写、每天清理超期分表。代码内调试：

```java
DaoManager dao = DaoManager.getInstance();
dao.enableSqlExecuteStats();
// ... 执行操作 ...
dao.getSqlExecuteStatsList().forEach(s -> log.info(s.genFullSqlInfo()));
dao.disableSqlExecuteStats();
```

## Helper 使用示例

```java
// Helper 风格（纯静态工具类，禁止 @Service/@Component）
public class UserHelper {
    private static final DaoManager dao = DaoManager.getInstance();

    public static ResponseData<User> createUser(User user) {
        user.setId(dao.getSequenceId(User.class));
        user.setCreateDate(SystemClock.nowDate());
        return dao.save(user);
    }

    public static ResponseData<User> updateUser(User user) {
        user.setModifyDate(SystemClock.nowDate());
        return dao.update(user);
    }

    public static ResponseData<PageList<User>> listByStatus(int status) {
        return dao.list(User.class, "SELECT * FROM user WHERE status=?", new Object[]{status});
    }

    public static ResponseData<Long> countByStatus(int status) {
        return dao.queryForValue(Long.class,
            "SELECT COUNT(*) FROM user WHERE status=?", new Object[]{status});
    }
}
```

## 事务管理

`TransactionManager` 与 Spring 事务体系独立。DaoManager 已委派事务 API，直接调用即可：

```java
DaoManager dao = DaoManager.getInstance();
TransactionManager tx = dao.beginTransaction();
try {
    dao.save(order);
    dao.save(orderItem);
    tx.commit();
} catch (Exception e) {
    tx.rollback();
    throw e;
}
```

隔离级别：`tx.setTransactionIsolation(TransactionManager.TRANSACTION_READ_COMMITTED)`
（仅显式调用后才应用到连接，未设置则保持驱动默认）。
