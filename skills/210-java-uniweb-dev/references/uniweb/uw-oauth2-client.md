# uw-oauth2-client — OAuth2客户端

**Maven 坐标**: `com.umtone:uw-oauth2-client`

轻量级 OAuth2 客户端，支持 Google/Apple/GitHub/微信/支付宝等平台，提供扫码登录和用户信息获取能力。

**配置前缀**: `uw.oauth2.client`

```yaml
uw:
  oauth2:
    client:
      redirect-uri: http://localhost:8080/ui/oauth2/redirect
      qrcode-uri: http://localhost:8080/oauth2/qrcode/
      providers:
        google:
          clientId: your-google-client-id
          clientSecret: your-google-client-secret
          # authUri, tokenUri, userInfoUri, authScope 使用默认值
        github:
          clientId: your-github-client-id
          clientSecret: your-github-client-secret
        apple:
          clientId: your-apple-service-id
          # Apple 不使用 clientSecret，需在 extParam 配置 teamId/keyId/p8Key 动态生成 JWT client_secret
          extParam:
            teamId: your-apple-team-id
            keyId: your-apple-key-id
            p8Key: -----BEGIN PRIVATE KEY-----\n...
        wechat:
          clientId: your-wechat-appid
          clientSecret: your-wechat-secret
        alipay:
          clientId: your-alipay-appid
          clientSecret: your-alipay-client-secret
          # 支付宝需配置 RSA2 私钥（PKCS8）用于网关签名
          extParam:
            privateKey: your-alipay-rsa2-private-key
```

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 网页授权登录 | `OAuth2ClientHelper.buildAuthUrl(provider, stateId)` | stateId 为空时内部自动生成；格式 `providerCode:authType:seqId`，分隔符为 `:` |
| 扫码登录 | `OAuth2ClientHelper.buildQrCode(provider)` | 内部生成 qrcode 类型 stateId，轮询 `getAuthState(stateId)` 检查状态 |
| 获取令牌 | `OAuth2ClientHelper.getToken(provider, code, stateId, null)` | 在授权回调中调用；stateId 必须处于 SCANNED 状态（CSRF 校验） |
| 获取用户信息 | `OAuth2ClientHelper.getUserInfo(provider, token)` | 先 getToken 再 getUserInfo；Apple 返回 NOT_SUPPORTED，回退用 token 字段 |
| 查询扫码状态 | `OAuth2ClientHelper.getAuthState(stateId)` | 返回 WAITING/SCANNED/CONFIRMED/EXPIRED/FAILED |
| 失效授权状态 | `OAuth2ClientHelper.invalidateAuthState(stateId)` | 流程结束后清理 state |

> **stateId 格式**：`providerCode:authType:seqId`，authType 取值为 `auth`（网页授权）或 `qrcode`（扫码授权），seqId 为 Snowflake 雪花 ID。分隔符为 `:`（避免与 providerCode 中的下划线冲突）。通常无需手动构造，调用 `buildAuthUrl`/`buildQrCode` 时传 null 即可自动生成。

## OAuth2ClientHelper 方法签名

> **包路径**：`uw.oauth2.client.OAuth2ClientHelper`

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `buildAuthUrl(provider, stateId)` | `ResponseData<String>` | 构建授权URL（跳转第三方登录页） |
| `buildQrCode(provider)` | `ResponseData<String>` | 构建二维码URL（扫码登录） |
| `getToken(provider, code, stateId, extParam)` | `ResponseData<OAuth2Token>` | 获取访问令牌（回调后调用） |
| `getUserInfo(provider, token)` | `ResponseData<OAuth2UserInfo>` | 获取用户信息 |
| `getAuthState(stateId)` | `OAuth2ClientAuthStatus` | 获取扫码状态（轮询用） |
| `invalidateAuthState(stateId)` | void | 失效授权状态 |
| `registerProvider(code, provider)` | void | 注册自定义 Provider（code + provider 两个参数） |
| `getProvider(code)` | `OAuth2Provider` | 获取 Provider |
| `getProviderMap()` | `Map<String, OAuth2Provider>` | 获取所有 Provider（只读视图） |
| `getConfigMap()` | `Map<String, ProviderConfig>` | 获取所有 Provider 配置（只读视图） |

## OAuth2Token

> **包路径**：`uw.oauth2.client.vo.OAuth2Token`

构造：`OAuth2Token.builder()` 或 `OAuth2Token.builder(copy)` — Builder 模式。也支持 `new OAuth2Token()` + setter。
提供 `toUserInfo()` 可快速转为 OAuth2UserInfo（从 token 字段映射）。

| 字段 | 类型 | JSON 字段 | 说明 |
|------|------|-----------|------|
| accessToken | String | access_token | 访问令牌 |
| refreshToken | String | refresh_token | 刷新令牌 |
| tokenType | String | token_type | 令牌类型（Bearer） |
| expiresIn | long | expires_in | 过期时间（秒） |
| scope | String | scope | 授权作用域 |
| idToken | String | id_token | ID令牌（OpenID Connect） |
| error | String | error | 错误代码 |
| errorDescription | String | error_description | 错误描述 |
| errorUri | String | error_uri | 错误URI |
| createTime | long | create_time | 令牌创建时间 |
| refreshTokenExpiresIn | long | refresh_token_expires_in | 刷新令牌过期时间（秒） |
| issuedAt | long | issued_at | 令牌颁发时间戳 |
| openId | String | — | 三方用户ID（解析自响应/id_token） |
| unionId | String | — | 三方统一ID（微信等） |
| username | String | — | 用户名 |
| email | String | — | 邮箱 |
| phone | String | — | 手机号 |
| avatar | String | — | 头像 |
| rawParams | `Map<String, Object>` | — | 原始响应参数（@JsonAnySetter 收集） |

## OAuth2UserInfo

> **包路径**：`uw.oauth2.client.vo.OAuth2UserInfo`

构造：`OAuth2UserInfo.builder()` 或 `OAuth2UserInfo.builder(copy)` — Builder 模式。也支持 `new OAuth2UserInfo()` + setter

| 字段 | 类型 | 说明 |
|------|------|------|
| providerCode | String | 认证提供者（google/wechat等） |
| openId | String | 三方用户ID |
| unionId | String | 三方统一ID（微信等） |
| username | String | 用户名 |
| email | String | 邮箱 |
| phone | String | 手机号 |
| avatar | String | 头像URL |
| gender | String | 性别 |
| area | String | 地区 |
| address | String | 地址 |
| rawParams | `Map<String, Object>` | 原始用户信息 |

## OAuth2StateId

> **包路径**：`uw.oauth2.client.vo.OAuth2StateId`

构造：`new OAuth2StateId(providerCode, authType, seqId)` — 格式：`providerCode:authType:seqId`（分隔符 `:`）

解析：`OAuth2StateId.parse(authStateId)` — 从字符串解析；解析失败返回字段全 null 的空对象

| 字段 | 类型 | 说明 |
|------|------|------|
| providerCode | String | 提供者编码（google/wechat等） |
| authType | String | 授权类型（auth / qrcode） |
| seqId | String | 序列ID（Snowflake 雪花 ID） |

## ProviderConfig

> **包路径**：`uw.oauth2.client.conf.OAuth2ClientProperties.ProviderConfig`

| 字段 | 类型 | 说明 |
|------|------|------|
| clientId | String | 应用ID |
| clientSecret | String | 应用密钥 |
| extParam | `Map<String, String>` | 扩展参数（Apple: teamId/keyId/p8Key；Alipay: privateKey） |
| authUri | String | 授权URL |
| tokenUri | String | 令牌URL |
| userInfoUri | String | 用户信息URL |
| authScope | String | 授权范围 |

> google/apple/wechat/alipay/github 五个内置 Provider 的 authUri/authScope/tokenUri/userInfoUri 已有默认值，YAML 中只需配 clientId/clientSecret（及必要的 extParam），缺省项自动合并默认值。

## OAuth2ClientAuthStatus 枚举

> **包路径**：`uw.oauth2.client.constant.OAuth2ClientAuthStatus`

stateId 在 GlobalCache 中的状态（有效期 5 分钟）：

| 值 | 说明 | 流转 |
|------|------|------|
| WAITING | 等待扫码 | 扫码登录二维码生成后初始状态 |
| SCANNED | 已扫码，等待确认 | 网页授权 buildAuthUrl 后、或扫码已识别 |
| CONFIRMED | 登录已确认 | 换取 token 成功后 |
| EXPIRED | 已过期 | stateId 不存在（超过 5 分钟或已清理） |
| FAILED | 登录失败 | providerCode 无法解析或缓存值非法 |

## OAuth2ClientResponseCode 错误码

> **包路径**：`uw.oauth2.client.constant.OAuth2ClientResponseCode`

| code | 含义 | 触发场景 |
|------|------|---------|
| success | 成功 | 正常返回 |
| not_supported | 不支持 | 如 Apple 调用 getUserInfo |
| invalid_provider | 无效的供应商 | providerCode 未注册/未配置 clientId |
| invalid_state_id | 无效的状态ID | stateId 不在 SCANNED 状态（CSRF 校验失败） |
| invalid_http_code | 无效的HTTP状态码 | 三方返回非 200 |
| http_request_failed | HTTP请求失败 | 请求异常或三方业务错误码 |
| server_error | 服务端错误 | 内部异常 |
| unknown_error | 未知错误 | 兜底 |

## Controller 使用示例

```java
@RestController
@RequestMapping("/oauth2")
public class OAuth2Controller {
    
    /**
     * 获取授权URL（网页登录）
     */
    @GetMapping("/auth-url")
    public ResponseData<String> getAuthUrl(@RequestParam String provider) {
        // stateId 传 null 即可自动生成（格式：providerCode:auth:seqId）
        ResponseData<String> response = OAuth2ClientHelper.buildAuthUrl(provider, null);
        return response;
    }
    
    /**
     * 获取二维码URL（扫码登录）
     */
    @GetMapping("/qrcode")
    public ResponseData<String> getQrCode(@RequestParam String provider) {
        return OAuth2ClientHelper.buildQrCode(provider);
    }
    
    /**
     * 授权回调处理
     */
    @GetMapping("/callback")
    public ResponseData<OAuth2UserInfo> callback(
            @RequestParam(required = false) String provider,
            @RequestParam String code,
            @RequestParam String state) {
        
        // 1. 获取访问令牌
        ResponseData<OAuth2Token> tokenResponse = OAuth2ClientHelper.getToken(
            provider, code, state, null);
        
        if (tokenResponse.isNotSuccess()) {
            return tokenResponse.raw();
        }
        
        OAuth2Token token = tokenResponse.getData();
        
        // 2. 获取用户信息
        ResponseData<OAuth2UserInfo> userInfoResponse = OAuth2ClientHelper.getUserInfo(
            provider, token);
        
        if (userInfoResponse.isNotSuccess()) {
            return userInfoResponse;
        }
        
        OAuth2UserInfo userInfo = userInfoResponse.getData();
        
        // 3. 处理登录逻辑（绑定用户或创建新用户）
        handleOAuth2Login(userInfo);
        
        return ResponseData.success(userInfo);
    }
    
    /**
     * 轮询扫码状态
     */
    @GetMapping("/status")
    public ResponseData<String> checkStatus(@RequestParam String stateId) {
        OAuth2ClientAuthStatus status = OAuth2ClientHelper.getAuthState(stateId);
        
        switch (status) {
            case CONFIRMED:
                // 登录成功，获取用户信息
                return ResponseData.success("CONFIRMED");
            case SCANNED:
                return ResponseData.success("SCANNED");
            case WAITING:
                return ResponseData.success("WAITING");
            case EXPIRED:
                return ResponseData.warn("EXPIRED", "二维码已过期");
            case FAILED:
                return ResponseData.error("FAILED", "登录失败");
            default:
                return ResponseData.error("UNKNOWN", "未知状态");
        }
    }
    
    /**
     * 自定义Provider示例（扩展支持其他平台）
     */
    @PostConstruct
    public void registerCustomProvider() {
        OAuth2Provider customProvider = new AbstractOAuth2Provider(
            "custom", providerConfig, redirectUri, qrcodeUri) {
            
            // 继承 AbstractOAuth2Provider 后，通常只需实现两个抽象方法即可：
            @Override
            protected ResponseData<OAuth2Token> parseTokenResponse(String responseBody) {
                // 自定义解析令牌响应
                return ResponseData.success(OAuth2Token.builder().accessToken("...").build());
            }
            
            @Override
            protected ResponseData<OAuth2UserInfo> parseUserInfoResponse(String responseBody) {
                // 自定义解析用户信息响应
                return ResponseData.success(OAuth2UserInfo.builder().openId("...").build());
            }
        };
        
        // registerProvider 需要两个参数：providerCode 与 provider 实例
        OAuth2ClientHelper.registerProvider("custom", customProvider);
    }
    
    private void handleOAuth2Login(OAuth2UserInfo userInfo) {
        // 根据openId查询是否已绑定用户
        // 未绑定则创建新用户或引导绑定
        // 已绑定则直接登录
    }
}
```

## 支持的Provider

| Provider | 类 | 特点 |
|---|---|---|
| `google` | StandardOAuth2Provider | 标准OAuth2，scope=openid email profile |
| `github` | StandardOAuth2Provider | 标准OAuth2，scope=user:email |
| `apple` | AppleOAuth2Provider | Sign in with Apple，需 extParam: teamId/keyId/p8Key；用户信息从 id_token 解析，不支持独立用户信息接口 |
| `wechat` | WechatOAuth2Provider | scope=snsapi_login；授权/换token用 appid+secret；用户信息以 query 携带 access_token+openid |
| `alipay` | AlipayOAuth2Provider | 需 extParam: privateKey（RSA2）；所有请求需签名；业务结果码 10000 表示成功 |
| 自定义 | 继承 AbstractOAuth2Provider | 用于其他标准或非标准 OAuth2 平台，实现 parseTokenResponse/parseUserInfoResponse 即可 |

> Provider 类名在 createProvider 中按 providerCode 小写路由：google/github 走 Standard，apple/wechat/alipay 走各自实现，其余默认走 Standard。

## 自动装配与依赖

- **自动配置**：`uw.oauth2.client.conf.OAuth2ClientAutoConfiguration`（已注册到 `spring.factories` / AutoConfiguration.imports），引入依赖后自动生效。
- **依赖**：`uw-httpclient`（HTTP 调用）、`uw-cache`（GlobalCache 存储 stateId）、`uw-common`（ResponseData/JsonUtils/Snowflake）、`auth0/java-jwt`（id_token 解析与 Apple client_secret 签发）。
- **Helper 设计**：`OAuth2ClientHelper` 核心方法为静态方法，配置在 Bean 构造时一次性注入静态字段，业务侧直接 `OAuth2ClientHelper.xxx()` 调用，无需注入。

## 注意事项

1. **stateId 分隔符为 `:`**（不是下划线），手写 stateId 时需保证三段格式 `providerCode:authType:seqId`，否则 parse 失败返回空对象。
2. **CSRF 防护**：`getToken` 会校验 stateId 必须处于 SCANNED 状态，失败会清理 state；切勿在换 token 前手动 invalidate。
3. **id_token 不验签**：`parseIdToken` 仅解码不校验签名。若上层用 openId 做账号绑定/登录，需调用方自行用 Provider 公钥验签后才能信任。
4. **Apple clientSecret** 由 p8 私钥动态生成 JWT，缓存 30 天自动刷新，参数缺失会抛 IllegalArgumentException。
5. **扫码登录流程**：手机端扫码后实际走的是网页授权流程（stateId 的 authType 为 qrcode），登录成功后 token 会暂存 GlobalCache 60 秒，由 PC 端轮询 CONFIRMED 后调 `getToken` 接口拉取（见项目 README 5.3 节）。
6. **HTTP 客户端信任全部证书**：内部 HTTP 客户端配置了 trustAll，仅适用于对接三方 OAuth 网关；生产环境如需严格校验请覆盖 `AbstractOAuth2Provider.JSON_INTERFACE_HELPER`。

