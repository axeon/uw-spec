### uw-notify-client — 通知客户端

**Maven 坐标**: `com.umtone:uw-notify-client`

基于 SSE（Server-Sent Events）的实时通知推送客户端，用于向指定用户或运营商推送 Web 通知。

**配置前缀**: `uw.notify`

```yaml
uw:
  notify:
    notify-center-host: http://uw-notify-center
```

#### NotifyClientHelper API

```java
public class NotifyClientHelper {
    
    /**
     * 推送Web通知
     * 
     * @param webNotifyMsg 通知消息对象
     * @return ResponseData
     */
    public static ResponseData pushNotify(WebNotifyMsg webNotifyMsg);
}
```

#### WebNotifyMsg 通知消息

```java
public class WebNotifyMsg implements Serializable {
    private long userId;              // 用户ID（0表示广播给所有用户）
    private long saasId;              // 运营商ID（0表示所有运营商）
    private NotifyBody notifyBody;    // 通知内容
    
    public WebNotifyMsg(long userId, long saasId, NotifyBody notifyBody);
    
    public static class NotifyBody implements Serializable {
        private String type;          // 通知类型
        private String subject;       // 消息标题
        private String content;       // 消息内容
        private Object data;          // 消息数据（任意对象）
        
        public NotifyBody(String type, Object data);
    }
}
```

#### 使用示例

```java
@Service
public class NotificationService {
    
    /**
     * 向指定用户推送通知
     */
    public void notifyUser(long userId, long saasId, String title, String message) {
        WebNotifyMsg.NotifyBody body = new WebNotifyMsg.NotifyBody();
        body.setType("SYSTEM");
        body.setSubject(title);
        body.setContent(message);
        body.setData(Map.of("timestamp", System.currentTimeMillis()));
        
        WebNotifyMsg msg = new WebNotifyMsg(userId, saasId, body);
        
        ResponseData response = NotifyClientHelper.pushNotify(msg);
        if (response.isSuccess()) {
            log.info("通知推送成功");
        }
    }
    
    /**
     * 广播通知给所有在线用户
     */
    public void broadcast(String type, Object data) {
        WebNotifyMsg msg = new WebNotifyMsg(
            0L,                           // userId=0 广播
            0L,                           // saasId=0 所有运营商
            new WebNotifyMsg.NotifyBody(type, data)
        );
        
        NotifyClientHelper.pushNotify(msg);
    }
    
    /**
     * 发送订单状态变更通知
     */
    public void notifyOrderStatus(long userId, long saasId, Order order) {
        WebNotifyMsg.NotifyBody body = new WebNotifyMsg.NotifyBody();
        body.setType("ORDER_STATUS_CHANGED");
        body.setSubject("订单状态变更");
        body.setContent("您的订单 " + order.getOrderNo() + " 状态已更新为 " + order.getStatus());
        body.setData(order);
        
        NotifyClientHelper.pushNotify(new WebNotifyMsg(userId, saasId, body));
    }
}
```

