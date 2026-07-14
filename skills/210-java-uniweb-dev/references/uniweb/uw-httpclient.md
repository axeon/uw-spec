# uw-httpclient — HTTP客户端

**Maven 坐标**: `com.umtone:uw-httpclient`
**底层实现**: OkHttp 5.3.x（`okhttp-jvm`，服务端 JVM 版）+ Jackson（JSON / XML 序列化）

> 本文档方法签名与 `uw.httpclient.http.HttpInterface` 实际源码一一对应，AI 生成代码时请严格按此文档调用，勿臆造方法名。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| JSON 接口 | `new JsonInterfaceHelper()` / `new JsonInterfaceHelper(HttpConfig)` | 默认 mediaType=application/json |
| XML 接口 | `new XmlInterfaceHelper()` / `new XmlInterfaceHelper(HttpConfig)` | 默认 mediaType=application/xml |
| GET | `helper.getForEntity(url, Class)` 或 `getForData(url)` | ForEntity 返回 HttpEntity，取 `.getValue()` |
| GET + 查询参数 | `helper.getForEntity(url, Class, Map<String,String> queryParam)` | 仅 value 非空的参数才拼到 URL |
| GET + Header | `helper.getForEntity(url, Class, Map<String,String> headers, Map<String,String> queryParam)` | — |
| POST 表单 | `helper.postFormForEntity(url, Class, Map<String,String> formData)` | application/x-www-form-urlencoded |
| POST JSON body | `helper.postBodyForEntity(url, Class, Object body)` | body 自动序列化为配置的 mediaType |
| PUT / PATCH | `helper.putXxxForEntity(...)` / `helper.patchXxxForEntity(...)` | 与 POST 同构（Form/Body 两种） |
| DELETE | `helper.deleteForEntity(url, Class)` | 无 body，可带 queryParam |
| 文件上传（multipart） | `helper.postFormFileForEntity(url, Class, formData, fileData)` | fileData 值支持 `byte[]`、`File`、其它(toString) |
| 文件下载 | `helper.getForData(url)` → `.getResponseBytes()` | 取 HttpData 的 responseBytes（二进制友好） |
| 泛型响应 | `helper.getForEntity(url, new TypeReference<List<User>>(){})` | 或 `JavaType` 重载 |
| 自定义 OkHttp Request | `helper.requestForEntity(Request, Class)` | 走底层原生 Request |
| 直接拿 OkHttpClient | `helper.getOkHttpClient()` | 兜底，可自由操作 |
| 自定义超时/并发 | `HttpConfig.builder().connectTimeout(...).build()` 传入构造器 | 见 HttpConfig |
| 默认请求头 | `HttpConfig.builder().defaultHeaders(map)` | 所有请求自动追加，业务同名头覆盖 |
| Cookie 持久化（会话） | `HttpConfig.builder().cookieJar(jar)` | 默认不持久化 |
| 应用/网络拦截器 | `HttpConfig.builder().addInterceptor(...)` / `.addNetworkInterceptor(...)` | 仅作用于本实例 |
| 取响应头 | `data.getResponseHeaders()` / `((HttpDefaultData)data).getResponseHeader(name)` | 大小写不敏感，`Map<String,List<String>>` |
| 取重试/重定向次数 | `data.getRetryCount()` | 自动启用，含连接失败重试与 follow-up |
| 自签名 HTTPS | `SSLContextUtils.getTruestAllSocketFactory()` + `getTrustAllManager()` | 见 SSL 章节 |

## 包路径

| 类 | 全限定名 |
|---|---|
| `HttpInterface` | `uw.httpclient.http.HttpInterface` |
| `JsonInterfaceHelper` | `uw.httpclient.json.JsonInterfaceHelper` |
| `XmlInterfaceHelper` | `uw.httpclient.xml.XmlInterfaceHelper` |
| `HttpConfig` | `uw.httpclient.http.HttpConfig` |
| `HttpEntity` | `uw.httpclient.http.HttpEntity` |
| `HttpData` / `HttpDefaultData` | `uw.httpclient.http.HttpData` / `uw.httpclient.http.HttpDefaultData` |
| `HttpDataLogLevel` | `uw.httpclient.http.HttpDataLogLevel` |
| `HttpDataProcessor` | `uw.httpclient.http.HttpDataProcessor` |
| `DataObjectMapper` | `uw.httpclient.http.DataObjectMapper` |
| `SSLContextUtils` | `uw.httpclient.util.SSLContextUtils` |
| `MediaTypes` | `uw.httpclient.util.MediaTypes` |
| `HttpBasicAuthenticator` | `uw.httpclient.util.HttpBasicAuthenticator` |
| `HttpRequestException` | `uw.httpclient.exception.HttpRequestException` |
| `DataMapperException` | `uw.httpclient.exception.DataMapperException` |

## 核心入口类

```java
// JSON（默认）
JsonInterfaceHelper helper = new JsonInterfaceHelper();
JsonInterfaceHelper helper = new JsonInterfaceHelper(HttpConfig);

// XML
XmlInterfaceHelper helper = new XmlInterfaceHelper();
XmlInterfaceHelper helper = new XmlInterfaceHelper(HttpConfig);
```

完整构造（JSON）：
```java
public JsonInterfaceHelper(HttpConfig httpConfig,
                           Class<? extends HttpData> httpDataCls,   // 自定义日志类，null 用 HttpDefaultData
                           HttpDataLogLevel httpDataLogLevel,        // 日志级别，null 默认 RECORD_RESPONSE
                           HttpDataProcessor httpDataProcessor);     // 数据处理器（加解密/日志等），可 null
```

## 方法命名规律

```
{httpMethod}{请求体形式}{返回形式}
  httpMethod: get / post / put / patch / delete
  请求体形式: Form(表单) | Body(请求体) | FormFile(含文件上传) | 无(get/delete)
  返回形式:   ForData(返回 HttpData) | ForEntity(返回 HttpEntity，含反序列化对象)
```

例：`postBodyForEntity` = POST + JSON body + 返回 HttpEntity；`postFormFileForData` = POST multipart + 返回 HttpData。

每个 `ForEntity` 方法都提供 **三套响应类型重载**：`Class<T>`、`TypeReference<T>`、`JavaType`；每套再各有一个带 `Map<String,String> headers` 的重载。下表为代表性签名，其余按规律类推。

## HttpInterface 方法签名

> 包路径：`uw.httpclient.http.HttpInterface`

### ForEntity（返回 `HttpEntity<D, T>`，含反序列化对象）

| 方法（Class 重载示例） | 说明 |
|------|------|
| `getForEntity(url, Class<T>)` | GET |
| `getForEntity(url, Class<T>, Map queryParam)` | GET + 查询参数 |
| `getForEntity(url, Class<T>, Map headers, Map queryParam)` | GET + Header + 参数 |
| `postFormForEntity(url, Class<T>, Map<String,String> formData)` | POST 表单 |
| `postFormForEntity(url, Class<T>, Map headers, Map formData)` | POST 表单 + Header |
| `postBodyForEntity(url, Class<T>, Object body)` | POST JSON/请求体 |
| `postBodyForEntity(url, Class<T>, Map headers, Object body)` | POST 请求体 + Header |
| `postFormFileForEntity(url, Class<T>, Map formData, Map<String,Object> fileData)` | POST multipart 上传 |
| `postFormFileForEntity(url, Class<T>, Map headers, Map formData, Map fileData)` | multipart 上传 + Header |
| `putFormForEntity(...)` / `putBodyForEntity(...)` | PUT，与 POST 同构 |
| `patchFormForEntity(...)` / `patchBodyForEntity(...)` | PATCH，与 POST 同构 |
| `deleteForEntity(url, Class<T>)` | DELETE |
| `deleteForEntity(url, Class<T>, Map queryParam)` | DELETE + 参数 |
| `deleteForEntity(url, Class<T>, Map headers, Map queryParam)` | DELETE + Header + 参数 |
| `requestForEntity(Request, Class<T>)` | 自定义 OkHttp Request + 反序列化 |

泛型/复杂类型响应：
```java
// TypeReference
helper.getForEntity(url, new TypeReference<List<User>>(){});
helper.getForEntity(url, new TypeReference<Map<String,User>>(){}, params);
// JavaType
JavaType type = helper.getJsonMapper().constructParametricType(List.class, User.class);
helper.getForEntity(url, type);
```

### ForData（返回 `HttpData`，不做响应反序列化，二进制友好）

| 方法 | 说明 |
|------|------|
| `getForData(url)` / `getForData(url, Map queryParam)` / `getForData(url, Map headers, Map queryParam)` | GET，可用于下载 |
| `postFormForData(url, Map formData)` / `(url, Map headers, Map formData)` | POST 表单 |
| `postBodyForData(url, Object body)` / `(url, Map headers, Object body)` | POST 请求体 |
| `postFormFileForData(url, Map formData, Map<String,Object> fileData)` / `(..., Map headers, ...)` | multipart 上传 |
| `putFormForData(...)` / `putBodyForData(...)` | PUT |
| `patchFormForData(...)` / `patchBodyForData(...)` | PATCH |
| `deleteForData(url)` / `(url, Map queryParam)` / `(url, Map headers, Map queryParam)` | DELETE |
| `requestForData(Request)` | 自定义 OkHttp Request |

## HttpEntity

> 包路径：`uw.httpclient.http.HttpEntity`

由 `*ForEntity` 方法返回，不直接 new。

| 方法 | 返回 | 说明 |
|------|------|------|
| `getHttpData()` | `HttpData` | 请求/响应日志数据（URL/状态码/响应字节等） |
| `getValue()` | V | **反序列化后的响应体**（注意是 getValue，不是 getBody） |

## HttpData 字段

| 字段 | 类型 | 说明 |
|---|---|---|
| `requestUrl` / `requestMethod` / `requestHeader` | String | 请求 URL / 方法 / 头 |
| `requestData` / `requestSize` | String / long | 请求体（受日志级别控制）/ 大小 |
| `statusCode` | int | HTTP 状态码 |
| `responseBytes` | byte[] | **响应原始字节（二进制优先）** |
| `getResponseData()` | String | 懒转换为字符串（UTF-8），优先返回已设置的 responseData |
| `responseSize` / `responseType` | long / String | 响应大小 / Content-Type |
| `responseHeaders` | `Map<String,List<String>>` | **完整响应头（大小写不敏感、不可变）**，同名多值头（如 `Set-Cookie`）用 List 表达 |
| `getResponseHeader(name)` | String | `HttpDefaultData` 便捷方法：取单个响应头首值，大小写不敏感 |
| `responseMessage` | String | HTTP 状态消息（reason phrase，如 "Not Found"） |
| `elapsedMillis` | long | 请求整体耗时（毫秒，基于 OkHttp 时间戳，含连接/重试/传输耗时），-1 未设置 |
| `retryCount` | int | **重试/重定向次数**（物理网络请求次数 - 1，含连接失败重试与 follow-up），自动启用，默认 0 |
| `errorInfo` | String | 错误信息 |
| `requestDate` / `responseDate` | Date | 请求/响应时间 |

> 自定义日志类：实现 `HttpData` 接口（或继承 `HttpDefaultData`），作为日志载体直接复用，避免拷贝。

## HttpDataLogLevel

> 包路径：`uw.httpclient.http.HttpDataLogLevel`

**响应数据始终记录**；本级别仅控制是否额外记录请求体。

| 枚举 | 是否记录请求体 | 说明 |
|---|---|---|
| `RECORD_RESPONSE`（默认） | 否 | 仅记录响应 |
| `RECORD_REQUEST` | 是 | 记录响应 + 请求体 |
| `RECORD_ALL` | 是 | 同 RECORD_REQUEST |

## HttpDataProcessor

> 包路径：`uw.httpclient.http.HttpDataProcessor<D extends HttpData, T>`

用于加解密、签名、日志上报等场景。三个回调：

```java
public interface HttpDataProcessor<D extends HttpData, T> {
    // 请求发送前（可拿到 requestBody / formData / headers，业务侧原始参数）
    void requestProcess(String requestBody, Map<String,String> formData, Map<String,String> headers);
    // 请求发送前（完整 Request 版，默认空实现，按需覆写）
    // 拿到构建完成的 okhttp3.Request（含已合并的业务头 + defaultHeaders、最终解析的 HttpUrl、method、body 引用）
    // 注意：不含 OkHttp 网络层注入的头（Host/Content-Length/Connection/Cookie 等），那些只在网络拦截器里可见
    // 适合做签名/验签/链路追踪。抛出的 DataMapperException 会直接冒泡（不包装），交由上层处理。
    default void requestProcess(okhttp3.Request request) {}
    // 收到响应后（可拿到 httpData 与响应 headers）
    void responseProcess(D httpData, Headers headers);
    // 请求完成后（通常用于日志上报，t 为反序列化对象或 null）
    void postProcess(D httpData, T t);
}
```

## HttpConfig

> 包路径：`uw.httpclient.http.HttpConfig`

```java
HttpConfig config = HttpConfig.builder()
    .connectTimeout(5000)
    .readTimeout(30000)
    .writeTimeout(10000)
    .retryOnConnectionFailure(false)   // 幂等性敏感的接口务必设为 false
    .maxRequests(64)                    // 全局最大并发（独立 dispatcher，互不串扰）
    .maxRequestsPerHost(32)             // 每主机最大并发
    .maxIdleConnections(50)             // 连接池最大空闲连接
    .keepAliveTimeout(300000)           // 空闲连接存活毫秒
    // .defaultHeaders(headerMap)        // 默认请求头，所有请求自动追加，业务同名头覆盖
    // .cookieJar(cookieJar)             // Cookie 持久化（会话/登录态）
    // .addInterceptor(appInterceptor)   // 应用拦截器（重试/重定向前介入）
    // .addNetworkInterceptor(netInterceptor) // 网络拦截器（重试/重定向后介入）
    // .sslSocketFactory(...) / .trustManager(...) / .hostnameVerifier(...)
    .build();
```

| 属性 | 类型 | 说明 |
|------|------|------|
| connectTimeout | long | 连接超时（毫秒） |
| readTimeout | long | 读超时（毫秒） |
| writeTimeout | long | 写超时（毫秒） |
| retryOnConnectionFailure | boolean | 连接失败重试（幂等敏感接口设 false） |
| maxRequests / maxRequestsPerHost | int | 全局/每主机最大并发 |
| maxIdleConnections / keepAliveTimeout | int / long | 连接池空闲连接数/存活时间 |
| defaultHeaders | `Map<String,String>` | 默认请求头，所有请求自动追加，业务侧同名头覆盖 |
| cookieJar | `okhttp3.CookieJar` | Cookie 持久化，为 null 时 OkHttp 默认不持久化 |
| interceptors / networkInterceptors | `List<Interceptor>` | 应用/网络拦截器链，仅作用于本实例 |
| sslSocketFactory / trustManager / hostnameVerifier | — | SSL 配置 |

> 自 2026/06 重构起：每个 HttpInterface 实例使用**独立的 Dispatcher**，配置 maxRequests/maxRequestsPerHost 不会污染全局或其他实例。

## SSL 自签名证书

> 包路径：`uw.httpclient.util.SSLContextUtils`

实际 API（注意方法名）：

| 方法 | 返回 | 说明 |
|---|---|---|
| `getTrustAllManager()` | `X509TrustManager` | 信任全部证书的 Manager |
| `getTruestAllSocketFactory()` | `SSLSocketFactory` | 信任全部证书的 SocketFactory |
| `getAllTruestContext()` | `SSLContext` | 信任全部证书的 SSLContext |

```java
HttpConfig config = HttpConfig.builder()
    .sslSocketFactory(SSLContextUtils.getTruestAllSocketFactory())
    .trustManager(SSLContextUtils.getTrustAllManager())
    .hostnameVerifier((hostName, session) -> true)
    .build();
JsonInterfaceHelper helper = new JsonInterfaceHelper(config);
```

> ⚠️ 信任全部证书仅适用于内网自签名/测试环境，生产环境须使用受信任证书库。

## HTTP Basic 认证

> 包路径：`uw.httpclient.util.HttpBasicAuthenticator`

```java
HttpConfig config = HttpConfig.builder()
    // ... 其他配置
    .build();
// Basic 认证通过 OkHttpClient 的 authenticator 设置，需自定义 Builder 或直接操作 client：
JsonInterfaceHelper helper = new JsonInterfaceHelper(config);
helper.getOkHttpClient().dispatcher(); // 兜底：可直接拿 client 做更多自定义
```

> `HttpBasicAuthenticator` 实现了 OkHttp `Authenticator`，已对 401 重试做防护（凭证无效时放弃，不再死循环）。

## 异常

| 异常 | 说明 |
|---|---|
| `HttpRequestException`（继承 `TaskPartnerException`） | 请求阶段异常：网络错误、URL 非法、IO 失败 |
| `DataMapperException`（继承 `TaskDataException`） | 序列化/反序列化异常：JSON/XML 转换失败、null 内容 |

两者均为 `RuntimeException` 子类（非受检，无需 throws 声明）。

**异常分类语义**（与 uw-task 调度框架对齐）：
- `HttpRequestException` → 接口方/网络错误（外部原因），uw-task 通常会重试。
- `DataMapperException` → 数据/序列化错误（内部原因），uw-task 通常不重试。
- `HttpDataProcessor` 抛出的 `DataMapperException` 会**直接冒泡**，不被本库吞掉或改写异常类型。

> `uw.task.exception` 包下的 `TaskDataException`/`TaskPartnerException` 是本库为避免强依赖 uw-task 而保留的同名桥接副本。运行在 uw-task 环境时类加载器优先加载 uw-task 的同名类，异常分类语义生效；独立运行回退到本地副本。

## 完整使用示例

```java
import uw.httpclient.json.JsonInterfaceHelper;
import uw.httpclient.http.*;
import tools.jackson.core.type.TypeReference;
import java.util.*;

public class HttpHelper {
    private static final JsonInterfaceHelper http = new JsonInterfaceHelper(
        HttpConfig.builder()
            .connectTimeout(5000).readTimeout(30000).writeTimeout(10000)
            .retryOnConnectionFailure(false)
            .build());

    // GET → 对象
    public static User getUser(long userId) {
        return http.getForEntity("https://api.example.com/users/" + userId, User.class).getValue();
    }

    // GET → 泛型列表 + 查询参数
    public static List<User> listUsers(String keyword, int page) {
        Map<String, String> params = new HashMap<>();
        params.put("keyword", keyword);
        params.put("page", String.valueOf(page));
        return http.getForEntity("https://api.example.com/users",
            new TypeReference<List<User>>() {}, params).getValue();
    }

    // POST JSON body
    public static User createUser(CreateUserRequest req) {
        return http.postBodyForEntity("https://api.example.com/users", User.class, req).getValue();
    }

    // POST 表单
    public static Token login(String user, String pass) {
        Map<String, String> form = new HashMap<>();
        form.put("username", user);
        form.put("password", pass);
        return http.postFormForEntity("https://api.example.com/login", Token.class, form).getValue();
    }

    // 自定义 Header
    public static User requestWithHeaders() {
        Map<String, String> headers = new HashMap<>();
        headers.put("Authorization", "Bearer token123");
        return http.getForEntity("https://api.example.com/users/1",
            User.class, headers, null).getValue();
    }

    // 文件上传（multipart）
    public static UploadResult upload(File file) {
        Map<String, Object> fileData = new HashMap<>();
        fileData.put("file", file);           // 值支持 byte[] / File / 其它(toString)
        return http.postFormFileForEntity("https://api.example.com/upload",
            UploadResult.class, null, fileData).getValue();
    }

    // 文件下载
    public static byte[] download(String fileUrl) {
        return http.getForData(fileUrl).getResponseBytes();
    }

    // 不关心响应体、只要状态码/日志
    public static HttpData delete(long userId) {
        return http.deleteForData("https://api.example.com/users/" + userId);
    }
}
```

## 关键约束（AI 生成代码必读）

1. **没有 `postForEntity`/`putForEntity`**：POST 请求体是 `postBodyForEntity`，表单是 `postFormForEntity`，含文件是 `postFormFileForEntity`。PUT/PATCH 同理（`putBodyForEntity`/`patchBodyForEntity` 等）。
2. **没有 `uploadForEntity`/`download`/`getBody()`**：上传用 `postFormFileForEntity`/`postFormFileForData`，下载用 `getForData(...).getResponseBytes()`，取反序列化结果用 `getValue()`。
3. **`HttpEntity` 取响应体是 `getValue()`**，不是 `getBody()`。
4. **`retryOnConnectionFailure`**：对下单/支付等幂等敏感接口必须设为 `false`，否则重试可能造成重复调用。
5. **泛型响应**：用 `TypeReference` 或 `JavaType` 重载，不能直接传 `List<User>.class`。
6. **二进制响应**：用 `getForData` + `getResponseBytes()`，避免不必要的字符串转换。
7. **取响应头**：用 `data.getResponseHeaders()`（`Map<String,List<String>>`，大小写不敏感）或 `HttpDefaultData.getResponseHeader(name)`，**不要**从 `getResponseType()`（只有 Content-Type）取。
8. **`retryCount` 自动启用**：无需配置，所有请求都能拿到。语义是"物理网络请求次数 - 1"，含连接失败重试与重定向 follow-up；即便 `retryOnConnectionFailure=false`，发生 302 重定向跟随时 retryCount 也会 >0。
9. **异常不吞**：`HttpDataProcessor` 抛出的 `DataMapperException` 会**直接冒泡**（不包装成 `HttpRequestException`），交由上层（uw-task/业务）按异常分类处理。生成 Processor 实现时不要在内部吞掉异常。
10. **OkHttp 5.x artifact**：服务端必须用 `okhttp-jvm`（5.0+ 主 `okhttp` artifact 为空 jar）。本库已通过 `uw-httpclient` 传递引入，下游无需自行声明。
