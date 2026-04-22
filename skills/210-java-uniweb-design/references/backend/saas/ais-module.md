## AIS 模块（Application Interface Service - 应用接口服务）

AIS 模块提供可插拔的应用接口服务管理框架，通过 Linker（链接器）机制实现不同服务提供商的统一接入。

### AIS 核心概念

| 概念 | 说明 |
|------|------|
| **LinkerType（链接器类型）** | 定义一类接口的抽象，如邮件、短信、支付 |
| **Linker（链接器）** | 具体的接口实现，如 SMTP 邮件、AWS SES 邮件 |
| **LinkerConfig（链接器配置）** | 运营商/商户的具体配置参数 |
| **ConfigMeta（配置元数据）** | 配置的摘要信息，用于快速检索 |

### AIS 架构设计

**顶层：AisHelper（帮助类）** — 缓存管理、实例获取、配置检索

↓

**核心层**

| 组件 | 职责 |
|------|------|
| LinkerType | 链接器类型定义（如邮件、短信、支付的抽象） |
| Linker | 链接器具体实现（如 SMTP 邮件、微信支付） |
| Config | 链接器配置数据（运营商/商户参数） |

↓

**实现层**

| Linker 实现 | 说明 |
|-------------|------|
| MsgMail | 邮件接口 |
| MsgSms | 短信接口 |
| Payment | 支付接口 |

### AisLinker 接口定义

```java
/**
 * AIS 链接器接口
 * 所有链接器必须实现此接口
 */
public interface AisLinker {
    /**
     * 使用配置初始化
     */
    void config(AisLinkerConfigData configData);

    /**
     * AIS 链接器类型名
     */
    String typeName();

    /**
     * AIS 链接器类型版本
     */
    String typeVersion();

    /**
     * AIS 链接器类型开发者信息
     */
    String typeDevInfo();

    /**
     * AIS 链接器名
     */
    String name();

    /**
     * AIS 链接器版本
     */
    String version();

    /**
     * AIS 链接器开发者信息
     */
    String devInfo();

    /**
     * 公开配置参数（所有人可见）
     */
    List<JsonConfigParam> pubParam();

    /**
     * API 配置参数（运营商及以上可见）
     */
    List<JsonConfigParam> apiParam();

    /**
     * 系统配置参数（仅管理员可见）
     */
    List<JsonConfigParam> sysParam();

    /**
     * 日志配置参数（仅管理员可见）
     */
    List<JsonConfigParam> logParam();
}
```

### BaseAisLinker 抽象类

```java
/**
 * AIS 链接器基类
 * 接口的实现需要继承此类，AIS 系统将会提供自动注册和适配能力
 */
public abstract class BaseAisLinker implements AisLinker {
    private AisLinkerConfigData configData;

    @Override
    public void config(AisLinkerConfigData configData) {
        this.configData = configData;
    }

    // 获取配置参数的方法
    public String getParam(String paramName) {
        return configData.getParam(paramName);
    }

    public int getIntParam(String paramName) {
        return configData.getIntParam(paramName);
    }

    // ... 更多参数获取方法
}
```

### AIS Helper API 详解

```java
// ========== 按 Linker 类查询 ==========
// 获取配置列表
static List<AisLinkerConfigMeta> listLinkerConfigMetaByLinker(
    Long saasId, Class<? extends AisLinker> linkerClass, long mchId, String configSid)

// 获取配置信息列表
static List<AisLinkerConfigInfo> listLinkerConfigInfoByLinker(
    Long saasId, Class<? extends AisLinker> linkerClass, long mchId, String configSid)

// 获取实例列表
static <A extends AisLinker> List<A> listLinkerInstanceByLinker(
    Long saasId, Class<? extends AisLinker> linkerClass, long mchId, String configSid)

// 获取单个实例（取第一个）
static <A extends AisLinker> A getFitLinkerInstanceByLinker(
    Long saasId, Class<? extends AisLinker> linkerClass, long mchId, String configSid)

// ========== 按 Type 类查询 ==========
static List<AisLinkerConfigMeta> listLinkerConfigMetaByType(
    Long saasId, Class<? extends AisLinker> typeClass, long mchId, String configSid)

static <A extends AisLinker> A getFitLinkerInstanceByType(
    Long saasId, Class<? extends AisLinker> typeClass, long mchId, String configSid)

// ========== 直接使用 ConfigId ==========
static AisLinkerConfigData getLinkerConfigData(long configId)
static <A extends AisLinker> A getLinkerInstance(long configId)

// ========== 缓存刷新 ==========
static void linkerConfigChangeNotify(long linkerConfigId)
static void linkerMetaChangeNotify(long saasId)

// ========== 使用示例 ==========
// 示例1：获取支付链接器
PaymentLinker paymentLinker = AisHelper.getFitLinkerInstanceByLinker(
    saasId, WechatPaymentLinker.class, mchId, null);

// 示例2：获取所有邮件链接器
List<MsgMailLinker> mailLinkers = AisHelper.listLinkerInstanceByType(
    saasId, MsgMailLinker.class, 0, null);

// 示例3：使用配置 ID 直接获取
AisLinkerConfigData config = AisHelper.getLinkerConfigData(configId);
```

### 实现 Linker 的步骤

1. **定义 Linker 接口**（可选，用于类型分组）：

```java
/**
 * 支付链接器类型接口
 */
public interface PaymentLinkerType extends AisLinker {
}
```

2. **创建 Linker 实现类**，继承 `BaseAisLinker`：

```java
@Service
public class WechatPaymentLinker extends BaseAisLinker {
    @Override
    public String typeName() {
        return "支付网关接口";
    }

    @Override
    public String typeVersion() {
        return "1.0.0";
    }

    @Override
    public String typeDevInfo() {
        return "axeon";
    }

    @Override
    public String name() {
        return "微信支付";
    }

    @Override
    public String version() {
        return "1.0.0";
    }

    @Override
    public String devInfo() {
        return "axeon";
    }

    @Override
    public List<JsonConfigParam> pubParam() {
        return Collections.emptyList();
    }

    @Override
    public List<JsonConfigParam> apiParam() {
        return Arrays.asList(
            new JsonConfigParam("appId", ParamType.STRING, "", "应用ID", null),
            new JsonConfigParam("mchId", ParamType.STRING, "", "商户号", null),
            new JsonConfigParam("apiKey", ParamType.STRING, "", "API密钥", null)
        );
    }

    @Override
    public List<JsonConfigParam> sysParam() {
        return Collections.emptyList();
    }

    @Override
    public List<JsonConfigParam> logParam() {
        return Collections.emptyList();
    }

    // 业务方法
    public ResponseData<FinPayOrderResponse> payOrder(PayOrderInfo payOrderInfo) {
        // 实现支付逻辑
    }
}
```

3. **系统自动注册**：启动时会自动扫描并注册到 saas-base-app

---

