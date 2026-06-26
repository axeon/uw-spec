# UniWeb 开发能力考核 — B 卷（准入考试）

> 考核目标：验证开发者对 UniWeb 框架**全部模块的实际运用能力**，重点考察"能否独立使用核心 API 完成开发任务"。
> 可参考 UniWeb 官方文档。建议用时 120 分钟。

---

# 一、选择题（每题 3 分，共 15 分）

**1. Helper 层按 ID 加载当前租户的 SaaS 实体（表含 `saas_id`），并要求状态为 ENABLED。应使用哪个 QueryParam？**

A. `new AuthIdQueryParam(saasId, id)`

B. `new AuthIdStateQueryParam(saasId, id, CommonState.ENABLED.getValue())`

C. `new IdStateQueryParam(id, CommonState.ENABLED.getValue())`

D. `new IdQueryParam(id)`

---

**2. FusionCache 初始化时 `CacheDataLoader` 的正确写法是？**

A. `(key) -> dao.load(Product.class, key).getData()`

B. `new CacheDataLoader<Long, Product>() { @Override public Product load(Long key) {...} }`

C. `new CacheDataLoader<>(Long.class, Product.class)`

D. `CacheDataLoader.of(key -> dao.load(...))`

---

**3. 某接口查询用户是否存在、不存在则新建。查询后判断是否走新建分支应使用？**

A. `isNotSuccess()` — 查不到就是非成功，直接走新建

B. `isError()` — 仅系统异常才中断；查不到返回 warn，data 为 null 可继续走新建

C. `isWarn()` — warn 就是查不到

D. `getData() == null`

---

**4. uw-httpclient 中发起 POST 请求发送 JSON body 并获取反序列化对象，正确方法名是？**

A. `postForEntity(url, clazz, body)`

B. `postBodyForEntity(url, clazz, body)`

C. `postJsonForEntity(url, clazz, body)`

D. `postForData(url, body)`

---

**5. TaskRunner 中第三方接口调用失败（如限流）应抛出哪个异常以触发重试？**

A. `TaskDataException`

B. `TaskPartnerException`

C. `RuntimeException`

D. `IOException`

---

# 二、代码实现题（共 80 分）

> 每题聚焦一个或两个模块，场景均为独立业务需求。

## 1. 新闻资讯管理 Controller + 发布流转（15 分）— 开发规范 + uw-dao + uw-auth-service + uw-common-app

**场景**：为平台管理员（ADMIN）角色开发新闻资讯管理接口。新闻状态：0=草稿、1=已发布、-1=已下架。

**要求**：
1. `$PackageInfo$` 类：声明 `/admin/news` 一级菜单权限（2 级路径）
2. `NewsController`（`@RequestMapping("/admin/news/info")`），实现：
   - `list(AuthQueryParam param)` — 分页查询当前租户新闻列表
   - `load(long id)` — 使用 `AuthIdQueryParam(getSaasId(), id)` 租户隔离加载
   - `publish(long id)` — 发布：先 `logRef`，用 `AuthIdStateQueryParam` 查状态为"草稿"(0) 的新闻，改为"已发布"(1)，链式 Function 版 `onSuccess` 内 update
   - `unpublish(long id)` — 下架：先 `logRef`，用 `AuthIdStateQueryParam` 查状态为"已发布"(1) 的新闻，改为"已下架"(-1)
3. `@MscPermDeclare(user = UserType.ADMIN, auth = AuthType.PERM)`，`@Tag` 只写本级功能名
4. 定义新闻状态枚举 `NewsState`（在 `constant/` 包下），替代硬编码数字
5. `publish` 方法需要 `SysDataHistoryHelper.saveHistory()` 记录发布操作

作答：

```java
// NewsState.java（枚举）


// $PackageInfo$.java


// NewsController.java


```

---

## 2. 第三方物流 API 对接 + 面单下载（10 分）— uw-httpclient

**场景**：需要对接顺丰物流 API，包含：查询快递轨迹、创建物流订单、下载电子面单（PDF 二进制）。

**要求实现 `LogisticsHelper`**：
1. 创建 `JsonInterfaceHelper`（`HttpConfig.builder()`：connectTimeout 5s, readTimeout 30s, `retryOnConnectionFailure(false)` — 物流下单幂等敏感）
2. `queryTrack(String trackingNo)` — GET 请求 `https://api.sf.com/track?no={trackingNo}`，返回 `TrackResult` 对象（用 `getForEntity`，取 `getValue()`）
3. `createOrder(LogisticsOrder order)` — POST JSON body 到 `https://api.sf.com/order`，带自定义 Header `X-Api-Key: xxx`（用 `postBodyForEntity`）
4. `downloadLabel(String url)` — 下载面单 PDF（二进制），用 `getForData()` + `getResponseBytes()` 返回 `byte[]`
5. 泛型响应处理：`queryTrackList(String[] nos)` — 批量查询，用 `TypeReference<List<TrackResult>>` 处理泛型

作答：

```java


```

---

## 3. 用户注册全流程安全防护（12 分）— uw-mfa + uw-common + uw-dao

**场景**：用户注册接口需要完整的安全防护链路。同一 IP 连续失败 3 次后要求验证码，10 次后屏蔽 IP。注册成功后发送短信验证码确认手机号。

**要求实现 `RegisterHelper.register(String mobile, String password, String ip, String captchaId, String captchaSign)`**：

1. **IP 安全检查**：`MfaFusionHelper.checkIpErrorLimit(ip)`
   - 返回 `error` → IP 已屏蔽，直接返回 `ResponseData.errorCode(...)` 拒绝
   - 返回 `warn` → 需要验证码，先校验 `MfaFusionHelper.verifyCaptcha(ip, captchaId, captchaSign)`，校验失败则返回错误
   - 返回 `success` → 继续
2. **手机号校验**：`ValidateUtils.isChinaMobile(mobile)`，不通过返回 `ResponseData.warnCode(RegisterResponseCode.INVALID_MOBILE)`
3. **密码强度校验**：`ValidateUtils.isStrongPassword(password, 8, 20)`，不通过返回错误
4. **密码加密**：`DigestUtils.signHex(password, DigestUtils.Algorithm.SHA_256)`
5. **查重 + 新建**：查询手机号是否已注册（注意 `isError()` 判定陷阱），未注册则新建用户
6. **注册失败**：`MfaFusionHelper.incrementIpErrorTimes(ip, "注册失败")`
7. **注册成功**：`MfaFusionHelper.clearIpErrorLimit(ip)` 清除计数 → 发送短信验证码 `MfaFusionHelper.sendDeviceCode(ip, saasId, MfaDeviceType.MOBILE_CODE.getValue(), mobile, captchaId, captchaSign)`
8. 操作日志：注册成功并生成 Token 后，`AuthServiceHelper.logRef(User.class, newUserId)` + `logInfo("新用户注册")`（注意：logRef 依赖 Web 请求上下文中的 Token，需在 Token 生成之后调用）
9. **定义 `RegisterResponseCode` 枚举**（在 `constant/` 包下）：实现 `ResponseCode` 接口，包含 `INVALID_MOBILE` / `WEAK_PASSWORD` / `PHONE_EXISTS` 三个枚举值，每个值通过 `EnumUtils.enumNameToDotCase()` 生成 code，并说明配套的 i18n 资源文件应放在哪个目录下（12 种语种）

作答：

```java


```

---

## 4. AI 智能客服 + 自定义工具（13 分）— uw-ai + uw-dao

**场景**：平台需要集成 AI 客服，用户提问后调用大模型回答。同时需要支持自定义工具——当用户问"我的订单怎么样了"时，AI 自动调用订单查询工具。

**要求实现 `AiCustomerHelper` + `OrderQueryTool`**：

1. **同步对话** `chat(String userMessage)`：
   - `AiChatGenerateParam.builder().configCode("customer-service").userPrompt(message).bindAuthInfo().build()`
   - `AiClientHelper.generate(param)` 返回 `ResponseData<String>`
2. **流式对话** `streamChat(String userMessage)`：
   - 返回 `Flux<String>`，用 `AiClientHelper.chatGenerate(param)`
3. **带工具的对话** `chatWithOrderTool(String userMessage, long userId)`：
   - 构建 `AiToolCallInfo` 列表（`toolCode` = 工具类类名 `OrderQueryTool`，非 toolName）
   - `param.toolList(tools).toolContext(Map.of("userId", userId))`
4. **实现 `OrderQueryTool implements AiTool<OrderQueryTool.Param, ResponseData<String>>`**：
   - `@Component` 注解（AiTool 是 Spring Bean，框架自动注册元数据）
   - `toolName()` = "订单查询"，`toolDesc()` = "根据订单号查询订单状态"，`toolVersion()` = "1.0.0"
   - `Param extends AiToolParam`：字段 `orderNo`(String)，用 `@Schema(title="订单号", description="订单号", requiredMode=REQUIRED)` 标注
   - `apply(Param param)` — 用 `dao.queryForObject()` 查订单，返回订单状态描述

作答：

```java
// AiCustomerHelper.java


// OrderQueryTool.java


```

---

## 5. ES 操作日志查询 + 聚合统计（10 分）— uw-log-es

**场景**：运维需要查询 API 访问日志、统计接口调用量。日志已由 uw-auth-service 自动写入 ES。

**要求实现 `ApiLogQueryHelper`**：

1. 定义日志类 `ApiAccessLog extends LogBaseVo`（字段：userId / apiUri / statusCode / responseMillis / requestDate / saasId）
2. `static {}` 块注册：`LogClient.getInstance().regLogObjectWithIndexPattern(ApiAccessLog.class, "yyyyMM")`（按月分索引）
3. `queryByUser(long userId, int page, int size)` — DSL 查询：
   - 构建 DSL JSON：`term(userId)` + `sort(requestDate desc)` + `from` + `size`
   - `logClient.dslQuery()` + `LogClient.mapQueryResponseToPageList()` 转分页
4. `countByApi()` — 聚合统计各 API 调用量：
   - DSL：`size:0` + `aggs.terms(field: "apiUri")`
   - `LogClient.convertAggBucketFlatMap()` 拉平结果

作答：

```java


```

---

## 6. 投票去重 + GlobalCache + GlobalHashSet（10 分）— uw-cache

**场景**：在线投票活动，每个用户只能投一次票，需要防止重复投票。投票结果需要缓存以支撑高并发读取。

**要求实现 `VoteHelper`**：

1. `static {}` 块无需初始化（GlobalCache/GlobalHashSet 不需要 static 初始化）
2. `vote(long userId, long candidateId)` — 投票：
   - `GlobalHashSet.add("votedUsers", userId)` 添加到已投票集合，返回 boolean
   - 返回 false 表示已投过票，返回 `ResponseData.warnCode(VoteResponseCode.ALREADY_VOTED)`
   - 投票成功后：`GlobalCounter.increment(VoteResult.class, candidateId)` 增加候选人票数
   - `GlobalCache.invalidate("voteResults", candidateId)` 失效结果缓存
3. `getResult(long candidateId)` — 获取票数：
   - `GlobalCache.get("voteResults", candidateId, new CacheDataLoader<Long, Long>(){...}, 60000L)` 从缓存获取，loader 内调 `GlobalCounter.get()`
4. `hasVoted(long userId)` — 是否已投票：`GlobalHashSet.contains("votedUsers", userId)`

作答：

```java


```

---

## 7. 腾讯云短信 Linker + 授权扣减（10 分）— SaaS AIS + SaaS AIP

**场景**：平台对接腾讯云短信，每次发送短信需扣减租户的短信授权额度。

**要求实现**：

1. `TencentSmsLinker extends BaseAisLinker`（`@Service`）：
   - `typeName()` 返回 "短信服务"，`name()` 返回 "腾讯云短信"
   - `apiParam()` 返回 4 个配置参数：`sdkAppId`(STRING) / `appKey`(STRING) / `signName`(STRING) / `templateId`(STRING)
   - `sendSms(String mobile, Map<String,String> params)` — 用 `getParam("sdkAppId")` / `getParam("appKey")` 取配置，调用腾讯云 API
2. `SmsSendHelper.sendVerifyCode(long saasId, long mchId, String mobile)`：
   - 先 `AipHelper.checkAndDeductLicense(saasId, "saas-base-app:sms", 1)` 扣减授权
   - 获取 Linker：`AisHelper.getFitLinkerInstanceByLinker(saasId, TencentSmsLinker.class, mchId, null)`
   - 调用 `linker.sendSms(mobile, params)` 发送
   - 发送失败时 `AipHelper.refundLicense(saasId, "saas-base-app:sms", 1)` 退还授权

作答：

```java
// TencentSmsLinker.java


// SmsSendHelper.java


```

---

# 三、简答论述题（共 25 分）

> 考察对 UniWeb 核心设计理念的理解。与 A/C 卷论述题完全不同。

## 1. FusionCache 与 GlobalCache 的选型决策（10 分）

uw-cache 提供了 FusionCache（本地+Redis 融合）和 GlobalCache（纯 Redis）两种缓存方案。请论述：

1. 两者的架构差异是什么？FusionCache 通过什么机制保证多实例一致性？
2. 以下场景分别应该选哪个？说明理由：
   - 商品详情缓存（读 QPS 8000/s，单条实体，变更不频繁）
   - 用户搜索结果缓存（列表数据，按条件变化，需秒级过期）
   - 临时验证码缓存（60 秒过期，用户维度）
3. FusionCache 为什么要求在 `static {}` 块中初始化，而 GlobalCache 不需要？

答：

---

## 2. 链式调用的三种 onSuccess 重载（10 分）

`ResponseData.onSuccess` 有 Function / Consumer / Runnable 三种重载。请论述：

1. 三种重载各自的返回值是什么？（Function 返回新的 ResponseData，Consumer/Runnable 返回自身）
2. 各举一个典型使用场景（如：加载→修改→更新用什么版本？保存后清缓存用什么版本？）
3. 为什么 Function 版失败时"跳过 function 直接返回自身"被称为"扁平链式"？这个机制如何消除 if-else 判空？

答：

---

## 3. SaaS Finance 余额体系（5 分）

SaaS Finance 提供预存款、授信、佣金三种余额类型。请论述：

1. 三种余额类型各自支持哪些资金操作？（预存款 vs 授信 vs 佣金）
2. 佣金收入为什么需要 `rebateIncomeStart()` + `rebateIncomeNotifySuccess()` 两步？这种设计有什么好处？

答：

---

*— B 卷结束 —*
