### uw-mfa — 多因素认证

**Maven 坐标**: `com.umtone:uw-mfa`

融合 IP 限制、验证码、设备码的多重认证库。

**配置前缀**: `uw.mfa`

```yaml
uw:
  mfa:
    # IP白名单，支持CIDR格式
    ip-white-list: "127.0.0.1,10.0.0.0/8,172.16.0.0/12,192.168.0.0/16"
    # IP限制配置
    ip-limit-seconds: 600          # 错误检查过期时间（秒）
    ip-limit-warn-times: 3         # 警告阈值次数
    ip-limit-error-times: 10       # 错误屏蔽阈值次数
    # 验证码配置
    captcha-expired-seconds: 180   # 验证码过期时间
    captcha-send-limit-seconds: 60 # 发送限制时间
    captcha-send-limit-times: 10   # 发送限制次数
    captcha-strategies: "StringCaptchaStrategy,CalculateCaptchaStrategy,SlidePuzzleCaptchaStrategy,ClickWordCaptchaStrategy,RotatePuzzleCaptchaStrategy"
    # 设备验证码配置
    device-code-expired-seconds: 300      # 过期时间
    device-code-default-length: 6         # 默认长度
    device-code-send-limit-seconds: 1800  # 发送限制时间
    device-code-send-limit-times: 10      # 发送限制次数
    device-code-verify-limit-seconds: 600 # 校验限制时间
    device-code-verify-error-times: 10    # 校验错误次数
    device-notify-subject: "设备验证码"
    device-notify-content: "设备验证码[$DEVICE_CODE$]，$EXPIRE_MINUTES$分钟后过期"
    # TOTP配置
    totp-algorithm: SHA1           # 算法：SHA1/SHA256/SHA512
    totp-secret-length: 32         # 密钥长度
    totp-code-length: 6            # 验证码长度
    totp-time-period: 30           # 时间窗口（秒）
    totp-time-period-discrepancy: 2 # 时间偏移量
    totp-verify-limit-seconds: 600  # 校验限制时间
    totp-verify-error-times: 10     # 校验错误次数
    totp-gen-qr: true              # 是否生成二维码
    totp-qr-size: 350              # 二维码尺寸
    totp-issuer: "uw-mfa"          # 签发人
    # Redis配置
    redis:
      database: 0
      host: 127.0.0.1
      port: 6379
```

#### MfaFusionHelper 统一认证入口

**API 说明**：

```java
public class MfaFusionHelper {
    // ========== IP限制相关 ==========
    
    // 检查IP是否在白名单中
    public static boolean checkIpWhiteList(String ip);
    
    // 检查IP错误限制，返回warn表示需要验证码，error表示已屏蔽
    public static ResponseData checkIpErrorLimit(String ip);
    
    // 递增IP错误次数
    public static void incrementIpErrorTimes(String ip, String remark);
    
    // 清除IP错误限制
    public static boolean clearIpErrorLimit(String ip);
    
    // 获取IP限制列表
    public static Set<String> getIpErrorLimitList();
    
    // ========== 验证码相关 ==========
    
    // 生成验证码（自动检测IP限制，warn状态才生成）
    public static ResponseData<CaptchaQuestion> generateCaptcha(String userIp, String captchaId);
    
    // 验证验证码
    public static ResponseData verifyCaptcha(String userIp, String captchaId, String captchaSign);
    
    // 获取验证码发送限制列表
    public static Set<String> getCaptchaSendLimitList();
    
    // 清除验证码发送限制
    public static boolean clearCaptchaSendLimit(String ip);
    
    // ========== 设备验证码相关 ==========
    
    // 发送设备验证码（自动IP检测）
    public static ResponseData sendDeviceCode(String userIp, long saasId, int deviceType, String deviceId);
    public static ResponseData sendDeviceCode(String userIp, long saasId, int deviceType, String deviceId, 
                                              String captchaId, String captchaSign);
    
    // 校验设备验证码
    public static ResponseData verifyDeviceCode(int deviceType, String deviceId, String deviceCode);
    public static ResponseData verifyDeviceCode(String userIp, int deviceType, String deviceId, String deviceCode);
    public static ResponseData verifyDeviceCode(String userIp, int deviceType, String deviceId, 
                                                String deviceCode, String captchaId, String captchaSign);
    
    // 获取设备验证码限制列表
    public static Set<String> getDeviceCodeSendLimitList();
    public static Set<String> getDeviceCodeVerifyLimitList();
    
    // 清除设备验证码限制
    public static boolean clearDeviceCodeSendLimit(String ip);
    public static boolean clearDeviceCodeVerifyLimit(String deviceId);
    
    // ========== TOTP相关 ==========
    
    // 生成TOTP密钥
    public static ResponseData<TotpSecretData> issueTotpSecret(String label);
    public static ResponseData<TotpSecretData> issueTotpSecret(String label, String issuer, int qrSize);
    
    // 校验TOTP验证码
    public static ResponseData verifyTotpCode(String userInfo, String totpSecret, String totpCode);
    public static ResponseData verifyTotpCode(String userIp, String userInfo, String totpSecret, String totpCode);
    public static ResponseData verifyTotpCode(String userIp, String userInfo, String totpSecret, 
                                               String totpCode, String captchaId, String captchaSign);
    
    // 生成恢复码
    public static String[] generateRecoveryCode(int amount);
    
    // 获取TOTP校验限制列表
    public static Set<String> getTotpVerifyLimitList();
    
    // 清除TOTP校验限制
    public static boolean clearTotpVerifyLimit(String userInfo);
}

// 验证码问题对象
public class CaptchaQuestion implements Serializable {
    private String captchaId;          // 验证码ID
    private long captchaTTL;           // 有效期（秒）
    private String captchaType;        // 类型：StringCaptcha/CalculateCaptcha/SlidePuzzleCaptcha/ClickWordCaptcha/RotatePuzzleCaptcha
    private String mainImageBase64;    // 主图片Base64
    private String subImageBase64;     // 子图片Base64（滑动/旋转用）
    private String subData;            // 附加数据（AES加密）
}

// TOTP密钥数据
public class TotpSecretData {
    private String secret;   // 密钥
    private String uri;      // 二维码URI（otpauth://）
    private String qr;       // 二维码图片Base64
}

// 设备类型常量
public enum MfaDeviceType {
    MOBILE_CODE(1),   // 手机短信
    EMAIL_CODE(2);    // 邮件
}
```

**使用示例**：

```java
@Service
public class LoginService {
    
    @Autowired
    private UserService userService;
    
    // 登录接口 - 带验证码检测
    public ResponseData login(String username, String password, String ip, 
                              String captchaId, String captchaSign) {
        // 1. 检查IP限制
        ResponseData ipCheck = MfaFusionHelper.checkIpErrorLimit(ip);
        if (ipCheck.isError()) {
            return ipCheck;  // IP已被屏蔽
        }
        
        // 2. 如果需要验证码，验证验证码
        if (ipCheck.isWarn()) {
            ResponseData captchaCheck = MfaFusionHelper.verifyCaptcha(ip, captchaId, captchaSign);
            if (captchaCheck.isNotSuccess()) {
                return captchaCheck;
            }
        }
        
        // 3. 验证用户名密码
        User user = userService.verifyPassword(username, password);
        if (user == null) {
            // 登录失败，递增IP错误次数
            MfaFusionHelper.incrementIpErrorTimes(ip, "密码错误");
            return ResponseData.errorMsg("用户名或密码错误");
        }
        
        // 4. 登录成功，清除IP限制
        MfaFusionHelper.clearIpErrorLimit(ip);
        
        return ResponseData.success(user);
    }
    
    // 发送短信验证码
    public ResponseData sendSmsCode(String mobile, String ip, String captchaId, String captchaSign) {
        // 自动检测IP限制和验证码
        return MfaFusionHelper.sendDeviceCode(
            ip, 
            1001L,  // saasId
            MfaDeviceType.MOBILE_CODE.getValue(), 
            mobile,
            captchaId, 
            captchaSign
        );
    }
    
    // 校验短信验证码
    public ResponseData verifySmsCode(String mobile, String code, String ip) {
        // 同时验证IP
        return MfaFusionHelper.verifyDeviceCode(
            ip,
            MfaDeviceType.MOBILE_CODE.getValue(),
            mobile,
            code
        );
    }
    
    // 生成TOTP密钥（绑定Google Authenticator）
    public ResponseData<TotpSecretData> generateTotpSecret(Long userId) {
        String label = "user:" + userId;
        return MfaFusionHelper.issueTotpSecret(label, "MyApp", 300);
    }
    
    // 校验TOTP验证码
    public ResponseData verifyTotp(Long userId, String totpSecret, String totpCode, String ip) {
        return MfaFusionHelper.verifyTotpCode(
            ip,
            "user:" + userId,
            totpSecret,
            totpCode
        );
    }
}
```

#### 独立Helper类

如需单独使用某项功能，可直接使用对应的Helper类：

```java
// IP限制
MfaIPLimitHelper.checkIpWhiteList(ip);
MfaIPLimitHelper.checkIpErrorLimit(ip);
MfaIPLimitHelper.incrementIpErrorTimes(ip, remark);
MfaIPLimitHelper.clearIpErrorLimit(ip);

// 验证码
MfaCaptchaHelper.generateCaptcha(userIp, captchaId);
MfaCaptchaHelper.verifyCaptcha(captchaId, captchaSign);

// 设备验证码
MfaDeviceCodeHelper.sendDeviceCode(userIp, saasId, deviceType, deviceId);
MfaDeviceCodeHelper.verifyDeviceCode(deviceType, deviceId, deviceCode);

// TOTP
MfaTotpHelper.issue(label);
MfaTotpHelper.verifyCode(userInfo, totpSecret, totpCode);
MfaTotpHelper.generateRecoveryCode(10);  // 生成10个恢复码
```

