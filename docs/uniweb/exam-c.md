# UniWeb 开发能力考核 — C 卷（转正考试）

> 考核目标：验证开发者能否**独立运用 UniWeb 框架完成完整业务系统**的设计与实现。重点考察跨模块综合运用、架构决策、复杂场景处理和错误恢复能力。
> 可参考 UniWeb 官方文档。满分 120 分。

---

# 一、方案选择题（每题 4 分，共 20 分）

> 以下每题给出一个复杂业务场景，选择最合适的综合方案。

**1. 某秒杀活动预计瞬时 QPS 20000/s，需保证：库存不超卖、每用户限抢一次、成功后异步创建订单。以下方案组合最合理的是？**

A. FusionCache 缓存商品 + synchronized 防超卖 + dao.save 同步建单

B. GlobalLocker 分布式锁 + GlobalHashSet 用户去重 + GlobalCounter 库存扣减 + TaskFactory.sendToQueue 异步建单

C. FusionCounter 库存计数 + 数据库唯一索引去重 + runTask 同步建单

D. GlobalCache 缓存库存 + GlobalSortedSet 排队 + runTaskLocal 本地建单

---

**2. 某多租户 SaaS 平台需要发送订单通知短信给用户，并上传发票 PDF 到对象存储。以下 API 组合正确的是？**

A. `MsgHelper.sendSms(MsgSmsVo)` 发送短信 + `SysOssHelper.genUploadParam(...)` 获取上传参数

B. `FinPaymentHelper.payOrder()` 发送短信 + `AisHelper.getFitLinkerInstanceByLinker()` 上传文件

C. `SaasInfoHelper.getSaasName()` 发送短信 + `LogClient.log()` 存储文件

D. `AuthServiceHelper.logInfo()` 记录短信 + `JsonInterfaceHelper.postBodyForEntity()` 上传文件

---

**3. 关于 UniWeb 的 `GlobalResponseAdvice` 全局响应包裹机制，以下说法正确的是？**

A. Controller 返回非 ResponseData 对象时，框架自动包裹为 `ResponseData.success(body)`；返回 null 自动包裹为 `ResponseData.warn()`；已是 ResponseData 时直接透传

B. 所有 Controller 方法必须手动将返回值包裹为 `ResponseData`，框架不做自动处理

C. `GlobalResponseAdvice` 仅在方法标注了 `@ResponseAdviceIgnore` 时才生效

D. String 类型返回值会直接输出原文，不会包裹为 ResponseData

---

**4. 某平台需要对接微信支付和支付宝，运营商可在后台切换。关于 AIS Linker 与 Finance 的协作方式，正确的是？**

A. Linker 只负责配置管理，支付逻辑全部在 `FinPaymentHelper` 中硬编码

B. 继承 `BaseAisLinker` 实现各支付通道 Linker（`apiParam()` 定义 appId/mchId/apiKey），`FinPaymentHelper.payOrder()` 内部通过 `AisHelper` 获取运营商配置的 Linker 实例执行支付

C. 每个支付通道写一个 Helper，用 if-else 切换

D. `FinPaymentHelper` 直接传入 appId/mchId/apiKey，不需要 Linker

---

**5. 关于 UniWeb 中"差量更新"机制，以下说法正确的是？**

A. `dao.update(entity)` 总是写入所有非主键字段，无论是否修改过

B. 框架 `load()` 后置 `_IS_LOADED=true`，此后每次 setter 调用 `addUpdateInfo` 记录变更，`update()` 仅对有变更字段生成 SET 子句；非 load 来源的 entity update 时写全部字段

C. `_IS_LOADED` 和 `_UPDATED_INFO` 字段开发者可以按需手动修改

D. 差量更新仅适用于有 `saas_id` 的 SaaS 实体

---

# 二、跨模块综合实现题（共 60 分）

> 所有场景与 A/B 卷完全不同，每题涉及 2-5 个模块协作。

## 1. 会员积分系统 — 签到 + 过期 + 缓存（12 分）

**涉及模块**：uw-dao + uw-cache + uw-task + SaaS AIP + uw-auth-service + uw-common

**场景**：设计会员积分系统。用户每日签到获得积分（防重复），积分每年底过期清零。

### 要求实现：

1. **签到接口** `SignInController.doSignIn()`（Guest 角色 `/guest/member/sign`）：
   - `GlobalLocker.tryLock()` 防当日重复签到 → `AipHelper.checkAndDeductLicense()` 扣减授权 → 差量更新积分 → 清缓存 → `logRef` + `logInfo` → `finally` 释放锁
2. **积分过期定时任务** `PointsExpireTask extends TaskCroner`（`@Component`）：
   - 每年 1 月 1 日执行，`RUN_TYPE_SINGLETON`，`BatchUpdateManager` 批量清零（先开事务），`logSysInfo()` 记录
3. **积分缓存** `PointsCacheHelper`：
   - FusionCache static 块初始化，`getPoints()` / `addPoints()`（更新+`FusionCache.put()`）/ `deductPoints()`（检查余额）

作答：

```java
// SignInController.doSignIn()


// PointsExpireTask


// PointsCacheHelper


```

---

## 2. 用户社区系统 — 发帖 + 敏感词 + 评论计数 + 父子联查（12 分）

**涉及模块**：uw-dao + uw-cache + uw-common-app + 开发规范 + uw-auth-service + uw-ai

**场景**：用户在社区发帖和评论。发帖时需 AI 内容审核和敏感词过滤，评论后实时更新帖子评论计数，帖子详情页需联查评论列表。

### 要求实现：

1. **发帖接口** `PostController.publish()`（Guest `/guest/community/post`）：
   - `@MscPermDeclare(user = UserType.GUEST, auth = AuthType.USER)`
   - 敏感词检查：横切 Helper `SensitiveWordHelper.check(content)`，不通过返回 `ResponseData.warnCode(PostResponseCode.SENSITIVE_CONTENT)`
   - AI 内容审核：`AiClientHelper.generate()` 调用大模型判断内容是否合规，不合规返回 `ResponseData.warnCode(PostResponseCode.AI_REJECTED)`
   - 入库：分配 `dao.getSequenceId()` → 设置 `createDate` → `dao.save()`
   - `AuthServiceHelper.logRef(Post.class, newId)` + `logInfo("发表帖子")`
   - **定义 `PostResponseCode` 枚举**（`constant/` 包）：实现 `ResponseCode` 接口，含 `SENSITIVE_CONTENT` / `AI_REJECTED` 枚举值，配套 i18n 资源文件目录（12 种语种）

2. **评论计数管理** `CommentCounterHelper`：
   - `static {}` 块配置 FusionCounter：`FusionCounter.config(Post.class, 60000L, 300000L, (postId, count) -> dao.execute("UPDATE post SET comment_count=? WHERE id=?", new Object[]{count, postId}))`
   - `increment(long postId)` — +1
   - `getCount(long postId)` — 获取（`FusionCounter.get(Post.class, postId, true)` 强制同步）

3. **帖子详情联查评论** `PostController.loadEx()`：
   - `PostEx extends Post` 含 `List<Comment> commentList`
   - `loadEx(long id)` — `dao.load(PostEx.class, id)` + `onSuccess` Consumer 查评论设值
   - 查评论：`dao.list(Comment.class, "SELECT * FROM comment WHERE post_id=? AND state=?", new Object[]{id, CommonState.ENABLED.getValue()})`
   - 空评论检查：为空时设空列表不报错

作答：

```java
// PostController.publish()


// CommentCounterHelper


// PostController.loadEx()


```

---

## 3. 商品管理与加载优化 — Vo/Ex + 缓存 + 多语言（12 分）

**涉及模块**：开发规范 + uw-dao + uw-cache + uw-common-app

**场景**：SAAS 运营商管理商品。商品列表只展示基础信息（不含成本价），详情页展示 SKU 列表，商品支持多语言，详情页高并发读取需要缓存。

### 要求实现：

1. **`ProductVo`**（继承 `DataEntity`）：仅用 `@ColumnMeta` 标注需输出字段（id/name/price/state），`cost_price` 不加注解即不映射。类级和字段级 `@Schema` 必须同时设置 `title` 和 `description`

2. **`ProductSkuEx extends Product`**：添加 `List<ProductSku> skuList` 字段

3. **懒加载方案**：父表标准 `list` 返回 `ProductVo`（`/saas/product/info`），SKU 独立 4 级路径子集 Controller（`/saas/product/info/sku`）。说明为什么选懒加载（SKU > 20 条/商品）

4. **商品缓存** `ProductCacheHelper`：
   - `static {}` 块初始化 FusionCache（Product.class，本地最大 10000 条，`cacheExpireMillis = -1`）
   - `CacheDataLoader<Long, Product>` 抽象类实现
   - `getProduct(long id)` — 缓存获取
   - `updateProduct(Product product)` — `dao.load()` 重新加载做差量更新 → `FusionCache.invalidate()`（不能用缓存对象做差量更新）

5. **多语言查询** `loadLang(long id)`：默认语言直接 `dao.load()`，非默认语言 LEFT JOIN `product_info_lang` + COALESCE 降级

作答：

```java
// ProductVo


// ProductSkuEx


// ProductSkuController（4级路径子集Controller）


// ProductCacheHelper


// loadLang 方法


```

---

## 4. 下单流程 — 防重 + 库存扣减 + 延迟取消 + 缓存（12 分）

**涉及模块**：uw-dao + uw-cache + uw-task + uw-auth-service + uw-common-app

**场景**：用户下单需要防止重复提交、安全扣减库存、30 分钟后自动取消未支付订单。

### 要求实现：

1. **下单接口** `OrderController.placeOrder()`（Guest `/guest/order/info`）：
   - `GlobalLocker.tryLock(Order.class, userId, 30000L)` 防重（stamp > 0 判定，`finally` 释放）
   - `dao.load(Product.class, productId)` 重新加载商品（不能用缓存对象做差量更新）→ 检查库存 → 减库存 → `dao.update()` 差量更新
   - 创建订单：分配序列 ID → 设置 `createDate`/`saasId`/状态 → `dao.save(order)`
   - 创建订单明细：批量保存 `OrderItem`
   - `TaskData.builder(OrderTimeoutTask.class, param).refId(orderId).taskDelay(30 * 60 * 1000).build()` → `taskFactory.sendToQueue()`
   - `FusionCache.invalidate(Product.class, productId)` 清商品缓存
   - `AuthServiceHelper.logRef(Order.class, orderId)` + `SysDataHistoryHelper.saveHistory(order, "用户下单")`

2. **超时取消任务** `OrderTimeoutTask extends TaskRunner`（`@Component`）：
   - `runTask()`：加载订单 → 检查是否仍为"待支付"状态
   - 取消订单 → 恢复库存（`dao.load()` 加载商品 → 增加库存 → 差量更新）→ 清缓存
   - 短信通知失败抛 `TaskPartnerException`（`retryTimesByPartner(3)`）
   - 订单已被手动处理抛 `TaskDataException`（不重试）

作答：

```java
// OrderController.placeOrder()


// OrderTimeoutTask


```

---

## 5. 订单管理与退款 — 状态流转 + Finance 退款 + ES 日志（12 分）

**涉及模块**：uw-dao + SaaS Finance + uw-auth-service + uw-common-app + uw-log-es + uw-task + SaaS AIS

**场景**：商户管理订单（发货/退款），退款涉及 Finance 余额退还、ES 日志记录和短信通知。

### 要求实现：

1. **商户订单 Controller** `MerchantOrderController`（MCH 角色 `/mch/order/info`）：
   - `$PackageInfo$` 声明 `/mch/order` 一级菜单
   - `ship(long id)` — 发货：`logRef` → `AuthIdStateQueryParam` 查"已支付"订单 → 改为"已发货" → 链式 Function 版 `onSuccess` 内 `dao.update()`
   - `list(AuthQueryParam param)` — `param.bindMchId()` 绑定当前商户

2. **退款处理** `RefundHelper.processRefund(long saasId, long orderId, long amountFen)`：
   - `logRef` + `logInfo("订单退款")`
   - `AuthIdStateQueryParam` 查"已支付"订单 → 改为"已取消"
   - 买家退款：`FinBalanceHelper.depositConsumeRefund(param)`（预存款退还）
   - 卖家佣金退回：`FinBalanceHelper.rebateIncomeNotifyRefund(param)`
   - 定义 `RefundLog extends LogBaseVo`，`LogClient.getInstance().log()` 写入 ES 退款日志
   - `SysDataHistoryHelper.saveHistory(order, "订单退款")`

3. **退款通知任务** `RefundNotifyTask extends TaskRunner`（`@Component`）：
   - 通过 `AisHelper.getFitLinkerInstanceByLinker()` 获取短信 Linker 发送退款通知
   - 发送失败抛 `TaskPartnerException`（重试 3 次）

作答：

```java
// $PackageInfo$


// MerchantOrderController


// RefundHelper.processRefund()


// RefundNotifyTask


```

---

# 三、架构设计论述题（共 40 分）

> 考察系统级设计决策能力。与 A/B 卷完全不同。

## 1. TaskFactory 任务执行策略选型（15 分）

TaskFactory 提供了 `sendToQueue` / `runTask` / `runTaskLocal` / `runTaskAsync` / `runQueue` 五种执行方式。它们在"执行位置×是否阻塞调用方×是否需要返回值"三个维度上各有差异。

请为以下 5 个场景选择最合适的执行策略，并说明理由：

| 场景 | 你的选择 | 理由 |
|------|---------|------|
| 30 分钟后自动取消未支付订单 | ? | ? |
| 用户支付成功后同步通知库存系统锁定库存（需确认结果） | ? | ? |
| 批量发送营销短信（百万级，不关心单条结果） | ? | ? |
| 高频短任务：实时计算用户积分（本机有 runner，需省 MQ） | ? | ? |
| 文件导出大报表（耗时 2-5 分钟，需返回下载链接） | ? | ? |

答：

---

## 2. SaaS 多租户数据安全体系（15 分）

某 SaaS 平台有租户（SAAS）和商户（MCH）两级，大部分业务表含 `saas_id`，部分含 `mch_id`。请论述：

1. **Controller 层**：`AuthQueryParam` / `AuthIdQueryParam` / `AuthIdStateQueryParam` 三者各自的使用场景和自动注入机制是什么？为什么 `AuthQueryParam` 仅限 Controller 层使用？

2. **Helper 层**：为什么不能直接用 `dao.load(Class, id)` 加载 SaaS 实体？正确的做法是什么？如果不遵守会导致什么安全风险？

3. **QueryParam 继承陷阱**：自定义 QueryParam 继承 `AuthPageQueryParam` 时，为什么禁止重新声明 `saasId` 字段？`QueryParamUtils.loadQueryParamMetaInfo` 是如何遍历继承链的？会导致什么具体 bug？

4. **RPC 鉴权**：内部 RPC 调用（`UserType.RPC`）的 `@MscPermDeclare` 应如何配置？与标准角色有什么区别？`ip-protected-paths` 在此场景下的作用是什么？

答：

---

## 3. 异常处理与重试策略体系（10 分）

UniWeb 在 DAO / Task / HTTP / 全局四个层面都有异常处理机制。请论述：

1. **DAO 层**：`dao.load()` 查不到数据返回 `warn` 而非抛异常。这种设计如何配合链式 `onSuccess` 实现自动跳过？对比传统 `try-catch` + 返回 null 的优势。

2. **Task 层**：三种异常策略（`TaskPartnerException` 重试 / `TaskDataException` 不重试 / 普通异常不重试）的设计理念。为什么程序 bug 不自动重试？

3. **HTTP 层**：`HttpRequestException` 继承 `TaskPartnerException`、`DataMapperException` 继承 `TaskDataException`。这种跨模块异常继承关系的设计意图是什么？在 uw-task 环境中运行时，异常分类语义如何生效？

4. **全局层**：`GlobalExceptionAdvice` 将 `TokenInvalidException`(401) / `TokenExpiredException`(498) / `TokenPermException`(403) 自动转为 HTTP 状态码。开发者在 Controller 中需要手动 try-catch 这些异常吗？为什么？

答：

---

*— C 卷结束 —*
