# uw-gateway-client — 网关客户端

**Maven 坐标**: `com.umtone:uw-gateway-client`
**配置前缀**: `uw.gateway`

网关管理客户端，通过 HTTP RPC 调用 gateway-center 管理运营商（SAAS）限速策略。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 设置运营商限速 | `GatewayClientHelper.updateSaasRateLimit(...)` | saasId>0、expireDate 必填不可为 null |
| 清除运营商限速 | `GatewayClientHelper.clearSaasRateLimit(saasId, remark)` | saasId>0 |

## GatewayClientHelper 方法签名

> **包路径**：`uw.gateway.client.GatewayClientHelper`

全部静态方法。共享 `RestClient` 由 `uw-auth-client` 提供（带鉴权拦截器）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `updateSaasRateLimit(saasId, limitSeconds, limitRequests, limitBytes, expireDate, remark)` | `ResponseData<Void>` | 设置运营商限速 |
| `clearSaasRateLimit(saasId, remark)` | `ResponseData<Void>` | 清除运营商限速 |

**`updateSaasRateLimit` 参数**：

| 参数 | 类型 | 说明 |
|------|------|------|
| saasId | long | 运营商 ID |
| limitSeconds | int | 限速统计窗口（秒） |
| limitRequests | int | 窗口内最大请求数 |
| limitBytes | int | 窗口内最大字节数 |
| expireDate | Date | 过期时间（必填，不可为 null） |
| remark | String | 备注信息 |

返回语义：`SUCCESS`=操作成功 / `ERROR`=操作失败。任何异常均被捕获并以 `ResponseData.errorMsg` 返回，**不会抛出**。

## Helper 使用示例

```java
public class GatewayHelper {

    public static ResponseData setRateLimit(long saasId) {
        // expireDate 必填，传 1 年后过期
        return GatewayClientHelper.updateSaasRateLimit(
            saasId, 1, 100, 1024 * 1024,
            new Date(System.currentTimeMillis() + 365L * 24 * 3600 * 1000), "防止接口滥用");
    }

    public static ResponseData setTempRateLimit(long saasId) {
        // 1小时后自动失效
        return GatewayClientHelper.updateSaasRateLimit(
            saasId, 60, 1000, 0,
            new Date(System.currentTimeMillis() + 3600_000L), "临时限速1小时");
    }

    public static ResponseData clearRateLimit(long saasId) {
        return GatewayClientHelper.clearSaasRateLimit(saasId, "恢复正常访问");
    }
}
```

## 接入步骤

1. 引入依赖 `com.umtone:uw-gateway-client`（依赖链含 `uw-auth-client`）。
2. 按需配置 `uw.gateway.gateway-center-host`（默认 `http://uw-gateway-center`）。
3. 确保容器中存在 `authRestClient`（由 `uw-auth-client` 提供）。
4. 启动后自动装配，直接调用静态方法。
