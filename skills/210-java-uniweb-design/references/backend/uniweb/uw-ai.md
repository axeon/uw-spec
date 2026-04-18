### uw-ai — AI集成模块

**Maven 坐标**: `com.umtone:uw-ai`

**配置前缀**: `uw.ai`

```yaml
uw:
  ai:
    ai-center-host: http://uw-ai-center
```

#### AiClientHelper API

```java
public class AiClientHelper {
    
    // ========== AI对话生成 ==========
    
    // 同步生成
    public static ResponseData<String> generate(AiChatGenerateParam param);
    
    // 流式生成（SSE）
    public static Flux<String> chatGenerate(AiChatGenerateParam param);
    
    // 生成并转换为Java对象
    public static <T> ResponseData<T> generateEntity(AiChatGenerateParam param, Class<T> clazz);
    
    // ========== AI工具 ==========
    
    // 获取工具元数据列表
    public static ResponseData<List<AiToolMeta>> listToolMeta(String appName);
    
    // 更新工具元数据
    public static ResponseData updateToolMeta(AiToolMeta aiToolMeta);
    
    // ========== AI翻译 ==========
    
    // 翻译列表
    public static ResponseData<AiTranslateResultData[]> translateList(AiTranslateListParam param);
    
    // 翻译Map
    public static ResponseData<AiTranslateResultData[]> translateMap(AiTranslateMapParam param);
}

// 对话生成参数
public class AiChatGenerateParam {
    private long saasId;           // 租户ID
    private long userId;           // 用户ID
    private int userType;          // 用户类型
    private String userInfo;       // 用户信息
    private long configId;         // AI配置ID（必填）
    private String userPrompt;     // 用户输入（必填）
    private String systemPrompt;   // 系统提示
    private List<AiToolCallInfo> toolList;  // 工具列表
    private Map<String,Object> toolContext; // 工具上下文
    private long[] ragLibIds;      // RAG知识库ID列表
    private MultipartFile[] fileList;  // 文件列表
    
    public void bindAuthInfo();    // 自动绑定当前登录用户信息
    
    public static Builder builder();
}

// Session初始化参数（多轮对话）
public class AiChatSessionParam {
    private long saasId;
    private long userId;
    private int userType;
    private String userInfo;
    private long configId;         // AI配置ID
    private String userPrompt;     // 用户输入
    private String systemPrompt;   // 系统提示
    private int windowSize;        // 窗口大小（保留历史消息数）
    private List<AiToolCallInfo> toolList;
    private long[] ragLibIds;
    
    public void bindAuthInfo();
    
    public static Builder builder();
}

// 工具调用信息
public class AiToolCallInfo {
    private String toolName;       // 工具名称
    private String toolVersion;    // 工具版本
}

// 翻译参数基类
public class AiTranslateBaseParam {
    private long configId;         // AI配置ID
    private String sourceLang;     // 源语言（auto表示自动检测）
    private String targetLang;     // 目标语言
    private String systemPrompt;   // 系统提示
}

// 列表翻译参数
public class AiTranslateListParam extends AiTranslateBaseParam {
    private String[] sourceArray;  // 待翻译文本数组
}

// Map翻译参数
public class AiTranslateMapParam extends AiTranslateBaseParam {
    private Map<String, String> sourceMap;  // 待翻译Map（key保留，value翻译）
}

// 翻译结果
public class AiTranslateResultData {
    private String source;         // 原文
    private String target;         // 译文
}

// 工具元数据
public class AiToolMeta {
    private String appName;        // 应用名称
    private String toolName;       // 工具名称
    private String toolDesc;       // 工具描述
    private String toolVersion;    // 工具版本
    private String paramSchema;    // 参数JSON Schema
}
```

#### AI工具扩展

```java
// 定义工具参数
@Data
public class WeatherToolParam extends AiToolParam {
    private String city;           // 城市
    private String date;           // 日期
}

// 实现AI工具
@Component
public class WeatherTool implements AiTool<WeatherToolParam, ResponseData<WeatherInfo>> {
    
    @Override
    public String toolName() {
        return "getWeather";
    }
    
    @Override
    public String toolDesc() {
        return "获取指定城市的天气信息";
    }
    
    @Override
    public String toolVersion() {
        return "1.0";
    }
    
    @Override
    public ResponseData<WeatherInfo> apply(WeatherToolParam param) {
        // 调用天气API
        WeatherInfo info = weatherService.getWeather(param.getCity(), param.getDate());
        return ResponseData.success(info);
    }
}

// 工具参数基类
public abstract class AiToolParam {
    // 工具参数基类，用于JSON Schema生成
}
```

#### 使用示例

```java
@Service
public class AiService {
    
    // 简单对话
    public String chat(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configId(1L)
            .userPrompt(message)
            .build();
        param.bindAuthInfo();  // 绑定当前用户信息
        
        ResponseData<String> response = AiClientHelper.generate(param);
        return response.getData();
    }
    
    // 带系统提示的对话
    public String chatWithSystemPrompt(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configId(1L)
            .systemPrompt("你是一个专业的Java开发助手，请用简洁的语言回答问题。")
            .userPrompt(message)
            .build();
        param.bindAuthInfo();
        
        return AiClientHelper.generate(param).getData();
    }
    
    // 流式对话（SSE）
    public Flux<String> streamChat(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configId(1L)
            .userPrompt(message)
            .build();
        param.bindAuthInfo();
        
        return AiClientHelper.chatGenerate(param);
    }
    
    // 结构化输出（生成Java对象）
    public UserIntent analyzeIntent(String message) {
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configId(1L)
            .systemPrompt("分析用户意图，提取关键信息")
            .userPrompt(message)
            .build();
        param.bindAuthInfo();
        
        ResponseData<UserIntent> response = AiClientHelper.generateEntity(param, UserIntent.class);
        return response.getData();
    }
    
    // 使用工具
    public String chatWithTools(String message) {
        List<AiToolCallInfo> tools = Arrays.asList(
            new AiToolCallInfo("getWeather", "1.0"),
            new AiToolCallInfo("sendEmail", "1.0")
        );
        
        AiChatGenerateParam param = AiChatGenerateParam.builder()
            .configId(1L)
            .userPrompt(message)
            .toolList(tools)
            .toolContext(Map.of("userId", AuthServiceHelper.getUserId()))
            .build();
        param.bindAuthInfo();
        
        return AiClientHelper.generate(param).getData();
    }
    
    // 批量翻译
    public Map<String, String> translateProducts(List<String> productNames) {
        AiTranslateListParam param = new AiTranslateListParam();
        param.setConfigId(2L);
        param.setSourceLang("zh");
        param.setTargetLang("en");
        param.setSourceArray(productNames.toArray(new String[0]));
        
        ResponseData<AiTranslateResultData[]> response = AiClientHelper.translateList(param);
        
        Map<String, String> result = new HashMap<>();
        for (AiTranslateResultData data : response.getData()) {
            result.put(data.getSource(), data.getTarget());
        }
        return result;
    }
    
    // 翻译Map
    public Map<String, String> translateMap(Map<String, String> contentMap) {
        AiTranslateMapParam param = new AiTranslateMapParam();
        param.setConfigId(2L);
        param.setSourceLang("auto");  // 自动检测
        param.setTargetLang("en");
        param.setSourceMap(contentMap);
        
        ResponseData<AiTranslateResultData[]> response = AiClientHelper.translateMap(param);
        
        Map<String, String> result = new HashMap<>();
        for (AiTranslateResultData data : response.getData()) {
            result.put(data.getSource(), data.getTarget());
        }
        return result;
    }
}

// 结构化输出示例
@Data
public class UserIntent {
    private String intent;         // 意图类型
    private double confidence;     // 置信度
    private List<String> entities; // 实体列表
    private Map<String, String> params;  // 参数
}
```

