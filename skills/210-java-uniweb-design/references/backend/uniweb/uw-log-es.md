### uw-log-es — Elasticsearch 日志客户端

**Maven 坐标**: `com.umtone:uw-log-es`

**配置前缀**: `uw.log.es`

```yaml
uw:
  log:
    es:
      server: http://localhost:9200
      username: admin
      password: admin
```

#### LogClient API

```java
public class LogClient {
    
    // 获取全局唯一实例
    public static LogClient getInstance();
    
    // ========== 日志注册 ==========
    
    // 注册日志类型（使用类名作为索引）
    public void regLogObject(Class<?> logClass);
    
    // 注册日志类型（自定义索引名）
    public void regLogObjectWithIndexName(Class<?> logClass, String index);
    
    // 注册日志类型（自定义索引模式）
    public void regLogObjectWithIndexPattern(Class<?> logClass, String indexPattern);
    
    // 注册日志类型（自定义索引名和模式）
    public void regLogObjectWithIndexNameAndPattern(Class<?> logClass, String index, String indexPattern);
    
    // ========== 索引名称获取 ==========
    
    public String getRawIndexName(Class<?> logClass);          // 原始索引名
    public String getQuotedRawIndexName(Class<?> logClass);    // 带引号的原始索引名
    public String getQueryIndexName(Class<?> logClass);        // 查询索引名
    public String getQuotedQueryIndexName(Class<?> logClass);  // 带引号的查询索引名
    
    // ========== 日志写入 ==========
    
    // 单条写入
    public <T extends LogBaseVo> void log(T source);
    
    // 批量写入
    public <T extends LogBaseVo> void bulkLog(List<T> sourceList);
    
    // ========== DSL查询 ==========
    
    // DSL查询（使用默认索引）
    public <T> SearchResponse<T> dslQuery(Class<T> tClass, String dslQuery);
    
    // DSL查询（指定索引）
    public <T> SearchResponse<T> dslQuery(Class<T> tClass, String index, String dslQuery);
    
    // SQL转DSL
    public String translateSqlToDsl(String sql, int startIndex, int resultNum, boolean isTrueCount);
    
    // ========== Scroll查询（大数据量） ==========
    
    // 开启scroll查询
    public <T> ScrollResponse<T> scrollQueryOpen(Class<T> tClass, String index, 
                                                  int scrollExpireSeconds, String dslQuery);
    
    // 获取下一批数据
    public <T> ScrollResponse<T> scrollQueryNext(Class<T> tClass, String index, 
                                                  String scrollId, int scrollExpireSeconds);
    
    // 关闭scroll查询
    public DeleteScrollResponse scrollQueryClose(String scrollId, String index);
    
    // ========== 聚合结果处理工具方法 ==========
    
    // 查询响应转分页列表
    public static <T> ESDataList<T> mapQueryResponseToEDataList(SearchResponse<T> response, 
                                                                 int startIndex, int pageSize);
    
    // 获取聚合值
    public static double getAggValue(Map<String, SearchResponse.Aggregation> aggMap, String aggName);
    
    // 转换聚合Bucket为Map（结构：agg->bucket）
    public static Map<String, List<Map<String, Object>>> convertAggBucketListMap(
        Map<String, SearchResponse.Aggregation> aggMap);
    
    // 转换聚合Bucket为Map（结构：agg->bucket->agg+bucket）
    public static Map<String, Map<String, Map<String, Double>>> convertAggBucketAggBucketFlatMap(
        Map<String, SearchResponse.Aggregation> aggMap);
    
    // 转换聚合Bucket为Map（结构：agg+bucket）
    public static Map<String, Double> convertAggBucketFlatMap(
        Map<String, SearchResponse.Aggregation> aggMap);
}

// 日志基类
public class LogBaseVo {
    private int logLevel;      // 日志级别：-1不记录，0普通，1详细
    private Date logDate;      // 日志时间
    
    public int getLogLevel();
    public void setLogLevel(int logLevel);
    public Date getLogDate();
    public void setLogDate(Date logDate);
}

// 日志级别枚举
public enum LogLevel {
    NONE(-1),      // 不记录
    NORMAL(0),     // 普通
    DETAIL(1);     // 详细
}

// 查询响应
public class SearchResponse<T> {
    private HitResponse<T> hitResponse;
    private Map<String, Aggregation> aggregations;
    
    public static class HitResponse<T> {
        private Total total;
        private List<Hit<T>> hits;
    }
    
    public static class Hit<T> {
        private String id;
        private T source;  // 日志对象
    }
    
    public static class Total {
        private long value;
    }
    
    public static class Aggregation {
        private double value;
        private List<Bucket> buckets;
        private Map<String, Aggregation> subAggregations;
    }
    
    public static class Bucket {
        private String key;
        private long docCount;
        private Map<String, Aggregation> subAggregations;
    }
}

// Scroll查询响应
public class ScrollResponse<T> {
    private String scrollId;
    private List<T> dataList;
    private long total;
    private boolean hasMore;
}

// 分页数据列表
public class ESDataList<T> {
    private List<T> data;
    private int startIndex;
    private int pageSize;
    private long total;
}
```

#### 使用示例

```java
@Service
public class LogService {
    
    @Autowired
    private LogClient logClient;
    
    // 定义日志对象
    @Data
    public class UserAccessLog extends LogBaseVo {
        private Long userId;
        private String action;
        private String ip;
        private Date accessTime;
        private String result;
    }
    
    // 初始化注册日志类型
    @PostConstruct
    public void init() {
        // 注册日志类型（索引名：user_access_log）
        logClient.regLogObject(UserAccessLog.class);
        
        // 或自定义索引名
        logClient.regLogObjectWithIndexName(UserAccessLog.class, "user_access");
        
        // 或按日期分片（索引模式：user_access_202401）
        logClient.regLogObjectWithIndexPattern(UserAccessLog.class, "user_access_yyyyMM");
    }
    
    // 记录单条日志
    public void recordAccess(Long userId, String action, String ip, String result) {
        UserAccessLog log = new UserAccessLog();
        log.setLogLevel(LogLevel.NORMAL.getValue());
        log.setLogDate(new Date());
        log.setUserId(userId);
        log.setAction(action);
        log.setIp(ip);
        log.setAccessTime(new Date());
        log.setResult(result);
        
        logClient.log(log);
    }
    
    // 批量记录日志
    public void batchRecordAccess(List<UserAccessLog> logs) {
        logClient.bulkLog(logs);
    }
    
    // DSL查询
    public ESDataList<UserAccessLog> queryAccessLogs(Long userId, Date startDate, Date endDate, 
                                                      int page, int size) {
        String dsl = "{" +
            "\"query\": {" +
            "  \"bool\": {" +
            "    \"must\": [" +
            "      {\"term\": {\"userId\": " + userId + "}}," +
            "      {\"range\": {\"accessTime\": {\"gte\": \"" + startDate + "\", \"lte\": \"" + endDate + "\"}}}" +
            "    ]" +
            "  }" +
            "}," +
            "\"sort\": [{\"accessTime\": \"desc\"}]," +
            "\"from\": " + (page - 1) * size + "," +
            "\"size\": " + size +
            "}";
        
        SearchResponse<UserAccessLog> response = logClient.dslQuery(UserAccessLog.class, dsl);
        return LogClient.mapQueryResponseToEDataList(response, (page - 1) * size, size);
    }
    
    // SQL转DSL查询
    public ESDataList<UserAccessLog> queryBySql(Long userId) {
        String sql = "SELECT * FROM user_access_log WHERE userId = " + userId + " ORDER BY accessTime DESC";
        String dsl = logClient.translateSqlToDsl(sql, 0, 20, true);
        
        SearchResponse<UserAccessLog> response = logClient.dslQuery(UserAccessLog.class, dsl);
        return LogClient.mapQueryResponseToEDataList(response, 0, 20);
    }
    
    // Scroll查询（大数据量）
    public void exportAllLogs(Date startDate, Date endDate) {
        String dsl = "{" +
            "\"query\": {" +
            "  \"range\": {\"accessTime\": {\"gte\": \"" + startDate + "\", \"lte\": \"" + endDate + "\"}}" +
            "}," +
            "\"size\": 1000" +  // 每批1000条
            "}";
        
        String index = logClient.getQueryIndexName(UserAccessLog.class);
        
        // 开启scroll查询
        ScrollResponse<UserAccessLog> scrollResponse = logClient.scrollQueryOpen(
            UserAccessLog.class, index, 60, dsl);
        
        while (scrollResponse != null && !scrollResponse.getDataList().isEmpty()) {
            // 处理数据
            List<UserAccessLog> logs = scrollResponse.getDataList();
            processLogs(logs);
            
            // 获取下一批
            if (scrollResponse.isHasMore()) {
                scrollResponse = logClient.scrollQueryNext(
                    UserAccessLog.class, index, scrollResponse.getScrollId(), 60);
            } else {
                break;
            }
        }
        
        // 关闭scroll
        if (scrollResponse != null) {
            logClient.scrollQueryClose(scrollResponse.getScrollId(), index);
        }
    }
    
    // 聚合查询
    public Map<String, Double> getActionStats(Date startDate, Date endDate) {
        String dsl = "{" +
            "\"query\": {" +
            "  \"range\": {\"accessTime\": {\"gte\": \"" + startDate + "\", \"lte\": \"" + endDate + "\"}}" +
            "}," +
            "\"aggs\": {" +
            "  \"action_count\": {" +
            "    \"terms\": {\"field\": \"action\"}" +
            "  }" +
            "}," +
            "\"size\": 0" +
            "}";
        
        SearchResponse<UserAccessLog> response = logClient.dslQuery(UserAccessLog.class, dsl);
        return LogClient.convertAggBucketFlatMap(response.getAggregations());
    }
    
    private void processLogs(List<UserAccessLog> logs) {
        // 处理日志数据
    }
}
```

