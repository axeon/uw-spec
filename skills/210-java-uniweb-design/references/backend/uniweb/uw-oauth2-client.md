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
          clientId: your-apple-client-id
          clientSecret: your-apple-client-secret
          extParam:
            privateKey: -----BEGIN EC PRIVATE KEY-----\n...
        wechat:
          clientId: your-wechat-appid
          clientSecret: your-wechat-secret
        alipay:
          clientId: your-alipay-appid
          clientSecret: your-alipay-private-key
          extParam:
            publicKey: your-alipay-public-key
```

## OAuth2ClientHelper API

```java
public class OAuth2ClientHelper {
    
    // ========== Provider管理 ==========
    
    // 注册Provider（自定义Provider）
    public static void registerProvider(String providerCode, OAuth2Provider provider);
    
    // 获取Provider
    public static OAuth2Provider getProvider(String providerCode);
    
    // 获取所有Provider Map
    public static Map<String, OAuth2Provider> getProviderMap();
    
    // 获取所有Provider配置
    public static Map<String, OAuth2ClientProperties.ProviderConfig> getConfigMap();
    
    // ========== 授权流程 ==========
    
    // 构建授权URL（跳转到第三方登录页）
    public static ResponseData<String> buildAuthUrl(String providerCode, String authStateId);
    
    // 构建二维码URL（用于扫码登录）
    public static ResponseData<String> buildQrCode(String providerCode);
    
    // 获取访问令牌（授权回调后调用）
    public static ResponseData<OAuth2Token> getToken(String providerCode, String authCode, 
                                                      String authStateId, Map<String, String> extParam);
    
    // 获取用户信息（使用访问令牌）
    public static ResponseData<OAuth2UserInfo> getUserInfo(String providerCode, OAuth2Token oAuth2Token);
    
    // 获取授权状态（用于轮询扫码状态）
    public static OAuth2ClientAuthStatus getAuthState(String authStateId);
    
    // 删除/失效授权状态
    public static void invalidateAuthState(String authStateId);
}
```

## OAuth2Token — 访问令牌

```java
public class OAuth2Token implements Serializable {
    private String accessToken;        // 访问令牌
    private String refreshToken;       // 刷新令牌
    private String tokenType;          // 令牌类型（Bearer）
    private long expiresIn;            // 过期时间（秒）
    private String scope;              // 授权作用域
    private String idToken;            // ID令牌（OpenID Connect）
    private String error;              // 错误代码
    private String errorDescription;   // 错误描述
    private long createTime;           // 创建时间
    private long refreshTokenExpiresIn;// 刷新令牌过期时间
    private long issuedAt;             // 颁发时间
    private String openId;             // 三方用户ID
    private String unionId;            // 三方统一ID
    private String username;           // 用户名
    private String email;              // 邮箱
    private String phone;              // 手机号
    private String avatar;             // 头像
    private Map<String, Object> rawParams; // 原始响应参数
    
    public static Builder builder();
    public static Builder builder(OAuth2Token copy);
}
```

## OAuth2UserInfo — 用户信息

```java
public class OAuth2UserInfo implements Serializable {
    private String providerCode;       // 认证提供者（google/wechat等）
    private String openId;             // 三方用户ID
    private String unionId;            // 三方统一ID（微信等）
    private String username;           // 用户名
    private String email;              // 邮箱
    private String phone;              // 手机号
    private String avatar;             // 头像URL
    private String gender;             // 性别
    private String area;               // 地区
    private String address;            // 地址
    private Map<String, Object> rawParams; // 原始用户信息
    
    public static Builder builder();
    public static Builder builder(OAuth2UserInfo copy);
}
```

## OAuth2StateId — 状态ID解析

```java
public class OAuth2StateId {
    private String providerCode;       // Provider名称
    private String authType;           // 认证类型
    private String seqId;              // 顺序ID
    
    public OAuth2StateId(String providerCode, String authType, String seqId);
    
    // 解析状态ID字符串（格式：providerCode_authType_seqId）
    public static OAuth2StateId parse(String authStateId);
    
    public String toString();          // 格式：providerCode_authType_seqId
}
```

## OAuth2ClientAuthStatus — 授权状态枚举

```java
public enum OAuth2ClientAuthStatus {
    WAITING,      // 等待扫码
    SCANNED,      // 已扫码，等待确认
    CONFIRMED,    // 登录已确认
    EXPIRED,      // 已过期
    FAILED        // 登录失败
}
```

## 使用示例

```java
@RestController
@RequestMapping("/oauth2")
public class OAuth2Controller {
    
    /**
     * 获取授权URL（网页登录）
     */
    @GetMapping("/auth-url")
    public ResponseData<String> getAuthUrl(@RequestParam String provider) {
        // 生成stateId（格式：providerCode_authType_seqId）
        String stateId = generateStateId(provider, "web");
        
        ResponseData<String> response = OAuth2ClientHelper.buildAuthUrl(provider, stateId);
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
            
            @Override
            public String buildAuthUrl(String authStateId) {
                // 自定义授权URL构建逻辑
                return "https://custom.com/oauth/authorize?...";
            }
            
            @Override
            public ResponseData<OAuth2Token> getToken(String authCode, String authStateId, 
                                                       Map<String, String> extParam) {
                // 自定义获取令牌逻辑
                return ResponseData.success(OAuth2Token.builder()...build());
            }
            
            @Override
            public ResponseData<OAuth2UserInfo> getUserInfo(OAuth2Token oAuth2Token) {
                // 自定义获取用户信息逻辑
                return ResponseData.success(OAuth2UserInfo.builder()...build());
            }
        };
        
        OAuth2ClientHelper.registerProvider("custom", customProvider);
    }
    
    private String generateStateId(String provider, String authType) {
        String seqId = System.currentTimeMillis() + "";
        return new OAuth2StateId(provider, authType, seqId).toString();
    }
    
    private void handleOAuth2Login(OAuth2UserInfo userInfo) {
        // 根据openId查询是否已绑定用户
        // 未绑定则创建新用户或引导绑定
        // 已绑定则直接登录
    }
}
```

## 支持的Provider

| Provider | 说明 | 特点 |
|---|---|---|
| `google` | Google登录 | 标准OAuth2，支持openid email profile |
| `github` | GitHub登录 | 标准OAuth2，支持user:email |
| `apple` | Apple登录 | Sign in with Apple，需处理privateKey |
| `wechat` | 微信登录 | 扫码登录，snsapi_login作用域 |
| `alipay` | 支付宝登录 | 需配置公钥/私钥 |
| `standard` | 标准OAuth2 | 用于其他标准OAuth2平台 |

