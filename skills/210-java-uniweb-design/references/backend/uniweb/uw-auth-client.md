### uw-auth-client — 认证客户端

**Maven 坐标**: `com.umtone:uw-auth-client`

内部 RPC 鉴权客户端，Auto-Configuration Starter。

**配置前缀**: `uw.auth.client`

```yaml
uw:
  auth:
    client:
      login-id: rpc-user
      login-pass: rpc-pass
      http-pool:
        max-total: 100000
        default-max-per-route: 10000
        connect-timeout: 30000
        connection-request-timeout: 180000
        socket-timeout: 30000
```

**注入 Bean**：
- `authRestTemplate` — 自动注入 Token 的 RestTemplate（@Primary）
- `authWebClient` — 自动注入 Token 的 WebClient（@Primary）
- `authClientTokenHelper` — Token 管理器，一般不用手动注入

**AuthClientTokenHelper API 详解**：

```java
// ========== Token 获取与管理 ==========
String getToken()              // 获取有效 Token（自动处理登录/刷新）
void invalidate()              // 使当前 Token 失效（强制重新获取）

// ========== 配置属性 ==========
// uw.auth.client 配置项：
- auth-center-host: http://uw-auth-center    // 认证中心地址
- login-id: your-service-account             // 服务账号
- login-pass: your-password                  // 服务密码
- saasId: 0                                  // SAAS ID
- userType: 10                               // 用户类型,默认RPC
```

**使用示例**：

```java
@Service
public class RemoteService {
    
    // 注入带鉴权的 RestTemplate（自动管理 Token）
    public RemoteService(RestTemplate authRestTemplate) {
        this.authRestTemplate = authRestTemplate;
    }
    
    public String callRemoteApi() {
        // 方式1：使用 authRestTemplate（推荐，自动注入 Token）
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);
        HttpEntity<String> entity = new HttpEntity<>("{}", headers);
        
        ResponseEntity<String> response = authRestTemplate.exchange(
            "http://uw-other-center/api/data",
            HttpMethod.GET,
            entity,
            String.class
        );
        return response.getBody();
        
    }
}
```