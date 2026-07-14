# uw-common — 通用工具类库

**Maven 坐标**: `com.umtone:uw-common`

## AI 决策速查

| 我要做什么 | 用什么 | 关键约束 |
|-----------|--------|---------|
| 成功返回 | `ResponseData.success(data)` | — |
| 业务校验失败 | `ResponseData.warnCode(ResponseCode)` 或 `warnCode("CODE","msg")` | 禁止 `warn("CODE","msg")`（泛型陷阱） |
| 系统异常 | `ResponseData.errorCode(ResponseCode)` 或 `errorCode("CODE","msg")` | 禁止 `error("CODE","msg")`（泛型陷阱） |
| 致命错误 | `ResponseData.fatalCode(ResponseCode)` | — |
| 判断成功 | `result.isSuccess()` | — |
| 判断非成功 | `result.isNotSuccess()` | 不要再叠加 getData()==null 判断 |
| 链式处理 | `result.onSuccess(data -> {...}).onError(data -> {...})`（块 lambda 消歧，禁造型） | — |
| 货币计算 | `MoneyUtils` | 金额以 long（分）为单位，禁止浮点 |
| 数据校验 | `ValidateUtils.isXxx(value)` | 覆盖字符串/整数/身份证/手机号等 |
| 数据脱敏 | `MaskUtils.maskXxx(value)` | 所有方法 mask 前缀；null 返回 null；手机/身份证用 `maskChinaMobile/maskChinaIdCard` |
| JSON序列化 | `JsonUtils.toString(object)` | — |
| JSON反序列化 | `JsonUtils.parse(json, Class)` 或 `parse(json, new TypeReference<List<T>>(){})` | parse 失败抛 RuntimeException 且 message **含原始数据**，对外边界必须 catch + 固定文案，禁止回传 `e.getMessage()` |
| AES加密 | `AESUtils.encryptString(key, data)` | 推荐自动IV版本（密文自带IV） |
| 当前时间 | `SystemClock.nowDate()` / `SystemClock.now()` | **不要用 `new Date()` / `System.currentTimeMillis()`**，createDate/modifyDate 赋值必用此 |
| 日期格式化 | `DateTools.dateToString(date, DateTools.DATE_TIME)` | 方法名是 `dateToString` 不是 `format` |
| 日期解析 | `DateTools.stringToDate(str)` 或 `stringToDate(str, pattern)` | 方法名是 `stringToDate` 不是 `parse` |
| 日期偏移 | `DateTools.offsetDay(date, n)` | 方法名不是 addDays |
| 当天开始 | `DateTools.beginOfToday(date)` | 方法名不是 getDayStart |
| 当天结束 | `DateTools.endOfToday(date)` | 方法名不是 getDayEnd |
| 摘要签名 | `DigestUtils.sign(msg, DigestUtils.Algorithm.SHA_256)` | 无独立 md5/sha256 方法，用 `Algorithm` 枚举 |
| HMAC签名 | `HmacUtils.sign(message, secret)` / `verify(...)` | HMAC-SHA256 |
| 雪花ID | `SnowflakeIdGenerator.getInstance().generateId()` | — |
| 位运算开关 | `BitConfigUtils.isOn/on/off(config, bitIndex)` | int 32位，long 64位 |
| 字符串拼接 | `StringTools.join(long[]/int[]/String[]/Collection, [sep])` | null/空返回空串，元素null跳过 |
| 数字串切分 | `StringTools.splitToLongArray/splitToIntArray(s, [sep])` | 自动trim、跳过空白段、脏数据逐条跳过+WARN，不走正则 |
| SQL占位符 | `StringTools.buildPlaceholders(count)` / `buildInClause(ids)` | 空集合兜底`(0)`避免语法错；仅服务端白名单ID |
| 命名风格转换 | `StringTools.toSnakeCase/toCamelCase/toKebabCase(str)` | 自动探测源格式，正确处理HTTPResponse缩写 |
| 字符串相似度 | `StringTools.levenshteinSimilarity/lcsSimilarity/ngramSimilarity(s1,s2)` | 返回[0,1]；中文场景用ChineseUtils万分制版本 |

## ResponseData

> **包路径**：`uw.common.response.ResponseData`

构造：`ResponseData.success(data)` / `ResponseData.warnCode(code)` / `ResponseData.errorCode(code)` / `ResponseData.fatalCode(code)`

| 字段 | 类型 | 说明 |
|------|------|------|
| time | long | 时间戳 |
| state | String | 响应状态（SUCCESS/WARN/ERROR/FATAL） |
| code | String | 响应状态码 |
| msg | String | 响应消息 |
| data | T（泛型） | 响应数据 |
| type | String | 响应数据类型 |

**状态判断**：`isSuccess()` / `isWarn()` / `isError()` / `isFatal()` / `isNotSuccess()`

**链式回调**（核心设计 — 每个方法有 3 种重载）：

| 重载 | 签名 | 返回值 | 典型用途 |
|------|------|--------|---------|
| **Function 版（转换）** | `onSuccess(Function<T, ResponseData<R>>)` | 新的 `ResponseData<R>` | **链式类型转换**：`dao.load()` → 修改 → `dao.update()`，返回 update 的 ResponseData |
| **Consumer 版（回调）** | `onSuccess(Consumer<T>)` | 自身 `ResponseData<T>` | 副作用操作：缓存更新、日志记录、数据组装 |
| **Runnable 版（无参）** | `onSuccess(Runnable)` | 自身 `ResponseData<T>` | 不需要 data 的操作 |

**所有 onXxx 方法一览**（每个都有 3 种重载）：

| 方法 | 触发条件 |
|------|---------|
| `onSuccess` | state == SUCCESS |
| `onWarn` | state == WARN |
| `onError` | state == ERROR |
| `onFatal` | state == FATAL |
| `onNotSuccess` | state != SUCCESS |
| `onNotError` | state != ERROR |

**Function 版（转换）核心机制**：
- **成功时**：执行 `function.apply(data)`，返回新的 `ResponseData<R>`（类型从 T 变为 R）
- **失败时**：跳过 function，直接返回自身（`this.raw()`，自动转型为 `ResponseData<R>`）
- **这是"扁平链式"的真正实现**：`dao.queryForObject()` → `onSuccess(product -> { return dao.update(product); })` 返回的是 update 的 ResponseData
- **消歧规则（强制）**：三个重载（Function/Consumer/Runnable）使**单参表达式 lambda 普遍歧义**（`x -> dao.update(x)`、`x -> cache.invalidate(x.getId())` 均报 `ambiguous`，与 lambda 体是否返回值无关）。**必须写成块 lambda**（Function 版加 `return`、Consumer 版不加）。**禁止 `(Function<...>)` / `(Consumer<...>)` 造型绕过**。无参 `() ->`（Runnable）不受影响。

**Consumer 版（回调）核心机制**：
- 执行 consumer 后返回自身，支持连续链式调用
- 适用于副作用操作（缓存失效、日志、数据组装）

**map 类型转换**：`<R> R map(Function<ResponseData<T>, R>)` — 将整个 ResponseData 转为任意类型 R。

**泛型陷阱**：`ResponseData.warn("CODE","msg")` 实际签名是 `warn(T t, String code)`，Java 推断 T=String，"CODE" 变成 data。✅ 正确：`warnCode("CODE","msg")`。

**使用示例**：
```java
// 1. 直接返回（最常见）
@GetMapping("/load")
public ResponseData<User> load(long id) {
    return dao.load(User.class, id);
}

// 2. Consumer 版：链式后处理（副作用）
@PostMapping("/save")
public ResponseData<User> save(@RequestBody User user) {
    user.setId(dao.getSequenceId(User.class));
    user.setCreateDate(SystemClock.nowDate());
    return dao.save(user).onSuccess(saved -> {
        FusionCache.invalidate(User.class, saved.getId());
    });
}

// 3. Function 版：链式类型转换（加载→修改→更新，返回update的ResponseData）
@PostMapping("/enable")
public ResponseData<Integer> enable(long id) {
    AuthServiceHelper.logRef(Product.class, id);
    return dao.queryForObject(Product.class, new AuthIdQueryParam(getSaasId(), id))
        .onSuccess(product -> {       // Function版：返回新的ResponseData
            product.setState(CommonState.ENABLED.getValue());
            product.setModifyDate(SystemClock.nowDate());
            return dao.update(product);  // 返回update的ResponseData<Integer>
        });
}

// 4. map：将ResponseData转为其他类型
String json = dao.load(User.class, id).map(resp -> JsonUtils.toString(resp));

// 5. 链式数据组装（父子表联查）
@GetMapping("/listEx")
public ResponseData<PageList<OrderEx>> listEx(AuthQueryParam param) {
    return dao.list(OrderEx.class, param).onSuccess(orders -> {
        if (orders.isEmpty()) return;
        long[] ids = orders.stream().mapToLong(OrderEx::getId).toArray();
        dao.list(OrderItem.class, "SELECT * FROM order_item WHERE order_id IN ("
            + StringTools.buildPlaceholders(ids.length) + ")", ids)
            .onSuccess(items -> {
                Map<Long, List<OrderItem>> map = items.stream()
                    .collect(Collectors.groupingBy(OrderItem::getOrderId));
                orders.forEach(o -> o.setItemList(map.getOrDefault(o.getId(), List.of())));
            });
    });
}

// 6. 条件更新链（加载→修改→更新）
@PostMapping("/enable")
public ResponseData<Integer> enable(long id) {
    AuthServiceHelper.logRef(Product.class, id);
    return dao.queryForObject(Product.class, new AuthIdQueryParam(getSaasId(), id))
        .onSuccess(product -> {
            product.setState(CommonState.ENABLED.getValue());
            product.setModifyDate(SystemClock.nowDate());
            return dao.update(product);
        });
}

// 7. 多步骤业务流程（加载→修改→更新）
public static ResponseData<User> resetPassword(long userId, String newPassword) {
    AuthServiceHelper.logRef(User.class, userId);
    return dao.load(User.class, userId)
        .onSuccess(user -> {
            user.setPassword(DigestUtils.signHex(newPassword, DigestUtils.Algorithm.SHA_256));
            user.setModifyDate(SystemClock.nowDate());
            return dao.update(user);
        });
}
```

### 链式调用模式对比

| 模式 | 使用方法 | 返回值 | 适用场景 |
|------|---------|--------|---------|
| 直接返回 | 无链式 | `dao.xxx()` 的 ResponseData | 简单 CRUD |
| Consumer 后处理 | `onSuccess(Consumer)` | 原始 ResponseData | 缓存失效、日志、数据组装 |
| Function 转换 | `onSuccess(Function)` | 新的 ResponseData | 加载→修改→更新，状态变更 |
| map 转换 | `map(Function)` | 任意类型 R | 提取/转换数据 |

## ResponseCode 接口

> **包路径**：`uw.common.response.ResponseCode`

业务枚举实现此接口实现类型安全的响应码管理，替代硬编码字符串。接口自带 i18n：实现类只需提供 `getMessage()`（兜底文案）与 `codePrefix()`，其余由接口 default 方法自动组装。

**接口方法**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `getMessage()` | String | **抽象**，兜底文案（未经 i18n 解析）。标注 `@JsonIgnore`，不直接序列化 |
| `codePrefix()` | String | 码前缀（可选，default null）。同时用于拼完整码与推导 i18n basename |
| `getCode()` | String | 完整响应码（default，由 `codePrefix + 枚举名点分` 自动拼接） |
| `getLocalizedMessage()` | String | **JSON 输出字段**（`@JsonProperty("message")`），按当前 Locale i18n，缺失回退 `getMessage()` |
| `getLocalizedMessage(Object... params)` | String | 带格式化参数的 i18n 消息（`@JsonIgnore`，业务调用，不参与序列化） |
| `messageSource()` | MessageSource | i18n 消息源（default，按 `codePrefix()` 自动加载并缓存） |

> ⚠️ **i18n 双方法模式（强制）**：`getMessage()` 是抽象原始值（`@JsonIgnore` 不输出），`getLocalizedMessage()` 是 default i18n 出口（`@JsonProperty("message")` 输出）。这样即便实现类误覆写 i18n 出口，原始文案也不会泄漏到 JSON。**Jackson 仅识别严格无参方法为 getter**——所以 `getLocalizedMessage()` 必须无参（变参重载 `getLocalizedMessage(Object...)` 不参与序列化）。

**i18n basename 自动推导**：`messageSource()` 按 `codePrefix()` 推导 basename = `i18n/messages/<codePrefix 点转下划线>`（如 `uw.validate` → `i18n/messages/uw_validate`），加载后缓存在 `MESSAGE_SOURCE_CACHE`（多枚举共享同 prefix 复用同一实例）。资源 key 约定为**短码**（枚举名点分小写，如 `NOT_NULL` → `not.null`，即接口 `code()` 的返回值，**非完整码** `getCode()`）。未配置 `codePrefix()` 或资源缺失时回退 `getMessage()` 兜底文案。

**枚举定义模板**（在 `{package}/constant/` 包下）：
```java
@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum GuestResponseCode implements ResponseCode {
    USER_NOT_FOUND("用户不存在"),
    PHONE_EXISTS("该手机号已注册"),
    ;

    private final String message;

    GuestResponseCode(String message) {
        this.message = message;
    }

    /** 兜底文案（原始值），供接口 getLocalizedMessage() 回退使用 */
    @Override
    public String getMessage() { return message; }

    /** 响应码前缀，同时作为 i18n basename 推导依据（uw.guest → i18n/messages/uw_guest） */
    @Override
    public String codePrefix() { return "uw.guest"; }
}
```

> 实现类**只需** `getMessage()` + `codePrefix()`，其余（`code`/`getCode()`/`messageSource()`）由接口自动派生或加载——**不要**手写 `MESSAGE_SOURCE` 静态块、`code` 字段、`getCode()`/`messageSource()` 覆写（历史样板，已内化到接口）。
>
> 完整 ResponseCode 模板（含 i18n 12 语种资源文件目录结构）见 [code-templates.md](code-templates.md)「枚举与响应码模板」。

## PageList

> **包路径**：`uw.common.data.PageList<T>`

分页列表数据容器，组合分页信息与泛型列表数据，实现 `Iterable<T>` 接口。

**构造**：`new PageList<>(List<T>, int startIndex, int resultNum, int sizeAll)` 或 `PageList.empty()`

| 字段 | 类型 | 说明 |
|------|------|------|
| startIndex | int | 起始索引 |
| resultNum | int | 每页大小 |
| size | int | 当前页实际条数 |
| sizeAll | int | 总数据量 |
| page | int | 当前页码 |
| pageCount | int | 总页数 |
| list | ArrayList\<T\> | 数据列表 |

**访问方法**：

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `get(int index)` | T | 按索引获取（越界返回 null） |
| `getFirst()` | T | 第一个元素 |
| `getLast()` | T | 最后一个元素 |
| `size()` | int | 当前页条数 |
| `sizeAll()` | int | 总数据量 |
| `isEmpty()` | boolean | 是否为空 |
| `isNotEmpty()` | boolean | 是否非空 |
| `list()` | List\<T\> | 获取数据列表 |
| `stream()` | Stream\<T\> | 获取 Stream |
| `contains(Object)` | boolean | 是否包含元素 |
| `indexOf(Object)` | int | 查找元素索引 |
| `iterator()` | Iterator\<T\> | 实现 Iterable，支持 for-each |

> 所有 `@JsonIgnore` 的 getter 方法不序列化到 JSON。JSON 输出仅含 startIndex / resultNum / size / sizeAll / page / pageCount / list。

**PageList 链式调用模式**（配合 ResponseData + DaoManager）：

```java
// 1. 空结果提前返回（避免后续 NPE）
return dao.list(User.class, param).onSuccess(users -> {
    if (users.isEmpty()) return;  // 空页自动跳过
    // 处理非空数据...
});

// 2. 遍历处理（PageList 实现 Iterable<T>）
return dao.list(User.class, param).onSuccess(users -> {
    for (User user : users) {  // 支持 for-each
        user.setDisplayName(user.getNickName());
    }
});

// 3. Stream 操作
return dao.list(User.class, param).onSuccess(users -> {
    List<Long> ids = users.stream().map(User::getId).collect(Collectors.toList());
    // 批量查询关联数据...
});

// 4. getFirst/getLast 快捷访问
return dao.queryForObject(User.class, param).onSuccess(user -> {
    // 单条查询也返回 ResponseData，链式一致
});
```

## PageRowSet

> **包路径**：`uw.common.data.PageRowSet`

分页行集数据容器。存储行列结构的二维数据，支持游标遍历和按列名/列索引访问。不依赖 `java.sql`，由 DAO 层负责 `ResultSet` 到 `PageRowSet` 的转换。

**构造**：`new PageRowSet(String[] columnNames, List<Object[]> list, int startIndex, int resultNum, int sizeAll)` 或 `PageRowSet.empty()`

**分页字段**：与 PageList 相同（startIndex / resultNum / size / sizeAll / page / pageCount）

**游标遍历**：

| 方法 | 说明 |
|------|------|
| `next()` | 移到下一行，返回是否有数据 |
| `previous()` | 移到上一行 |
| `absolute(int index)` | 定位到指定位置 |
| `remove()` | 删除当前行 |

**按列名取值**（游标模式，需先 `next()`）：

| 方法 | 返回类型 |
|------|---------|
| `get(String colName)` | Object |
| `getBoolean(String colName)` | boolean |
| `getInt(String colName)` | int |
| `getLong(String colName)` | long |
| `getDouble(String colName)` | double |
| `getFloat(String colName)` | float |
| `getString(String colName)` | String |
| `getBigInteger(String colName)` | BigInteger |
| `getBigDecimal(String colName)` | BigDecimal |
| `getBytes(String colName)` | byte[] |
| `getDate(String colName)` | java.util.Date |

> 同名方法也支持按列索引（int colIndex）访问。

**类型转换**：

```java
// PageRowSet → PageList（游标遍历 + Function 映射）
PageList<UserVO> result = pageRowSet.map(row -> {
    UserVO vo = new UserVO();
    vo.setId(row.getLong("id"));
    vo.setName(row.getString("user_name"));
    vo.setCreateDate(row.getDate("create_date"));
    return vo;
});
```

## MoneyUtils

> **包路径**：`uw.common.util.MoneyUtils`

所有金额以 **分（long）** 为单位，避免浮点误差。溢出或除零时抛出 `ArithmeticException`。提供静态方法和链式调用（Chain）两套 API。

**静态方法**：

| 方法 | 说明 |
|------|------|
| `add(long a, long b)` | 安全相加 |
| `subtract(long a, long b)` | 安全相减 |
| `sum(long... values)` | 多值求和 |
| `multiply(long amount, long factor)` | 乘以整数倍数 |
| `multiplyBps(long amount, long rateBps)` | 乘以万分比（850 = 8.5%） |
| `multiplyRatio(long amount, long num, long den)` | 乘以比率（分子/分母） |
| `multiplyRate(long amount, double rate)` | 乘以 double 倍率（汇率） |
| `multiplyRate(long amount, String rate)` | 乘以 String 倍率（汇率，避免精度丢失） |
| `divideHalfUp(long dividend, long divisor)` | 四舍五入除法 |
| `ceilDiv(long dividend, long divisor)` | 向上取整除法（天花板） |
| `divideRate(long amount, double rate)` | 除以倍率 |
| `allocate(long total, long[] weights)` | 按比例分摊（尾差兜底，合计=total） |
| `toYuan(long cents)` | 分 → 元字符串（"1.99"） |
| `fromYuan(String yuan)` | 元字符串 → 分（199） |
| `toChinese(long cents)` | 分 → 中文大写（"贰佰伍拾伍元伍角整"） |

**链式调用（Chain）**：

```java
// 链式入口：of(分) 或 ofYuan("元")
long fee = MoneyUtils.of(10000L)       // 100.00 元
    .multiply(3)                        // × 3 件
    .multiplyRate("0.85")               // 85 折
    .add(500)                           // + 5 元手续费
    .cent();                            // → 25550（255.50 元）

String display = MoneyUtils.of(19900)
    .multiplyBps(850)                   // 85 折
    .yuan();                            // → "169.15"

String cn = MoneyUtils.of(214748364700L)
    .chinese();                         // → "贰拾壹亿肆仟柒佰肆拾捌万叁仟陆佰肆拾柒元整"
```

**Chain 方法**：`add(long)` / `add(Chain)` / `subtract(long)` / `subtract(Chain)` / `multiply(long)` / `multiplyBps(long)` / `multiplyRate(double)` / `multiplyRate(String)` / `divideHalfUp(long)` / `ceilDiv(long)` / `divideRate(double)` / `divideRate(String)` / `cent()` → long / `yuan()` → String / `chinese()` → String

## ValidateUtils

> **包路径**：`uw.common.util.ValidateUtils`

所有方法返回 `boolean`，不抛异常。null 输入统一返回 false。

**字符串校验**：

| 方法 | 说明 |
|------|------|
| `isNotEmpty(String)` | 非空字符串 |
| `isNotBlank(String)` | 非空白字符串 |
| `isLengthInRange(String, int min, int max)` | 长度在闭区间内（null视为0） |
| `isDigits(String)` | 纯数字（0~9） |
| `isLetters(String)` | 纯英文字母 |
| `isAlphanumeric(String)` | 字母+数字组合 |
| `isStrongPassword(String, int min, int max)` | 密码强度（至少含字母+数字） |

**数值校验**：

| 方法 | 说明 |
|------|------|
| `isInteger(String)` | 合法整数（含正负号，long范围内） |
| `isPositiveInteger(String)` | 正整数（>0，无前导零） |
| `isNonNegativeInteger(String)` | 非负整数（>=0） |
| `isDecimal(String)` | 合法浮点数 |
| `isPositiveDecimal(String)` | 正浮点数 |
| `isDecimalWithScale(String, int maxScale)` | 浮点数且小数位不超过 maxScale |
| `isInRange(String, double min, double max)` | 数值在闭区间内 |

**日期时间校验**：

| 方法 | 说明 |
|------|------|
| `isDate(String)` | 日期 yyyy-MM-dd（含闰年校验） |
| `isDate(String, String pattern)` | 指定格式日期 |
| `isDateInRange(String, LocalDate start, LocalDate end)` | 日期范围 |
| `isTime(String)` | 时间 HH:mm:ss |
| `isTime(String, String pattern)` | 指定格式时间 |
| `isDateTime(String)` | 日期时间 yyyy-MM-dd HH:mm:ss（严格解析） |
| `isDateTime(String, String pattern)` | 指定格式日期时间 |

**网络校验**：

| 方法 | 说明 |
|------|------|
| `isEmail(String)` | 邮箱（RFC 5321，<=254字符） |
| `isUrl(String)` | URL（http/https/ftp，<=2048字符） |
| `isIpv4(String)` | IPv4 地址 |
| `isIpv6(String)` | IPv6 地址（标准8段全称） |
| `isIp(String)` | IPv4 或 IPv6（合并判断） |

**中国业务校验**：

| 方法 | 说明 |
|------|------|
| `isChinaMobile(String)` | 中国手机号（11位，1开头） |
| `isChinaIdCard(String)` | 中国身份证号（18位，含校验位） |
| `isChinaName(String)` | 中文姓名（2-20个汉字） |
| `isChinaUscc(String)` | 统一社会信用代码（18位） |
| `isChinaPlateNo(String)` | 车牌号（含新能源） |

**使用示例**：
```java
if (!ValidateUtils.isChinaMobile(phone)) {
    return ResponseData.warnCode(BizResponseCode.INVALID_PHONE);
}
if (!ValidateUtils.isChinaIdCard(idCard)) {
    return ResponseData.warnCode(BizResponseCode.INVALID_ID_CARD);
}
```

## MaskUtils

> **包路径**：`uw.common.util.MaskUtils`

数据脱敏工具。null 输入返回 null，不抛异常。**所有方法统一 `mask` 前缀**。过短无法保留明文片段时整体返回 `FULL_MASK`（`****`）。脱敏仅用于回显/日志/展示，**不可作为加密手段**。

> ⚠️ 本类**没有** `desensitize` / `hide` 等方法名，也**没有**不带 mask 前缀的 `email`/`phone`/`ipv4` 等方法（避免与取值类方法混淆）。所有公开方法都是 `maskXxx`。

**两类 API**：

| API | 中间掩码行为 | 适用场景 |
|-----|------------|---------|
| `mask(input, pre, suf, maskStr)` / `maskSecret(secret)` | **固定掩码串**，不随原长变化（不泄漏长度结构） | 凭证/密钥/Token |
| `maskByLength(input, pre, suf)` + 业务语义方法 | **按原文长度逐位填星** | 展示类固定格式字段（手机号展示 `138****5678`） |

**通用方法**：

| 方法 | 说明 |
|------|------|
| `mask(String, int pre, int suf)` | 保留前后指定位，中间填默认 `****` |
| `mask(String, int pre, int suf, String maskStr)` | 自定义中间掩码串 |
| `maskByLength(String, int pre, int suf)` | 按原长逐位填星 |
| `maskSecret(String)` | 凭证脱敏（前4后4固定掩码，长度≤8整体 `****`） |

**业务语义化方法**（内部走 `maskByLength`）：

| 方法 | 示例 |
|------|------|
| `maskChinaMobile(mobile)` | 13812345678 → 138****5678 |
| `maskTelephone(tel)` | 01012345678 → 0101*****78 |
| `maskChinaIdCard(idCard)` | 110101199001011234 → 110101********1234 |
| `maskPassport(passport)` | E12345678 → E1*****78 |
| `maskChinaName(name)` | 欧阳修 → 欧**；单字 → `****` |
| `maskBankCard(bankCard)` | 6222021234567890 → 6222********7890 |
| `maskEmail(email)` | alice@example.com → a****@example.com（域名完整保留） |
| `maskChinaUscc(uscc)` | 911101081234561234 → 9111**********1234 |
| `maskChinaTaxNo(taxNo)` | 同 USCC，兼容 15/18/20 位税号 |
| `maskChinaPlateNo(plateNo)` | 京A12345 → 京A*****（前2位保留） |
| `maskAddress(addr)` | 保留前6位（省市区），其余填星 |
| `maskImei(imei)` | 490154203237518 → 490154*******18（前6 TAC 后2） |
| `maskWechatId(wxId)` | alice_wx → a******x（首+尾，长度≤2 整体 `****`） |
| `maskIpv4(ip)` | 192.168.1.100 → 192.168.*.*（非标准 IPv4 整体 `****`） |

```java
// 日志脱敏（凭证用固定掩码，不泄漏长度）
log.info("login failed, token={}", MaskUtils.maskSecret(token));

// 接口返回脱敏（手机号展示用按长度填星）
user.setMobile(MaskUtils.maskChinaMobile(user.getMobile()));
```

## JsonUtils

> **包路径**：`uw.common.util.JsonUtils`

| 方法 | 说明 |
|------|------|
| `toString(Object)` | 对象 → JSON字符串 |
| `toPrettyString(Object)` | 美化输出 |
| `parse(String, Class<T>)` | JSON → 对象 |
| `parse(String, TypeReference<T>)` | JSON → 泛型对象（如 List<User>） |
| `parseTree(String)` | JSON → JsonNode 树模型 |
| `convert(Object, Class<T>)` | 对象类型转换（如 Map → POJO） |

> ⚠️ **parse/convert 失败会抛 `RuntimeException`，且 message 拼接了原始数据**
>
> 所有 `parse`/`convert` 方法在解析失败时抛 `RuntimeException`，其 message 形如 `"<Jackson错误消息>! data: <原始JSON>"` —— **同时包含 Jackson 内部解析细节和原始输入数据**。
>
> 这带来两类风险：
> 1. **原始数据泄露**：原始 JSON（可能含用户敏感输入：密码、token、个人信息）被拼进异常 message，沿调用栈传播，任何一层的日志/响应若不脱敏都会泄露。
> 2. **框架细节泄露**：Jackson 错误消息暴露服务端使用的 JSON 序列化实现。
>
> **强制规范**（与 KryoUtils 同等约束，见下方「KryoUtils」章节）：
> - 面向外部的边界（Controller / RPC / 对外响应）调用 `parse`/`convert` 时**必须 try-catch**，对外用**固定文案**（如 `"invalid request data."`），**禁止把异常 message 回传客户端**（会泄露原始数据 + Jackson 细节）。
> - 需要排查时，在 catch 块内自行决定如何记录原始数据（建议脱敏后记录），不要依赖异常 message。
> - 反面教材（禁止）：`return ResponseData.errorCode("parse failed: " + e.getMessage())` —— 直接把原始 JSON 数据回传给客户端。

## KryoUtils

> **包路径**：`uw.common.util.KryoUtils`

高性能二进制序列化，四种路径：整对象反射（`serialize(Object)`）、lambda 手工（`serialize(Consumer)`）、BiFunction 完整版（`serialize(BiConsumer)`）、KryoData 接口式（`serializeData/deserializeData`）。

> ⚠️ Kryo 反序列化要求**具体实现类**，不能用接口类型（List/Map/Set 要用 ArrayList/LinkedHashMap/HashSet）；`CacheDataLoader<V>` 的 V 必须是具体类。

> ⚠️ **异常策略与对外报错安全（强制）**
>
> `KryoUtils` 的所有方法**不吞异常**：反序列化失败（字段错位、类型不符、数据损坏，典型如版本升级后旧协议 token 残留 Redis/MQ）时，直接抛 `KryoException` / `KryoBufferUnderflowException`，由**调用方** catch。
>
> 此类异常的 message 携带序列化框架内部关键字（`kryo`、`Buffer underflow`、字段读取位置等），**禁止出现在对外响应**——否则暴露服务端序列化实现与协议结构，成为攻击者构造畸形 payload 探测/绕过鉴权的信息泄露面。
>
> **强制规范**：
> 1. 面向外部的边界（Controller / RPC / 对外响应）必须 catch `KryoException`，不得让其冒泡到 `GlobalExceptionAdvice` 后原样回传客户端。
> 2. 对外响应消息用**固定文案**（如 `"token invalid or corrupted."`），**禁止拼接 `e.getMessage()` / `e.toString()`**；详细原因（含 kryo 关键字）只写服务端日志。
> 3. 缓存 / MQ 批量读取（`GlobalHashSet` / `GlobalSortedSet` / `GlobalCache`）**逐条 catch 跳过**脏数据降级 WARN，单条损坏不得拖垮整批；单元素读取保持抛异常由调用方决策。
>
> 参考实现：`MscTokenService.verifyAuthToken` / `parseRefreshToken` / `loadAuthTokenData` / `getInvalidTokenList`、`GlobalCache.get`、`GlobalHashSet` / `GlobalSortedSet` 批量方法。完整说明见 `KryoUtils` 类注释与 [dev-standards.md](dev-standards.md)「缓存使用规范」。

## DateTools

> **包路径**：`uw.common.util.DateTools`

| 方法 | 说明 | 注意 |
|------|------|------|
| `dateToString(Date, String format)` | 按指定格式格式化 | 方法名是 `dateToString` 不是 `format` |
| `stringToDate(String)` | 按默认格式 yyyy-MM-dd HH:mm:ss 解析 | 方法名是 `stringToDate` 不是 `parse` |
| `stringToDate(String, String format)` | 按指定格式解析 | — |
| `offsetDay(Date, int)` | 偏移天数 | 不是 addDays |
| `offsetMonth/offsetYear/offsetHour/offsetMinute/offsetSecond` | 各粒度偏移 | — |
| `beginOfToday(Date)` | 当天开始 00:00:00 | 不是 getDayStart |
| `endOfToday(Date)` | 当天结束 23:59:59 | 不是 getDayEnd |
| `beginOfMonth/beginOfYear/beginOfYesterday/beginOfTomorrow` | 区间起点 | 同系列有 endXxx |
| `daysDiff/hoursDiff/minutesDiff/monthsDiff/yearsDiff` | 时间差 | — |
| `dateToLocalDate/localDateToDate` | Date ↔ LocalDate 互转 | — |
| `dayOfMonth/dayOfWeek` | 取日/星期 | — |

> ⚠️ **DateTools 没有 `now()` / `nowDate()`**。获取当前时间用 [`SystemClock`](#systemclock) 的 `now()` / `nowDate()`。

**日期格式常量**（直接用常量名，不要手写格式串）：

| 常量 | 值 |
|------|------|
| `DATE_TIME` | yyyy-MM-dd HH:mm:ss |
| `DATE_MILLIS` | yyyy-MM-dd HH:mm:ss.SSS |
| `DATE` | yyyy-MM-dd |
| `TIME` | HH:mm:ss |
| `DATE_SIMPLE` | yyyyMMdd |
| `DATE_TIME_SIMPLE` | yyyyMMddHHmmss |
| `MONTH` | yyyy-MM |
| `DATE_TIME_ISO` | yyyy-MM-dd'T'HH:mm:ssZZ |

> 常量名是 `DATE_TIME` / `DATE` / `TIME` 等，不是 `FORMAT_DEFAULT` / `FORMAT_DATE`。

## SystemClock

> **包路径**：`uw.common.util.SystemClock`

高性能动态系统时钟。调用频率低时直接读系统时钟，频率高时（>10/ms）自动切换为定时器刷新的缓存值。**`createDate` / `modifyDate` 赋值一律用 `SystemClock.nowDate()`**，禁止 `new Date()` / `System.currentTimeMillis()`。

| 方法 | 返回类型 | 说明 |
|------|---------|------|
| `now()` | long | 当前时间戳（毫秒），高频场景比 `System.currentTimeMillis` 快约 40 倍 |
| `nowDate()` | Date | 当前 Date（createDate/modifyDate 赋值用） |
| `elapsedMillis(long startTime)` | long | 从 startTime 到当前的耗时 |
| `elapsedMillis(long startTime, long endTime)` | long | 两时间戳之差 |

## AESUtils

> **包路径**：`uw.common.util.AESUtils`

| 方法 | 说明 |
|------|------|
| `generateKey(int keySize)` | 生成密钥（128/192/256位） |
| `generateIv()` | 生成IV（16字节） |
| `encryptString(byte[] key, String data)` | 自动IV加密（推荐，密文自带IV） |
| `decryptString(byte[] key, String encrypted)` | 自动IV解密 |
| `encryptString(byte[] key, byte[] iv, String data)` | 指定IV加密 |
| `decryptString(byte[] key, byte[] iv, String encrypted)` | 指定IV解密 |

## BitConfigUtils

> **包路径**：`uw.common.util.BitConfigUtils`

| 方法 | 说明 |
|------|------|
| `isOn(int/long config, int bitIndex)` | 检查开关是否开启 |
| `on(int/long config, int... bitIndex)` | 开启开关 |
| `off(int/long config, int bitIndex)` | 关闭开关 |
| `countOn(int/long config)` | 已开启数量 |

## BizAESBox

> **包路径**：`uw.common.util.BizAESBox`

业务 AES 加解密盒子。通过配置文件管理密钥和向量，确保多次加密结果一致。

| 方法 | 说明 |
|------|------|
| `getInstance(String configPath)` | 获取实例（配置文件缓存，线程安全） |
| `encrypt(String data)` | 加密 |
| `decrypt(String encrypted)` | 解密 |
| `genAesConfig()` | 生成 AES 密钥+向量配置字符串（静态） |

**配置文件格式**（如 `bizaes.properties`）：
```properties
aes.key=Base64编码的密钥
aes.iv=Base64编码的向量（可选）
```

```java
BizAESBox aesBox = BizAESBox.getInstance("bizaes.properties");
String encrypted = aesBox.encrypt("敏感数据");
String decrypted = aesBox.decrypt(encrypted);
```

## ChineseUtils

> **包路径**：`uw.common.util.ChineseUtils`

汉字拼音与相似度工具类。

| 方法 | 说明 |
|------|------|
| `convertToPinyin(String, String separator)` | 转拼音 |
| `getShortPinyin(String)` | 拼音首字母缩写 |
| `hasMultiPinyin(char)` | 多音字检测 |
| `similarDegree(String, String)` | N-gram 相似度（0~10000） |
| `lcsSimilarDegree(String, String)` | LCS 最长公共子序列相似度（0~10000） |
| `ngramSimilarDegree(String, String)` | N-gram 余弦相似度（0~10000） |
| `toSBC(String)` | 半角 → 全角 |
| `toDBC(String)` | 全角 → 半角 |

> 相似度返回值范围 0~10000（10000 = 完全相同）。`ngramSimilarDegree` 对少量字符差异更敏感，适合区分高度相似但不同的名称。
>
> ⚠️ 相似度算法实现与 `toSBC`/`toDBC` 全角半角转换均已委托 [StringTools](#stringtools)（`StringTools.lcsSimilarity/ngramSimilarity/toFullWidth/toHalfWidth`），ChineseUtils 保留万分制 int 返回值以兼容历史调用方。新代码可直接用 StringTools 的 [0,1] double 版本。

## StringTools

> **包路径**：`uw.common.util.StringTools`

通用字符串工具，覆盖「数组/集合 ⇄ 字符串互转」「命名风格转换」「相似度」「转义」「随机串」五类高频场景。所有方法空安全（null 不抛 NPE）。校验用 `ValidateUtils`、脱敏用 `MaskUtils`，本类不重复实现。

**Join（数组/集合 → 字符串）**：

| 方法 | 说明 |
|------|------|
| `join(long[]/int[]/String[]/Collection, [sep])` | 拼接为字符串，null/空返回 `""`，元素 null 跳过 |
| `joinMap(Map, entrySep, kvSep)` | Map → `k1=v1&k2=v2`。独立方法名（非 join 重载），避免三参 String 与 Collection 版重载歧义 |
| `surround(joined, sep)` | 首尾各补一个分隔符：`surround("1,2,3", ",")` → `,1,2,3,`。需首尾补分隔符时自行组合 `surround(join(arr,sep), sep)` |
| `buildPlaceholders(count[, ph, sep])` | SQL 占位符 `?,?,?` |
| `buildInClause(long[]/Collection)` | `(1,2,3)`，空集合兜底 `(0)` 避免 SQL 语法错。仅服务端白名单 ID，不接受外部未清洗输入 |

**Split（字符串 → 数组/集合）**：

| 方法 | 说明 |
|------|------|
| `splitToLongArray/splitToIntArray(s, [sep])` | → `long[]/int[]`，自动 trim、跳过空白段、脏数据逐条跳过+WARN |
| `splitToStringArray/splitToLongList/splitToIntList/splitToStringList(s, [sep])` | 各类型切分 |
| `split(s, char, omitEmpty)` / `split(s, char, limit)` | 单字符切分（避免正则意外）/ 限制段数 |
| `splitLines(s)` | 统一处理 `\n`/`\r\n`/`\r` |
| `splitMap(s, entrySep, kvSep)` | `joinMap` 逆操作，kvSep 以首次出现为准 |

> 切分底层用 indexOf+substring 实现，不走正则引擎，默认分隔符兼容逗号+各类空白（合并连续分隔符）。脏数据逐条跳过+WARN，符合批量读取降级约定。

**命名风格转换**：

| 方法 | 说明 |
|------|------|
| `toCamelCase(str, sep)` | 按指定分隔符转驼峰（做大小写规范化） |
| `toSnakeCase` / `toKebabCase` / `toPascalCase` / `toMacroCase` / `toTrainCase` / `toDotCase` / `toPathCase` / `toFlatCase` | 自动探测源格式输出对应风格 |
| `convertCase(str, CaseStyle)` | 命名风格统一互转入口，`CaseStyle` 是 StringTools 内嵌枚举 |
| `capFirst(s)` / `uncapFirst(s)` | 首字母大/小写 |

> 自动探测规则：所有分隔符 `-_. /` 视为边界 + 大小写边界，能正确处理连续大写缩写（`HTTPResponse` → `http_response`、`userID` → `user_id`）。

**默认值 / 截断 / 填充**：`defaultIfBlank(str, def)` / `defaultIfBlank(supplier)` / `defaultIfEmpty(supplier)`（supplier 惰性求值）；`truncate(str, max, suffix)` / `truncateByWidth(str, maxWidth)`（中文算 2 宽度）；`padStart/padEnd(str, min, ch)`。

**格式化 / 清理**：`format(template, Map)`（命名参数 `${name}`，未提供 key 原样保留）；`format(template, args...)`（null 安全版 `String.format`）；`clean(str)`（清理零宽/BOM/全角空格/控制字符）；`normalizeSpace(str)`；`toHalfWidth/toFullWidth`；`extractChinese/hasCJK`。

**包含 / 计数 / 判等**：`containsAny/containsAll/containsIgnoreCase`；`countMatches(str, sub)`（不重叠）；`startsWithAny/endsWithAny`；`equals/equalsIgnoreCase/trimToEmpty/trimToNull`（均 null 安全）。

**转义 / 编码**：`escapeHtml` / `escapeJson` / `escapeXml` / `escapeRegex`；`unicodeEncode/unicodeDecode`（非 ASCII ⇄ `\uXXXX`）。

**路径 / 文件名**：

| 方法 | 说明 |
|------|------|
| `getFileName(path)` | 跨平台提取文件名（同时处理 `/` `\`） |
| `getFileExtension(name)` / `removeFileExtension(name)` | 扩展名（小写）/ 去扩展名 |
| `normalizePath(path)` | 合并连续分隔符、解析 `.`/`..`；绝对路径根之上的 `..` 剔除（`/../a` → `/a`） |
| `safeFileName(name[, ch])` | 去除非法字符 `< > : " / \ \| ? *`，默认替换为 `_` |

**随机串**（均基于 SecureRandom）：`randomNumeric(len)` / `randomAlphanumeric(len)` / `randomChinese(len)`（CJK 常用汉字区，测试数据用）；`randomUuidSimple()`（无横线 32 位）；`randomToken(len)`（URL-safe Base64 无填充）。

**文本 / 相似度**：

| 方法 | 算法 | 适用场景 |
|------|------|---------|
| `levenshteinDistance(s1,s2)` | 编辑距离（int） | "打错几次字" |
| `levenshteinSimilarity(s1,s2)` | 编辑距离换算 [0,1] | 拼写纠错 / OCR |
| `lcsSimilarity(s1,s2)` | 最长公共子序列 [0,1] | 简称匹配全称 |
| `ngramSimilarity(s1,s2)` | bigram 余弦 [0,1] | 区分高度相似但不同（万达嘉华 vs 万达瑞华） |
| `similarity(s1,s2)` | `levenshteinSimilarity` 别名 | 默认场景 |
| `diff(oldStr, newStr)` | 行级 LCS 差异标记 | `- `/`+ `/`  ` 前缀 |

> 三种相似度算法维度不同不可互替：编辑距离看「操作量」，LCS/N-gram 看「重合度」。ChineseUtils 的万分制版本（`lcsSimilarDegree/ngramSimilarDegree`）委托本类算法。

## DigestUtils

> **包路径**：`uw.common.util.DigestUtils`

消息摘要工具类。**没有独立的 `md5()`/`sha256()` 方法**，统一通过 `sign(msg, Algorithm)` 调用，算法由 `Algorithm` 枚举指定。

| 方法 | 说明 |
|------|------|
| `sign(String msg, Algorithm)` | 摘要签名，返回 Base64 字符串 |
| `signHex(String msg, Algorithm)` | 摘要签名，返回十六进制字符串 |
| `bytesToHex(byte[])` | 字节数组转十六进制 |

**Algorithm 枚举值**：

| 枚举 | 算法 | 说明 |
|------|------|------|
| `MD5` | MD5 | 不安全，仅兼容旧系统 |
| `SHA` | SHA-1 | 较弱，仅历史系统 |
| `SHA_256` | SHA-256 | **推荐使用** |
| `SHA_384` | SHA-384 | 高强度 |
| `SHA_512` | SHA-512 | 最高强度 |
| `SHA3_256` | SHA3-256 | 最新标准，推荐 |
| `SHA3_512` | SHA3-512 | 最新标准，最高强度 |

```java
String hash = DigestUtils.signHex(password, DigestUtils.Algorithm.SHA_256);
```

## HmacUtils

> **包路径**：`uw.common.util.HmacUtils`

HMAC-SHA256 签名工具类。

| 方法 | 说明 |
|------|------|
| `sign(String message, String secret)` | HMAC-SHA256 签名，返回十六进制字符串 |
| `verify(String message, String secret, String signature)` | 验证签名 |

## CurrencyUtils

> **包路径**：`uw.common.util.CurrencyUtils`

货币币种工具类。

| 字段/方法 | 说明 |
|----------|------|
| `CURRENCY_DEFAULT` | 默认币种 CNY |
| `getAvailableCurrencies()` | 获取可用货币集合 |
| `getCurrency(String code)` | 获取指定币种（异常返回 null） |
| `getCurrency(String code, Currency default)` | 获取指定币种（异常返回默认值） |

## LimitedVirtualThreadExecutor

> **包路径**：`uw.common.util.LimitedVirtualThreadExecutor`

背压限制虚拟线程执行器，基于虚拟线程（`Thread.ofVirtual`）+ `Semaphore` 限制最大并发数。两种模式：

| 模式 | queueCapacity | 行为 |
|------|---------------|------|
| 快速模式 | = 0 | 无队列缓冲，并发满直接走 `CallPolicy` |
| 队列模式 | > 0 | 有界无锁队列（JCTools `MpmcArrayQueue`）缓冲；并发满先入队，队列满才走 `CallPolicy` |

队列模式下 worker 跑**接力循环**：完成当前 task 后非阻塞 `poll` 队列取下一个（有则继续、不释放许可，消除"等外部 submit"的空窗；无则释放许可退出）。worker 退出与 submit 入队时均有扫尾补偿（`drainQueueIfPresent`），保证"入队成功 ⇒ 必有 worker 最终消费"，避免关闭时孤儿任务静默丢失。

虚拟线程以 `threadNamePrefix + 序号` 命名（如 "my-task-0"），便于在 JMC/JFR、线程堆栈与日志中区分任务来源；前缀建议以分隔符（如 "-"）结尾。

**构造**（4 个重载）：

| 构造 | 模式 | 默认策略 |
|------|------|---------|
| `(String threadNamePrefix, int maxConcurrency)` | 快速 | `BlockPolicy` |
| `(String threadNamePrefix, int maxConcurrency, CallPolicy)` | 快速 | 指定 |
| `(String threadNamePrefix, int maxConcurrency, int queueCapacity)` | queueCapacity>0 队列 / =0 快速 | 队列 `FailFastPolicy` / 快速 `BlockPolicy` |
| `(String threadNamePrefix, int maxConcurrency, int queueCapacity, CallPolicy)` | 全参 | 指定（null→`BlockPolicy`） |

> 参数容错：`threadNamePrefix` 空→默认值；`maxConcurrency<=0`→1；`queueCapacity<0`→1。

**方法**：

| 方法 | 说明 |
|------|------|
| `submit(Runnable)` | 提交任务（执行器已关闭抛 `IllegalStateException`） |
| `shutdown()` | 优雅关闭：不再接受新任务，已提交任务（含队列缓冲）执行完毕 |
| `shutdownNow()` | 立即关闭：中断运行中任务，**返回**队列中未执行任务 `List<Runnable>` |
| `isShutdown()` | 是否已调用 shutdown/shutdownNow |
| `awaitTermination(long, TimeUnit)` | 等待所有任务完成或超时 |
| `getActiveCount()` | 正在执行的任务数（近似值，任务执行口径） |
| `getIdleCount()` | 空闲并发槽位数（`semaphore.availablePermits()`，当前可立即 `startWorker` 不进队列/不被拒的新任务数） |
| `getMaximumPoolSize()` | 最大并发数（= Semaphore 许可数） |
| `getQueueCapacity()` | 队列容量（快速模式返回 0） |
| `getQueueSize()` | 队列待执行任务数（快速模式返回 0） |
| `getRejectedCount()` | 累计被拒绝任务数（走 `CallPolicy` 拒绝的） |

> ⚠️ 本类**没有** `getQueuedTasks()` 方法（查队列大小用 `getQueueSize()`）。查可用许可数用 `getIdleCount()`，查并发上限用 `getMaximumPoolSize()`。

> **active/idle 口径差异（勿用减法估算空闲）**：`getActiveCount()` 只统计已进入 `task.run()` 的 worker（任务执行口径），不含「已 `tryAcquire` 持有许可、但虚拟线程尚未 mount 进 `run()`」的 worker（脉冲 submit 时虚拟线程排队等载体），也不含接力循环中 poll 间隙的 worker。因此**禁止用 `getMaximumPoolSize() - getActiveCount()` 估算空闲**——会高估空闲、导致批量偏大。`getIdleCount()` 直接读 `semaphore.availablePermits()`，精确反映可立即占用的并发槽位。判断"是否还能接受新任务/是否达上限"用 `getIdleCount()`；观察"实际执行负载"用 `getActiveCount()`。

**调用策略**（`CallPolicy` 接口，对齐 JDK `RejectedExecutionHandler`）：

| 策略类 | 行为 | 默认用于 |
|--------|------|---------|
| `BlockPolicy` | 阻塞等待直到获取许可 | 队列模式 |
| `CallerRunsPolicy` | 由调用者线程直接执行（背压） | 快速模式 |
| `FailFastPolicy` | 抛出 `RejectedExecutionException` | — |
| `DiscardPolicy` | 静默丢弃任务 | — |

```java
// 队列模式：并发 100、队列 1000，队列满则阻塞提交线程（BlockPolicy）
LimitedVirtualThreadExecutor executor = new LimitedVirtualThreadExecutor("my-task-", 100, 1000);
executor.submit(() -> doSomething());
executor.shutdown();
executor.awaitTermination(5, TimeUnit.SECONDS);

// 快速模式 + 丢弃策略（异步更新访问时间，丢了无所谓）
LimitedVirtualThreadExecutor updateExecutor =
    new LimitedVirtualThreadExecutor("update-", 20, new LimitedVirtualThreadExecutor.DiscardPolicy());
updateExecutor.submit(() -> updateLastVisitTime(id));
```

> `shutdownNow()` 返回的 `List<Runnable>` 含队列中未开始执行的任务，调用方可遍历回收（如写回 Redis zset），避免丢失——对齐 `TaskDelayerContainer` 关闭时 drain + 写回 zset 的语义。

## 其他工具类

| 类名 | 核心方法 | 用途 |
|------|---------|------|
| `RSAUtils` | `genKeyPair(keySize)` / `encrypt()` / `decrypt()` / `sign()` / `checkSign()` | RSA 非对称加密签名 |
| `IpMatchUtils` | `sortList(ipList)` / `matches(sortedList, ip)` / `parseInetAddress(ip)` | IP 段匹配（**先 sortList 排序再 matches 二分查找**），`IpRange` 支持 CIDR/范围 |
| `ByteArrayUtils` | `intToByteArray` / `byteArrayToInt` / `longToByteArray` / `hexToByteArray` / `byteArrayToString`（均含 LittleEndian 版） | 数值 ↔ 字节数组大小端转换，底层协议开发 |
| `NumCodeUtils` | `confuseNum(num)` / `clarifyNum(enc)` | 数字编码混淆 |
| `EnumUtils` | `getEnumMap(basePackage)` / `enumNameToDotCase` / `enumNameToHyphenCase` / `enumNameToCamelCase` | 枚举名称转换（点号/连字符/驼峰），用于 ResponseCode code 生成 |
| `SnowflakeIdGenerator` | `getInstance().generateId()` | 分布式雪花ID（machineId: 环境变量 → HOSTNAME hash → UUID 降级） |
| `ResponseCodeUtils` | `toProperties(Class)` / `toPropertyString(Class)` | 将 ResponseCode 枚举导出为 i18n Properties |
| `ExceptionUtils` | `exceptionToString(Throwable)` | 过滤框架堆栈的异常格式化（GlobalExceptionAdvice 使用） |

> DigestUtils 见 [上文独立章节](#digestutils)，SystemClock 见 [上文独立章节](#systemclock)。

---

## constant 包 — 常量/枚举/响应码

> 从 uw-common-app 迁入，包路径 `uw.common.constant`。

### CommonConstants

| 常量 | 值 | 说明 |
|------|-----|------|
| `EMPTY` / `SPACE` / `NULL` | `""` / `" "` / `"null"` | 空白常量 |
| `EMPTY_JSON` | `"{}"` | 空 JSON |
| `COLON` / `COMMA` / `SEMICOLON` | `:` / `,` / `;` | 分隔符 |
| `ACCEPT_LANG` | `"Accept-Language"` | 请求头语言 |
| `UTF_8` | `"UTF-8"` | 字符编码 |

### CommonState — 通用状态枚举

| 值 | 含义 |
|------|------|
| DELETED(-1) | 标记删除 |
| DISABLED(0) | 禁用 |
| ENABLED(1) | 启用 |

方法：`valueOf(int)` / `getValue()` / `getLabel()`

### CommonResponseCode — 通用响应码

实现 ResponseCode，codePrefix = `uw.common`，i18n：`i18n/messages/uw_common`。

| 枚举值 | 默认消息 |
|--------|---------|
| ENTITY_LIST/LOAD/SAVE/UPDATE/DELETE_ERROR | 数据列表/加载/保存/更新/删除失败 |
| ENTITY_EXISTS_ERROR | 数据已存在 |
| ENTITY_NOT_FOUND_ERROR | 数据未找到 |
| ENTITY_STATE_ERROR | 数据状态错误 |

### ValidateResponseCode — 校验响应码

实现 ResponseCode，codePrefix = `uw.validate`，i18n：`i18n/messages/uw_validate`。

| 枚举值 | 默认消息 |
|--------|---------|
| NOT_NULL / NOT_EMPTY | 不能为NULL / 不能为空 |
| VALUE_TOO_SMALL / VALUE_TOO_LARGE | 不能小于最小值 / 不能大于最大值 |
| LENGTH_TOO_SHORT / LENGTH_TOO_LONG | 不能小于最小长度 / 不能大于最大长度 |
| DATA_FORMAT_ERROR / REGEX_FORMAT_ERROR | 数据格式错误 / 正则校验格式错误 |

---

## helper 包 — 工具类

> 从 uw-common-app 迁入，包路径 `uw.common.helper`。

### JsonConfigHelper — JSON配置参数

全部静态方法。通过 `JsonConfigParam`（枚举）定义配置参数，构建 `JsonConfigBox` 获取强类型参数。

| 方法 | 说明 |
|------|------|
| `buildParamBox(List<JsonConfigParam>, String json)` | 从 JSON 构建 |
| `buildParamBox(List<JsonConfigParam>, Map data)` | 从 Map 构建 |
| `validateConfigData(List<JsonConfigParam>, Map/String)` | 校验配置数据，返回 `ResponseData<List<ValidateResult>>` |

> ⚠️ 参数定义（`List<JsonConfigParam>`）必须是编译期枚举（如 `MyConfig.values()`），`JsonConfigParam` 是接口，Jackson 无法反序列化。

### SchemaValidateHelper — Schema注解校验

基于 `@Schema` 注解自动校验 VO 对象（Caffeine 缓存反射元数据）。

```java
List<ValidateResult> errors = SchemaValidateHelper.validate(form);
```

| @Schema 属性 | 校验行为 |
|------|---------|
| `requiredMode = REQUIRED` | 非空校验 |
| `minimum` / `maximum` | 数值范围 |
| `minLength` / `maxLength` | 字符串长度 |
| `pattern` | 正则校验 |

### QueryParamHelper — URL查询参数构建

将 QueryParam 对象属性展开为 URI 查询参数（Caffeine 缓存反射元数据）。

```java
String url = QueryParamHelper.buildUriWithParams("/api/list", queryParam);
// → /api/list?name=foo&$pg=1&$rn=20
```

自动映射 PageQueryParam 魔法参数（PAGE→$pg 等），过滤 Auth 系参数，支持数组/Iterable 展开。

---

## vo 包 — 值对象

> 从 uw-common-app 迁入，包路径 `uw.common.vo`。

### JsonConfigBox — JSON配置参数盒子

从 `Map<String, String>` 获取强类型参数值。`EMPTY_PARAM_BOX` 常量用于空配置。

| 方法类别 | 说明 |
|---------|------|
| `getParam(name[, default])` | String，默认空串 |
| `getIntParam` / `getLongParam` / `getFloatParam` / `getDoubleParam` | 数值类型（均含 default 版 + 数组版） |
| `getBooleanParam` | boolean（含 default + 数组） |
| `getMapParam` | `Map<String, String>` |

> 所有 getXxx 方法也支持传入 `JsonConfigParam` 枚举作为参数。

### JsonConfigParam — 配置参数定义接口

使用枚举实现。实现类只需提供 `getParamData()`（返回 `ParamData`），`getKey()` 由枚举名自动派生，`getType/getValue/getTitle/getRegex` 由接口 default 委托。

| 方法 | 返回 | 说明 |
|------|------|------|
| `getParamData()` | ParamData | **唯一抽象方法**（type/value/title/regex） |
| `getKey()` | String | **default**，枚举名→点分隔（`MAX_TOKENS`→`max.tokens`） |
| `getTitle()` | String | **default** 原始标题（`@JsonIgnore`） |
| `getLocalizedTitle()` | String | **default** i18n 出口（`@JsonProperty("title")`），缺失回退 getTitle() |
| `configPrefix()` | String | **default null**，i18n 资源前缀（可选） |

**ParamType 枚举**：STRING/TEXT/TEXT_RICH/INT/LONG/BOOLEAN/FLOAT/DOUBLE/DATE/TIME/DATETIME/ENUM/MAP（及 SET_ 集合变体）。

```java
@JsonFormat(shape = JsonFormat.Shape.OBJECT)
public enum SystemConfig implements JsonConfigParam {
    SITE_NAME(ParamType.STRING, "MySite", "站点名称", null),
    MAX_UPLOAD_SIZE(ParamType.INT, "10485760", "最大上传大小", null),
    ;
    private final ParamData paramData;
    SystemConfig(ParamType type, String value, String title, String regex) {
        this.paramData = new ParamData(type, value, title, regex);
    }
    @Override public ParamData getParamData() { return paramData; }
}
```

> ⚠️ 枚举必须标注 `@JsonFormat(shape = JsonFormat.Shape.OBJECT)`，否则 Jackson 序列化为枚举名而非对象。

### ValidateResult — 校验结果

| 字段 | 类型 | 说明 |
|------|------|------|
| name | String | 属性名 |
| title | String | 属性描述 |
| errorCode | String | 完整错误码 |
| errorMsg | String | 国际化错误信息 |
| refData | String | 参考数据 |
