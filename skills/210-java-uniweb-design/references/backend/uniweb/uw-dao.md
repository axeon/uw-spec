### uw-dao — 数据访问层

**Maven 坐标**: `com.umtone:uw-dao`

自研 ORM/DAO 框架，比 Hibernate 更高效，比 MyBatis 更简单。

**核心特性**：

| 特性 | 说明 |
|---|---|
| 注解驱动 ORM | `@TableMeta` / `@ColumnMeta`，反射元数据全局缓存 |
| 差量更新 | `DataUpdateInfo` 字段级变更追踪，UPDATE 只更新已修改字段 |
| 多数据源路由 | 按表名前缀路由到不同连接池，支持读写分离 |
| HikariCP 连接池 | 每数据源独立池，支持配置 min/max 连接 |
| 多方言支持 | MySQL / Oracle / SQL Server 分页方言自动识别 |
| 自动分表 | 按日期(day/month/year)或 ID 范围自动创建分片表 |
| 分布式序列 | DB 乐观锁序列 + Redis INCR 高速序列 |
| 事务管理 | 轻量 `TransactionManager`，支持标准隔离级别 |
| 批量更新 | `BatchUpdateManager` 复用 PreparedStatement |
| SQL 执行监控 | 慢 SQL 统计写入 `dao_sql_stats_YYYYMMDD` 表 |
| 双入口设计 | `DaoFactory`（抛异常）/ `DaoManager`（ResponseData 包装） |

**配置前缀**: `uw.dao`

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

**核心类**：
- `DaoManager` — ResponseData 包装风格DAO入口（推荐）
- `DaoFactory` — 抛异常风格DAO入口
- `DataEntity` — 实体基类，注解 `@TableMeta` / `@ColumnMeta`
- `QueryParam` — 条件查询参数
- `DataSet` — SQL 查询结果集
- `DataList` — 分页数据列表
- `BatchUpdateManager` — 批量更新管理器

**DaoManager API 详解**（ResponseData 包装风格，推荐）：

```java
// ========== 获取实例 ==========
static DaoManager getInstance()  // 获取线程安全实例

// ========== 序列生成 ==========
long getSequenceId(Class<?> entityCls)   // 根据实体类获取序列
long getSequenceId(String seqName)       // 根据表名获取序列

// ========== 保存（Insert）==========
// 单条保存
<T extends DataEntity> ResponseData<T> save(T entity)
<T extends DataEntity> ResponseData<T> save(String connName, T entity)
<T extends DataEntity> ResponseData<T> save(T entity, String tableName)
<T extends DataEntity> ResponseData<T> save(String connName, T entity, String tableName)

// 批量保存
<T extends DataEntity> ResponseData<List<T>> save(List<T> entityList)
<T extends DataEntity> ResponseData<List<T>> save(String connName, List<T> entityList)
<T extends DataEntity> ResponseData<List<T>> save(List<T> entityList, String tableName)
<T extends DataEntity> ResponseData<List<T>> save(String connName, List<T> entityList, String tableName)

// ========== 更新（Update）==========
// 根据主键更新（差量更新，只更新修改过的字段）
<T extends DataEntity> ResponseData<T> update(T entity)
<T extends DataEntity> ResponseData<T> update(String connName, T entity)
<T extends DataEntity> ResponseData<T> update(T entity, String tableName)
<T extends DataEntity> ResponseData<T> update(String connName, T entity, String tableName)

// 根据条件更新
<T extends DataEntity> ResponseData<Integer> update(T entity, QueryParam queryParam)
<T extends DataEntity> ResponseData<Integer> update(String connName, T entity, QueryParam queryParam)
<T extends DataEntity> ResponseData<Integer> update(String connName, T entity, String tableName, QueryParam queryParam)

// ========== 删除（Delete）==========
// 根据主键删除
<T extends DataEntity> ResponseData<Integer> delete(T entity)
<T extends DataEntity> ResponseData<Integer> delete(String connName, T entity)
<T extends DataEntity> ResponseData<Integer> delete(T entity, String tableName)
<T extends DataEntity> ResponseData<Integer> delete(String connName, T entity, String tableName)

// 根据条件删除
<T extends DataEntity> ResponseData<Integer> delete(Class<T> entityCls, QueryParam queryParam)
<T extends DataEntity> ResponseData<Integer> delete(String connName, Class<T> entityCls, QueryParam queryParam)
<T extends DataEntity> ResponseData<Integer> delete(String connName, Class<T> entityCls, String tableName, QueryParam queryParam)

// ========== 查询（Select）==========
// 根据主键加载
<T> ResponseData<T> load(Class<T> entityCls, Serializable id)
<T> ResponseData<T> load(String connName, Class<T> entityCls, Serializable id)
<T> ResponseData<T> load(Class<T> entityCls, String tableName, Serializable id)
<T> ResponseData<T> load(String connName, Class<T> entityCls, String tableName, Serializable id)

// 列表查询
<T> ResponseData<DataList<T>> list(Class<T> entityCls, String selectSql)
<T> ResponseData<DataList<T>> list(Class<T> entityCls, String selectSql, int startIndex, int resultNum)
<T> ResponseData<DataList<T>> list(Class<T> entityCls, String selectSql, int startIndex, int resultNum, boolean autoCount)
<T> ResponseData<DataList<T>> list(String connName, Class<T> entityCls, String selectSql)

// 带参数列表查询
<T> ResponseData<DataList<T>> list(Class<T> entityCls, String selectSql, Object[] paramList)

// QueryParam 查询
<T> ResponseData<DataList<T>> list(Class<T> entityCls, QueryParam queryParam)
<T> ResponseData<DataList<T>> list(String connName, Class<T> entityCls, QueryParam queryParam)

// 单条查询
<T> ResponseData<T> single(Class<T> entityCls, String selectSql)
<T> ResponseData<T> single(Class<T> entityCls, String selectSql, Object[] paramList)

// SQL 执行
ResponseData<Integer> execute(String sql)
ResponseData<Integer> execute(String sql, Object[] paramList)

// ========== 使用示例 ==========
@Service
public class UserService {
    private final DaoManager daoManager = DaoManager.getInstance();
    
    // 保存用户
    public ResponseData<User> createUser(User user) {
        user.setId(daoManager.getSequenceId(User.class));
        return daoManager.save(user);
    }
    
    // 更新用户（自动差量更新）
    public ResponseData<User> updateUser(User user) {
        return daoManager.update(user);
    }
    
    // 根据ID查询
    public ResponseData<User> getById(Long id) {
        return daoManager.load(User.class, id);
    }
    
    // 条件查询
    public ResponseData<DataList<User>> listByStatus(int status, int page, int size) {
        UserQueryParam param = new UserQueryParam();
        param.setStatus(status);
        param.SORT_NAME("createDate").SORT_TYPE(SORT_DESC);
        return daoManager.list(User.class, param);
    }
    
    // 执行自定义SQL
    public ResponseData<Integer> updateStatus(Long id, int status) {
        String sql = "UPDATE user SET status = ? WHERE id = ?";
        return daoManager.execute(sql, new Object[]{status, id});
    }
}
```

**DataEntity 接口详解**：

```java
// 实体类必须实现 DataEntity 接口
public interface DataEntity {
    String ENTITY_TABLE();        // 返回实体对应的表名
    String ENTITY_NAME();         // 返回实体名称（用于日志）
    Serializable ENTITY_ID();     // 返回实体主键值
    DataUpdateInfo GET_UPDATED_INFO();  // 获取字段更新信息（差量更新用）
    void CLEAR_UPDATED_INFO();    // 清除更新信息
}

// ========== 实体类示例 ==========
@TableMeta(tableName = "user", tableType = "table")
public class User implements DataEntity {
    
    @ColumnMeta(columnName = "id", primaryKey = true)
    private Long id;
    
    @ColumnMeta(columnName = "user_name")
    private String userName;
    
    @ColumnMeta(columnName = "status")
    private Integer status;
    
    // 实现 DataEntity 接口方法
    @Override
    public String ENTITY_TABLE() { return "user"; }
    
    @Override
    public String ENTITY_NAME() { return "用户"; }
    
    @Override
    public Serializable ENTITY_ID() { return id; }
    
    @JsonIgnore
    private DataUpdateInfo dataUpdateInfo;
    
    @Override
    public DataUpdateInfo GET_UPDATED_INFO() { return dataUpdateInfo; }
    
    @Override
    public void CLEAR_UPDATED_INFO() { dataUpdateInfo = null; }
    
    // Getters 和 Setters（setter 中需要记录更新字段）
    public void setUserName(String userName) {
        if (dataUpdateInfo == null) dataUpdateInfo = new DataUpdateInfo();
        dataUpdateInfo.addUpdate("user_name", this.userName, userName);
        this.userName = userName;
    }
}
```

**QueryParam API 详解**：

```java
// ========== 排序常量 ==========
static final int SORT_NONE = 0   // 不排序
static final int SORT_ASC = 1    // 升序
static final int SORT_DESC = 2   // 降序

// ========== 链式配置方法 ==========
P SELECT_SQL(String sql)                    // 设置自定义 SELECT SQL
P LIKE_QUERY_ENABLE(boolean enable)         // 是否启用 LIKE 查询
P LIKE_QUERY_PARAM_MIN_LEN(int len)         // LIKE 查询参数最小长度
P ADD_EXT_COND_SQL(String sql)              // 添加附加 WHERE 条件
P ADD_EXT_COND(String expr, Object value)   // 添加附加条件参数
P CLEAR_EXT_COND_SQL()                      // 清除附加条件
P SORT_NAME(String... names)                // 设置排序字段
P SORT_TYPE(int... types)                   // 设置排序类型

// ========== 查询注解 @QueryMeta ==========
@QueryMeta(expr = "user_name like ?")        // LIKE 查询
@QueryMeta(expr = "status = ?")              // 等值查询
@QueryMeta(expr = "create_date >= ? and create_date <= ?")  // 范围查询
@QueryMeta(expr = "id in (select user_id from user_role where role_id = ?)")  // 子查询

// ========== 使用示例 ==========
public class UserQueryParam extends QueryParam<UserQueryParam> {
    
    @QueryMeta(expr = "user_name like ?")
    private String userName;
    
    @QueryMeta(expr = "status = ?")
    private Integer status;
    
    @QueryMeta(expr = "create_date >= ?")
    private Date startDate;
    
    @QueryMeta(expr = "create_date <= ?")
    private Date endDate;
    
    // 允许的排序属性映射
    @Override
    public Map<String, String> ALLOWED_SORT_PROPERTY() {
        Map<String, String> map = new HashMap<>();
        map.put("userName", "user_name");
        map.put("createDate", "create_date");
        return map;
    }
    
    // Getters 和 Setters
}

// Controller 中使用
@GetMapping("/list")
public ResponseData<DataList<User>> list(
        @RequestParam(required = false) String userName,
        @RequestParam(required = false) Integer status,
        @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd") Date startDate,
        @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd") Date endDate,
        @RequestParam(defaultValue = "createDate") String sortName,
        @RequestParam(defaultValue = "2") int sortType) {
    
    UserQueryParam param = new UserQueryParam();
    param.setUserName(userName);
    param.setStatus(status);
    param.setStartDate(startDate);
    param.setEndDate(endDate);
    param.SORT_NAME(sortName).SORT_TYPE(sortType);
    
    return daoManager.list(User.class, param);
}
```

**BatchUpdateManager 批量更新**：

```java
// ========== 获取实例 ==========
BatchUpdateManager getBatchUpdateManager(String connName)

// ========== BatchUpdateManager 方法 ==========
void setBatchSize(int size)                    // 设置批量大小
int getBatchSize()                             // 获取批量大小
List<String> getBatchList()                    // 获取待执行的 SQL 列表
Map<String, List<Integer>> submit()            // 提交执行

// ========== 使用示例 ==========
public void batchUpdateUsers(List<User> users) {
    BatchUpdateManager batchManager = daoManager.getBatchUpdateManager(null);
    batchManager.setBatchSize(100);  // 每100条提交一次
    
    for (User user : users) {
        String sql = String.format(
            "UPDATE user SET status = %d WHERE id = %d",
            user.getStatus(), user.getId()
        );
        batchManager.getBatchList().add(sql);
    }
    
    try {
        Map<String, List<Integer>> result = batchManager.submit();
        // result 包含每个 SQL 的执行结果
    } catch (TransactionException e) {
        log.error("批量更新失败", e);
    }
}
```

