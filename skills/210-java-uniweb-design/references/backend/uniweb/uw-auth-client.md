### uw-auth-client — 认证客户端

**Maven 坐标**: `com.umtone:uw-auth-client`

内部 RPC 鉴权客户端。

**配置前缀**: `uw.auth.client`

```yaml
uw:
  auth:
    client:
      login-id: rpc-user
      login-pass: rpc-pass
```

**注入 Bean**：
- `authRestTemplate` — 自动注入 Token 的 RestTemplate（@Primary）
- `authWebClient` — 自动注入 Token 的 WebClient（@Primary）


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