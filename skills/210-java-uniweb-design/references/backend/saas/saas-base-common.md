## 核心基础模块（saas-base-common）

saas-base-common 是所有 SaaS 微服务的基础依赖，GroupId 为 `saas`，Parent POM 版本通过 `${revision}` 管理。提供SAAS租户管理、商户管理、消息通知、对象存储等基础能力。

### 模块依赖关系

**Maven 坐标**: `saas:saas-base-common`

```xml
<dependency>
    <groupId>saas</groupId>
    <artifactId>saas-base-common</artifactId>
    <version>${revision}</version>
</dependency>
```

**核心依赖**:
- `com.umtone:uw-common` - UniWeb 通用工具类库
- `com.umtone:uw-common-app` - Web 应用公共类库
- `com.umtone:uw-cache` - 缓存框架
- `com.umtone:uw-httpclient` - HTTP 客户端
- `com.umtone:uw-task` - 任务调度框架
- `com.umtone:uw-mydb-client` - 分库分表中间件
- `com.umtone:uw-notify-client` - 通知服务客户端


### 核心 Helper 类

| Helper 类 | 功能 | 主要方法 |
|-----------|------|----------|
| `SaasInfoHelper` | SAAS租户信息 | `getSaasInfo()`, `getSaasName()`, `getSaasCurrency()` |
| `SaasMchHelper` | SAAS商户信息 | `getMchInfo()`, `getMchName()`, `incrementSaleStats()` |
| `MsgHelper` | 系统消息通知 | `sendSms()`, `sendMail()` |
| `SysOssHelper` | 系统对象存储 | `genUploadParam()`, `getDownloadUrl()` |
| `SysDictHelper` | 系统字典数据 | `listDictByParentCode()` |
| `SaasDictHelper` | SAAS字典数据 | `listDictByParentCode()` |
| `SysAreaHelper` | 系统地区信息 | `getSaasAreaInfoByAreaCode()`, `getCityAreaCode()` |
| `LocaleHelper` | 系统国际化 | `getResolvedLocale()` |

### SaasInfoHelper API 详解

```java
// ========== SAAS租户查询 ==========
// 获取所有租户 ID 列表
static List<Long> getAllSaasIdList()

// 获取租户信息
static SaasInfoVo getSaasInfo(long saasId)

// 获取租户名称
static String getSaasName(long saasId)

// 获取租户币种
static String getSaasCurrency(long saasId)

// ========== 租户管理 ==========
// 启用租户
static void enable(long saasId)

// 停用租户
static void disable(long saasId)

// 缓存变更通知
static void publishChangeNotify(long saasId)
```

### SaasMchHelper API 详解

```java
// ========== SAAS商户查询 ==========
// 获取商户信息
static SaasMchInfoVo getMchInfo(long saasId, long mchId)

// 获取商户名称
static String getMchName(long saasId, long mchId)

// 获取商户币种
static String getMchCurrency(long saasId, long mchId)

// 获取商户等级信息
static SaasMchGradeVo getSaasMchLevel(long saasId, int mchType, long gradeLevel)

// ========== SAAS商户管理 ==========
// 保存商户信息
static ResponseData saveMchInfo(SaasMchInfoVo saasMchInfo)

// 增加销量统计
static void incrementSaleStats(long saasId, long distributorMchId, 
    long supplierMchId, int productType, int orderNum, int orderTimes, long orderPrice)

// 缓存变更通知
static void publishChangeNotify(long saasId, long mchId)
```

### MsgHelper API 详解

```java
// ========== 系统消息通知发送 ==========
// 发送短信
static ResponseData sendSms(MsgSmsVo msgSms)

// 发送邮件
static ResponseData sendMail(MsgMailVo msgMail)

// ========== 使用示例 ==========
// 发送短信
MsgSmsVo sms = new MsgSmsVo();
sms.setSaasId(saasId);
sms.setPhone("13800138000");
sms.setContent("您的验证码是：123456");
ResponseData result = MsgHelper.sendSms(sms);

// 发送邮件
MsgMailVo mail = new MsgMailVo();
mail.setSaasId(saasId);
mail.setTo("user@example.com");
mail.setSubject("欢迎注册");
mail.setContent("<h1>欢迎</h1>");
ResponseData result = MsgHelper.sendMail(mail);
```

### SysOssHelper API 详解

```java
// ========== 文件上传 ==========
// 获取预上传链接和参数
static ResponseData genUploadParam(long saasId, String configSid, 
    String refType, String refId, String fileName, long fileSize, 
    int accessType, Date expireDate)

// ========== 文件下载 ==========
// 获取下载链接
static ResponseData getDownloadUrl(long saasId, String configSid, 
    String filename, int ttl)

// 获取 OSS 站点地址
static ResponseData getOssSite(long saasId, String configSid)
```

---


