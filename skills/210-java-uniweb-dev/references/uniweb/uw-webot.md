# uw-webot — Web 自动化框架

**Maven 坐标**：`com.umtone:uw-webot`　**配置前缀**：`uw.webot`

基于 Microsoft Playwright 的 Web 自动化框架。采用 **Hybrid 混合模式**：Browser 进程级别复用（同一个 BrowserInstance 可承载多个 BrowserTab），BrowserTab 级别的一次性 Context（每次新建 BrowserContext + Page，`close()` 后销毁、**不复用**，因为 Playwright Context 池化复用经实测不稳定）。

> ⚠️ **重要事实**：BrowserTab 的 `close()` 是**销毁**而非归还复用。不要在文档或注释里写"归还到池中复用"。只有 Browser 进程本身由池持有并跨 Tab 共享。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 获取管理器 | Spring 注入 `WebotManager`（或 `WebotManager.getInstance()`） | getInstance 在 enabled=false 时返回 null |
| 创建会话 | `webotManager.createSession(SessionConfig)` | 不传参则用默认配置 |
| 关闭会话 | `webotManager.destroySession(sessionId)` | — |
| 打开页签（一次性 Context） | `webotManager.openBrowserTab(session)` | **必须** try-with-resources，close 会销毁 Context |
| 执行操作（推荐） | `webotManager.execute(session, tab -> {...})` | 自动 try-with-resources |
| 导航 | `tab.navigate(url)` | 返回 Response |
| 等待加载 | `tab.waitForLoadState(LoadState)` | — |
| 取当前 URL/标题/HTML | `tab.url()` / `tab.title()` / `tab.content()` | — |
| 执行 JS | `tab.evaluate(expr)` 或 `tab.evaluate(expr, arg)` | — |
| 截图 | `tab.screenshot(Page.ScreenshotOptions)` | 返回 byte[] |
| 存储状态 | `tab.getStorageStateJson()` | 返回 cookies+localStorage JSON |
| 自定义操作 | `tab.execute((ctx, page) -> {...})` / `tab.consume(...)` | 直接拿 Playwright 原生 Page/Context |
| 注册初始化脚本 | `tab.addInitScript(script)` | 必须在 navigate 前；反检测自动调用 |
| 识别验证码 | `webotManager.getCaptchaManager().getCaptchaService("key").recognizeImageCaptcha(bytes)` | — |
| 应用反检测 | 会话配 `stealthConfigKey`，openBrowserTab 自动应用 | 通过 addInitScript 注入 |

## 架构

```
WebotManager (Spring Bean / 静态单例)
    ├── BrowserBotPool               (按 BrowserConfig 标签分组)
    │       └── BrowserGroup         (组内 BrowserInstance 轮询负载均衡)
    │              └── BrowserInstance (专属单线程 executor，Playwright+Browser)
    │                     └── BrowserTab (一次性 Context + Page)
    ├── SessionService               (GlobalSessionServiceImpl, 基于 FusionCache)
    ├── CaptchaManager               (Map<String, CaptchaService>: OCR/2Captcha/Capsolver)
    ├── StealthManager               (Map<String, StealthService>)
    └── ProxyManager                 (Map<String, ProxyService>: LocalProxyPoolImpl)
```

资源模型：
- `BrowserInstance` 持有一个单线程 `ExecutorService`，**所有 Playwright 操作**都通过 `submitAndWait` 串行提交到该线程，保证 Playwright 线程安全。
- `selectBrowserInstance()` 轮询组内实例，选中第一个 `isActive() && activeTabCount < maxTabsPerBrowser` 的；都不满则按需新建至上限 `maxBrowsersPerGroup`。

## 配置（application.yml）

字段严格以 `uw.webot.conf.WebotProperties` 及各 `*Config` 类为准：

```yaml
uw:
  webot:
    enabled: true
    bot-pool:
      max-browsers-per-group: 5     # 每个 BrowserGroup 最大 Browser 数 (1-20)
      max-tabs-per-browser: 20      # 每个 Browser 最大 Tab 数 (1-50)
    session:
      distributed: false            # true 则 FusionCache 走分布式
      default-session:
        expire-time: P30D           # ISO-8601 duration
        browser-config:
          browser-type: chromium    # chromium | firefox | webkit
          headless: true
          viewport-width: 1920
          viewport-height: 1080
          java-script-enabled: true
          # user-agent / locale / timezone / args 可选
    # 验证码/代理/反检测 均为 Map<String, XxxConfig>，key 由 SessionConfig.*ConfigKey 引用
    captcha: {}                      # 示例见下
    stealth: {}
    proxy: {}
```

多服务配置示例（key 即业务方在 SessionConfig 引用的名称）：

```yaml
    captcha:
      default:
        service-type: capsolver      # ocr | twocaptcha | capsolver
        api-key: ${CAPSOLVER_KEY}
        max-timeout: PT2M
    stealth:
      default:
        hide-web-driver: true
        webgl-spoofing: true
        plugin-spoofing: true
        font-spoofing: true
        fingerprint-randomization: false
    proxy:
      default:
        max-failures: 3
        health-check-interval: PT5M
        servers:
          - type: HTTPS              # http | https | socks4 | socks5
            host: proxy.example.com
            port: 8080
            username: ""
            password: ""
```

## WebotManager（`uw.webot.WebotManager`）

构造：由 `WebotAutoConfiguration` 装配为 Spring Bean；也提供 `WebotManager.getInstance()` 静态访问（enabled=false 时为 null）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `createSession()` | WebotSession | 用默认 SessionConfig 创建会话 |
| `createSession(SessionConfig)` | WebotSession | 创建会话（自动绑定 proxy/stealth/captcha key） |
| `getSession(sessionId)` | WebotSession | 取会话 |
| `updateSession(WebotSession, Duration ttl)` | void | 续期 |
| `destroySession(sessionId)` | void | 销毁会话 |
| `openBrowserTab(WebotSession)` | BrowserTab | **新建**一次性 Context+Page；若配了 stealthConfigKey 自动注入反检测 initScript |
| `execute(WebotSession, WebotFunction<T>)` | T | 自动 openBrowserTab + try-with-resources |
| `execute(WebotSession, WebotConsumer)` | void | 无返回值版本 |
| `getCaptchaManager()` | CaptchaManager | — |
| `getStealthManager()` | StealthManager | — |
| `getProxyManager()` | ProxyManager | — |
| `getSessionManager()` | SessionService | — |
| `getStats()` | BrowserBotPool.PoolStats | 池统计 |
| `getProperties()` | WebotProperties | — |

## WebotSession（`uw.webot.WebotSession`）

由 `createSession()` 返回，不直接构造（有 Builder 但一般无需手建）。

| 字段 | 类型 | 说明 |
|------|------|------|
| sessionId | String | 雪花 ID |
| createTime | long | 创建时间戳 |
| lastAccessTime | long | 最后访问时间戳（volatile） |
| expirationTime | long | 过期时间戳（volatile；实际过期由 FusionCache TTL 控制） |
| browserConfig | BrowserConfig | 浏览器配置 |
| captchaConfigKey | String | 验证码服务 key |
| stealthConfigKey | String | 反检测服务 key |
| proxyServer | ProxyConfig.ProxyServer | 解析后的代理（由 proxyConfigKey 填充） |
| storageStateJson | String | 上下文存储状态 JSON（用于恢复登录态） |
| extParam | `Map<String,Object>` | 业务扩展参数（ConcurrentHashMap） |

## SessionConfig（`uw.webot.session.SessionConfig`）

`SessionConfig.builder().browserConfig(...).stealthConfigKey("default").build()`。

| 字段 | 类型 | 说明 |
|------|------|------|
| expireTime | Duration | 会话过期时间，默认 30 天 |
| browserConfig | BrowserConfig | 浏览器配置，默认 chromium+headless |
| captchaConfigKey | String | 引用 `uw.webot.captcha.<key>` |
| stealthConfigKey | String | 引用 `uw.webot.stealth.<key>` |
| proxyConfigKey | String | 引用 `uw.webot.proxy.<key>` |
| extParam | `Map<String,Object>` | 初始扩展参数 |

## BrowserConfig（`uw.webot.core.BrowserConfig`）

`BrowserConfig.builder().browserType(CHROMIUM).headless(true).build()`。注意 Builder 的 `javaScriptEnabled` 默认与字段一致（true）。

| 属性 | 类型 | 说明 |
|------|------|------|
| browserType | BrowserType | CHROMIUM / FIREFOX / WEBKIT，默认 CHROMIUM |
| headless | boolean | 默认 true |
| viewportWidth | int | 默认 1920 |
| viewportHeight | int | 默认 1080 |
| userAgent | String | 自定义 UA |
| locale | String | 语言 |
| timezone | String | 时区 |
| args | `List<String>` | 浏览器启动参数 |
| javaScriptEnabled | boolean | 默认 true |
| executablePath | String | 自定义浏览器可执行路径 |

`getBrowserGroupTag()` 生成 Group 标签：`browserType[:headless][:executablePath]`，不同标签会落到不同 Group（各有独立 Browser 进程）。

## BrowserTab（`uw.webot.core.BrowserTab`）

由 `openBrowserTab()` / `execute()` 返回，实现 `Closeable`。所有方法内部 submit 到 BrowserInstance 单线程。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `navigate(url)` | Response | 导航 |
| `navigate(url, Page.NavigateOptions)` | Response | 带选项导航 |
| `waitForLoadState(LoadState)` | void | 等待加载状态 |
| `url()` | String | 当前 URL |
| `title()` | String | 页面标题 |
| `content()` | String | 完整 HTML |
| `reload()` / `reload(options)` | Response | 刷新 |
| `screenshot(Page.ScreenshotOptions)` | byte[] | 截图 |
| `evaluate(expr)` | Object | 执行 JS |
| `evaluate(expr, arg)` | Object | 执行 JS（带参） |
| `getStorageStateJson()` | String | 取 cookies+localStorage JSON |
| `addInitScript(script)` | void | 注册初始化脚本（navigate 前；反检测内部用） |
| `consume(BiConsumer<BrowserContext,Page>)` | void | 直接操作原生 Page/Context |
| `execute(BiFunction<BrowserContext,Page,T>)` | T | 直接操作并返回结果 |
| `executeAsync(BiFunction<...>)` | Future<T> | 异步 |
| `close()` | void | **销毁** Context+Page，从 instance 注销 |

> BrowserTab **没有**元素操作方法（click/fill/getInnerText 等）。需要元素操作时，用 `tab.execute((ctx, page) -> page.querySelector(...).fill(...))` 直接走 Playwright 原生 Page API。

## CaptchaService（`uw.webot.captcha.CaptchaService`）

通过 `webotManager.getCaptchaManager().getCaptchaService(key)` 获取。实现：`LocalOcrCaptchaServiceImpl`（OCR，**未集成真实引擎，直接返回失败**）、`TwoCaptchaServiceImpl`、`CapsolverServiceImpl`。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `recognizeImageCaptcha(byte[])` | CaptchaResult | 图片验证码 |
| `recognizeImageCaptcha(byte[], CaptchaOptions)` | CaptchaResult | 带选项 |
| `recognizeBase64Captcha(base64[, options])` | CaptchaResult | Base64 图片 |
| `solveReCaptchaV2(siteKey, pageUrl)` | CaptchaResult | — |
| `solveReCaptchaV3(siteKey, pageUrl, action, minScore)` | CaptchaResult | — |
| `solveHCaptcha(siteKey, pageUrl)` | CaptchaResult | — |
| `solveGeeTest(gt, challenge, apiServer, pageUrl)` | CaptchaResult | — |
| `isAvailable()` | boolean | API key 是否配置 |
| `getBalance()` | double | 账户余额 |
| `getServiceType()` | String | "ocr" / "2captcha" / "capsolver" |

`CaptchaResult` record：`success` / `code` / `errorMessage` / `solveTimeMillis` / `cost`，工厂 `success(...)` / `failure(msg)`。

## ProxyService（`uw.webot.proxy.ProxyService`）

通过 `webotManager.getProxyManager().getProxyService(key)` 获取，实现 `LocalProxyPoolImpl`。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `getProxy(ProxyType)` | ProxyInfo | 轮询取代理（**命中代理也保留在池中循环复用**） |
| `releaseProxy(ProxyInfo)` | void | 本实现 no-op（getProxy 不取出代理） |
| `markProxyFailed(ProxyInfo)` | void | 累加 failureCount |
| `checkProxyHealth(ProxyInfo)` | boolean | 同步健康检查（HTTP GET httpbin，10s 超时） |
| `getStatistics()` | ProxyPoolStatistics | 统计 |
| `shutdown()` | void | 清空池 |

`ProxyInfo`：`proxyServer` / `weight` / `failureCount` / `lastUsedTime` / `lastHealthCheckTime` / `lastHealthCheckResult`。

## StealthService（`uw.webot.stealth.StealthService`）

通过 `WebotManager` 自动应用（会话配 stealthConfigKey 即可），也可 `stealthManager.apply(tab, key)` 手动调。

反检测脚本通过 **`BrowserTab.addInitScript`** 注入，在每个新页面任何站点脚本前执行：
- `hideWebDriver`：删除 `navigator.webdriver`、伪装 languages/chrome/permissions/Notification
- `webglSpoofing`：覆盖 `WebGLRenderingContext.getParameter`（vendor/renderer 指纹）
- `fingerprintRandomization`：随机 UA / 屏幕分辨率 / 时区 / 覆盖 `Intl.DateTimeFormat`、`Date`
- `pluginSpoofing`：伪造 navigator.plugins / mimeTypes
- `fontSpoofing`：canvas measureText / getImageData 加噪

> 反检测是"尽力而为"，高级反爬仍可能识别。指纹数据需与 UA 平台自洽，否则更可疑。

## Helper 使用示例

```java
public class WebCrawlerHelper {

    @Autowired
    private WebotManager webotManager;   // 推荐注入；静态场景可用 WebotManager.getInstance()

    /** 基础爬取：用 execute 自动管理 BrowserTab 生命周期 */
    public String crawlPage(String url) throws Exception {
        SessionConfig config = SessionConfig.builder()
                .browserConfig(BrowserConfig.builder()
                        .browserType(BrowserType.CHROMIUM)
                        .headless(true).build())
                .stealthConfigKey("default")
                .build();
        WebotSession session = webotManager.createSession(config);
        try {
            return webotManager.execute(session, tab -> {
                tab.navigate(url);
                tab.waitForLoadState(LoadState.NETWORKIDLE);
                return tab.content();
            });
        } finally {
            webotManager.destroySession(session.getSessionId());
        }
    }

    /** 截图：手动 try-with-resources 管理 BrowserTab */
    public byte[] takeScreenshot(String url) throws Exception {
        WebotSession session = webotManager.createSession(SessionConfig.builder().build());
        try (BrowserTab tab = webotManager.openBrowserTab(session)) {
            tab.navigate(url);
            tab.waitForLoadState(LoadState.NETWORKIDLE);
            return tab.screenshot(new Page.ScreenshotOptions().setFullPage(true));
        } finally {
            webotManager.destroySession(session.getSessionId());
        }
    }

    /** 元素操作：通过 execute 拿原生 Page 走 Playwright API */
    public void submitForm(String url, String name) throws Exception {
        WebotSession session = webotManager.createSession();
        try {
            webotManager.execute(session, tab -> {
                tab.navigate(url);
                tab.waitForLoadState(LoadState.NETWORKIDLE);
                tab.execute((ctx, page) -> {
                    page.fill("#name", name);
                    page.click("#submit");
                    return null;
                });
                return null;
            });
        } finally {
            webotManager.destroySession(session.getSessionId());
        }
    }

    /** 验证码识别 */
    public String recognize(byte[] imgBytes) {
        CaptchaService svc = webotManager.getCaptchaManager().getCaptchaService("default");
        CaptchaResult r = svc.recognizeImageCaptcha(imgBytes);
        return r.success() ? r.code() : null;
    }
}
```

## 注意事项

1. **BrowserTab 不可复用**：每次 `openBrowserTab` 都新建 Context+Page，`close()` 销毁。不要持有 tab 跨方法长期使用。
2. **反检测时机**：必须在 `navigate` 前注入 initScript；`openBrowserTab` 已在返回前完成注入（配了 stealthConfigKey 时）。
3. **OCR 默认失败**：未集成真实 OCR 引擎，`LocalOcrCaptchaServiceImpl` 直接返回 failure，生产请用 2captcha/capsolver。
4. **代理池不取走代理**：`getProxy` 命中代理也保留在池中，无需 release；`LocalProxyPoolImpl` 健康检查走 httpbin 同步请求，可能 10s 阻塞。
5. **submitAndWait 超时**：默认 60s，超时会 cancel 任务并把 BrowserInstance 标记失效异步重建——长耗时导航请用带超时重载或调高默认值。
6. **线程模型**：BrowserInstance 单线程串行执行所有 Playwright 操作；同一 instance 上的多个 tab 操作会串行排队。
7. **会话存储**：`WebotSession.storageStateJson` 可在 createSession 前设置以恢复登录态，或 `tab.getStorageStateJson()` 取出保存。
