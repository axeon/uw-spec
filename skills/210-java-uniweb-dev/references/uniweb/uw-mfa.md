# uw-mfa — 多因素认证

**Maven 坐标**: `com.umtone:uw-mfa`

融合 **IP 限制 / Captcha / 设备验证码 / TOTP** 的多重认证库，对外统一入口为 `MfaFusionHelper`（全静态方法）。

**核心约定**：
- **三态响应**：`success` / `warn`（需验证码提示）/ `error`（已屏蔽），对应 `ResponseData.isSuccess()` / `isWarn()` / `isError()`。
- **白名单豁免**：白名单内 IP 全程豁免 —— `checkIpErrorLimit` 直接放行，`incrementIpErrorTimes` 不计入。
- **一次性消费**：Captcha 答案与设备验证码校验时通过 Redis `getAndDelete` 消费，防重放。

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
    device-notify-mobile-api: "http://saas-base/rpc/msg/sendSms"  # 短信发送API
    device-notify-email-api: "http://saas-base/rpc/msg/sendMail"  # 邮件发送API
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

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 检查IP白名单 | `MfaFusionHelper.checkIpWhiteList(ip)` | 返回 boolean |
| 检查IP错误限制 | `MfaFusionHelper.checkIpErrorLimit(ip)` | warn=需验证码，error=已屏蔽 |
| 递增IP错误次数 | `MfaFusionHelper.incrementIpErrorTimes(ip, remark)` | 密码错误时调用 |
| 生成验证码 | `MfaFusionHelper.generateCaptcha(ip, captchaId)` | warn 状态才生成 |
| 验证验证码 | `MfaFusionHelper.verifyCaptcha(ip, captchaId, captchaSign)` | — |
| 发送设备验证码 | `MfaFusionHelper.sendDeviceCode(ip, saasId, deviceType, deviceId, captchaId, captchaSign)` | 含Captcha校验 |
| 校验设备验证码 | `MfaFusionHelper.verifyDeviceCode(ip, deviceType, deviceId, code)` | — |
| 生成TOTP密钥 | `MfaFusionHelper.issueTotpSecret(label)` | 返回含 QR Base64 |
| 校验TOTP | `MfaFusionHelper.verifyTotpCode(ip, userInfo, secret, code)` | — |
| 生成恢复码 | `MfaFusionHelper.generateRecoveryCode(amount)` | — |

## MfaFusionHelper 方法签名

> **包路径**：`uw.mfa.MfaFusionHelper`

全部静态方法，无需实例化。

**IP限制**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `checkIpWhiteList(ip)` | boolean | IP是否在白名单 |
| `checkIpErrorLimit(ip)` | ResponseData | 白名单直接success；warn=需验证码，error=已屏蔽 |
| `incrementIpErrorTimes(ip, remark)` | void | 递增IP错误次数（白名单豁免，不计入） |
| `clearIpErrorLimit(ip)` | boolean | 清除IP限制 |
| `getIpErrorLimitList()` | `Set<String>` | 获取IP限制列表 |
| `countMfaInfo()` | long | 统计MFA限制信息条数（Redis dbSize） |

**验证码**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `generateCaptcha(ip, captchaId)` | `ResponseData<CaptchaQuestion>` | 生成验证码（warn状态才生成） |
| `verifyCaptcha(ip, captchaId, captchaSign)` | ResponseData | 验证验证码 |
| `getCaptchaSendLimitList()` | `Set<String>` | 获取验证码发送限制列表 |
| `clearCaptchaSendLimit(ip)` | boolean | 清除验证码发送限制 |

**设备验证码**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `sendDeviceCode(ip, saasId, deviceType, deviceId, captchaId, captchaSign)` | ResponseData | 发送设备验证码（默认长度模板） |
| `sendDeviceCode(..., codeLen)` | ResponseData | 指定验证码长度 |
| `sendDeviceCode(..., codeLen, notifySubject, notifyContent)` | ResponseData | 完整参数 |
| `verifyDeviceCode(deviceType, deviceId, code)` | ResponseData | 仅校验设备验证码 |
| `verifyDeviceCode(ip, deviceType, deviceId, code)` | ResponseData | 校验设备验证码+IP |
| `verifyDeviceCode(ip, deviceType, deviceId, code, captchaId, captchaSign)` | ResponseData | 校验设备验证码+IP+Captcha |
| `getDeviceCodeSendLimitList()` | `Set<String>` | 获取发送限制列表 |
| `getDeviceCodeVerifyLimitList()` | `Set<String>` | 获取校验限制列表 |
| `clearDeviceCodeSendLimit(ip)` | boolean | 清除发送限制 |
| `clearDeviceCodeVerifyLimit(deviceId)` | boolean | 清除校验限制 |

**TOTP**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `issueTotpSecret(label)` | `ResponseData<TotpSecretData>` | 生成TOTP密钥 |
| `issueTotpSecret(label, issuer, qrSize)` | `ResponseData<TotpSecretData>` | 自定义签发人+二维码尺寸 |
| `verifyTotpCode(userInfo, secret, code)` | ResponseData | 校验TOTP |
| `verifyTotpCode(ip, userInfo, secret, code)` | ResponseData | 校验TOTP（同时验证IP） |
| `verifyTotpCode(ip, userInfo, secret, code, captchaId, captchaSign)` | ResponseData | 校验TOTP+验证码 |
| `generateRecoveryCode(amount)` | String[] | 生成恢复码 |
| `getTotpVerifyLimitList()` | `Set<String>` | 获取TOTP校验限制列表 |
| `clearTotpVerifyLimit(userInfo)` | boolean | 清除TOTP校验限制 |

## CaptchaQuestion

> **包路径**：`uw.mfa.captcha.vo.CaptchaQuestion`

| 字段 | 类型 | 说明 |
|------|------|------|
| captchaId | String | 验证码ID |
| captchaTTL | long | 有效期（秒） |
| captchaType | String | 类型（策略类名）：StringCaptchaStrategy/CalculateCaptchaStrategy/SlidePuzzleCaptchaStrategy/ClickWordCaptchaStrategy/RotatePuzzleCaptchaStrategy |
| mainImageBase64 | String | 主图片 Base64 |
| subImageBase64 | String | 子图片 Base64（滑动/旋转用） |
| subData | String | 附加数据（AES加密） |

## TotpSecretData

> **包路径**：`uw.mfa.totp.vo.TotpSecretData`

| 字段 | 类型 | 说明 |
|------|------|------|
| secret | String | 密钥 |
| uri | String | 二维码URI（otpauth://） |
| qr | String | 二维码图片Base64 |

## MfaDeviceType 枚举

> **包路径**：`uw.mfa.constant.MfaDeviceType`

| 枚举 | 值 | 说明 |
|------|-----|------|
| `TOTP_RECOVERY_CODE` | 20 | TOTP恢复码登录 |
| `TOTP_CODE` | 21 | TOTP验证码登录 |
| `EMAIL_CODE` | 22 | Email验证码（sendDeviceCode支持） |
| `MOBILE_CODE` | 23 | 手机短信验证码（sendDeviceCode支持） |

> `sendDeviceCode` 仅支持 `MOBILE_CODE` 与 `EMAIL_CODE`，其他类型返回 `DEVICE_TYPE_ERROR`。

## HmacAlgorithm 枚举

> **包路径**：`uw.mfa.constant.HmacAlgorithm`

| 枚举 | JCE算法 | otpauth标签 | 说明 |
|------|---------|-------------|------|
| `SHA1` | HmacSHA1 | SHA1 | 默认，兼容Google Authenticator |
| `SHA256` | HmacSHA256 | SHA256 | 安全性更高 |
| `SHA512` | HmacSHA512 | SHA512 | 兼容性较差 |

## MfaResponseCode 响应码

> **包路径**：`uw.mfa.constant.MfaResponseCode`，codePrefix=`uw.mfa`，i18n资源=`i18n/messages/uw_mfa`

| 枚举 | 说明 |
|------|------|
| `IP_AUTH_ERROR` / `IP_LIMIT_WARN` / `IP_LIMIT_ERROR` | IP授权/警告/屏蔽 |
| `CAPTCHA_*` | Captcha欠费/发送超限/生成错误/信息丢失/校验错误 |
| `DEVICE_CODE_*` | 设备码欠费/发送超限/校验超限/发送失败/信息丢失/校验错误/类型错误 |
| `TOTP_*` | TOTP密钥生成/匹配/丢失、验证码丢失/校验错误/校验超限、恢复码校验错误 |

## 独立 Helper 类

| 类 | 功能 |
|------|------|
| `MfaIPLimitHelper` | IP限制（checkIpWhiteList/checkIpErrorLimit/incrementIpErrorTimes/clearIpErrorLimit/countMfaInfo） |
| `MfaCaptchaHelper` | 验证码（generateCaptcha/verifyCaptcha） |
| `MfaDeviceCodeHelper` | 设备验证码（sendDeviceCode/verifyDeviceCode） |
| `MfaTotpHelper` | TOTP（issue/verifyCode/generateRecoveryCode） |

## 注意事项

1. **`verifyCaptcha` 不可直接对外暴露**，否则会被重放/重试攻击，应由业务接口内部调用。
2. **设备码发送失败会清理验证码**：通知服务故障时已写入 Redis 的验证码会被删除，避免残留绕过发送限制。
3. **独立 Redis 连接**：`mfaRedisTemplate` 使用独立连接池与 database，与业务 Redis 隔离。
4. **校验错误限制是独立的**：设备码/TOTP 校验错误限制基于 deviceId/userInfo，与 IP 限制相互独立。

## Helper 使用示例

```java
public class LoginHelper {

    // 完整登录流程（IP检测 + 验证码 + MFA）
    public static ResponseData login(String username, String password, String ip,
                              String captchaId, String captchaSign) {
        ResponseData ipCheck = MfaFusionHelper.checkIpErrorLimit(ip);
        if (ipCheck.isError()) return ipCheck;
        if (ipCheck.isWarn()) {
            ResponseData captchaCheck = MfaFusionHelper.verifyCaptcha(ip, captchaId, captchaSign);
            if (captchaCheck.isNotSuccess()) return captchaCheck;
        }
        User user = UserHelper.verifyPassword(username, password);
        if (user == null) {
            MfaFusionHelper.incrementIpErrorTimes(ip, "密码错误");
            return ResponseData.errorMsg("用户名或密码错误");
        }
        MfaFusionHelper.clearIpErrorLimit(ip);
        return ResponseData.success(user);
    }

    // 发送短信验证码（含Captcha校验）
    public static ResponseData sendSmsCode(String mobile, String ip, String captchaId, String captchaSign) {
        return MfaFusionHelper.sendDeviceCode(ip, 1001L, MfaDeviceType.MOBILE_CODE.getValue(), mobile, captchaId, captchaSign);
    }

    // 校验短信验证码（同时验证IP）
    public static ResponseData verifySmsCode(String mobile, String code, String ip) {
        return MfaFusionHelper.verifyDeviceCode(ip, MfaDeviceType.MOBILE_CODE.getValue(), mobile, code);
    }

    // 生成TOTP密钥（绑定Google Authenticator）
    public static ResponseData<TotpSecretData> generateTotp(Long userId) {
        return MfaFusionHelper.issueTotpSecret("user:" + userId, "MyApp", 300);
    }

    // 校验TOTP验证码
    public static ResponseData verifyTotp(Long userId, String totpSecret, String totpCode, String ip) {
        return MfaFusionHelper.verifyTotpCode(ip, "user:" + userId, totpSecret, totpCode);
    }
}
```
