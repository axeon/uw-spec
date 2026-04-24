# uw-tinyurl-client — 短链接客户端

**Maven 坐标**: `com.umtone:uw-tinyurl-client`

短链接生成与解析客户端，用于将长 URL 转换为短链接，支持密语保护和过期时间设置。

## TinyurlClientHelper API

```java
public class TinyurlClientHelper {
    
    /**
     * 生成短链接
     * 
     * @param tinyurlParam 短链接参数
     * @return ResponseData<String> 生成的短链接码
     */
    public static ResponseData<String> generate(TinyurlParam tinyurlParam);
}
```

## TinyurlParam 短链接参数

```java
public class TinyurlParam implements Serializable {
    private long saasId;           // 运营商ID
    private String objectType;     // 对象类型（用于分类统计）
    private long objectId;         // 对象ID
    private String url;            // 原始长URL
    private String secretTips;     // 密语提示（如设置了密语）
    private String secretData;     // 密语（访问时需要输入）
    private Date expireDate;       // 过期时间
    
    public static Builder builder();
}
```

## 使用示例

```java
@Service
public class ShortUrlService {
    
    /**
     * 生成普通短链接
     */
    public String createShortUrl(long saasId, String longUrl) {
        TinyurlParam param = TinyurlParam.builder()
            .saasId(saasId)
            .objectType("LINK")
            .url(longUrl)
            .build();
        
        ResponseData<String> response = TinyurlClientHelper.generate(param);
        
        if (response.isSuccess()) {
            String shortCode = response.getData();
            return "https://t.example.com/" + shortCode;
        }
        return null;
    }
    
    /**
     * 生成带密语的短链接
     */
    public String createSecretUrl(long saasId, String longUrl, String secret) {
        TinyurlParam param = TinyurlParam.builder()
            .saasId(saasId)
            .objectType("SECRET_LINK")
            .url(longUrl)
            .secretTips("请输入访问密码")
            .secretData(secret)
            .expireDate(new Date(System.currentTimeMillis() + 7 * 24 * 3600 * 1000))  // 7天过期
            .build();
        
        ResponseData<String> response = TinyurlClientHelper.generate(param);
        return response.getData();
    }
    
    /**
     * 为订单生成短链接
     */
    public String createOrderShortUrl(long saasId, long orderId, String orderDetailUrl) {
        TinyurlParam param = TinyurlParam.builder()
            .saasId(saasId)
            .objectType("ORDER")
            .objectId(orderId)
            .url(orderDetailUrl)
            .expireDate(new Date(System.currentTimeMillis() + 30 * 24 * 3600 * 1000))  // 30天过期
            .build();
        
        ResponseData<String> response = TinyurlClientHelper.generate(param);
        return "https://t.example.com/" + response.getData();
    }
}
```

