# uw-notify-client — 通知客户端

**Maven 坐标**: `com.umtone:uw-notify-client`
**配置前缀**: `uw.notify`

Web 通知推送客户端，通过 HTTP RPC 调用 notify-center 推送通知，notify-center 再经 SSE（Server-Sent Events）等实时通道下发至前端。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 推送通知(指定用户) | `NotifyClientHelper.pushNotify(new WebNotifyMsg(userId, saasId, body))` | userId>0 定向 |
| 广播(全用户) | userId 设为 `0` | 0 表示该运营商下广播 |
| 广播(全运营商) | saasId 设为 `0` | 0 表示所有运营商 |

## NotifyClientHelper 方法签名

> **包路径**：`uw.notify.client.NotifyClientHelper`

全部静态方法。共享 `RestClient` 由 `uw-auth-client` 提供（带鉴权拦截器）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `pushNotify(WebNotifyMsg webNotifyMsg)` | `ResponseData<Void>` | 推送 Web 通知 |

返回语义：`SUCCESS`=推送成功 / `ERROR`=推送失败。任何异常均被捕获并以 `ResponseData.errorMsg` 返回，**不会抛出**。

## WebNotifyMsg

> **包路径**：`uw.notify.client.vo.WebNotifyMsg`

构造：`new WebNotifyMsg(long userId, long saasId, NotifyBody body)` — userId=0 广播该运营商，saasId=0 所有运营商

| 字段 | 类型 | 说明 |
|------|------|------|
| userId | long | 用户ID，0 表示该运营商下广播 |
| saasId | long | 运营商编号，0 表示所有运营商 |
| notifyBody | NotifyBody | 通知内容 |

### NotifyBody（内部类）

构造：`new WebNotifyMsg.NotifyBody(String type, Object data)` 或 `new NotifyBody(type, subject, content, data)` 或无参 + setter

| 字段 | 类型 | 说明 |
|------|------|------|
| type | String | 通知类型（业务自定义，如 `"SYSTEM"`） |
| subject | String | 消息标题 |
| content | String | 消息内容 |
| data | Object | 消息数据（任意对象） |

## Helper 使用示例

```java
public class NotificationHelper {

    public static ResponseData<Void> notifyUser(long userId, long saasId, String title, String message) {
        WebNotifyMsg.NotifyBody body = new WebNotifyMsg.NotifyBody();
        body.setType("SYSTEM");
        body.setSubject(title);
        body.setContent(message);
        body.setData(Map.of("timestamp", System.currentTimeMillis()));
        return NotifyClientHelper.pushNotify(new WebNotifyMsg(userId, saasId, body));
    }

    public static void broadcast(String type, Object data) {
        NotifyClientHelper.pushNotify(new WebNotifyMsg(0L, 0L, new WebNotifyMsg.NotifyBody(type, data)));
    }
}
```

## 接入步骤

1. 引入依赖 `com.umtone:uw-notify-client`（依赖链含 `uw-auth-client`）。
2. 按需配置 `uw.notify.notify-center-host`（默认 `http://uw-notify-center`）。
3. 确保容器中存在 `authRestClient`（由 `uw-auth-client` 提供）。
4. 启动后自动装配，直接调用静态方法。
