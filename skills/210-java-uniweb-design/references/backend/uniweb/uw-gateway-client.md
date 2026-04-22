### uw-gateway-client — 网关客户端

**Maven 坐标**: `com.umtone:uw-gateway-client`

网关管理客户端，用于调用 uw-gateway-center 的 RPC 接口，管理运营商限速策略。

**配置前缀**: `uw.gateway`

#### GatewayClientHelper API

```java
public class GatewayClientHelper {
    
    /**
     * 设置运营商限速
     * 
     * @param saasId        运营商ID
     * @param limitSeconds  限速时间窗口（秒）
     * @param limitRequests 限制请求数
     * @param limitBytes    限制字节数
     * @param expireDate    过期时间
     * @param remark        备注说明
     * @return ResponseData
     */
    public ResponseData updateSaasRateLimit(long saasId, int limitSeconds, int limitRequests, 
                                            int limitBytes, Date expireDate, String remark);
    
    /**
     * 清除运营商限速
     * 
     * @param saasId  运营商ID
     * @param remark  备注说明
     * @return ResponseData
     */
    public ResponseData clearSaasRateLimit(long saasId, String remark);
}
```

#### 使用示例

```java
@Service
public class GatewayService {
    
\
    /**
     * 为运营商设置限速：每秒最多100请求，1MB流量
     */
    public void setRateLimit(long saasId) {
        ResponseData response = GatewayClientHelper.updateSaasRateLimit(
            saasId,           // 运营商ID
            1,                // 1秒窗口
            100,              // 最多100请求
            1024 * 1024,      // 最多1MB
            null,             // 无过期时间
            "防止接口滥用"     // 备注
        );
        
        if (response.isSuccess()) {
            log.info("限速设置成功");
        }
    }
    
    /**
     * 清除运营商限速
     */
    public void clearRateLimit(long saasId) {
        gatewayClientHelper.clearSaasRateLimit(saasId, "恢复正常访问");
    }
}
```

