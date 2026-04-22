### uw-common — 通用工具类库

**Maven 坐标**: `com.umtone:uw-common`

提供项目基础支撑能力：

| 类 | 功能 |
|---|---|
| `ResponseData<T>` | 统一响应数据封装，支持函数式链式调用（`onSuccess`/`onError`/`onFatal`） |
| `BitConfigUtils` | 位运算配置工具，int 存储 32 个开关，long 存储 64 个开关 |
| `DateUtils` | 灵活的日期处理工具 |
| `JsonUtils` | JSON 序列化/反序列化（基于 Jackson） |
| `AESUtils` / `RSAUtils` / `DigestUtils` | 加密解密与签名工具 |
| `SnowflakeIdGenerator` | 分布式雪花 ID 生成器 |
| `IpMatchUtils` | IP 匹配工具，支持 CIDR 格式 |
| `ByteArrayUtils` | 字节数组操作工具 |
| `NumCodeUtils` | 数字编码混淆工具 |
| `EnumUtils` | 枚举转换工具 |

**ResponseData 核心概念**：

```
ResponseData 封装结构：
- time: 时间戳
- state: 响应状态 (SUCCESS/WARN/ERROR/FATAL)
- code: 响应状态码
- msg: 响应消息
- data: 响应数据（泛型）
- type: 响应数据类型
```

**ResponseData API 详解**：

```java
// ========== 构造方法 ==========
// 静态构造器
static <T> ResponseData<T> of(long time, T data, String state, String code, String msg)

// 无数值成功
static <T> ResponseData<T> success()
// 带结果成功返回值
static <T> ResponseData<T> success(T t)

// 警告状态（业务逻辑警告，如参数校验失败）
static <T> ResponseData<T> warn(String msg)
static <T> ResponseData<T> warn(String code, String msg)
static <T> ResponseData<T> warn(String code, String msg, T t)

// 错误状态（系统错误，如数据库连接失败）
static <T> ResponseData<T> error(String msg)
static <T> ResponseData<T> error(String code, String msg)
static <T> ResponseData<T> error(String code, String msg, T t)

// 致命错误（严重系统故障）
static <T> ResponseData<T> fatal(String msg)
static <T> ResponseData<T> fatal(String code, String msg)
static <T> ResponseData<T> fatal(String code, String msg, T t)

// ========== 状态判断 ==========
boolean isSuccess()   // 是否成功
boolean isWarn()      // 是否警告
boolean isError()     // 是否错误
boolean isFatal()     // 是否致命错误
boolean isNotSuccess() // 是否非成功（警告/错误/致命）

// ========== 函数式链式调用 ==========
// 成功时执行
ResponseData<T> onSuccess(Consumer<T> consumer)
// 警告时执行
ResponseData<T> onWarn(Consumer<T> consumer)
// 错误时执行
ResponseData<T> onError(Consumer<T> consumer)
// 致命错误时执行
ResponseData<T> onFatal(Consumer<T> consumer)
// 非成功时执行
ResponseData<T> onNotSuccess(Consumer<T> consumer)

// ========== 使用示例 ==========
// 示例1：基础使用
public ResponseData<User> getUser(Long id) {
    User user = userService.findById(id);
    if (user == null) {
        return ResponseData.warn("USER_NOT_FOUND", "用户不存在");
    }
    return ResponseData.success(user);
}

// 示例2：链式处理
return userService.update(user)
    .onSuccess(updated -> log.info("用户更新成功: {}", updated.getId()))
    .onError(error -> log.error("用户更新失败: {}", error.getMsg()));

// 示例3：Controller 层统一返回
@GetMapping("/detail")
public ResponseData<User> detail(@RequestParam Long id) {
    return userService.getById(id);
}
```

**BitConfigUtils API 详解**：

```java
// ========== int 类型配置（32个开关） ==========
// 检查指定开关是否开启
static boolean isOn(int config, int configType)
// 开启指定开关
static int on(int config, int configType)
// 批量开启多个开关
static int on(int config, int... configTypes)
// 关闭指定开关
static int off(int config, int configType)
// 计算已开启的开关数量
static int countOn(int config)

// ========== long 类型配置（64个开关） ==========
static boolean isOn(long config, int configType)
static long on(long config, int configType)
static long on(long config, int... configTypes)
static long off(long config, int configType)
static int countOn(long config)
static List<Integer> calcOnPosList(long config)  // 计算开启的位置列表

// ========== 使用示例 ==========
// 定义开关常量
public static final int SWITCH_EMAIL_NOTIFY = 0;   // 邮件通知
public static final int SWITCH_SMS_NOTIFY = 1;     // 短信通知
public static final int SWITCH_PUSH_NOTIFY = 2;    // 推送通知

// 开启多个开关
int config = BitConfigUtils.on(0, SWITCH_EMAIL_NOTIFY, SWITCH_SMS_NOTIFY);

// 检查开关状态
if (BitConfigUtils.isOn(config, SWITCH_EMAIL_NOTIFY)) {
    // 发送邮件
}

// 关闭开关
config = BitConfigUtils.off(config, SWITCH_SMS_NOTIFY);
```

**DateUtils API 详解**：

```java
// ========== 日期格式常量 ==========
String FORMAT_DEFAULT = "yyyy-MM-dd HH:mm:ss"
String FORMAT_DATE = "yyyy-MM-dd"
String FORMAT_DATETIME = "yyyy-MM-dd HH:mm:ss"
String FORMAT_TIME = "HH:mm:ss"
String FORMAT_YEAR_MONTH = "yyyy-MM"
String FORMAT_MONTH_DAY = "MM-dd"
// ... 更多格式常量

// ========== 格式化方法 ==========
// 格式化为默认格式
static String format(Date date)
// 按指定格式格式化
static String format(Date date, String pattern)
// 格式化为日期部分
static String formatDate(Date date)
// 格式化为时间部分
static String formatTime(Date date)

// ========== 解析方法 ==========
// 解析默认格式字符串
static Date parse(String dateStr)
// 按指定格式解析
static Date parse(String dateStr, String pattern)

// ========== 日期计算 ==========
// 添加天数
static Date addDays(Date date, int days)
// 添加月数
static Date addMonths(Date date, int months)
// 添加小时
static Date addHours(Date date, int hours)
// 获取日期开始（00:00:00）
static Date getDayStart(Date date)
// 获取日期结束（23:59:59）
static Date getDayEnd(Date date)
// 获取两个日期相差天数
static int diffDays(Date date1, Date date2)

// ========== 使用示例 ==========
// 格式化当前时间
String now = DateUtils.format(new Date());
// 解析日期
Date date = DateUtils.parse("2025-01-15 10:30:00");
// 计算3天后
Date threeDaysLater = DateUtils.addDays(new Date(), 3);
// 获取今天开始时间
Date todayStart = DateUtils.getDayStart(new Date());
```

**JsonUtils API 详解**：

```java
// ========== JSON 解析 ==========
// 解析为对象
static <T> T parse(String json, Class<T> classType)
// 解析为泛型对象（支持复杂类型如 List<User>）
static <T> T parse(String json, TypeReference<T> typeReference)
// 解析为 JsonNode 树模型
static JsonNode parseTree(String json)

// ========== JSON 序列化 ==========
// 对象转为 JSON 字符串
static String toString(Object object)
// 美化输出（带缩进）
static String toPrettyString(Object object)
// 字节数组转为 JSON 字符串
static String toString(byte[] bytes)

// ========== 高级转换 ==========
// 对象转为另一种类型（如 Map 转 POJO）
static <T> T convert(Object fromValue, Class<T> toValueType)
static <T> T convert(Object fromValue, TypeReference<T> toValueTypeRef)

// ========== 使用示例 ==========
// 对象转 JSON
User user = new User();
String json = JsonUtils.toString(user);

// JSON 转对象
User parsedUser = JsonUtils.parse(json, User.class);

// 支持泛型集合
List<User> users = JsonUtils.parse(json, new TypeReference<List<User>>() {});

// Map 转 POJO
Map<String, Object> map = new HashMap<>();
User userFromMap = JsonUtils.convert(map, User.class);
```

**AESUtils API 详解**：

```java
// ========== 密钥生成 ==========
// 生成 AES 密钥（支持 128/192/256 位）
static byte[] generateKey(int keySize)
// 生成初始化向量（IV，16字节）
static byte[] generateIv()

// ========== 指定 IV 加密（密文不带 IV） ==========
static byte[] encrypt(byte[] key, byte[] iv, byte[] data)
static String encryptString(byte[] key, byte[] iv, String data)

// ========== 指定 IV 解密 ==========
static byte[] decrypt(byte[] key, byte[] iv, byte[] encrypted)
static String decryptString(byte[] key, byte[] iv, String encrypted)

// ========== 自动 IV 加密（密文带 IV） ==========
static byte[] encrypt(byte[] key, byte[] data)
static String encryptString(byte[] key, String data)

// ========== 自动 IV 解密（密文带 IV） ==========
static byte[] decrypt(byte[] key, byte[] encrypted)
static String decryptString(byte[] key, String encrypted)

// ========== 使用示例 ==========
// 生成密钥
byte[] key = AESUtils.generateKey(256);

// 方式1：自动 IV（推荐，密文自带 IV）
String encrypted = AESUtils.encryptString(key, "敏感数据");
String decrypted = AESUtils.decryptString(key, encrypted);

// 方式2：指定 IV（需要自行管理 IV）
byte[] iv = AESUtils.generateIv();
String encrypted2 = AESUtils.encryptString(key, iv, "敏感数据");
String decrypted2 = AESUtils.decryptString(key, iv, encrypted2);
```

**SnowflakeIdGenerator API 详解**：

```java
// ========== 获取实例 ==========
// 获取单例实例
static SnowflakeIdGenerator getInstance()

// ========== ID 生成 ==========
// 生成唯一 ID
synchronized long generateId()

// ========== 使用示例 ==========
// 获取生成器实例
SnowflakeIdGenerator generator = SnowflakeIdGenerator.getInstance();

// 生成唯一 ID
long id = generator.generateId();

// 用于数据库主键
user.setId(generator.generateId());
```

**其他工具类速查**：

| 类名 | 核心方法 | 用途 |
|------|---------|------|
| `RSAUtils` | `generateKeyPair()` / `encrypt()` / `decrypt()` / `sign()` / `verify()` | RSA 非对称加密签名 |
| `DigestUtils` | `md5()` / `sha1()` / `sha256()` / `hmacSha256()` | 哈希摘要计算 |
| `IpMatchUtils` | `match(ip, pattern)` / `isInRange(ip, cidr)` | IP 匹配，支持 CIDR |
| `ByteArrayUtils` | `toHex()` / `fromHex()` / `concat()` / `slice()` | 字节数组操作 |
| `NumCodeUtils` | `encode()` / `decode()` | 数字编码混淆 |
| `EnumUtils` | `getEnum()` / `getEnumMap()` / `getEnumList()` | 枚举转换工具 |

