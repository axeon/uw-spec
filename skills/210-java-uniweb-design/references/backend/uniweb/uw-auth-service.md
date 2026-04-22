### uw-auth-service — 认证服务端

**Maven 坐标**: `com.umtone:uw-auth-service`

为 Web 应用提供统一认证、授权和日志记录功能。

**配置前缀**: `uw.auth.service`

```yaml
uw:
  auth:
    service:
      auth-center-host: http://uw-auth-center
      app-label: 我的应用
```

**权限声明注解 `@MscPermDeclare`**：

```java
@MscPermDeclare(
    user = UserType.ADMIN,        // 用户类型
    auth = AuthType.PERM,         // 授权类型
    log = ActionLog.ALL           // 日志级别
)
```

**USER 类型体系**：

| 类型 | 值 | 说明 | URL路径 |
|------|-----|------|---------|
| ANY | 0 | 任意用户 | `/open/*` |
| GUEST | 1 | C端访客 | `/guest/*` |
| RPC | 10 | 内部RPC服务 | `/rpc/*` |
| ROOT | 100 | 超级管理员（强制MFA） | `/root/*` |
| OPS | 110 | 开发运维（强制MFA） | `/ops/*` |
| ADMIN | 200 | 平台管理员 | `/admin/*` |
| SAAS | 300 | SAAS运营商 | `/saas/*` |
| MCH | 310 | SAAS商户 | `/mch/*` |

**AUTH 类型体系**：

| 类型 | 值 | 说明 |
|------|-----|------|
| NONE | 0 | 不验证鉴权, 仅用于特殊场景（如公开接口） | |
| TEMP | 1 | 临时授权, 仅用于特殊场景，MFA登录前状态 |
| USER | 2 | 仅验证用户类型, 不验证权限 |
| PERM | 3 | 验证用户类型和权限，会直接注册为菜单权限 |
| SUDO | 6 | 超级用户权限，会直接注册为菜单权限 |

**LOG 类型体系**：

| 类型 | 值 | 说明 |
|------|-----|------|
| NONE | -1 | 不记录日志 |
| BASE | 0 | 记录基础信息，仅包含IP,请求方法、路径、状态码、响应时间 |
| REQUEST | 1 | 仅记录请求日志 |
| RESPONSE | 2 | 仅记录响应日志 |
| ALL | 3 | 记录请求和响应日志 |
| CRIT | 9 | 记录全部数据,同时使用AuthCriticalLogStorage记录到关键存储器 |

**@MscPermDeclare 使用范例**：

```java

// ========== 使用示例 ==========
// 示例1：Controller 类级别声明
@RestController
@RequestMapping("/admin/user")
@MscPermDeclare(name = "用户管理", user = UserType.ADMIN, auth = AuthType.PERM)
public class UserController {
    
    // 示例2：方法级别声明（继承类配置，可覆盖）
    @GetMapping("/list")
    @MscPermDeclare(name = "用户列表", auth = AuthType.PERM, log = ActionLog.REQUEST)
    public ResponseData<DataList<User>> list(AuthQueryParam param) {
        return userService.list(param);
    }
    
    // 示例3：关键操作记录完整日志
    @PostMapping("/save")
    @MscPermDeclare(name = "保存用户", auth = AuthType.PERM, log = ActionLog.ALL)
    public ResponseData<User> save(@RequestBody User user) {
        return userService.save(user);
    }
    
    // 示例4：不验证权限（仅用于特殊场景）
    @GetMapping("/public/info")
    @MscPermDeclare(user = UserType.ANY, auth = AuthType.NONE)
    public ResponseData<String> publicInfo() {
        return ResponseData.success("公开信息");
    }
}

// 一级菜单权限声明，仅root/ops/admin/saas/mch带后台管理功能的用户角色
package uw.order.center.controller.admin;

@RestController
public class $PackageInfo$ {
    @MscPermDeclare(user = UserType.ADMIN)
    @Operation(summary = "订单管理模块", description = "订单管理模块")
    @GetMapping("/admin/order")
    public void info() {

    }

}

```

**AuthServiceHelper API 详解**：

```java
// ========== 应用信息获取 ==========
static long getAppId()                    // 获取当前应用ID
static String getAppLabel()               // 获取应用标签
static String getAppName()                // 获取应用名称
static String getAppVersion()             // 获取应用版本
static String getAppHost()                // 获取应用主机
static int getAppPort()                   // 获取应用端口
static String getAppInfo()                // 获取应用信息（name:version）
static String getAppHostInfo()            // 获取应用主机信息
static Map<String, Integer> getAppPermMap() // 获取应用权限映射

// ========== 当前用户信息获取 ==========
static AuthTokenData getContextToken()    // 获取当前线程的 Token 数据
static long getUserId()                   // 获取当前用户ID
static String getUserName()               // 获取当前用户名
static String getRealName()               // 获取用户真实姓名
static String getNickName()               // 获取用户昵称
static String getMobile()                 // 获取手机号
static String getEmail()                  // 获取邮箱
static long getGroupId()                  // 获取用户组ID
static int getUserType()                  // 获取用户类型
static int getUserGrade()                 // 获取用户等级
static long getSaasId()                   // 获取当前 SAAS ID
static long getMchId()                    // 获取当前商户ID
static String getLoginIp()                // 获取登录IP
static String getRemoteIp()               // 获取远程IP（考虑代理）

// ========== Token 管理 ==========
static void invalidToken(InvalidTokenData invalidToken)  // 使 Token 失效
static String genAnonymousToken(long saasId, long mchId) // 生成匿名 Token
static int getTokenType()                 // 获取当前 Token 类型

// ========== RPC 调用 ==========
static AuthServiceRpc getAuthServiceRpc() // 获取认证服务 RPC 客户端
static int getSaasUserLimit(long saasId)  // 获取 SAAS 用户数量限制

// ========== 系统日志记录 ==========
static void logSysInfo(String apiCode, String apiName, String apiIp, 
                       long saasId, Class<?> bizTypeClass, Serializable bizId, 
                       String bizLog, ResponseData<?> responseData)

static void logSysInfo(String apiCode, String apiName, String apiIp, 
                       long saasId, String bizType, Serializable bizId, 
                       String bizLog, ResponseData<?> responseData)

```

**AuthTokenData 令牌数据结构**：

```java
// 核心字段
int tokenType              // Token 类型（Access/Refresh）
int userType               // 用户类型（GUEST/ADMIN/SAAS 等）
long userId                // 用户ID
long saasId                // SAAS 运营商ID
long mchId                 // 商户ID
long groupId               // 用户组ID
int isMaster               // 是否管理员（0/1）
String userName            // 登录名
String realName            // 真实姓名
String nickName            // 昵称
String mobile              // 手机号
String email               // 邮箱
String userIp              // 登录IP
int userGrade              // 用户等级
long expireAt              // Token 过期时间戳
long createAt              // Token 创建时间戳
Set<Integer> permSet       // 权限ID集合
Map<String, String> configMap  // 用户配置

```