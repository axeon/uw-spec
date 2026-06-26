# UniWeb 开发能力实战考核 — A 卷（模拟考试）

> 本考核不设时间限制。满分 120 分。可参考 UniWeb 官方文档。
>
> 考核目标：检验开发者能否**综合运用 UniWeb 框架**完成真实业务系统的设计与实现。所有题目均为场景驱动，需要你理解 UniWeb 各模块的设计理念并灵活组合运用。

---

# Part 1 — 基础模块实战（每题 5 分，共 50 分）

> 本部分每个场景聚焦一个核心模块，考察你对模块 API 和设计理念的掌握。

---

## 1.1 电商商品管理 — Controller + DAO 链式调用

**业务场景**：你需要为某 SaaS 电商平台开发商品管理模块，面向 SAAS 运营商角色。

**要求实现**：

1. `$PackageInfo$` 类：声明 `/saas/product` 一级菜单权限
2. `ProductController` 类（`@RequestMapping("/saas/product/info")`），实现以下接口：

| 接口 | 业务逻辑 |
|------|---------|
| `list` | 分页查询当前租户商品列表（`AuthQueryParam` 自动注入 saasId） |
| `load` | 按 ID 加载商品详情，使用 `AuthIdQueryParam` 确保租户隔离 |
| `save` | 新建商品：分配序列 ID → 设置 `createDate` / `saasId` → 保存 → 成功后清 `FusionCache` 缓存 |
| `enable` | 启用商品：先 `logRef` 记录业务引用 → 用 `AuthIdStateQueryParam` 查 DISABLED 状态商品 → 改为 ENABLED → 更新 |
| `disable` | 禁用商品：与 enable 对称 |
| `delete` | 删除商品：先 `logRef` → 加载 → 删除 → 清缓存 |

**考核要点**：
- `$PackageInfo$` 的 2 级路径声明
- `@MscPermDeclare` / `@Operation` / `@Tag` 注解的正确使用
- `onSuccess` Function 版与 Consumer 版的区分使用
- `CommonState` 枚举替代硬编码状态值
- `AuthServiceHelper.logRef()` 的调用时机
- `AuthServiceHelper.getSaasId()` 获取当前租户

---

## 1.2 订单缓存层 — FusionCache + GlobalLocker

**业务场景**：商品详情页 QPS 约 5000/s，需要设计缓存层减轻数据库压力。同时，商品更新时需要防止并发覆盖。

**要求实现 `ProductCacheHelper`**：

1. `static {}` 块初始化 FusionCache：
   - 缓存 `Product.class`，本地最大 5000 条，`cacheExpireMillis = -1`（永久，仅靠 Redis TTL 兜底）
   - `CacheDataLoader<Long, Product>` 实现 `load()` 方法（`new CacheDataLoader<Long, Product>(){...}`，禁止 lambda）
2. `getProduct(long productId)` — 从 FusionCache 获取
3. `updateProduct(Product product)` — 更新数据库 + 失效缓存
4. `batchGetProducts(List<Long> ids)` — 批量获取（循环调用 FusionCache.get，利用本地缓存）
5. `safeUpdatePrice(long productId, long newPrice)` — 使用 `GlobalLocker` 加分布式锁安全更新价格：
   - `tryLock` 获锁（超时 30s）
   - 锁内 `dao.load(Product.class, productId)` 重新加载（不能用缓存对象，缓存对象 `_IS_LOADED=false`，差量更新失效）→ 改价 → 更新 → 失效缓存
   - `keepLock` 续期（长操作保护）
   - `finally` 中 `unlock`

**考核要点**：
- FusionCache 在 `static {}` 块初始化的设计原因
- `CacheDataLoader` 是抽象类不能用 lambda
- `cacheExpireMillis = -1` 的性能考量
- `GlobalLocker` stamp > 0 判定获锁、`keepLock` 续期、`finally` 释放
- 缓存失效用 `invalidate` 而非手动删除

---

## 1.3 超时订单处理 — TaskRunner 队列任务

**业务场景**：用户下单后 30 分钟未支付需自动取消。取消操作涉及：释放库存、退还优惠券、发送通知。通知可能因短信网关限流失败，需自动重试。

**要求实现**：

1. `OrderTimeoutTask extends TaskRunner<OrderTimeoutParam, Void>`：
   - `@Component` 注解（TaskRunner 是 Spring Bean）
   - `runTask()` 实现：检查订单状态 → 取消订单 → 释放库存 → 退优惠券 → 发通知
   - 短信网关失败时抛 `TaskPartnerException`（触发重试）
   - 订单状态异常（已被手动取消）时抛 `TaskDataException`（不重试）
   - `initConfig()` 配置：`retryTimesByPartner(3)`、`consumerNum(5)`、限流策略

2. 在 `OrderHelper` 中编写触发方法：
   - 下单时用 `TaskData.builder()` 构建任务数据，设置 `taskDelay(30 * 60 * 1000)`（30 分钟延迟）
   - 调用 `taskFactory.sendToQueue(taskData)` 异步入队

**考核要点**：
- TaskRunner 是 Spring Bean（`@Component`）vs Helper 纯静态
- `TaskPartnerException`（第三方失败，重试）vs `TaskDataException`（数据错误，不重试）的区分
- `retryTimesByPartner` 的语义（N+1 次执行）
- `TaskData.builder()` + `taskDelay()` 延迟任务
- `sendToQueue` 异步入队 vs `runTask` 同步执行

---

## 1.4 第三方短信网关对接 — AIS Linker

**业务场景**：平台需要对接多个短信服务商（阿里云、腾讯云、华为云），运营商可在后台切换配置。

**要求实现**：

1. 编写 `AliyunSmsLinker extends BaseAisLinker`：
   - `@Service` 注解（Linker 是 Spring Bean）
   - 实现 `typeName()` / `typeVersion()` / `name()` / `version()` / `devInfo()` / `typeDevInfo()`
   - `apiParam()` 返回配置参数定义：`accessKeyId` / `accessKeySecret` / `signName` / `templateCode`
   - `pubParam()` / `sysParam()` / `logParam()` 返回各自层级的参数
   - 业务方法 `sendSms(String mobile, Map<String, String> params)` — 使用 `getParam()` / `getBooleanParam()` 获取配置后调用阿里云 API

2. 在业务 Helper 中获取 Linker 实例并发送短信：
   - `AisHelper.getFitLinkerInstanceByLinker(saasId, AliyunSmsLinker.class, mchId, null)`
   - 调用 `linker.sendSms(mobile, params)`

**考核要点**：
- AIS Linker 三层配置参数的可见性设计：pubParam（公开）/ apiParam（运营商）/ sysParam（管理员）
- Linker 是 Spring Bean（`@Service`），框架自动扫描注册
- `AisHelper.getFitLinkerInstanceByLinker()` 获取适配的 Linker 实例
- `BaseAisLinker` 的 `getParam()` / `getBooleanParam()` 等类型化取值方法

---

## 1.5 用户认证与操作日志 — AuthServiceHelper

**业务场景**：管理员需要通过 Admin Controller 执行用户封禁操作，并完整记录操作日志（操作人、被操作对象、操作内容、请求响应）。

**要求实现 `banUser(long userId, String reason)` 接口**：

1. 路径：`/admin/user/info/ban`
2. 操作日志完整记录：
   - `@MscPermDeclare` 设置 `log = ActionLog.ALL`（记录请求+响应）
   - `AuthServiceHelper.logRef(User.class, userId)` 绑定业务引用
   - `AuthServiceHelper.logInfo("封禁用户，原因：" + reason)` 追加业务描述
3. 获取当前操作者信息：`AuthServiceHelper.getUserId()` / `getRealName()` / `getLoginIp()`
4. 执行封禁：`dao.queryForObject(User.class, new AuthIdQueryParam(getSaasId(), userId))` 租户隔离加载 → 设置 `state = CommonState.DELETED.getValue()` → `dao.update()` 差量更新
5. 记录数据历史：`SysDataHistoryHelper.saveHistory(user, "封禁用户")`

**考核要点**：
- `ActionLog.ALL` 的日志记录范围
- `logRef`（绑定业务引用）与 `logInfo`（追加业务描述）的配合使用
- `AuthServiceHelper` 获取当前用户上下文信息
- `SysDataHistoryHelper.saveHistory()` 记录数据变更历史

---

## 1.6 聚合支付与余额 — SaaS Finance

**业务场景**：商城用户下单后选择微信支付。支付成功后，订单金额需从买家预存款余额扣除，同时给卖家增加佣金收入。

**要求实现 `OrderPayHelper`**：

1. `createWechatPay(long saasId, long mchId, long orderId, long amountFen)` — 创建微信支付订单：
   - 构造 `FinPayOrderRequest`（saasId / mchId / bizOrderId / bizOrderType / payChannel / payTradeType / orderAmount / orderSubject）
   - `payChannel = TypeFinPayChannel.WECHAT.getValue()`，`payTradeType = TypeFinPayTrade.NATIVE.getValue()`
   - 调用 `FinPaymentHelper.payOrder(request)` 返回支付链接

2. `processPaymentSuccess(long saasId, long buyerMchId, long sellerMchId, long amountFen)` — 支付成功后处理：
   - 买家预存款扣款：`FinBalanceHelper.depositConsume(param)`
   - 卖家佣金收入：`FinBalanceHelper.rebateIncomeStart(param)` → `rebateIncomeNotifySuccess(param)`
   - 汇率转换（如需）：`FinCurrencyHelper.getCurrencyRate(saasId, "USD", "CNY")` + `calcCurrencyRate(rate, amountFen)`

**考核要点**：
- `FinPayOrderRequest` 的正确构造
- 预存款 `depositConsume` vs 佣金 `rebateIncomeStart` + `rebateIncomeNotifySuccess` 的操作差异
- 金额单位统一为分（long）
- `FinCurrencyHelper` 汇率查询与计算

---

## 1.7 商品多语言 — 数据层国际化

**业务场景**：商品名称和描述需要支持中文/英文/日文三种语言。数据库有主表 `product_info` 和翻译表 `product_info_lang`。

**要求实现**：

1. 翻译表 `product_info_lang` 的字段设计（`product_id` / `lang` / `product_name` / `product_desc` / `state`）
2. `listLang(AuthQueryParam param)` Controller 方法：
   - 获取当前语言：`LocaleHelper.getResolvedLanguageTag()`
   - 默认语言优化：当前语言 == 默认语言时，直接调标准 `list`（无需 LEFT JOIN）
   - 非默认语言：LEFT JOIN `product_info_lang`，`COALESCE` 降级取值
   - API 命名 `listLang` 与标准 `list` 区分

**考核要点**：
- 翻译表命名规范（`{主表名}_lang`）
- 外键字段命名（`{实体}_id`，如 `product_id`）
- LEFT JOIN + COALESCE 降级逻辑
- `LocaleHelper.getResolvedLanguageTag()` 从 `Accept-Language` 解析
- 默认语言优化（避免不必要的 JOIN）
- API 命名 `listLang` / `loadLang`

---

## 1.8 AI 智能客服 — uw-ai 集成

**业务场景**：平台需要集成 AI 客服，用户提问后调用大模型回答。同时需要支持自定义工具（如查询订单状态）。

**要求实现 `AiCustomerHelper`**：

1. `chat(String userMessage)` — 同步对话：
   - `AiChatGenerateParam.builder().configCode("customer-service").userPrompt(message).bindAuthInfo().build()`
   - `AiClientHelper.generate(param)` 返回回复

2. `streamChat(String userMessage)` — 流式对话（SSE）：
   - 返回 `Flux<String>`，用 `AiClientHelper.chatGenerate(param)`

3. `chatWithOrderTool(String userMessage, long userId)` — 带工具的对话：
   - 构建 `AiToolCallInfo` 列表（`toolCode` = 工具类类名）
   - `param.toolList(tools).toolContext(Map.of("userId", userId))`

4. 实现 `OrderQueryTool implements AiTool<OrderQueryTool.Param, ResponseData<String>>`：
   - `@Component` 注解（AiTool 是 Spring Bean）
   - `toolName()` / `toolDesc()` / `toolVersion()` / `apply(param)`
   - Param 继承 `AiToolParam`，用 `@Schema` 注解描述参数

**考核要点**：
- `AiClientHelper.generate()`（同步）vs `chatGenerate()`（流式）
- `bindAuthInfo()` 自动填充认证四元组
- `AiToolCallInfo` 的 `toolCode` 对应工具类类名
- `AiTool` 是 Spring Bean（`@Component`），框架自动注册元数据

---

## 1.9 ES 操作日志分析 — uw-log-es

**业务场景**：所有 API 操作日志通过 uw-auth-service 自动写入 ES。现在需要开发一个管理界面，查询和统计操作日志。

**要求实现 `AccessLogHelper`**：

1. 定义日志类 `ApiAccessLog extends LogBaseVo`（含 userId / apiUri / statusCode / responseMillis 等字段）
2. `static {}` 块中注册日志类型：`logClient.regLogObjectWithIndexPattern(ApiAccessLog.class, "yyyyMM")`（按月分索引）
3. `queryLogs(long userId, int page, int size)` — DSL 查询 + 分页：
   - 构建 DSL JSON（term 查询 + sort + from + size）
   - `logClient.dslQuery()` + `LogClient.mapQueryResponseToPageList()`
4. `countByApi(int days)` — 聚合统计：
   - DSL 聚合查询（`size:0` + `aggs`）
   - `LogClient.convertAggBucketFlatMap()` 拉平聚合结果
5. `exportAll(long startMs, long endMs)` — Scroll 大数据导出：
   - `scrollQueryOpen` → 循环 `scrollQueryNext` → `finally` 中 `scrollQueryClose`

**考核要点**：
- 日志类继承 `LogBaseVo`，启动期 `regLogObjectWithIndexPattern` 注册
- `dslQuery` + `mapQueryResponseToPageList` 分页查询
- 聚合查询 + `convertAggBucketFlatMap`
- Scroll 游标查询必须 `finally` 关闭

---

## 1.10 登录安全防护 — uw-mfa

**业务场景**：用户登录接口需要防护暴力破解。连续密码错误 3 次后要求输入验证码，10 次后屏蔽该 IP。

**要求实现 `SecureLoginHelper.login()`**：

1. 检查 IP 限制：`MfaFusionHelper.checkIpErrorLimit(ip)`
   - 返回 `error` → IP 已屏蔽，直接返回拒绝
   - 返回 `warn` → 需要验证码，先校验 `verifyCaptcha()`
   - 返回 `success` → 白名单或未触发限制，继续
2. 校验用户名密码
3. 密码错误：`MfaFusionHelper.incrementIpErrorTimes(ip, "密码错误")`
4. 登录成功：`MfaFusionHelper.clearIpErrorLimit(ip)` 清除计数
5. 发送短信验证码（二次验证）：`MfaFusionHelper.sendDeviceCode(ip, saasId, MfaDeviceType.MOBILE_CODE.getValue(), mobile, captchaId, captchaSign)`

**考核要点**：
- MFA 三态响应：`success`（放行）/ `warn`（需验证码）/ `error`（已屏蔽）
- `checkIpErrorLimit` → `verifyCaptcha` → 业务逻辑 → `incrementIpErrorTimes` / `clearIpErrorLimit` 的完整流程
- `sendDeviceCode` 发送设备验证码（含 Captcha 前置校验）
- 白名单 IP 全程豁免

---

# Part 2 — 跨模块综合实战（每题 15 分，共 45 分）

> 本部分要求你综合运用多个 UniWeb 模块协作完成业务流程。

---

## 2.1 SaaS 多租户商品上架全流程

**业务场景**：租户管理员要上架一个新商品，涉及：授权检查 → 商品入库 → 缓存预热 → 发送通知 → 记录日志。

**要求设计并实现 `ProductPublishHelper.publishProduct()`**，完整流程如下：

| 步骤 | 涉及模块 | 实现要点 |
|------|---------|---------|
| ① 授权检查 | SaaS AIP | `AipHelper.checkAndDeductLicense(saasId, "product-publish", 1)` 检查并扣减"商品上架"授权额度 |
| ② 商品入库 | uw-dao | 分配序列 ID → 设置 createDate/saasId → `dao.save(product)` |
| ③ 缓存预热 | uw-cache | `FusionCache.put(Product.class, id, product)` 主动写入缓存 |
| ④ 数据历史 | uw-common-app | `SysDataHistoryHelper.saveHistory(product, "商品上架")` |
| ⑤ 发送通知 | SaaS AIS | 通过 `AisHelper.getFitLinkerInstanceByLinker()` 获取短信 Linker，发送通知给关注用户 |
| ⑥ 操作日志 | uw-auth-service | `AuthServiceHelper.logRef(Product.class, product.getId())` + `logInfo("商品上架")` |
| ⑦ 异步任务 | uw-task | `taskFactory.sendToQueue()` 发送"商品索引重建"异步任务 |

**额外要求**：
- 如果步骤 ① 授权扣减失败，整个流程终止并返回错误
- 步骤 ②-⑥ 中任何一步失败，需要回滚授权额度（`AipHelper.refundLicense()`）
- 步骤 ⑦ 使用 `TaskData.builder()` 构建延迟任务

**考核要点**：
- AIP 授权扣减 + 退还的原子性保障
- 多模块协作的错误处理与回滚策略
- `FusionCache.put()` 主动预热 vs `get()` 被动加载
- `SysDataHistoryHelper` / `AuthServiceHelper.logRef` / `TaskFactory.sendToQueue` 的组合使用

---

## 2.2 电商订单全生命周期（Controller + Helper + Task + Cache + Finance）

**业务场景**：实现订单从创建到完成的完整流程，涉及多个 UniWeb 模块协作。

**要求设计以下组件**：

### 2.2.1 订单创建 `OrderHelper.createOrder()`

1. 分布式锁防重：`GlobalLocker.tryLock()` 防止用户重复提交
2. 扣减库存：`dao.load(Product.class, productId)` 重新加载（不能用缓存对象做差量更新）→ 检查库存 → 减库存 → 差量更新
3. 创建订单：分配序列 ID → 设置时间/状态 → `dao.save(order)`
4. 创建订单明细：批量保存 `OrderItem` 列表
5. 发送延迟任务：30 分钟超时自动取消（`TaskData.builder().taskDelay()`）
6. 清缓存 + 记日志 + 数据历史
7. `finally` 释放锁

### 2.2.2 订单超时取消 `OrderTimeoutTask extends TaskRunner`

1. 加载订单 → 检查是否仍为"待支付"状态
2. 取消订单 → 恢复库存 → 退还优惠券
3. 发送取消通知（可能失败 → `TaskPartnerException` 重试）
4. 订单已被手动处理 → `TaskDataException` 不重试

### 2.2.3 订单支付 `OrderController.pay()`

1. Guest 角色，路径 `/guest/order/info/pay`
2. 调用 `OrderPayHelper.createWechatPay()` 创建支付
3. 支付回调：更新订单状态 → 买家扣款 → 卖家收入 → 发通知

### 2.2.4 订单列表（父子表联查） `OrderController.listEx()`

1. 联查订单 + 订单明细（`OrderInfoEx extends OrderInfo`）
2. Guest 角色，`@MscPermDeclare(user = UserType.GUEST, auth = AuthType.USER)`
3. 空页检查 + IN 查询 + 分组设值

**考核要点**：
- `GlobalLocker` 防重 + `finally` 释放
- 差量更新（load → modify → update）
- `TaskRunner` + `TaskPartnerException` / `TaskDataException`
- Guest 权限（`AuthType.USER`）与标准角色权限（`AuthType.PERM`）的区别
- 父子表联查 `listEx` 的完整实现
- `FinBalanceHelper` 买家扣款 / 卖家收入的配合

---

## 2.3 多语言内容管理平台

**业务场景**：构建一个支持多语言的内容管理平台，包含文章管理、评论、标签、多语言翻译、AI 辅助翻译。

**要求设计以下能力**：

1. **文章管理 Controller**（Admin 角色）：
   - CRUD + 启用/禁用 + 软删除
   - `listEx` 联查评论数和标签
   - `listLang` 多语言查询（LEFT JOIN `_lang` 表 + COALESCE）

2. **AI 辅助翻译 Helper**：
   - 调用 `AiClientHelper.translateList()` 批量翻译文章标题和摘要
   - 或 `AiClientHelper.translateMap()` 翻译键值对
   - 翻译结果写入 `_lang` 表

3. **评论管理 Controller**（Guest 角色）：
   - Guest 发表评论：`@MscPermDeclare(user = UserType.GUEST, auth = AuthType.USER)`
   - 敏感词检查：调用 `SensitiveWordHelper`（横切 Helper）
   - 评论后更新文章评论计数（`FusionCounter.increment()`）

4. **定时统计 Task**：
   - `TaskCroner` 每小时统计各文章阅读量/评论数
   - 结果写入 ES（`LogClient.log()`）
   - 通过 `TaskCronerConfig.RUN_TYPE_SINGLETON` 保证全局单例执行

**考核要点**：
- 多语言数据层方案（`_lang` 表 + LEFT JOIN + COALESCE）
- `listEx` 父子表联查 + `listLang` 多语言查询的组合
- `AiClientHelper.translateList()` / `translateMap()` 的运用
- Guest `AuthType.USER` 权限模型
- 横切 Helper（SensitiveWordHelper）的设计理念
- `FusionCounter` 计数器的高频累加 + 定期同步
- `TaskCroner` + `RUN_TYPE_SINGLETON` 定时任务
- ES 日志写入

---

# Part 3 — 架构设计与方案论述（每题 5 分，共 25 分）

> 本部分考察你对 UniWeb 架构理念的理解深度，以及面对复杂场景时的技术决策能力。

---

## 3.1 缓存架构设计

**场景**：某 SaaS 平台有以下缓存需求，请为每个场景选择合适的 uw-cache 组件并说明理由。

| 场景 | 读 QPS | 数据特征 | 一致性要求 | 你的方案 |
|------|--------|---------|-----------|---------|
| 商品详情 | 8000/s | 单条实体，变更不频繁 | 最终一致 | ? |
| 商品搜索结果 | 2000/s | 列表数据，按条件变化 | 秒级过期 | ? |
| 用户购物车 | 500/s | 临时数据，用户维度 | 实时 | ? |
| 秒杀库存计数 | 10000/s | 高频递增 | 强一致 | ? |
| 分布式防重 | — | 请求去重 | — | ? |
| 延迟取消订单 | — | 30 分钟后触发 | — | ? |

请逐行说明你的选型理由，以及初始化和使用方式。

---

## 3.2 权限体系设计

**场景**：某平台有以下角色和功能需求，请设计 Controller 包结构和路径层级。

| 角色 | 功能模块 |
|------|---------|
| SAAS 运营商 | 商品管理、订单管理、租户配置 |
| 商户 | 商品管理、订单管理 |
| 平台管理员 | 内容审核、用户管理、系统监控 |
| C 端用户 | 浏览商品、下单、评论、个人中心 |
| 内部 RPC | 数据同步、支付回调 |

**要求**：
1. 画出完整的 controller 包结构（到 module 级）
2. 为每个角色模块标注 `$PackageInfo$` 路径和 `@MscPermDeclare` 配置
3. 说明 Guest 角色与其他角色的核心区别（权限模型、路径层级、`$PackageInfo$`）
4. 指出 1:N 子集关系（如商品-SKU、订单-明细）应如何设计 Controller 路径

---

## 3.3 DAO 无数据返回值语义体系

**场景**：UniWeb 的 `DaoManager` 对"无数据/0行"的返回语义因方法而异，这是框架核心设计之一。

**请论述**：

1. `dao.load(Class, id)` 和 `dao.queryForObject()` 查不到数据时返回什么状态？`dao.list()` 无数据时返回什么？两者为什么不同？

2. 写操作（save/update/delete）影响行数 < 1 时返回什么状态？

3. 在"查无数据需要新建"的业务场景中，应使用 `isError()` 还是 `isNotSuccess()` 判断？误用会导致什么后果？

4. 链式 `onSuccess` / `onWarn` / `onNotSuccess` 如何利用这些状态实现自动短路跳过？

---

## 3.4 GlobalResponseAdvice 与异常处理体系

**场景**：UniWeb 通过 `GlobalResponseAdvice` 和 `GlobalExceptionAdvice` 实现全自动的响应包裹和异常处理，开发者无需在 Controller 中 try-catch。

**请论述**：

1. `GlobalResponseAdvice` 如何处理不同类型的 Controller 返回值？（非 ResponseData 对象 / null / 已是 ResponseData / String / ResponseEntity）

2. 哪些异常类型对应哪些 HTTP 状态码？（`TokenInvalidException` / `TokenExpiredException` / `TokenPermException` / `TokenServiceException`）

3. 在什么场景下需要用 `@ResponseAdviceIgnore` 跳过包裹？举一个实际例子。

4. 开发者在 Controller 中需要手动 try-catch 这些 Token 异常吗？为什么？

---

## 3.5 SaaS 计费体系设计

**场景**：某 SaaS 平台需要设计完整的计费体系，包含：功能订阅、按量计费、余额管理三层。

**请论述以下设计方案**：

1. **功能订阅层（AIP App Vendor）**：租户订阅"商城模块"（按年计费），订阅到期后自动停用。说明 Vendor 实现方式、`initData()` / `destroyData()` 的作用。

2. **按量计费层（AIP License Vendor）**：短信/邮件按条计费，预充值额度。说明 `checkAndDeductLicense` 的调用时机和并发安全保障。

3. **余额管理层（Finance）**：预存款充值消费、佣金收入分配。说明三种余额类型的资金流转关系。

4. **支付通道层（AIS Linker）**：对接微信/支付宝，运营商可切换。说明 Linker 与 Finance 的协作方式。

---

*— A 卷结束 —*
