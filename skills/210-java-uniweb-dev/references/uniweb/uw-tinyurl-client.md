# uw-tinyurl-client — 短链接客户端

**Maven 坐标**: `com.umtone:uw-tinyurl-client`
**配置前缀**: `uw.tinyurl`

短链接生成客户端，通过 HTTP RPC 调用 tinyurl-center 生成长链接对应的短链码，支持密语保护与过期时间。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 生成短链接 | `TinyurlClientHelper.generate(TinyurlParam)` | 返回 `ResponseData<String>`，data 为短链码 |
| 带密语 | param 设置 `.secretData(secret)` | 访问时需输入密语 |
| 带过期 | param 设置 `.expireDate(date)` | null 表示永不过期 |

## TinyurlClientHelper 方法签名

> **包路径**：`uw.tinyurl.client.TinyurlClientHelper`

全部静态方法。共享 `RestClient` 由 `uw-auth-client` 提供（带鉴权拦截器）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `generate(TinyurlParam param)` | `ResponseData<String>` | 生成短链（data 为短链码） |

返回语义：`SUCCESS`=生成成功，data 为短链码 / `ERROR`=生成失败。任何异常均被捕获并以 `ResponseData.errorMsg` 返回，**不会抛出**。

## TinyurlParam

> **包路径**：`uw.tinyurl.client.vo.TinyurlParam`

构造：推荐 Builder — `TinyurlParam.builder().saasId(saasId).url(longUrl).build()`；也支持 `new TinyurlParam()` + setter，或 `TinyurlParam.builder(copy)` 基于已有对象拷贝修改。

| 字段 | 类型 | 说明 |
|------|------|------|
| saasId | long | 运营商ID |
| objectType | String | 对象类型（分类统计用，如 `"LINK"`） |
| objectId | long | 对象ID |
| url | String | 原始长URL（必填） |
| secretTips | String | 密语提示 |
| secretData | String | 密语（访问时需输入） |
| expireDate | Date | 过期时间，null 表示永不过期 |

## Helper 使用示例

```java
public class ShortUrlHelper {

    public static String createShortUrl(long saasId, String longUrl) {
        TinyurlParam param = TinyurlParam.builder()
            .saasId(saasId).objectType("LINK").url(longUrl).build();
        ResponseData<String> response = TinyurlClientHelper.generate(param);
        return response.isSuccess() ? "https://t.example.com/" + response.getData() : null;
    }

    public static String createSecretUrl(long saasId, String longUrl, String secret) {
        TinyurlParam param = TinyurlParam.builder()
            .saasId(saasId).objectType("SECRET_LINK").url(longUrl)
            .secretTips("请输入访问密码").secretData(secret)
            .expireDate(new Date(System.currentTimeMillis() + 7L * 24 * 3600 * 1000))
            .build();
        ResponseData<String> response = TinyurlClientHelper.generate(param);
        return response.isSuccess() ? response.getData() : null;
    }
}
```

## 接入步骤

1. 引入依赖 `com.umtone:uw-tinyurl-client`（依赖链含 `uw-auth-client`）。
2. 按需配置 `uw.tinyurl.tinyurl-center-host`（默认 `http://uw-tinyurl-center`）。
3. 确保容器中存在 `authRestClient`（由 `uw-auth-client` 提供）。
4. 启动后自动装配，直接调用静态方法。
