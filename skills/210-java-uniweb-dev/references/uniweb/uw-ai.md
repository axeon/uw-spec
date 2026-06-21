# uw-ai — AI集成模块

`uw-ai` 是 UW Base 的 AI 集成客户端模块，封装了与 AI 服务中心（`uw-ai-center`）的交互，提供对话生成（同步/流式）、结构化输出、翻译、图片生成、模型与 API 配置查询，以及 AI 工具的自动注册与执行。

- **Maven 坐标**：`com.umtone:uw-ai`
- **配置前缀**：`uw.ai`
- **入口**：静态门面 `uw.ai.AiClientHelper`（由 `UwAiAutoConfiguration` 自动装配注入底层 RPC Bean）

```yaml
uw:
  ai:
    ai-center-host: http://uw-ai-center   # AI 服务中心地址
```

> 应用名通过 `project.name` 注入（`UwAiProperties.appName`），启动时按应用维度注册/拉取工具元数据。

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 同步对话 | `AiClientHelper.generate(param)` | 返回 `ResponseData<String>` |
| 流式对话（SSE） | `AiClientHelper.chatGenerate(param)` | 返回 `Flux<String>` |
| 结构化输出 | `AiClientHelper.generateEntity(param, Class)` | 系统提示会追加 JSON Schema 说明 |
| 带工具对话 | param 设置 `.toolList(...)` + `.toolContext(...)` | 工具类需为 Spring Bean 并实现 `AiTool` |
| 批量翻译（数组） | `AiClientHelper.translateList(param)` | `textList` 为待翻译文本 |
| Map 翻译 | `AiClientHelper.translateMap(param)` | key 保留，value 翻译 |
| 生成图片 | `AiClientHelper.generateImage(param)` | `userPrompt` 为提示词，可选 `sessionId` |
| 查询模型/API 配置 | `AiClientHelper.listModelInfoByXxx(...)` | 按租户/API/ID/Code/Type 维度查询 |
| 自定义 AI 工具 | 实现 `AiTool<P,R>` + `@Component` | P 继承 `AiToolParam`，启动自动注册 |
| 绑定用户信息 | `param.bindAuthInfo()` 或 Builder 链 `.bindAuthInfo()` | 自动填充当前登录用户认证四元组 |

> **统一约定**：`configId` 与 `configCode` 二选一定位 AI 配置；推荐 `configCode`（语义化、跨环境）。认证信息 `saasId/userId/userType/userInfo` 通过 `bindAuthInfo()` 绑定，调用时透传给服务中心。

## AiClientHelper 方法签名

全部为静态方法，无需实例化（依赖由 Spring 自动装配注入静态字段）。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `generate(AiChatGenerateParam)` | `ResponseData<String>` | 同步生成对话 |
| `chatGenerate(AiChatGenerateParam)` | `Flux<String>` | 流式生成（SSE 文本片段） |
| `generateEntity(AiChatGenerateParam, Class<T>)` | `ResponseData<T>` | 生成并转换为 Java 对象 |
| `translateList(AiTranslateListParam)` | `ResponseData<AiTranslateResultData[]>` | 数组翻译 |
| `translateMap(AiTranslateMapParam)` | `ResponseData<AiTranslateResultData[]>` | Map 翻译 |
| `generateImage(AiImageGenerateParam)` | `ResponseData<AiImageResultData>` | 生成图片 |
| `listModelInfoBySaas(Long saasId, Long mchId)` | `ResponseData<List<AiModelConfigVo>>` | 按租户/商户查模型配置 |
| `listModelInfoByApi(Long apiId)` | `ResponseData<List<AiModelConfigVo>>` | 按 API 配置 ID 查模型配置 |
| `listModelInfoById(Long id)` | `ResponseData<AiModelConfigVo>` | 按模型配置 ID 查询 |
| `listModelInfoByCode(String configCode)` | `ResponseData<AiModelConfigVo>` | 按配置代码查询 |
| `listModelInfoByType(String modelType, String modelTag)` | `ResponseData<List<AiModelConfigVo>>` | 按模型类型/标签查询 |
| `listModelApiBySaas(Long saasId, Long mchId)` | `ResponseData<List<AiModelApiVo>>` | 按租户/商户查 API 配置 |
| `getModelApiById(Long id)` | `ResponseData<AiModelApiVo>` | 按 ID 查 API 配置 |
| `getModelApiByCode(String apiCode)` | `ResponseData<AiModelApiVo>` | 按配置代码查 API 配置 |
| `listToolMeta(String appName)` | `ResponseData<List<AiToolMeta>>` | 获取应用下工具元数据 |
| `updateToolMeta(AiToolMeta)` | `ResponseData` | 更新工具元数据 |

> `modelType` 取值：`CHAT / EMBEDDING / RERANK / TTS / OCR`。

## 对话参数

### AiChatGenerateParam

构造：`AiChatGenerateParam.builder()...build()`（Builder），也支持 `new AiChatGenerateParam()` + setter。

| 字段 | 类型 | 说明 |
|------|------|------|
| configId | long | AI 配置 ID（与 configCode 二选一） |
| configCode | String | AI 配置代码（与 configId 二选一） |
| userPrompt | String | 用户输入 |
| systemPrompt | String | 系统提示 |
| toolList | `List<AiToolCallInfo>` | 工具调用信息列表 |
| toolContext | `Map<String,Object>` | 工具上下文（透传给工具执行） |
| ragLibIds | `long[]` | RAG 知识库 ID 列表 |
| fileList | `MultipartFile[]` | 文件列表 |
| saasId | long | 租户 ID（`bindAuthInfo` 自动填充） |
| userId | long | 用户 ID（`bindAuthInfo` 自动填充） |
| userType | int | 用户类型（`bindAuthInfo` 自动填充） |
| userInfo | String | 用户信息（`bindAuthInfo` 自动填充） |

### AiChatSessionParam / AiChatMsgParam（多轮会话）

`AiChatSessionParam` 用于会话初始化，`AiChatMsgParam` 用于会话内追加消息。两者字段集与 `AiChatGenerateParam` 基本一致：

- `AiChatSessionParam` 额外字段：`windowSize`（窗口大小，保留历史消息数）。
- `AiChatMsgParam` 额外字段：`sessionId`（会话 ID）。

> 这两个参数类当前仅作为数据载体，本模块 RPC 未直接暴露入口；多轮会话由 uw-ai-center 侧管理。

### AiToolCallInfo

构造：`new AiToolCallInfo(toolCode, returnDirect)`，或 Builder。

| 字段 | 类型 | 说明 |
|------|------|------|
| toolCode | String | 工具代码（对应 `AiTool` 实现类的类名） |
| returnDirect | boolean | 是否将工具结果直接返回给用户（不经大模型二次加工） |

> ⚠️ 注意：`toolCode` 对应工具类的**类名**（`Class.getSimpleName()` 由服务中心解析），不是 `toolName()`，也不是 `toolVersion`。

## 翻译参数

### AiTranslateBaseParam（基类）

`AiTranslateListParam` 与 `AiTranslateMapParam` 的共同基类。

| 字段 | 类型 | 说明 |
|------|------|------|
| configId | long | AI 配置 ID |
| configCode | String | AI 配置代码（与 configId 二选一，两个子类 Builder 均支持） |
| systemPrompt | String | 系统提示 |
| langList | `List<String>` | 目标语言列表 |
| saasId/userId/userType/userInfo | — | 认证信息（`bindAuthInfo` 自动填充） |

### AiTranslateListParam（继承基类）

| 字段 | 类型 | 说明 |
|------|------|------|
| textList | `List<String>` | 待翻译的文本列表 |

构造：`AiTranslateListParam.builder().configId(1L).textList(...).langList(...).bindAuthInfo().build()`

### AiTranslateMapParam（继承基类）

| 字段 | 类型 | 说明 |
|------|------|------|
| textMap | `LinkedHashMap<String,String>` | 待翻译的 Map（key 为变量名，value 为待翻译文本） |

构造：`AiTranslateMapParam.builder().configId(1L).textMap(...).langList(...).bindAuthInfo().build()`

### AiTranslateResultData

每个目标语言一条结果。

| 字段 | 类型 | 说明 |
|------|------|------|
| lang | String | 目标语言 |
| resultMap | `Map<String,String>` | 翻译结果（key 对应原文/变量名，value 为译文） |

> ⚠️ 返回的是**数组**（`AiTranslateResultData[]`），每个元素对应一个目标语言，而非单个 key-value。

## 图片生成

### AiImageGenerateParam

构造：`AiImageGenerateParam.builder().configCode("default-image").prompt("...").bindAuthInfo().build()`

| 字段 | 类型 | 说明 |
|------|------|------|
| configId | long | AI 配置 ID（与 configCode 二选一） |
| configCode | String | AI 配置代码 |
| sessionId | long | 会话 ID，>0 保存到指定会话，否则自动创建 |
| userPrompt | String | 图片提示词（必填，Builder 用 `.prompt(...)` 设置） |
| saasId/userId/userType/userInfo | — | 认证信息 |

### AiImageResultData

| 字段 | 类型 | 说明 |
|------|------|------|
| sessionId | long | 会话 ID |
| imageUrlList | `List<String>` | 生成的图片 URL 列表 |

## 模型与 API 配置 VO

### AiModelConfigVo（模型配置）

字段：`id / saasId / mchId / apiId / vendorClass / modelType / modelTag / configCode / configName / configDesc / modelName / state / createDate / modifyDate`。

### AiModelApiVo（API 连接配置）

字段：`id / saasId / mchId / apiCode / apiName / apiDesc / apiUrl / apiKey / state / createDate / modifyDate`。

## AI 工具扩展

实现 `AiTool<P, R>` 接口并标注 `@Component`：P 继承 `AiToolParam`（内置认证四元组），R 通常为 `ResponseData<T>`。

| 方法 | 说明 |
|------|------|
| `toolName()` | 工具名称（展示用） |
| `toolDesc()` | 工具描述（给大模型理解用） |
| `toolVersion()` | 工具版本（升级时递增，框架据此判断是否需同步元数据） |
| `apply(P param)` | 工具执行逻辑（来自 `Function`） |
| `getParamType()` | 参数类型（默认由泛型反射获取） |
| `convertParam(String toolTip)` | 参数转换（默认 JSON 反序列化为 P） |

> 启动时 `UwAiAutoConfiguration` 扫描所有 `AiTool` Bean，与服务中心按 `toolClass + toolVersion` 比对，新增或版本升级时自动同步工具元数据（含基于 Swagger 注解生成的输入/输出 JSON Schema）。

**AiTool 实现示例**：
```java
@Component
public class WeatherTool implements AiTool<WeatherTool.Param, ResponseData<String>> {

    @Override public String toolName() { return "天气查询工具"; }
    @Override public String toolDesc() { return "查询指定城市的当前天气"; }
    @Override public String toolVersion() { return "1.0.0"; }

    @Override
    public ResponseData<String> apply(Param param) {
        return ResponseData.success(weatherService.getWeather(param.getCity()));
    }

    public static class Param extends AiToolParam {
        @io.swagger.v3.oas.annotations.media.Schema(
            description = "城市名称", requiredMode = Schema.RequiredMode.REQUIRED)
        private String city;
        public String getCity() { return city; }
        public void setCity(String city) { this.city = city; }
    }
}
```

工具元数据 `AiToolMeta` 字段：`id / appName / toolClass / toolVersion / toolName / toolDesc / toolInput / toolOutput`（后两者为输入/输出的 JSON Schema 字符串）。

## 工具执行入口

服务中心回调执行工具时，调用本应用的 `POST /rpc/ai/tool/execute`（`AiToolExecuteController`），以 RPC 用户身份（`UserType.RPC`）执行。

请求体 `AiToolExecuteParam`：

| 字段 | 类型 | 说明 |
|------|------|------|
| toolId | long | 工具 ID |
| toolClass | String | 工具类全限定名 |
| toolInput | String | 工具输入（JSON 字符串，由 `convertParam` 反序列化） |

## Helper 使用示例

```java
public class AiCallHelper {

    // 同步对话
    public static String chat(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configCode("default-chat")
            .userPrompt(message)
            .bindAuthInfo()
            .build();
        return AiClientHelper.generate(param).getData();
    }

    // 带系统提示的对话
    public static String chatWithSystemPrompt(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configCode("default-chat")
            .systemPrompt("你是一个专业的Java开发助手，请用简洁的语言回答问题。")
            .userPrompt(message)
            .bindAuthInfo()
            .build();
        return AiClientHelper.generate(param).getData();
    }

    // 流式对话（SSE）
    public static Flux<String> streamChat(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configCode("default-chat")
            .userPrompt(message)
            .bindAuthInfo()
            .build();
        return AiClientHelper.chatGenerate(param);
    }

    // 结构化输出（生成 Java 对象）
    public static UserIntent analyzeIntent(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configCode("default-chat")
            .systemPrompt("分析用户意图，提取关键信息")
            .userPrompt(message)
            .bindAuthInfo()
            .build();
        return AiClientHelper.generateEntity(param, UserIntent.class).getData();
    }

    // 带工具的对话（toolCode 对应工具类的类名）
    public static String chatWithTools(String message) {
        List<AiToolCallInfo> tools = List.of(
            new AiToolCallInfo("WeatherTool", false)
        );
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configCode("default-chat")
            .userPrompt(message)
            .toolList(tools)
            .toolContext(Map.of("userId", AuthServiceHelper.getUserId()))
            .bindAuthInfo()
            .build();
        return AiClientHelper.generate(param).getData();
    }

    // 批量翻译（数组）：每个目标语言返回一条 AiTranslateResultData
    public static Map<String, String> translateProducts(List<String> productNames) {
        AiTranslateListParam param = AiTranslateListParam.builder()
            .configCode("default-translate")
            .textList(productNames)
            .langList(List.of("en"))
            .bindAuthInfo()
            .build();
        ResponseData<AiTranslateResultData[]> resp = AiClientHelper.translateList(param);
        return resp.getData()[0].getResultMap(); // 取目标语言 en 的翻译结果
    }

    // Map 翻译（key 保留，value 翻译）
    public static Map<String, String> translateMap(Map<String, String> contentMap) {
        AiTranslateMapParam param = AiTranslateMapParam.builder()
            .configCode("default-translate")
            .textMap(new LinkedHashMap<>(contentMap))
            .langList(List.of("en"))
            .bindAuthInfo()
            .build();
        ResponseData<AiTranslateResultData[]> resp = AiClientHelper.translateMap(param);
        return resp.getData()[0].getResultMap();
    }

    // 生成图片
    public static AiImageResultData genImage(String prompt) {
        AiImageGenerateParam param = AiImageGenerateParam.builder()
            .configCode("default-image")
            .prompt(prompt)
            .bindAuthInfo()
            .build();
        return AiClientHelper.generateImage(param).getData();
    }

    // 按配置代码查询模型配置
    public static AiModelConfigVo getModel(String configCode) {
        return AiClientHelper.listModelInfoByCode(configCode).getData();
    }
}
```
