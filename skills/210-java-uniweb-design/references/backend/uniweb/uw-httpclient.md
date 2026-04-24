# uw-httpclient — HTTP客户端

**Maven 坐标**: `com.umtone:uw-httpclient`

基于 OkHttp 的 HTTP 客户端类库。

| 特性 | 说明 |
|---|---|
| HTTP方法 | 完整 GET/POST/PUT/DELETE |
| 序列化 | 自动 JSON/XML 序列化 |
| 日志 | 完整请求/响应日志记录（HttpData） |
| SSL | 支持自签名证书（`SSLContextUtils`） |
| 文件 | 上传下载支持 |

## 核心入口类

```java
// JSON接口帮助类
public class JsonInterfaceHelper extends HttpInterface {
    public JsonInterfaceHelper();
    public JsonInterfaceHelper(HttpConfig httpConfig);
    public JsonInterfaceHelper(HttpConfig httpConfig, Class<? extends HttpData> httpDataCls);
    public JsonInterfaceHelper(HttpConfig httpConfig, Class<? extends HttpData> httpDataCls, 
                               HttpDataLogLevel httpDataLogLevel);
}

// XML接口帮助类
public class XmlInterfaceHelper extends HttpInterface {
    public XmlInterfaceHelper();
    public XmlInterfaceHelper(HttpConfig httpConfig);
    // ... 同上
}
```

## HttpInterface API

```java
public class HttpInterface {
    
    // ========== 通用请求方法 ==========
    
    // 自定义Request请求，返回HttpData
    public <D extends HttpData> D requestForData(final Request request);
    
    // 自定义Request请求，返回Entity（带响应体转换）
    public <D extends HttpData, T> HttpEntity<D, T> requestForEntity(final Request request, Class<T> responseType);
    public <D extends HttpData, T> HttpEntity<D, T> requestForEntity(final Request request, TypeReference<T> typeRef);
    public <D extends HttpData, T> HttpEntity<D, T> requestForEntity(final Request request, JavaType javaType);
    
    // ========== GET请求 ==========
    
    public <D extends HttpData> D getForData(String url);
    public <D extends HttpData> D getForData(String url, Map<String, String> queryParam);
    public <D extends HttpData> D getForData(String url, Map<String, String> headers, Map<String, String> queryParam);
    
    public <D extends HttpData, T> HttpEntity<D, T> getForEntity(String url, Class<T> responseType);
    public <D extends HttpData, T> HttpEntity<D, T> getForEntity(String url, Class<T> responseType, 
                                                                  Map<String, String> queryParam);
    public <D extends HttpData, T> HttpEntity<D, T> getForEntity(String url, Class<T> responseType, 
                                                                  Map<String, String> headers, 
                                                                  Map<String, String> queryParam);
    
    // ========== POST请求 ==========
    
    public <D extends HttpData> D postForData(String url, Object requestBody);
    public <D extends HttpData> D postForData(String url, Map<String, String> headers, Object requestBody);
    
    public <D extends HttpData, T> HttpEntity<D, T> postForEntity(String url, Class<T> responseType, 
                                                                   Object requestBody);
    public <D extends HttpData, T> HttpEntity<D, T> postForEntity(String url, Class<T> responseType, 
                                                                   Map<String, String> headers, 
                                                                   Object requestBody);
    
    // ========== PUT请求 ==========
    
    public <D extends HttpData> D putForData(String url, Object requestBody);
    public <D extends HttpData, T> HttpEntity<D, T> putForEntity(String url, Class<T> responseType, 
                                                                  Object requestBody);
    
    // ========== DELETE请求 ==========
    
    public <D extends HttpData> D deleteForData(String url);
    public <D extends HttpData, T> HttpEntity<D, T> deleteForEntity(String url, Class<T> responseType);
    
    // ========== 文件上传下载 ==========
    
    // 上传文件
    public <D extends HttpData, T> HttpEntity<D, T> uploadForEntity(String url, Class<T> responseType, 
                                                                     String formName, File file);
    public <D extends HttpData, T> HttpEntity<D, T> uploadForEntity(String url, Class<T> responseType, 
                                                                     String formName, File file, 
                                                                     Map<String, String> extraParam);
    
    // 下载文件
    public byte[] download(String url);
    public byte[] download(String url, Map<String, String> queryParam);
}

// HttpEntity包装类
public class HttpEntity<D extends HttpData, T> {
    private D httpData;    // HTTP请求响应数据
    private T body;        // 转换后的响应体
    
    public D getHttpData();
    public T getBody();
}

// HttpData接口（日志数据）
public interface HttpData {
    String getRequestUrl();        // 请求URL
    String getRequestMethod();     // 请求方法
    String getRequestHeader();     // 请求头
    String getRequestData();       // 请求数据
    long getRequestSize();         // 请求大小
    int getStatusCode();           // 响应状态码
    String getResponseData();      // 响应数据
    long getResponseSize();        // 响应大小
    String getResponseType();      // 响应类型
    String getErrorInfo();         // 错误信息
    Date getRequestDate();         // 请求时间
    Date getResponseDate();        // 响应时间
}
```

## HttpConfig配置

```java
public class HttpConfig {
    private long connectTimeout;           // 连接超时（毫秒）
    private long readTimeout;              // 读超时（毫秒）
    private long writeTimeout;             // 写超时（毫秒）
    private boolean retryOnConnectionFailure;  // 连接失败重试
    private int maxRequestsPerHost;        // 每主机最大并发
    private int maxRequests;               // 全局最大并发
    private int maxIdleConnections;        // 连接池最大空闲连接
    private long keepAliveTimeout;         // 空闲连接存活时间
    private SSLSocketFactory sslSocketFactory;  // SSL
    private X509TrustManager trustManager;      // 证书管理
    private HostnameVerifier hostnameVerifier;  // 主机名验证
    
    public static Builder builder();
}
```

## 使用示例

```java
@Service
public class HttpService {
    
    // 简单GET请求
    public User getUser(Long userId) {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        HttpEntity<HttpData, User> entity = helper.getForEntity(
            "https://api.example.com/users/" + userId,
            User.class
        );
        return entity.getBody();
    }
    
    // 带查询参数的GET请求
    public List<User> listUsers(String keyword, int page) {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        
        Map<String, String> params = new HashMap<>();
        params.put("keyword", keyword);
        params.put("page", String.valueOf(page));
        
        HttpEntity<HttpData, List<User>> entity = helper.getForEntity(
            "https://api.example.com/users",
            new TypeReference<List<User>>() {},
            params
        );
        return entity.getBody();
    }
    
    // POST请求（JSON）
    public User createUser(CreateUserRequest request) {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        HttpEntity<HttpData, User> entity = helper.postForEntity(
            "https://api.example.com/users",
            User.class,
            request
        );
        return entity.getBody();
    }
    
    // 自定义配置
    public void customConfigRequest() {
        HttpConfig config = HttpConfig.builder()
            .connectTimeout(5000)     // 5秒连接超时
            .readTimeout(30000)       // 30秒读超时
            .maxRequestsPerHost(50)   // 每主机50并发
            .maxIdleConnections(10)   // 10个空闲连接
            .build();
        
        JsonInterfaceHelper helper = new JsonInterfaceHelper(config);
        // ... 发送请求
    }
    
    // 文件上传
    public String uploadFile(File file) {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        HttpEntity<HttpData, UploadResult> entity = helper.uploadForEntity(
            "https://api.example.com/upload",
            UploadResult.class,
            "file",
            file
        );
        return entity.getBody().getUrl();
    }
    
    // 文件下载
    public byte[] downloadFile(String fileUrl) {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        return helper.download(fileUrl);
    }
    
    // 自定义Header
    public void requestWithHeaders() {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        
        Map<String, String> headers = new HashMap<>();
        headers.put("Authorization", "Bearer token123");
        headers.put("X-Custom-Header", "value");
        
        HttpEntity<HttpData, User> entity = helper.getForEntity(
            "https://api.example.com/users/1",
            User.class,
            headers,
            null
        );
    }
    
    // 使用OkHttp原生Request（高级用法）
    public void rawRequest() {
        JsonInterfaceHelper helper = new JsonInterfaceHelper();
        
        Request request = new Request.Builder()
            .url("https://api.example.com/users")
            .header("Authorization", "Bearer token")
            .post(RequestBody.create(MediaType.parse("application/json"), "{}"))
            .build();
        
        HttpData httpData = helper.requestForData(request);
        System.out.println(httpData.getResponseData());
    }
}
```

## SSL自签名证书支持

```java
// 信任所有证书（仅测试使用）
SSLContext sslContext = SSLContextUtils.createTrustAllSSLContext();

HttpConfig config = HttpConfig.builder()
    .sslSocketFactory(sslContext.getSocketFactory())
    .trustManager(SSLContextUtils.TRUST_ALL_MANAGER)
    .hostnameVerifier(SSLContextUtils.ALLOW_ALL_HOSTNAME_VERIFIER)
    .build();

JsonInterfaceHelper helper = new JsonInterfaceHelper(config);
```

