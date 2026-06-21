# uw-log-es — Elasticsearch 日志客户端

**Maven 坐标**: `com.umtone:uw-log-es`
**包路径根**: `uw.log.es`
**配置前缀**: `uw.log.es`
**自动装配入口**: `uw.log.es.LogClientAutoConfiguration`（Spring `@Configuration`，容器销毁时通过 `@PreDestroy` 关闭写入链路）

## 快速决策

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 获取客户端 | `LogClient.getInstance()` | 静态单例，Spring 装配后可用 |
| 注册日志类型 | `logClient.regLogObject(Class)` | 启动期（static 块 / @PostConstruct）完成，索引名按类名 lower_underscore 推导 |
| 自定义索引名 | `regLogObjectWithIndexName(Class, index)` | — |
| 按时间分索引 | `regLogObjectWithIndexPattern(Class, "yyyyMM")` | 写入追加时间后缀，查询用 `原名_*` |
| 单条写入 | `logClient.log(logObject)` | 须继承 LogBaseVo；`logLevel <= NONE(-1)` 不写入 |
| 批量写入 | `logClient.bulkLog(List)` | 空列表直接返回 |
| DSL 查询 | `logClient.dslQuery(Class, dsl)` | 用注册的查询索引；返回 SearchResponse |
| 指定索引 DSL | `logClient.dslQuery(Class, index, dsl)` | 自定义索引查询 |
| SQL 转 DSL | `logClient.translateSqlToDsl(sql, from, size, trueCount)` | SQL 不可含 limit；表名需转义 |
| 大数据量导出 | `scrollQueryOpen → scrollQueryNext → scrollQueryClose` | **必须关闭 scroll**；dsl 不可含 from |
| 聚合单值 | `LogClient.getAggValue(aggMap, name)` | 静态方法，缺失返回 0d |
| 聚合桶拉平 | `LogClient.convertAggBucketFlatMap(aggMap)` | agg+key → 值 |
| 转分页 | `LogClient.mapQueryResponseToPageList(resp, from, size)` | 返回 PageList，永不 null |

> **READ_ONLY 模式**：配置 `mode: READ_ONLY` 时不启动后台写入线程，仅可查询。写入方法静默返回。

## 配置

```yaml
uw:
  log:
    es:
      server: http://localhost:9200   # 为空时不写入，仅查询
      username: admin                  # 与 password 同时配置时启用 Http Basic
      password: admin
      mode: READ_WRITE                 # READ_ONLY | READ_WRITE
      app-info-overwrite: true         # 用应用信息覆写日志体 appInfo/appHost
      max-flush-in-seconds: 10         # 后台 flush 间隔（秒）
      max-kilo-bytes-of-batch: 8192    # 触发立即 flush 的 buffer 阈值（KB）
      max-batch-threads: 5             # 批量提交线程数
      max-batch-queue-size: 20         # 批量线程池队列容量
      connect-timeout: 30000           # 连接超时（毫秒）
      read-timeout: 30000              # 读超时（毫秒）
      write-timeout: 30000             # 写超时（毫秒）
      es-bulk: /_bulk?filter_path=took,errors  # bulk api 路径
```

`appInfo` 取自 `${spring.application.name}:${project.version}`，`appHost` 取自 `${spring.cloud.nacos.discovery.ip}:${server.port}`，均有默认值，未接 nacos 不报错。

## LogClient API

> **包路径**: `uw.log.es.LogClient`

**实例方法**

| 方法 | 返回 | 说明 |
|------|------|------|
| `regLogObject(Class)` | void | 注册，索引名按类名推导 |
| `regLogObjectWithIndexName(Class, index)` | void | 自定义索引名 |
| `regLogObjectWithIndexPattern(Class, indexPattern)` | void | 自定义时间滚动模式 |
| `regLogObjectWithIndexNameAndPattern(Class, index, indexPattern)` | void | 索引名 + 模式 |
| `getRawIndexName(Class)` | String | 原始索引名；未注册 null |
| `getQuotedRawIndexName(Class)` | String | `"原始名"`（裸引号，用于写入 _index） |
| `getQueryIndexName(Class)` | String | 查询索引名（有模式时为 `原名_*`）；未注册 null |
| `getQuotedQueryIndexName(Class)` | String | `\"原名*\"`（转义引号，用于 ES SQL from 子句） |
| `log(LogBaseVo)` | void | 单条写入，logLevel<=NONE 跳过 |
| `bulkLog(List<LogBaseVo>)` | void | 批量写入，空列表跳过 |
| `dslQuery(Class, dsl)` | `SearchResponse<T>` | 用注册查询索引查询；异常 null |
| `dslQuery(Class, index, dsl)` | `SearchResponse<T>` | 指定索引查询；异常 null |
| `translateSqlToDsl(sql, from, size, trueCount)` | String | SQL→DSL；from>0 附加 from，trueCount 附加 track_total_hits |
| `scrollQueryOpen(Class, index, expireSeconds, dsl)` | `ScrollResponse<T>` | 开启游标，dsl 不可含 from |
| `scrollQueryNext(Class, index, scrollId, expireSeconds)` | `ScrollResponse<T>` | 取下一批；index 仅辨识用 |
| `scrollQueryClose(scrollId, index)` | `DeleteScrollResponse` | 关闭游标；index 仅辨识用 |

**静态工具方法**

| 方法 | 返回 | 说明 |
|------|------|------|
| `getInstance()` | LogClient | 全局单例 |
| `mapQueryResponseToPageList(resp, from, size)` | `PageList<T>` | SearchResponse→PageList，永 null |
| `getAggValue(aggMap, name)` | double | 单值聚合，缺失 0d |
| `convertAggBucketListMap(aggMap)` | `Map<String,List<Map<String,Object>>>` | agg→桶列表，桶含 name/count/子聚合 |
| `convertAggBucketFlatMap(aggMap)` | `Map<String,Double>` | agg+桶key→值，单值聚合 agg→值 |
| `convertAggBucketAggBucketFlatMap(aggMap)` | `Map<String,Map<String,Map<String,Double>>>` | 三层拉平：agg→桶key→子聚合 |

## LogBaseVo

> **包路径**: `uw.log.es.LogBaseVo`（抽象基类，日志对象须继承）

| 字段 | Java 类型 | ES 字段 | 说明 |
|------|----------|---------|------|
| timestamp | String | `@timestamp` | ISO8601 时间，写入时自动补齐 |
| appInfo | String | appInfo | 应用名:版本，appInfoOverwrite 时覆写 |
| appHost | String | appHost | ip:port，appInfoOverwrite 时覆写 |
| logLevel | int | logLevel | 见 LogLevel；<=NONE 不写入 |

```java
@Data
public static class UserAccessLog extends LogBaseVo {
    private Long userId;
    private String action;
}
```

> LogLevel 枚举（`uw.log.es.vo.LogLevel`）：NONE(-1) / BASE(0) / REQUEST(1) / RESPONSE(2) / ALL(3)

## SearchResponse\<T\>

> **包路径**: `uw.log.es.vo.SearchResponse`

| 字段 | 类型 | 说明 |
|------|------|------|
| took | long | 请求耗时（毫秒） |
| timedOut | boolean | 是否超时 |
| shards | Shards | 分片信息（total/successful/skipped/failed） |
| hitResponse | `HitResponse<T>` | 命中结果 |
| aggregations | `Map<String, Aggregation>` | 聚合结果 |

**HitResponse\<T\>**：total(`Total`：value[long] + relation) / maxScore(Float) / hits(`List<Hit<T>>`)

**Hit\<T\>**：index / id / type / score(Float) / source(T) / fields / highlight

**Aggregation**：value(double) / valueAsString / count(long) / min / max / avg / sum / docCountErrorUpperBound / sumOtherDocCount / buckets(`List<Bucket>`)

**Bucket**：key / keyAsString / docCount(long) / subAggregations(`Map<String,Aggregation>`，经 `@JsonAnySetter` 捕获桶下子聚合)

## ScrollResponse\<T\>

> **包路径**: `uw.log.es.vo.ScrollResponse`（继承 SearchResponse）

| 字段 | 类型 | 说明 |
|------|------|------|
| scrollId | String | 游标 ID，传给 scrollQueryNext / scrollQueryClose |

> 命中数据通过继承的 `hitResponse.hits` 获取，**没有** dataList / hasMore / total 字段。判断是否结束：取下一批后 `hitResponse.hits` 为空即结束。

## BulkResponse / DeleteScrollResponse

> **包路径**: `uw.log.es.vo`

- **BulkResponse**：errors(boolean) / took(long) — bulk 提交后用于判断是否出错
- **DeleteScrollResponse**：succeeded(boolean) / numFreed(int) — 关闭游标结果

## PageList\<T\>

> 来自 `uw-common`（`uw.common.data.PageList`）

| 方法 | 返回 | 说明 |
|------|------|------|
| `list()` | `List<T>` | 数据列表 |
| `size()` | int | 当前页条数 |
| `sizeAll()` | int | 总条数（int） |
| `get(int)` | T | 按页内索引取元素 |
| `isEmpty()` / `isNotEmpty()` | boolean | — |

## Helper 使用示例

```java
public class AccessLogHelper {
    private static final LogClient logClient = LogClient.getInstance();

    @Data
    public static class UserAccessLog extends LogBaseVo {
        private Long userId;
        private String action;
        private String ip;
        private long accessTime;   // 毫秒时间戳
        private String result;
    }

    static {
        // 启动期注册，按时间分索引（yyyyMM）
        logClient.regLogObjectWithIndexPattern(UserAccessLog.class, "yyyyMM");
    }

    // 写入单条日志（logLevel>0 才写入）
    public static void recordAccess(long userId, String action, String ip, String result) {
        UserAccessLog log = new UserAccessLog();
        log.setLogLevel(LogLevel.BASE.getValue());
        log.setUserId(userId);
        log.setAction(action);
        log.setIp(ip);
        log.setAccessTime(System.currentTimeMillis());
        log.setResult(result);
        logClient.log(log);
    }

    // 批量写入
    public static void batchRecord(List<UserAccessLog> logs) {
        logClient.bulkLog(logs);
    }

    // DSL 查询 + 分页
    public static PageList<UserAccessLog> queryLogs(long userId, int page, int size) {
        int from = (page - 1) * size;
        String dsl = "{\"query\":{\"term\":{\"userId\":" + userId + "}},"
                + "\"sort\":[{\"accessTime\":\"desc\"}],\"from\":" + from + ",\"size\":" + size + "}";
        SearchResponse<UserAccessLog> resp = logClient.dslQuery(UserAccessLog.class, dsl);
        return LogClient.mapQueryResponseToPageList(resp, from, size);
    }

    // SQL 转 DSL 查询（表名需用 getQuotedQueryIndexName 转义）
    public static PageList<UserAccessLog> queryBySql(long userId, int page, int size) {
        int from = (page - 1) * size;
        String index = logClient.getQuotedQueryIndexName(UserAccessLog.class);
        String dsl = logClient.translateSqlToDsl(
                "select * from " + index + " where userId = " + userId, from, size, true);
        SearchResponse<UserAccessLog> resp = logClient.dslQuery(
                UserAccessLog.class, logClient.getQueryIndexName(UserAccessLog.class), dsl);
        return LogClient.mapQueryResponseToPageList(resp, from, size);
    }

    // Scroll 大数据量导出
    public static void exportAll(long startMs, long endMs) {
        String index = logClient.getQueryIndexName(UserAccessLog.class);
        String dsl = "{\"query\":{\"range\":{\"accessTime\":{\"gte\":" + startMs + ",\"lte\":" + endMs + "}}},\"size\":1000}";
        ScrollResponse<UserAccessLog> scroll = logClient.scrollQueryOpen(UserAccessLog.class, index, 60, dsl);
        try {
            while (scroll != null && scroll.getHitResponse() != null
                    && !scroll.getHitResponse().getHits().isEmpty()) {
                for (SearchResponse.Hit<UserAccessLog> hit : scroll.getHitResponse().getHits()) {
                    process(hit.getSource());
                }
                scroll = logClient.scrollQueryNext(UserAccessLog.class, index, scroll.getScrollId(), 60);
            }
        } finally {
            if (scroll != null) {
                logClient.scrollQueryClose(scroll.getScrollId(), index);
            }
        }
    }

    // 聚合查询
    public static Map<String, Double> countByAction() {
        String dsl = "{\"size\":0,\"aggs\":{\"by_action\":{\"terms\":{\"field\":\"action\"}}}}";
        SearchResponse<UserAccessLog> resp = logClient.dslQuery(UserAccessLog.class, dsl);
        return resp == null ? Collections.emptyMap()
                : LogClient.convertAggBucketFlatMap(resp.getAggregations());
    }
}
```

## 注意事项

1. **必须注册**：未注册的日志类型写入会输出 warn 并丢弃，查询索引返回 null。
2. **scroll 必须关闭**：未关闭会占用 ES 端上下文资源，建议 try-finally。
3. **scroll dsl 不可含 from**：ES 限制。
4. **SQL 不可含 limit**：由 `resultNum` 参数控制。
5. **批量写入不丢数据**：buffer 在后台按时间/字节阈值 flush；应用关闭时 `@PreDestroy` 会 flush 残留 buffer 并等待批量线程结束。
6. **int 溢出**：`track_total_hits=true` 时 total.value 可超 Integer.MAX_VALUE，已为 long；PageList.sizeAll 仍为 int。
