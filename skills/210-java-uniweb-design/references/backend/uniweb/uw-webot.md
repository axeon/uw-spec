### uw-webot — Web自动化框架

**Maven 坐标**: `com.umtone:uw-webot`

基于 Microsoft Playwright 的高性能 Web 自动化框架，采用 Hybrid 混合模式设计，支持浏览器实例复用和页签级隔离，提供验证码识别、反检测、代理池等企业级功能。

**配置前缀**: `uw.webot`

```yaml
uw:
  webot:
    enabled: true
    bot-pool:
      max-browsers-per-group: 5      # 每种浏览器类型最大实例数
      max-tabs-per-browser: 20       # 每个浏览器实例最大页签数
    session:
      distributed: false             # 是否启用分布式会话
      default-session:
        expire-time: P30D            # 会话默认过期时间（ISO-8601格式）
    # 验证码配置
    captcha:
      default:
        service-type: OCR            # OCR/TWOCAPTCHA/CAPSOLVER
        api-key: ""                  # 第三方服务API密钥
    # 代理配置
    proxy:
      default:
        type: HTTP                   # HTTP/HTTPS/SOCKS4/SOCKS5
        servers:
          - host: 127.0.0.1
            port: 8080
            username: ""             # 可选认证
            password: ""
    # 反检测配置
    stealth:
      default:
        enabled: true
        webdriver-hide: true
        webgl-spoof: true
        canvas-noise: true
```

#### 核心概念

**Hybrid 混合模式架构**:
```
WebotManager (单例)
    ├── BrowserBotPool
    │       ├── BrowserGroup (chromium)
    │       │       ├── BrowserInstance 1 → 多个 BrowserTab
    │       │       └── BrowserInstance 2 → 多个 BrowserTab
    │       ├── BrowserGroup (firefox)
    │       └── BrowserGroup (webkit)
    ├── CaptchaManager
    ├── StealthManager
    └── ProxyManager
```

- **BrowserGroup**: 按浏览器类型分组（Chromium/Firefox/WebKit）
- **BrowserInstance**: 实际的浏览器进程实例
- **BrowserTab**: 页签级隔离单元（独立的 Cookie/Storage/代理）

#### WebotManager API

```java
public class WebotManager {
    
    // ========== 单例获取 ==========
    public static WebotManager getInstance();
    
    // ========== 会话管理 ==========
    
    // 创建新会话
    public WebotSession createSession(SessionConfig sessionConfig);
    
    // 获取会话
    public WebotSession getSession(String sessionId);
    
    // 刷新会话过期时间
    public boolean touchSession(String sessionId);
    
    // 删除会话
    public boolean removeSession(String sessionId);
    
    // ========== 浏览器操作 ==========
    
    // 获取浏览器页签（需手动关闭）
    public BrowserTab openBrowserTab(WebotSession webotSession) throws TimeoutException;
    
    // 执行操作并自动管理资源（推荐）
    public <T> T execute(WebotSession webotSession, WebotFunction<T> function) throws Exception;
    public void execute(WebotSession webotSession, WebotConsumer consumer) throws Exception;
    
    // ========== 验证码服务 ==========
    public CaptchaService getCaptchaService(String configName);
    
    // ========== 代理服务 ==========
    public ProxyService getProxyService(String configName);
    
    // ========== 反检测服务 ==========
    public StealthService getStealthService(String configName);
}
```

#### WebotSession 会话

```java
public class WebotSession implements Serializable {
    private String sessionId;              // 会话ID
    private long createTime;               // 创建时间
    private long lastAccessTime;           // 最后访问时间
    private long expirationTime;           // 过期时间
    private BrowserConfig browserConfig;   // 浏览器配置
    private String captchaConfigKey;       // 验证码配置key
    private String stealthConfigKey;       // 反检测配置key
    private ProxyConfig.ProxyServer proxyServer;  // 代理服务器
    private String storageStateJson;       // 浏览器存储状态（用于恢复）
    private Map<String, Object> extParam;  // 扩展参数
    
    public static Builder builder();
}
```

#### BrowserConfig 浏览器配置

```java
public class BrowserConfig {
    private BrowserType browserType = BrowserType.CHROMIUM;  // 浏览器类型
    private boolean headless = true;                         // 无头模式
    private int viewportWidth = 1920;                        // 视口宽度
    private int viewportHeight = 1080;                       // 视口高度
    private String userAgent;                                // User-Agent
    private String locale;                                   // 语言设置
    private String timezone;                                 // 时区
    private List<String> args;                               // 启动参数
    private boolean javaScriptEnabled = true;                // 启用JS
    private String executablePath;                           // 自定义浏览器路径
    
    public static Builder builder();
}

public enum BrowserType {
    CHROMIUM,    // Chrome/Edge
    FIREFOX,     // Firefox
    WEBKIT       // Safari
}
```

#### BrowserTab 浏览器页签

```java
public class BrowserTab implements Closeable {
    
    // ========== 页面导航 ==========
    public void navigate(String url);
    public void navigate(String url, LoadState waitUntil);
    public void waitForLoadState(LoadState state);
    public String getUrl();
    public String getTitle();
    
    // ========== 元素操作 ==========
    public ElementHandle querySelector(String selector);
    public List<ElementHandle> querySelectorAll(String selector);
    public void click(String selector);
    public void fill(String selector, String value);
    public void type(String selector, String text);
    public void press(String key);
    public void selectOption(String selector, String value);
    
    // ========== 内容获取 ==========
    public String getTextContent(String selector);
    public String getInnerText(String selector);
    public String getInnerHtml(String selector);
    public String getAttribute(String selector, String name);
    public byte[] screenshot();
    public byte[] screenshot(String selector);
    
    // ========== 等待操作 ==========
    public void waitForSelector(String selector);
    public void waitForSelector(String selector, WaitForSelectorOptions options);
    public void waitForTimeout(long timeout);
    
    // ========== JavaScript执行 ==========
    public Object evaluate(String expression);
    public <T> T evaluate(String expression, Class<T> returnType);
    public Object evaluateHandle(String expression);
    
    // ========== 资源管理 ==========
    @Override
    public void close();  // 归还到连接池
    
    // ========== 高级操作 ==========
    public void setExtraHTTPHeaders(Map<String, String> headers);
    public void setViewportSize(int width, int height);
    public Page getPage();  // 获取原生Playwright Page对象
}
```

#### 验证码服务

```java
public interface CaptchaService {
    // 识别图片验证码
    CaptchaResult recognizeImageCaptcha(byte[] imageData);
    CaptchaResult recognizeImageCaptcha(byte[] imageData, CaptchaOptions options);
    
    // 识别Base64图片验证码
    CaptchaResult recognizeBase64Captcha(String base64Image);
    CaptchaResult recognizeBase64Captcha(String base64Image, CaptchaOptions options);
    
    // 解决ReCaptcha V2
    CaptchaResult solveRecaptchaV2(String siteKey, String pageUrl);
    
    // 解决ReCaptcha V3
    CaptchaResult solveRecaptchaV3(String siteKey, String pageUrl, String action);
    
    // 解决hCaptcha
    CaptchaResult solveHCaptcha(String siteKey, String pageUrl);
}

public enum CaptchaServiceType {
    OCR,           // 本地OCR识别（Tesseract）
    TWOCAPTCHA,    // 2Captcha服务
    CAPSOLVER      // Capsolver AI服务
}

public class CaptchaResult {
    private boolean success;       // 是否成功
    private String text;           // 识别结果
    private String errorCode;      // 错误码
    private String errorMessage;   // 错误信息
}
```

#### 代理服务

```java
public interface ProxyService {
    // 获取一个代理
    ProxyConfig.ProxyServer getProxy();
    
    // 获取指定类型的代理
    ProxyConfig.ProxyServer getProxy(ProxyType type);
    
    // 报告代理失败
    void reportFailure(ProxyConfig.ProxyServer proxy);
    
    // 健康检查
    boolean healthCheck(ProxyConfig.ProxyServer proxy);
}

public enum ProxyType {
    HTTP, HTTPS, SOCKS4, SOCKS5
}

public class ProxyConfig {
    private ProxyType type;
    private List<ProxyServer> servers;
    
    public static class ProxyServer {
        private String host;
        private int port;
        private String username;  // 可选
        private String password;  // 可选
    }
}
```

#### 反检测服务

```java
public class StealthService {
    // 应用反检测脚本
    public void applyStealth(BrowserTab browserTab);
}

public class StealthConfig {
    private boolean enabled = true;
    private boolean webdriverHide = true;      // 隐藏WebDriver属性
    private boolean webglSpoof = true;         // WebGL指纹伪装
    private boolean canvasNoise = true;        // Canvas指纹随机化
    private boolean fontsMask = true;          // 字体列表伪装
    private boolean pluginsMask = true;        // 插件列表伪装
}
```

#### 使用示例

```java
@Service
public class WebCrawlerService {
    
    @Autowired
    private WebotManager webotManager;
    
    /**
     * 基础页面抓取
     */
    public String crawlPage(String url) throws Exception {
        // 创建会话配置
        SessionConfig sessionConfig = SessionConfig.builder()
            .browserConfig(BrowserConfig.builder()
                .browserType(BrowserType.CHROMIUM)
                .headless(true)
                .viewportWidth(1920)
                .viewportHeight(1080)
                .build())
            .stealthConfigKey("default")  // 启用反检测
            .build();
        
        // 创建会话
        WebotSession session = webotManager.createSession(sessionConfig);
        
        // 使用 try-with-resources 自动管理资源
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate(url);
            tab.waitForLoadState(LoadState.NETWORKIDLE);
            
            // 获取页面标题
            String title = tab.getTitle();
            
            // 提取内容
            String content = tab.getInnerText("article");
            
            return content;
        }
    }
    
    /**
     * 使用 execute 方法简化资源管理
     */
    public String crawlWithExecute(String url) throws Exception {
        WebotSession session = webotManager.createSession(
            SessionConfig.builder()
                .browserConfig(BrowserConfig.builder()
                    .browserType(BrowserType.CHROMIUM)
                    .headless(true)
                    .build())
                .build()
        );
        
        return webotManager.execute(session, tab -> {
            tab.navigate(url);
            tab.waitForLoadState(LoadState.NETWORKIDLE);
            return tab.getInnerText("body");
        });
    }
    
    /**
     * 表单自动填写和提交
     */
    public void autoFillForm(String url) throws Exception {
        WebotSession session = webotManager.createSession(SessionConfig.builder().build());
        
        webotManager.execute(session, tab -> {
            tab.navigate(url);
            
            // 填写表单
            tab.fill("#username", "myuser");
            tab.fill("#password", "mypass");
            tab.selectOption("#country", "China");
            
            // 点击提交
            tab.click("#submit");
            
            // 等待跳转
            tab.waitForLoadState(LoadState.NETWORKIDLE);
        });
    }
    
    /**
     * 使用代理访问
     */
    public String crawlWithProxy(String url) throws Exception {
        SessionConfig config = SessionConfig.builder()
            .browserConfig(BrowserConfig.builder().build())
            .proxyConfigKey("default")  // 使用配置的代理
            .build();
        
        WebotSession session = webotManager.createSession(config);
        
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate(url);
            return tab.getInnerText("body");
        }
    }
    
    /**
     * 验证码识别示例
     */
    public String solveCaptcha(String imageUrl) throws Exception {
        // 获取验证码服务
        CaptchaService captchaService = webotManager.getCaptchaService("default");
        
        // 下载验证码图片
        byte[] imageData = downloadImage(imageUrl);
        
        // 识别
        CaptchaResult result = captchaService.recognizeImageCaptcha(imageData);
        
        if (result.isSuccess()) {
            return result.getText();
        }
        return null;
    }
    
    /**
     * 截图示例
     */
    public byte[] takeScreenshot(String url) throws Exception {
        WebotSession session = webotManager.createSession(SessionConfig.builder().build());
        
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate(url);
            tab.waitForLoadState(LoadState.NETWORKIDLE);
            
            // 全页截图
            return tab.screenshot();
        }
    }
    
    /**
     * 执行JavaScript
     */
    public Object executeJs(String url, String jsCode) throws Exception {
        WebotSession session = webotManager.createSession(SessionConfig.builder().build());
        
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate(url);
            return tab.evaluate(jsCode);
        }
    }
    
    /**
     * 会话状态保存和恢复（保持登录状态）
     */
    public void crawlWithSessionPersistence(String url) throws Exception {
        // 创建会话并登录
        WebotSession session = webotManager.createSession(SessionConfig.builder().build());
        
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate("https://example.com/login");
            tab.fill("#username", "user");
            tab.fill("#password", "pass");
            tab.click("#login");
            tab.waitForLoadState(LoadState.NETWORKIDLE);
        }
        
        // 保存会话状态（包含Cookie/LocalStorage）
        String sessionId = session.getSessionId();
        
        // 后续使用同一会话（保持登录）
        WebotSession savedSession = webotManager.getSession(sessionId);
        
        try (BrowserTab tab = webotManager.openBrowserTab(savedSession)) {
            tab.navigate(url);  // 已登录状态
            // ...
        }
    }
}
```

---

