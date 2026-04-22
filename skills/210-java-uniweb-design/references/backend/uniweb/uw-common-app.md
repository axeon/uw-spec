### uw-common-app — Web应用公共类库

**Maven 坐标**: `com.umtone:uw-common-app`

基于 uw-auth-service 和 uw-dao 的后台应用通用功能：

| 功能 | 说明 |
|---|---|
| 自动化权限注入 | 基于 AuthServiceHelper + QueryParam，注入 saasId/mchId/groupId/userId 权限数据 |
| 关键操作日志 | `SysCritLog`，日志级别 BASE/REQUEST/RESPONSE/ALL/CRIT |
| 数据历史记录 | `SysDataHistoryHelper`，数据变更回滚支持 |
| JSON配置管理 | `JsonConfigHelper` + `JsonConfigBox` + `JsonConfigParam` |
| i18n国际化 | 内置 12 种语言支持 |

**预置 QueryParam**：
- `IdQueryParam` — id 查询参数
- `IdStateQueryParam` — id + 状态查询参数
- `AuthQueryParam` — Auth 权限查询参数
- `AuthIdQueryParam` — Auth + id 查询参数
- `AuthIdStateQueryParam` — Auth + id + 状态查询参数

**QueryParam API 详解**：

```java
// ========== AuthQueryParam 构造与绑定 ==========
// 默认构造器，自动从当前请求上下文绑定 saasId
new AuthQueryParam()

// 指定 saasId 构造器（用于非 Web 环境）
new AuthQueryParam(Long saasId)

// 链式绑定当前用户信息
AuthQueryParam bindUserId()    // 绑定当前用户ID
AuthQueryParam bindMchId()     // 绑定当前商户ID
AuthQueryParam bindUserType()  // 绑定当前用户类型

// 链式设置参数
AuthQueryParam saasId(Long saasId)
AuthQueryParam mchId(Long mchId)
AuthQueryParam userId(Long userId)

// ========== 使用示例 ==========
// 示例1：基础查询（自动注入 saasId）
@GetMapping("/list")
public ResponseData<DataList<User>> list(AuthQueryParam param) {
    // param 自动包含当前用户的 saasId
    return daoManager.list(User.class, param);
}

// 示例2：绑定更多权限字段
@GetMapping("/my-list")
public ResponseData<DataList<Order>> myList(AuthQueryParam param) {
    param.bindUserId().bindMchId();
    // 现在 param 包含 saasId + userId + mchId
    return daoManager.list(Order.class, param);
}

// 示例3：手动设置（非 Web 环境）
AuthQueryParam param = new AuthQueryParam(100L)
    .mchId(200L)
    .userId(300L);
```

**SysDataHistoryHelper API 详解**：

```java
// ========== 保存历史记录 ==========
// 保存实体历史（自动提取实体ID和名称）
static void saveHistory(DataEntity dataEntity)

// 保存实体历史并添加备注
static void saveHistory(DataEntity dataEntity, String remark)

// 完整参数保存
static void saveHistory(Serializable entityId, Object dataEntity, String entityName, String remark)

// ========== 使用示例 ==========
// 示例1：简单保存
@PostMapping("/update")
public ResponseData<User> update(@RequestBody User user) {
    // 更新前保存历史
    SysDataHistoryHelper.saveHistory(user, "更新前");
    
    ResponseData<User> result = userService.update(user);
    
    // 更新后保存历史
    result.onSuccess(updated -> SysDataHistoryHelper.saveHistory(updated, "更新后"));
    
    return result;
}

// 示例2：自定义实体信息
SysDataHistoryHelper.saveHistory(
    orderId,           // 实体ID
    order,             // 实体数据
    "订单信息",         // 实体名称
    "用户修改订单"       // 备注
);
```

**JsonConfigHelper API 详解**：

```java
// ========== 构建配置盒子 ==========
// 从参数列表和数据 Map 构建
static ResponseData<JsonConfigBox> buildParamBox(List<JsonConfigParam> configParamList, Map<String, String> configDataMap)

// 从参数列表和数据 JSON 构建
static ResponseData<JsonConfigBox> buildParamBox(List<JsonConfigParam> configParamList, String configDataJson)

// 从参数 JSON 和数据 JSON 构建
static ResponseData<JsonConfigBox> buildParamBox(String configParamJson, String configDataJson)

// ========== JsonConfigBox 参数获取 ==========
// 字符串参数
String getParam(String paramName)
String getParam(String paramName, String defaultValue)
String[] getParams(String paramName)

// 整数参数
int getIntParam(String paramName)
int getIntParam(String paramName, int defaultValue)
int[] getIntParams(String paramName)

// 长整数参数
long getLongParam(String paramName)
long getLongParam(String paramName, long defaultValue)
long[] getLongParams(String paramName)

// 双精度浮点参数
double getDoubleParam(String paramName)
double getDoubleParam(String paramName, double defaultValue)

// 布尔参数
boolean getBooleanParam(String paramName)
boolean getBooleanParam(String paramName, boolean defaultValue)

// Map 参数
Map<String, String> getMapParam(String paramName)

// 泛型对象参数（支持复杂类型）
<T> T getObjectParam(String paramName, Class<T> clazz)
<T> T getObjectParam(String paramName, TypeReference<T> typeReference)

// ========== 使用示例 ==========
// 示例1：定义配置参数枚举
public enum SystemConfig implements JsonConfigParam {
    SITE_NAME("siteName", ParamType.STRING, "MySite", "站点名称", null),
    MAX_UPLOAD_SIZE("maxUploadSize", ParamType.INT, "10485760", "最大上传大小(字节)", null),
    ALLOWED_TYPES("allowedTypes", ParamType.SET_STRING, "[\"jpg\",\"png\"]", "允许的文件类型", null);

    private final ParamData paramData;

    SystemConfig(String key, ParamType type, String value, String desc, String regex) {
        this.paramData = new ParamData(key, type, value, desc, regex);
    }

    @Override
    public ParamData getParamData() {
        return paramData;
    }
}

// 示例2：构建配置盒子
List<JsonConfigParam> params = Arrays.asList(SystemConfig.values());
String configData = "{\"siteName\":\"MyWebsite\",\"maxUploadSize\":\"20971520\"}";

ResponseData<JsonConfigBox> boxResult = JsonConfigHelper.buildParamBox(params, configData);
boxResult.onSuccess(box -> {
    String siteName = box.getParam("siteName");
    int maxSize = box.getIntParam("maxUploadSize");
    Set<String> types = box.getObjectParam("allowedTypes", new TypeReference<Set<String>>() {});
});
```