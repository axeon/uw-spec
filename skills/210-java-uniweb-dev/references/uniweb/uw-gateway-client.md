# uw-gateway-client — 网关客户端

**Maven 坐标**: `com.umtone:uw-gateway-client`
**配置前缀**: `uw.gateway`

网关管理客户端，通过 HTTP RPC 调用 gateway-center 管理运营商（SAAS）限速策略。限速维度（按 IP / SAAS / 用户类型 / 用户 ID，及其带 URI 的组合）由调用方通过 `SaasRateLimitParam.limitType` 指定。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 设置运营商限速 | `GatewayClientHelper.updateSaasRateLimit(SaasRateLimitParam)` | saasId>0、limitType 合法、expireDate 必填且晚于当前、remark 必填 |
| 清除运营商限速 | `GatewayClientHelper.clearSaasRateLimit(saasId, remark)` | saasId>0 |

## GatewayClientHelper 方法签名

> **包路径**：`uw.gateway.client.GatewayClientHelper`

全部静态方法。共享 `RestClient` 由 `uw-auth-client` 提供（带鉴权拦截器）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `updateSaasRateLimit(SaasRateLimitParam param)` | `ResponseData<Void>` | 设置运营商限速，参数通过 vo + builder 传入 |
| `clearSaasRateLimit(long saasId, String remark)` | `ResponseData<Void>` | 清除运营商限速 |

返回语义：`SUCCESS`=操作成功 / `ERROR`=参数非法或操作失败。任何 RPC 异常均被捕获并以 `ResponseData.errorMsg` 返回，**不会抛出**。

业务级校验（limitType 合法性、按用户维度时 userType/userId 必填、limitSeconds>0、expireDate 晚于当前等）由 gateway-center 侧统一执行；client 仅做必填前置校验（param/saasId/expireDate/remark 非空）。

## SaasRateLimitParam

> **包路径**：`uw.gateway.client.vo.SaasRateLimitParam`，通过 `SaasRateLimitParam.builder()` 链式构造。

| 字段 | 类型 | 默认 | 必填 | 说明 |
|------|------|------|------|------|
| `saasId` | long | — | 是 | 运营商 ID（>0） |
| `limitType` | int | — | 是 | 限速类型裸值，对应服务端 `MscAclRateLimitType` 的 value（见下表） |
| `userType` | int | -1 | 仅按用户维度时 | 用户类型，仅 USER_TYPE/USER_ID/及其 URI 组合类型需要 |
| `userId` | long | -1 | 仅按用户维度时 | 用户 ID，同上 |
| `limitSeconds` | int | — | 是（limitType≠NONE） | 限速统计窗口（秒），>0 |
| `limitRequests` | int | — | 是（limitType≠NONE） | 窗口内最大请求数（与 limitBytes 至少一个>0） |
| `limitBytes` | int | — | 是（limitType≠NONE） | 窗口内最大字节数（与 limitRequests 至少一个>0） |
| `expireDate` | Date | — | 是 | 过期时间，须晚于当前时间 |
| `remark` | String | — | 是 | 备注（非空） |

**limitType 合法值**（裸值，对应服务端 `uw.gateway.center.acl.rate.constant.MscAclRateLimitType`）：

| 值 | 含义 | 是否需要 userType/userId |
|----|------|--------------------------|
| -1 | NONE 不限速 | 否 |
| 0 | IP 限速 | 否 |
| 1 | SAAS_LEVEL 限速 | 否 |
| 2 | USER_TYPE 限速 | 是 |
| 3 | USER_ID 限速 | 是 |
| 11 | SAAS_LEVEL + URI 限速 | 否 |
| 12 | USER_TYPE + URI 限速 | 是 |
| 13 | USER_ID + URI 限速 | 是 |

> ⚠️ client 项目内不含 `MscAclRateLimitType` 枚举类，调用方直接传裸值（如 `0`、`3`），不要引用该类。

## Helper 使用示例

```java
import uw.gateway.client.GatewayClientHelper;
import uw.gateway.client.vo.SaasRateLimitParam;
import uw.common.response.ResponseData;
import java.util.Date;

public class GatewayHelper {

    public static ResponseData setIpRateLimit(long saasId) {
        // 按 IP 限速：每 1 秒 100 请求、1MB，1 年后过期
        return GatewayClientHelper.updateSaasRateLimit(
            SaasRateLimitParam.builder()
                .saasId(saasId)
                .limitType(0)              // 0=IP限速
                .limitSeconds(1)
                .limitRequests(100)
                .limitBytes(1024 * 1024)
                .expireDate(new Date(System.currentTimeMillis() + 365L * 24 * 3600 * 1000))
                .remark("防止接口滥用")
                .build());
    }

    public static ResponseData setUserRateLimit(long saasId, int userType, long userId) {
        // 按用户 ID 限速：针对特定用户，1 小时后自动失效
        return GatewayClientHelper.updateSaasRateLimit(
            SaasRateLimitParam.builder()
                .saasId(saasId)
                .limitType(3)              // 3=USER_ID限速
                .userType(userType)        // 按用户维度时必填
                .userId(userId)            // 按用户维度时必填
                .limitSeconds(60)
                .limitRequests(1000)
                .expireDate(new Date(System.currentTimeMillis() + 3600_000L))
                .remark("临时限速1小时")
                .build());
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
