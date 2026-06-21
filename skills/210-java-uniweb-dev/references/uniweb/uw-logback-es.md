# uw-logback-es — Logback ES Appender

**Maven 坐标**: `com.umtone:uw-logback-es`

无需 Logstash，直接将日志批量发送到 Elasticsearch。基于 Logback 的自定义 Appender，支持批量提交、JMX 监控和异常堆栈压缩。

## 工作原理

- 业务线程在 `append()` 中把单条日志编码到临时 buffer，再加锁追加到全局 `okio.Buffer`（锁持有时间极短）。
- 后台监控线程 `ElasticsearchDaemonExporter`（默认 500ms 一轮）判断 flush 条件，满足时把 flush 任务提交到线程池。
- flush 触发条件二选一：buffer 达到 `maxKiloBytesOfBatch`（单位 KB），或距上次 flush 超过 `maxFlushInSeconds` 秒。
- flush 任务通过 ES `_bulk` 接口以 NDJSON 批量写入；HTTP 失败仅记录错误，本批数据会丢失（无重试/落盘）。
- 所有 JSON 字段值均经过转义，避免破坏 NDJSON 结构。

## 配置方式

在 `logback-spring.xml` 中配置。**标签名必须与 setter 名一致**（logback 按 setter 名注入，如 `<esServer>` 对应 `setEsServer`），否则属性不会生效。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>
    <springProfile name="prod">
        <appender name="ES" class="uw.logback.es.appender.ElasticSearchAppender">
            <!-- ES 服务地址(必填) -->
            <esServer>http://localhost:9200</esServer>
            <!-- ES bulk 接口路径(可选,默认 /_bulk?filter_path=took,errors) -->
            <esBulk>/_bulk?filter_path=took,errors</esBulk>
            <!-- 索引名(可选,默认等于 appInfo) -->
            <esIndex>my-app-logs</esIndex>
            <!-- 索引时间后缀(可选,支持 FastDateFormat 模式,如 _yyyy-MM-dd) -->
            <esIndexSuffix>_yyyy-MM-dd</esIndexSuffix>
            <!-- ES 认证用户名(可选,与密码同时配置才生效) -->
            <esUsername>elastic</esUsername>
            <!-- ES 认证密码(可选) -->
            <esPassword>changeme</esPassword>
            <!-- 应用名称(必填),写入 appInfo 字段;esIndex 为空时同时作为索引名 -->
            <appInfo>${APP_NAME:-unknown}</appInfo>
            <!-- 主机标识(可选),写入 appHost 字段 -->
            <appHost>${HOSTNAME:-unknown}</appHost>
            <!-- 批量提交触发阈值,单位 KB:buffer 达到此 KB 数触发 flush(默认 8192,即 8MB) -->
            <maxKiloBytesOfBatch>8192</maxKiloBytesOfBatch>
            <!-- 定时 flush 间隔,单位:秒(默认 10) -->
            <maxFlushInSeconds>10</maxFlushInSeconds>
            <!-- 批量 flush 线程池最大线程数(默认 5) -->
            <maxBatchThreads>5</maxBatchThreads>
            <!-- 批量 flush 线程池队列容量(默认 20) -->
            <maxBatchQueueSize>20</maxBatchQueueSize>
            <!-- 异常堆栈输出最大深度(默认 20,小于 10 会被提升到 10) -->
            <maxDepthPerThrowable>20</maxDepthPerThrowable>
            <!-- 折叠的堆栈类名前缀(逗号分隔,匹配帧被合并为 [N skipped]) -->
            <excludeThrowableKeys>java.base,org.spring,jakarta,org.apache,com.mysql,okhttp,com.fasterxml,uw.auth.service.filter</excludeThrowableKeys>
            <!-- 开启 JMX 监控(默认 false) -->
            <jmxMonitoring>true</jmxMonitoring>
        </appender>
        <root level="INFO">
            <appender-ref ref="ES"/>
        </root>
    </springProfile>
</configuration>
```

## 配置项清单

| XML 标签 | 属性 | 必填 | 默认值 | 说明 |
|---|---|---|---|---|
| `esServer` | esServer | 是 | - | ES 服务地址 |
| `esBulk` | esBulk | 否 | `/_bulk?filter_path=took,errors` | bulk 接口路径 |
| `esIndex` | esIndex | 否 | =appInfo | 索引名 |
| `esIndexSuffix` | esIndexSuffix | 否 | - | 索引时间后缀(FastDateFormat) |
| `esUsername` | esUsername | 否 | - | Basic 认证用户名 |
| `esPassword` | esPassword | 否 | - | Basic 认证密码 |
| `appInfo` | appInfo | 是 | - | 应用名(写入 appInfo;esIndex 为空时同时作索引名) |
| `appHost` | appHost | 否 | - | 主机标识(写入 appHost) |
| `maxKiloBytesOfBatch` | maxKiloBytesOfBatch | 否 | 8192 (8MB) | buffer 达到此 KB 数触发 flush |
| `maxFlushInSeconds` | maxFlushInSeconds | 否 | 10 | 定时 flush 间隔(秒) |
| `maxBatchThreads` | maxBatchThreads | 否 | 5 | flush 线程池最大线程数 |
| `maxBatchQueueSize` | maxBatchQueueSize | 否 | 20 | flush 线程池队列容量 |
| `maxDepthPerThrowable` | maxDepthPerThrowable | 否 | 20 | 堆栈输出最大深度 |
| `excludeThrowableKeys` | excludeThrowableKeys | 否 | 见代码 | 折叠的堆栈类名前缀(逗号分隔) |
| `jmxMonitoring` | jmxMonitoring | 否 | false | 是否开启 JMX |

## 写入 ES 的日志字段

| 字段 | 来源 | 备注 |
|---|---|---|
| `@timestamp` | 事件时间戳 | ISO8601 带毫秒/时区 |
| `appInfo` | appInfo | |
| `appHost` | appHost | |
| `level` | 日志级别 | INFO/WARN/ERROR 等 |
| `logger` | logger 名 | 通常为类全名 |
| `message` | 格式化后的消息 | 已 JSON 转义 |
| `stack_trace` | 异常代理 | 仅事件携带异常时存在,已折叠/转义 |

## 注意事项

- **HTTP 同步阻塞**：`HTTP_INTERFACE` 配置为信任全部证书 + 10s 超时 + 连接失败重试。ES 慢或不可达时，单次 flush 可能阻塞数十秒，占用 flush 线程。
- **失败丢数据**：flush 失败（网络异常或非 200）仅通过 logback StatusManager 记录错误，本批数据无重试/落盘兜底，会被丢弃。
- **静态共享堆栈配置**：`ThrowableProxyUtils` 的 `MaxDepthPerThrowable`/`ExcludeThrowableKeys` 为静态变量，多个 appender 实例配置时会互相覆盖。
- **JMX 错误观测**：appender 自身的错误（如配置缺失、flush 失败）写入 logback `StatusManager`，需在 logback 配置中加 `debug="true"` 或 `<statusListener>` 才能看到。
