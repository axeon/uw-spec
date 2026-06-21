# 开发规范

## 代码生成与开发流程

**代码生成机制**：

entity、dto、controller 基础代码通过 uw-code-center 工具自动生成。开发人员必须对自动生成的代码进行裁剪：

- **DTO 裁剪**：精确控制搜索条件字段和排序字段，确保接口参数符合业务需求
- **Controller 裁剪**：实现权限控制逻辑，移除不需要的功能接口，优化接口设计

**开发重点与代码组织**：

- 开发核心为 **service 层** 和 **controller 层**实现
- 针对非 CRUD 的复杂业务逻辑，必须封装在 **service 层**或 **Helper 工具类**中，供 controller 层调用

### Vo/Ex 规范

| 类型 | 命名 | 用途 | 约束 |
|------|------|------|------|
| 基础 Vo | `{Entity}Vo extends DataEntity` | 裁剪敏感字段（password/phone/openid 等） | 使用 `@TableMeta` + `@ColumnMeta`，仅标注需输出的字段，框架自动映射；敏感字段不加 `@ColumnMeta` 即不映射 |
| 扩展 Ex | `{Entity}Ex extends {Entity}` | 附加关联数据（如明细列表、附加计数） | 继承父 Entity，自动继承 `@TableMeta`/`@ColumnMeta`，框架可直接 `dao.load(ExClass, id)` |

- Vo/Ex 均可直接 `dao.load(VoClass/ExClass, id)`，无需手动 new + set
- Vo/Ex 统一在 **vo 包**下创建并管理

> 完整 Vo/Ex 代码模板见 [code-templates.md](code-templates.md)「Vo 模板」。

## Controller 规范

### 路径规范

**关键约束**：
- controller包层级最多两级，一级为用户角色名，二级为模块名（一级菜单）。
- 所有权限的第一级路径必须为用户角色名。
- **路径禁止使用 `-`（短横线）和 `_`（下划线）**。错误示例：`/user-profile`、`/order_item`；正确示例：`/userProfile`、`/orderItem`。

**GUEST用户权限**:
- **路径格式**：`/{role}/{一级菜单}/{二级菜单}/{功能权限}`

**非GUEST用户权限**：
- **路径格式**：`/{role}/{一级菜单}/{二级菜单}/{功能权限}` 或 `/{role}/{一级菜单}/{二级菜单}/{功能子集}/{功能子权限}`
- 一级菜单权限由包下面的$PackageInfo$.java类定义，此定义的路径必须为2级。
- 二级菜单权限由包下面的XXXController.java类定义，此定义的路径必须为3级。
- **1:N关系中的子集**需要定义为功能子集权限，由包下面的XXXController.java类定义，此定义的路径必须为4级。
- 每个URL路径必须存在父级路径定义，否则权限菜单无法正常显示。

### @Tag 命名规范

`@Tag(name = "本级功能名")`，只写当前 Controller 管理的功能名称，不包含角色和菜单层级前缀。路径层级已由 `@RequestMapping` 和 `$PackageInfo$` 定义，`@Tag` 仅用于 Swagger 分组展示。

```java
// ✅ 正确：name 只写本级功能，不带"-"/"_"
@Tag(name = "敏感词管理", description = "敏感词增删改查管理")

// ❌ 错误：name 包含完整菜单层级
@Tag(name = "SAAS-内容管理-敏感词管理", description = "SAAS端内容管理-敏感词增删改查列管理")
```

### 权限声明规范

所有接口必须使用 `@MscPermDeclare` 注解声明权限。**不同角色的 user 和 auth 参数不同**：

| 角色 | user | auth | 说明 |
|------|------|------|------|
| SAAS | `UserType.SAAS` | `AuthType.PERM` | 验证权限，注册菜单 |
| MCH | `UserType.MCH` | `AuthType.PERM` | 验证权限，注册菜单 |
| ADMIN | `UserType.ADMIN` | `AuthType.PERM` | 验证权限，注册菜单 |
| ROOT | `UserType.ROOT` | `AuthType.PERM` | 验证权限，注册菜单 |
| OPS | `UserType.OPS` | `AuthType.PERM` | 验证权限，注册菜单 |
| RPC | `UserType.RPC` | `AuthType.NONE` | 内部调用无需鉴权 |
| **GUEST** | `UserType.GUEST` | `AuthType.USER` | **仅验证登录，不验证权限** |

> 完整角色映射表、$PackageInfo$ 模板、Controller 方法体模板见 [code-templates.md](code-templates.md)「Controller 模板」。

### 响应格式规范

所有 Controller 方法返回值必须使用 `ResponseData` 包装（单对象 `ResponseData<T>`、分页列表 `ResponseData<PageList<T>>`、无数据 `ResponseData`）。

### 参数类型规范

方法参数一律使用基本类型（`long`、`int`、`boolean`），禁止使用包装类型（`Long`、`Integer`、`Boolean`）。仅当参数明确需要接受 `null` 值时（Javadoc 标注"可选"），才允许使用包装类型。

| 场景 | 类型 | 示例 |
|------|------|------|
| ID参数（必填） | `long` | `getById(long id)` |
| 数组参数（必填） | `long[]` | `batchPublish(long saasId, long[] ids)` |
| 可选筛选参数（需判null） | `Long` | `suggestQuestions(String keyword, Long categoryId)` |
| CacheDataLoader泛型 | `Long` | `CacheDataLoader<Long, Entity>` — 泛型必须用包装类型 |

此规范适用于 Controller 方法参数、Helper 方法参数、以及所有手写代码中的方法参数。代码生成器产出的 entity/dto 中的字段类型不在此规范约束范围内。

### 实体类规范

实体类继承 `DataEntity`，使用 `@TableMeta` 和 `@ColumnMeta` 注解。

**差量更新机制**（框架核心，代码生成器产出的 Entity 已内置）：

1. 框架 `load` 后反射置 `_IS_LOADED=true`
2. 此后每次 setter 调用 `DataUpdateInfo.addUpdateInfo(info, "col", oldVal, newVal, _IS_LOADED)` 记录变更
3. `update(entity)` 仅对有变更的字段生成 `SET col=?`，未改字段不入 SQL
4. 非 load 来源（`_IS_LOADED=false`）的 entity，`addUpdateInfo` 不记录（初始赋值），update 时写全部非主键字段

**关键约束**：
- 两个 `transient` 字段 `_IS_LOADED` / `_UPDATED_INFO` 由框架维护，**勿手动修改**
- `@ColumnMeta.columnName` 大小写不敏感（框架内部统一用小写匹配）
- setter 必须返回 `this`（链式），并在内部调用 `addUpdateInfo` 记录变更
- Entity 由代码生成器产出，**保留不动，不手动修改**

> 完整实体类模板见 [code-templates.md](code-templates.md)「Entity 实体类模板」。

### @Schema 注解规范

所有 Entity、Dto、Vo 类及其字段的 `@Schema` 注解**必须同时设置 `title` 和 `description`**：

```java
// ✅ 正确：同时设置 title 和 description
@Schema(title = "用户信息", description = "用户信息")
public class GuestInfoVo {
    @Schema(title = "主键", description = "主键")
    private long id;
}

// ❌ 错误：只有 description，缺少 title
@Schema(description = "用户信息")
public class GuestInfoVo {
    @Schema(description = "主键")
    private long id;
}
```

此规范适用于所有 `@Schema` 注解位置：Entity 类级/字段级、Dto 类级/字段级、Vo 类级/字段级。

## DAO 数据访问规范

- 优先使用 `DaoManager`（ResponseData 链式lambda风格），避免使用 `DaoFactory`（抛异常风格）
- `DaoManager` 通过 `DaoManager.getInstance()` 静态获取，无需 Spring 注入
- 查询参数使用 QueryParam 系列类（`IdQueryParam`、`AuthQueryParam`、`AuthIdStateQueryParam` 等），**仅限 Controller 层使用**（依赖 Auth 上下文）
- Helper 层按ID加载**SaaS实体（有saas_id字段）**时使用 `dao.queryForObject(Entity.class, new AuthIdQueryParam(saasId, id))`（自动拼接 `WHERE id=? AND saas_id=?`，无需手写SQL）。**对于无 saas_id 的非SaaS实体可直接 `dao.load(Entity.class, id)`。**需附带 state 条件时使用 `AuthIdStateQueryParam(saasId, id, state)`
- 批量操作使用 `BatchUpdateManager`

### 链式调用规范

DaoManager 所有方法返回 `ResponseData<T>`，这是框架的核心设计模式。通过链式 `onSuccess` / `onError` / `onWarn` / `onNotSuccess` 回调，实现零中间变量、自动错误传播的简洁代码。

**onXxx 全家族**（每个回调都有 3 种重载：Function 转换 / Consumer 副作用 / Runnable 无参）：

| 回调 | 触发条件 | 典型用途 |
|------|---------|---------|
| `onSuccess` | state == SUCCESS | 成功后处理（最常用） |
| `onWarn` | state == WARN | 数据未找到等业务告警分支 |
| `onError` | state == ERROR | 系统异常分支 |
| `onFatal` | state == FATAL | 致命错误分支 |
| `onNotSuccess` | state != SUCCESS | 通用失败兜底（WARN/ERROR/FATAL 都触发） |
| `onNotError` | state != ERROR | 非 ERROR 情况（含 SUCCESS/WARN/FATAL） |

**三种重载**（以 `onSuccess` 为例，其余 onXxx 同构）：

| 重载 | 签名 | 返回值 | 适用场景 |
|------|------|--------|---------|
| **Function 版（转换）** | `onSuccess(Function<T, ResponseData<R>>)` | 新的 `ResponseData<R>` | 链式类型转换：`dao.load()` → 修改 → `return dao.update()` |
| **Consumer 版（回调）** | `onSuccess(Consumer<T>)` | 自身 `ResponseData<T>` | 副作用：缓存失效、日志、数据组装 |
| **Runnable 版（无参）** | `onSuccess(Runnable)` | 自身 `ResponseData<T>` | 不需要 data 的操作 |

> **Function 版核心机制**：成功时执行 function 返回新的 ResponseData（类型 T→R）；失败时跳过 function 直接返回自身（自动转型），这是"扁平链式"的真正实现。`dao.queryForObject(...).onSuccess(e -> dao.update(e))` 返回的是 update 的 ResponseData。

| 规则 | 说明 | 示例 |
|------|------|------|
| **直接返回** | 简单 CRUD 直接 `return dao.xxx()` | `return dao.list(User.class, param);` |
| **链式后处理** | 需要额外操作时用 `onSuccess` | `return dao.save(user).onSuccess(u -> cache.invalidate(u.getId()));` |
| **自动跳过** | 链中任何一步 WARN/ERROR，后续 `onSuccess` 自动跳过 | 无需 `if (result.isSuccess())` 判空 |
| **禁止 if-else 判断** | 禁止 `if (result.isSuccess()) { ... } else { ... }` | 用 `.onSuccess(...)` / `.onNotSuccess(...)` 替代 |
| **扁平链式** | 禁止嵌套 `onSuccess` 内再嵌套 `onSuccess`，用 `return dao.xxx()` 展平 | 见下方说明 |
| **日志前置** | `logRef()` 放在方法体开头（`return` 之前），不嵌套在 `onSuccess()` 内 | 避免漏记日志 |
| **空页检查** | `onSuccess` 内第一行 `if (list.isEmpty()) return;` | 父子表联查必须检查空页 |
| **原地设值** | `onSuccess` 内直接修改 ResponseData 中的对象引用 | `orders.forEach(o -> o.setItems(...))` |
| **状态分支用 onWarn/onError** | 区分"数据未找到"与"系统异常"用 `onWarn` / `onError`，而非嵌套 if | `dao.load(...).onWarn(w -> log.warn("未找到"))` |

> 完整链式调用代码范例见 [uw-dao.md](uw-dao.md)「DaoManager 链式调用设计」和 [uw-common.md](uw-common.md)「ResponseData 链式回调」。

### 父子表查询方案

当存在 1:N 父子表关系（如订单 OrderInfo + 订单明细 OrderItem）时，提供两种查询方案：

| 方案 | API 命名 | 实现位置 | 适用场景 |
|------|---------|---------|---------|
| **联查方案** | `listEx` | Controller 内 `onSuccess` 嵌套，无需 Helper | 父列表需直接嵌套子数据（如订单列表展示商品明细） |
| **懒加载方案** | `list` + 子集 Controller `list` | 父表标准 `list`，子表独立 4 级路径 Controller | 子数据量大或按需展开，前端二次请求 |

**联查方案（`listEx`）规范**：

| 约束 | 说明 |
|------|------|
| Ex 类继承父 Entity | `OrderInfoEx extends OrderInfo`，ORM 通过继承的 `@TableMeta` 自动映射所有父字段 |
| API 命名 `listEx` | 与标准 `list` 区分，表示返回扩展数据（含子表） |
| 直接写在 Controller | 逻辑简单（2次查询+组装），无需 Helper |
| `onSuccess` 链式嵌套 | 外层查父表，内层 IN 查子表，失败/WARN 时链自动跳过 |
| 原地设值 | `forEach(o -> o.setItems(...))` 修改 ResponseData 内部同一引用 |
| SQL 模式 | 1次 `dao.list(ExClass, param)` + 1次 `dao.list(ChildClass, inSql, ids)`，共2次查询 |

**懒加载方案规范**：

| 约束 | 说明 |
|------|------|
| 父表标准 `list` | 无子数据，直接 `dao.list(Entity, param)` |
| 子集独立 Controller | 4 级路径（如 `/saas/order/item`），前端按需调用 |
| 遵循 gencode-trim-guide §2.D 子集实体裁剪规则 | |

**方案选择决策**：

| 条件 | 选择 |
|------|------|
| 父列表页面需要直接展示子数据 | **联查方案**（`listEx`） |
| 子数据量大（>20条/父记录）或按需展开 | **懒加载方案**（独立子集 Controller） |
| 两种都需要（不同页面/角色） | **两个都提供**：`list`（标准）+ `listEx`（联查） |

完整代码示例见 [code-templates.md](code-templates.md)「DAO 数据访问模板」。

### 多语言数据查询方案

系统多语言统一方案，涵盖程序层和数据层：

| 层级 | 类型 | 实现方式 | 说明 |
|------|------|---------|------|
| **程序层** | 错误码/枚举标签 | ResponseCode + i18n 资源文件（12种语种） | 见 code-templates.md「枚举与响应码模板」 |
| **数据层** | 用户业务数据 | `_lang` 翻译表 + LEFT JOIN + COALESCE 自动降级 | 本节 + code-templates.md「DAO 数据访问模板」 |

> **程序层多语言**已在 code-templates.md「枚举与响应码模板」完整定义（ResponseCode 枚举 + `ResourceBundleMessageSource` + 12 种语种资源文件），此处不再重复。

**数据层多语言规范**：

| 约束 | 说明 |
|------|------|
| 翻译表命名 | `{主表名}_lang`，如 `product_info_lang` |
| 不需要 Ex 类 | COALESCE 别名与主表字段同名，ORM 自动映射回主 Entity 同名字段 |
| API 命名 `listLang` / `loadLang` | 与标准 `list`/`load`、父子表 `listEx` 区分 |
| lang 通过 `LocaleHelper.getResolvedLanguageTag()` 获取 | 从 `Accept-Language` 请求头自动解析，白名单校验，不通过 `@RequestParam` 传入 |
| 默认语言优化 | `getResolvedLanguageTag()` 与 `getDefaultLanguageTag()` 一致时，直接调标准 `list`/`load`，无需 LEFT JOIN |
| LocaleHelper 位于 `service/` 包 | 纯 static 工具类，与 Helper 同包，按项目语种配置 `SUPPORTED_LANGUAGE_TAGS` |
| LEFT JOIN 条件含 `lang` + `state=1` | `l.{entity}_id=t.id AND l.lang=? AND l.state=1` |
| 外键字段使用 `{实体}_id` | 如 `product_id`，不用 `ref_id` |
| 降级逻辑 | COALESCE：翻译表无记录时自动返回主表默认语言值 |
| _lang 表标准 CRUD | 由代码生成器生成，作为子集实体（4 级路径 Controller）处理 |

完整代码示例见 [code-templates.md](code-templates.md)「DAO 数据访问模板」。

## Helper 设计规范

Helper 是纯静态工具类，**不是 Spring Bean**，不加 `@Component`，不使用构造器注入。

**创建条件**（三条件满足至少一项才创建 Helper）：

| 条件 | 说明 | 示例 |
|------|------|------|
| 逻辑复杂 | 状态机、多步流程、计算、复杂校验 | 内容审核（多状态流转）、回答采纳（锁+状态+积分） |
| 功能性 | 缓存、分布式锁、事务等横切关注点 | 详情缓存（FusionCache）、并发操作（GlobalLocker） |
| 多处调用 | 2个以上 Controller 或其他 Helper 调用 | 发送通知（多处触发）、用户信息查询（多处引用） |

**不建 Helper 的场景**：简单 CRUD（list/get/save/update/delete/enable/disable）直接在 Controller 中调 `DaoManager.getInstance()` 即可。

**Helper 两种类型**：

| 类型 | 识别维度 | 示例 |
|------|---------|------|
| **模块级 Helper** | 按数据库表/模块识别，一个模块一个 Helper | PostQuestionHelper、PostAnswerHelper |
| **横切 Helper** | 按 PRD 中跨模块的公共业务规则识别，一个规则一个 Helper | SensitiveWordHelper、MsgNotifyHelper |

> 横切 Helper 识别方法：通读 PRD，找出"在多个模块中出现相同描述"的业务规则。

> 完整 Helper 代码模板（含 import/类结构/缓存/方法签名）见 [code-templates.md](code-templates.md)「Helper 模板」。

**核心约束**：纯静态工具类，`DaoManager.getInstance()` 静态获取，禁止 `@Component` + 构造器注入。

## 枚举与响应码规范

### 枚举类组织

在 `{package}/constant/` 包下定义业务枚举类，替代代码中的硬编码数字和字符串。

**必须定义枚举的场景**：
- **状态值**：`state` 字段的 0/1/2 等数字 → 使用 `CommonState` 或自定义枚举
- **类型值**：`noticeType`、`refType`、`penaltyType` 等分类字段
- **响应码**：`warnCode`/`errorCode` 的字符串参数 → 使用 `ResponseCode` 枚举

> 完整枚举定义模板、ResponseCode 模板、i18n 资源文件模板见 [code-templates.md](code-templates.md)「枚举与响应码模板」。

### CommonState 使用规范

`uw-common-app` 提供的 `CommonState` 枚举（ENABLED=1, DISABLED=0, DELETED=-1），**Helper 和 Controller 中必须使用此枚举替代硬编码**：

```java
// ❌ 错误：硬编码数字
entity.setState(1);
entity.setState(0);

// ✅ 正确：使用 CommonState
entity.setState(CommonState.ENABLED.getValue());
entity.setState(CommonState.DISABLED.getValue());
```

### ResponseCode 使用规范

`ResponseData.warnCode()` 和 `ResponseData.errorCode()` 支持传入 `ResponseCode` 枚举，**业务响应码必须定义为枚举，禁止硬编码字符串**：

```java
// ❌ 错误：硬编码字符串，散落在各 Helper 中无法集中管理
return ResponseData.warnCode("USER_NOT_FOUND", "用户不存在");
return ResponseData.errorCode("UPDATE_FAILED", "更新失败");

// ✅ 正确：使用 ResponseCode 枚举，类型安全、集中管理
return ResponseData.warnCode(GuestResponseCode.USER_NOT_FOUND);
return ResponseData.errorCode(CommonResponseCode.ENTITY_UPDATE_ERROR);
```

**枚举分层**：
- 通用场景（实体不存在、保存失败等）→ 使用 `CommonResponseCode`（uw-common-app 提供）
- 业务场景（密码错误、积分不足等）→ 在 `{package}/constant/` 包下定义业务 `ResponseCode` 枚举
- 同一业务的响应码放在同一个枚举类中（如 `GuestResponseCode`、`PostResponseCode`、`CmsResponseCode`）

### ResponseCode i18n 资源文件规范

每个业务 `ResponseCode` 枚举必须配套 i18n 资源文件（12 种语种），资源文件位置：`src/main/resources/{枚举类全路径}/`。

> 完整 i18n 资源文件目录结构和示例见 [code-templates.md](code-templates.md)「枚举与响应码模板」。

## 缓存使用规范

- 高频读取场景使用 `FusionCache`（本地 + Redis 融合缓存）
- Kryo 序列化**必须使用具体实现类**，不能使用接口类型（如 List/Map/Set），必须传 ArrayList/LinkedHashMap/HashSet 等；`CacheDataLoader<V>` 的 V 必须是具体类
- Caffeine 设定过期时间后性能劣化 200 倍，建议 `cacheExpireMillis` 保持 -1（永久），仅靠 Redis TTL 兜底
- **FusionCache 必须在 static 块中初始化**：所有使用 FusionCache 的 Helper 必须在 `static {}` 块中一次性完成 `FusionCache.config(new FusionCache.Config(...), new CacheDataLoader<>() {...})` 初始化，包括缓存参数（容量、过期时间）和 CacheDataLoader.load() 实现。GlobalCache 不需要 static 初始化（使用行内 CacheDataLoader）。
- **`CacheDataLoader` 是抽象类**，不是函数式接口，**禁止用 lambda**，必须 `new CacheDataLoader<K,V>(){...}`
- **判断存在用 `containsKey`**，没有 `getIfPresent` / `exists`；查本地缓存条目数用 `localCacheSize`（不是 `size`）
- **全量清空**：`FusionCache.invalidate(Class, null)`（key 传 null）或 `GlobalCache.invalidate(cacheName, null)`，按前缀批量失效用 `invalidatePrefix` / `invalidateKeys`（内部走 SCAN，注意大 keyset 阻塞）

**FusionCache vs GlobalCache 选型**：

| 维度 | FusionCache | GlobalCache |
|------|------------|-------------|
| 存储 | 本地 + Redis 融合 | 纯 Redis |
| 适用 | 单条实体详情（高频读取） | 列表、临时数据 |
| 初始化 | 必须在 `static {}` 块 | 行内 CacheDataLoader |
| 失效 | `FusionCache.invalidate(Class, key)` | `GlobalCache.invalidate(cacheName, key)` |
| 是否占 JVM 内存 | 是（Caffeine） | 否 |

**其他缓存组件选型**：

| 组件 | 用途 | 关键约束 |
|------|------|---------|
| `GlobalLocker` | 分布式锁 | stamp > 0 才持有锁；`finally` 中 `unlock`；长任务必须 `keepLock` 续期（执行时间不可超过 lockTimeMillis）；unlock/keepLock 已是 Lua CAS 原子操作 |
| `FusionCounter` | 高性能计数器（本地+Redis融合） | 必须 `static {}` config；本地与 Redis 有 syncGlobalMillis 级延迟；强一致读用 `get(Class, id, true)` |
| `GlobalCounter` | 纯 Redis 计数器 | increment/decrement/get/set/delete |
| `GlobalHashSet` | 去重/随机抽取 | `contains` 会序列化 item 后比对，序列化协议与 add/remove 一致 |
| `GlobalSortedSet` | 延迟任务/定时触发 | score 通常为时间戳，按范围查询和删除 |

> 完整缓存组件方法签名与示例见 [uw-cache.md](uw-cache.md)。

## 响应包裹与异常处理规范

- **`GlobalResponseAdvice` 自动包裹**：所有 Controller 返回值自动包裹为 `ResponseData`，开发者无需手动包装（返回 null → `warn()`；返回业务对象 → `success(body)`；已是 ResponseData → 直接透传）。**正常情况下 Controller 直接 `return` 业务对象或 `ResponseData<T>` 即可**。
- **`GlobalExceptionAdvice` 自动处理异常**：所有未捕获异常自动转为 ResponseData 错误响应（TokenInvalidException→401、TokenPermException→403、TokenPayException→402、TokenExpiredException→498、TokenServiceException→503、TokenSudoException→426、其他→500），**Controller 无需 try-catch**。
- **跳过包裹**：文件下载等场景在类或方法上加 `@ResponseAdviceIgnore`（`uw.auth.service.annotation.ResponseAdviceIgnore`）。
- **权限匹配不支持路径变量**：`@MscPermDeclare` 的 PERM/SUDO 校验基于精确 URI + 请求方法，**不支持 `{id}` 等路径变量**。带路径变量的接口用 `auth = AuthType.NONE/USER`，或拆分为固定路径。
- **`user()` 为单值**：必须精确匹配一种 `UserType`，不支持数组。多类型访问需拆分接口。

## 任务框架规范

定时任务（`TaskCroner`）和队列任务（`TaskRunner`）由 `uw-task` 提供，配合服务端 `uw-task-center` 实现动态配置、状态上报与告警。

> **关键区别**：`TaskCroner` / `TaskRunner` 是 **Spring Bean**（必须加 `@Component`，由框架管理生命周期），**与 Helper（纯静态工具类，禁止 `@Component`）不同**。任务类放在 `task/` 包下，按 `task-project` 包名前缀扫描。

| 任务类型 | 基类 | 触发方式 | 多实例区分 |
|---------|------|---------|-----------|
| 定时任务 | `TaskCroner` | cron 表达式 | `taskParam` |
| 队列任务 | `TaskRunner<TP, RD>` | MQ 队列消费 | `taskTag` |

**TaskRunner 是单例**：多消费者线程并发调用同一实例的 `runTask`，**勿使用非线程安全的实例变量**。

**TaskFactory 获取**：`@Autowired private TaskFactory taskFactory;` 注入，或 `TaskFactory.getInstance()` 静态获取。

**run 系列方法选型**：

| 方法 | 执行位置 | 阻塞调用方 | 返回结果 | 本机无 runner | 典型用途 |
|------|---------|-----------|---------|--------------|---------|
| `sendToQueue` | 远程（入队） | 否 | 无 | 入队（正常） | 标准异步入队，不关心结果 |
| `runQueue` | 本地优先，否则入队 | 否 | 无 | 入队 | 高频短任务优化，省 MQ |
| `runTask` | AUTO 自动选 | 是 | 有 | 回退远程 RPC | 需立即同步拿结果 |
| `runTaskLocal` | 仅本地 | 是 | 有 | **抛异常** | 强制本机执行 |
| `runTaskAsync` | AUTO 自动选 | 否（Future） | 有（get 时） | 回退远程 RPC | 异步拿结果 |

> **禁止复用 taskData**：`runTask` / `runTaskLocal` / `runTaskAsync` / `runQueue` 会向传入的 taskData 写入 id/queueDate/runType 等运行期字段，每次必须新建。远程 RPC 默认超时 180 秒，避免在 Web 请求线程高频同步调用。

**重试与异常处理**：

| 异常类 | 状态 | 触发重试 | 使用场景 |
|--------|------|---------|---------|
| `TaskPartnerException` | STATE_FAIL_PARTNER | ✅ 按 `retryTimesByPartner` | 第三方接口限流、网络异常 |
| （框架自动） | STATE_FAIL_CONFIG | ✅ 按 `retryTimesByOverrated` | 超过流量限制 |
| `TaskDataException` | STATE_FAIL_DATA | ❌ 不重试 | 参数错误、数据格式错误 |
| 其他未捕获异常 | STATE_FAIL_PROGRAM | ❌ 不重试 | 程序 bug |

> **没有 `retryTimesByProgram`**，程序异常不重试。`retryTimes = N` 时总计执行 N+1 次（1 次初始 + N 次重试）。

> 完整任务类模板（TaskCroner/TaskRunner/TaskData）见 [code-templates.md](code-templates.md)「任务框架模板」，API 细节见 [uw-task.md](uw-task.md)。

## AI 集成规范

`uw-ai` 模块封装与 AI 服务中心（`uw-ai-center`）的交互，统一入口为静态门面 `uw.ai.AiClientHelper`。涵盖同步/流式对话、结构化输出、批量翻译、图片生成、模型/API 配置查询，以及 AI 工具扩展。

**核心原则**：

| 原则 | 说明 |
|------|------|
| **统一入口** | 一律通过 `AiClientHelper` 静态方法调用，不绕过门面直接 RPC |
| **configCode 优先** | `configId` 与 `configCode` 二选一定位配置，**优先用 `configCode`**（语义化、跨环境）；禁止业务代码硬编码 configId，确需运行时决定则从 AiConfig 表动态读取 |
| **bindAuthInfo 强制** | 认证信息（saasId/userId/userType/userInfo）通过 `param.bindAuthInfo()` 自动绑定当前登录用户，**禁止手动 `setSaasId`** |
| **结果必检查** | `generate`/`generateEntity` 等返回 `ResponseData`，**必须检查 `isNotSuccess()` 并写降级逻辑**，禁止 `.getData()` 后直接使用 |
| **真实调用不可降级为 DB 查询** | Javadoc 步骤标注 `[调用AI]` 的，必须调用 `AiClientHelper`；外部服务不可用时降级是"写降级逻辑"，不是省略或退化为数据库查询 |
| **流式场景用流式 API** | 对话/SSE 场景用 `chatGenerate()` 返回 `Flux<String>`，禁止用同步 `generate()` 处理流式需求 |

### 调用方式选型

| 场景 | 方法 | 返回类型 | 关键约束 |
|------|------|---------|---------|
| 同步对话 | `generate(param)` | `ResponseData<String>` | 需要完整结果时使用 |
| 流式对话（SSE） | `chatGenerate(param)` | `Flux<String>` | 对话场景默认用此，前端逐字渲染 |
| 结构化输出 | `generateEntity(param, Class<T>)` | `ResponseData<T>` | 系统提示会自动追加 JSON Schema 说明，返回值按 Class 反序列化 |
| 批量翻译（数组） | `translateList(param)` | `ResponseData<AiTranslateResultData[]>` | 每个目标语言一条结果，**返回是数组**不是单 Map |
| Map 翻译 | `translateMap(param)` | `ResponseData<AiTranslateResultData[]>` | key 保留、value 翻译；`textMap` 必须用 `LinkedHashMap` |
| 图片生成 | `generateImage(param)` | `ResponseData<AiImageResultData>` | `userPrompt` 必填（Builder 用 `.prompt(...)`） |
| 查询模型配置 | `listModelInfoByCode(configCode)` 等 | `ResponseData<AiModelConfigVo>` | modelType 取值：CHAT/EMBEDDING/RERANK/TTS/OCR |

### 调用流程模板

```
1. 构建 Param（Builder 链式）
   ├─ configCode("xxx")        ← 优先 configCode，不硬编码 configId
   ├─ systemPrompt(...)         ← 可选，结构化输出/翻译按需
   ├─ userPrompt(...) / textList / textMap / prompt
   └─ bindAuthInfo()            ← 最后调用，自动填充认证四元组
2. 调用 AiClientHelper.xxx(param)
3. 检查 ResponseData.isNotSuccess() → 降级处理
4. 取 .getData() 使用
```

### AiTool 扩展规范

自定义 AI 工具须实现 `AiTool<P, R>` 接口 + `@Component`：P 继承 `AiToolParam`（内置认证四元组），R 通常为 `ResponseData<T>`。

| 约束 | 说明 |
|------|------|
| **必须是 Spring Bean** | 加 `@Component`，由 `UwAiAutoConfiguration` 启动时扫描注册（与 Helper 的禁止 @Component 相反） |
| **toolVersion 升级递增** | 框架按 `toolClass + toolVersion` 比对同步元数据，功能变更必须递增 version |
| **toolCode = 类名** | `AiToolCallInfo` 的 toolCode 对应工具类**类名**（`Class.getSimpleName()`），不是 `toolName()` / `toolVersion` |
| **参数 Schema** | 内部 Param 类用 `@Schema` 标注（遵循 title+description 规范），框架据此生成输入/输出 JSON Schema |
| **工具执行入口** | 服务中心回调 `POST /rpc/ai/tool/execute`，以 `UserType.RPC` 身份执行，工具类无需自行暴露接口 |

### 降级与可靠性

- **降级原则**：AI 服务不可用（`isNotSuccess()`）时返回提示信息/默认值，**不是删除调用步骤**
- **批量翻译防丢**：`translateList`/`translateMap` 返回数组，取结果时按目标语言索引（`resp.getData()[0]`），注意判空
- **configId 动态化**：确需运行时决定配置时，从 AiConfig 表查询：`dao.queryForObject(AiConfig.class, ...)`

> 完整 AI 调用代码模板（同步/流式/结构化/降级 + AiTool）见 [code-templates.md](code-templates.md)「AI 集成模板」，API 细节见 [uw-ai.md](uw-ai.md)。

## 认证授权规范

- 内部 RPC 调用使用 `@Qualifier("authRestClient")` 注入带鉴权的 RestClient
- Token 自动管理（自动 login/refresh/重试），无需手动设置 Authorization 请求头
- 收到 401 或 498 时自动刷新 Token 并重试一次

## 单元测试规范

> TDD 通用方法论（Red-Green-Refactor、AI 原生循环）见 [tdd-guide.md](../../0-init/references/tdd-guide.md)。本节仅描述 210 专属的测试策略和约束。

### 测试原则

| 原则 | 说明 |
|------|------|
| **全链路测试** | 使用 `@SpringBootTest` 启动完整 Spring 上下文，测试真实数据库交互 |
| **单 Context** | 所有测试继承 `BaseIntegrationTest`，共享同一个 Spring Context，只启动一次 |
| **数据隔离** | 使用测试数据前缀（`TEST_` + 时间戳）+ `@AfterEach` 手动清理，不用事务回滚 |
| **测试用户** | 使用 `TestAuthUtils.setTestUser()` 设置测试用户（saasId=mchId=userId=666） |
| **真实依赖** | DaoManager/FusionCache/GlobalLocker 均使用真实实例，测试真实数据库读写 |
| **禁止 Mock** | 禁止 MockMvc/TestRestTemplate/Mockito，所有测试使用真实数据库交互 |

### 测试分层

| 层级 | 测试目标 | 技术方案 |
|------|---------|---------|
| Helper 测试 | 业务逻辑 + 数据访问 | `@SpringBootTest` + 真实数据库，继承 `BaseIntegrationTest` |
| Controller 测试 | API 契约 + SQL 正确性 | `@SpringBootTest` + 直接注入 Controller Bean + 反射注入 AuthTokenData，继承 `{role}ControllerTest` |

> **Controller 测试架构**：**禁止 MockMvc 和 TestRestTemplate**。`AuthServiceFilter` 拦截所有 HTTP 请求要求 RPC Token 验证，测试环境无法提供。正确方式是直接注入 Controller Bean + 通过反射 `AuthServiceHelper.setContextToken()` 注入 AuthTokenData。详见 [code-templates.md](code-templates.md)「单元测试模板」。

### 按方法类型测试用例数

| 方法类型 | 基础用例 | 边界用例 | 异常用例 | 合计 |
|---------|---------|---------|---------|------|
| listXxx | 正常分页查询 1 | 空结果 1 | - | 2 |
| getXxx | 正常查询 1 | 不存在ID 1 | - | 2 |
| saveXxx | 正常新增 1 | 唯一性冲突 1 | 必填字段缺失 1 | 3 |
| updateXxx | 正常修改 1 | 不存在ID 1 | 状态不允许修改 1 | 3 |
| deleteXxx | 正常删除 1 | 不存在ID 1 | 关联数据阻止删除 1 | 3 |
| enableXxx / disableXxx | 正常操作 1 | 不存在ID 1 | 重复操作 1 | 3 |
| 业务流程方法 | 正常流程 1 | 每个分支 1 | 每个异常 1 | 3-5 |

### 测试命名规范

| 规则 | 格式 | 示例 |
|------|------|------|
| 测试类 | `{Class}Test` | `ProductHelperTest` |
| 测试方法 | `test{Method}_{Scenario}_{ExpectedResult}` | `testSaveProduct_NameDuplicate_ThrowException` |
| 测试方法（简化） | `test{Method}_{ExpectedResult}` | `testGetById_NotFound_ReturnWarn` |

## AI Coding 禁忌清单（全局生效）

> 以下陷阱来自各模块文档汇总，AI 编码时**必须逐条检查**，违反即为 Bug。

### 禁止重复造轮子（优先使用框架工具类）

> AI 编码时**必须优先使用** `uw-common` 和 `uw-common-app` 提供的工具类，**禁止自行实现**相同功能。以下场景已有现成方案：

| 禁止自己写 | 必须用 | 所在包 |
|-----------|--------|--------|
| `System.currentTimeMillis()` | `SystemClock.now()` / `SystemClock.nowDate()` | `uw.common.util.SystemClock` |
| `new SimpleDateFormat(...)` | `DateUtils.dateToString()` / `DateUtils.stringToDate()` | `uw.common.util.DateUtils` |
| `new ObjectMapper()` / 手写 JSON 序列化 | `JsonUtils.toString()` / `JsonUtils.parse()` | `uw.common.util.JsonUtils` |
| 手写 MD5/SHA256 | `DigestUtils.signHex(msg, Algorithm.SHA_256)` — 无独立 md5/sha256 方法 | `uw.common.util.DigestUtils` |
| 手写 AES 加密 | `AESUtils.encryptString()` / `BizAESBox` | `uw.common.util` |
| 手写金额计算（double/BigDecimal） | `MoneyUtils`（long 分单位） | `uw.common.util.MoneyUtils` |
| 手写邮箱/手机号/身份证正则 | `ValidateUtils.isEmail()` / `isChinaMobile()` 等 | `uw.common.util.ValidateUtils` |
| 手写 IP 匹配/CIDR | `IpMatchUtils.sortList()` 后 `matches()` | `uw.common.util.IpMatchUtils` |
| `Math.random()` / `Random` 生成 ID | `SnowflakeIdGenerator.getInstance().generateId()` | `uw.common.util.SnowflakeIdGenerator` |
| 手写位运算开关 | `BitConfigUtils.isOn()` / `on()` / `off()` | `uw.common.util.BitConfigUtils` |
| 硬编码状态值 0/1/-1 | `CommonState.ENABLED/DISABLED/DELETED` | `uw.common.app.constant.CommonState` |
| 硬编码错误消息字符串 | `ResponseData.warnCode(CommonResponseCode.XXX)` | `uw.common.app.constant.CommonResponseCode` |
| 手写 `@Schema` 校验逻辑 | `SchemaValidateHelper.validate(entity)` | `uw.common.app.helper.SchemaValidateHelper` |
| 手写 URL 查询参数拼接 | `QueryParamHelper.buildUriWithParams(url, param)` | `uw.common.app.helper.QueryParamHelper` |
| 手写数据变更历史记录 | `SysDataHistoryHelper.saveHistory(entity, "操作")` | `uw.common.app.helper.SysDataHistoryHelper` |
| 手写 JSON 配置管理 | `JsonConfigHelper` / `JsonConfigBox` | `uw.common.app.helper` / `uw.common.app.vo` |
| `UUID.randomUUID()` 做业务 ID | `dao.getSequenceId(Class)` — 分布式序列 | `uw.dao.DaoManager` |
| 手写日志记录（Web 场景） | `AuthServiceHelper.logRef()` / `logInfo()` | `uw.auth.service.AuthServiceHelper` |
| 手写系统日志（非 Web 场景） | `AuthServiceHelper.logSysInfo(...)` | `uw.auth.service.AuthServiceHelper` |
| 手写数据脱敏 | `MaskUtils.maskChinaMobile()` / `maskChinaIdCard()` / `maskSecret()` 等 | `uw.common.util.MaskUtils` |
| 手写 HmacSha256 | `HmacUtils.sign()` / `verify()` | `uw.common.util.HmacUtils` |

### ResponseData 模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `ResponseData.warn("CODE","msg")` | `ResponseData.warnCode("CODE","msg")` 或 `warnCode(ResponseCode)` — 实际签名是 `warn(T t, String code)`，"CODE"变成data而非状态码 |
| `ResponseData.error("CODE","msg")` | `ResponseData.errorCode("CODE","msg")` 或 `errorCode(ResponseCode)` — 同 warn 泛型陷阱 |
| `isNotSuccess() \|\| getData()==null` | `if (result.isNotSuccess())` — getData()==null 不会额外为 true |

**ResponseData 状态选择决策**：

| 场景 | 使用 | 示例 |
|------|------|------|
| 操作成功（有数据） | `success(data)` | `ResponseData.success(entity)` |
| 操作成功（无数据） | `success()` | `ResponseData.success()` |
| 业务校验失败 | `warnCode()` | `ResponseData.warnCode(CommonResponseCode.ENTITY_NOT_FOUND_ERROR)` |
| 系统异常 | `errorCode()` | `ResponseData.errorCode(CommonResponseCode.ENTITY_SAVE_ERROR)` |
| 致命错误 | `fatalCode()` | `ResponseData.fatalCode(...)` |
| 数据不存在（dao.load/list 查无结果） | 框架自动返回 WARN | 检查 `result.getData() != null` 而非 `isSuccess()` |

### DaoManager 模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `dao.executeCommand(sql, params)` | `dao.execute(sql, params)` — 方法已重命名 |
| `dao.list("WHERE ...")` | `dao.list(Class, "SELECT * FROM table WHERE ...")` — SQL 必须完整 |
| `DataList.isEmpty()` | `data.size() == 0` — PageList 没有 isEmpty() |
| `dao.load(SaaSEntity, id)` 无 saasId | `dao.queryForObject(Class, new AuthIdQueryParam(saasId, id))` |
| `dao.list + LIMIT 1 + get(0)` | `dao.queryForObject(Class, "SELECT * FROM t WHERE ... LIMIT 1", params)` |
| `dao.list("SELECT *") + size()` 计数 | `dao.queryForValue(Long.class, "SELECT COUNT(*) FROM t WHERE ...", params)` |
| `dao.delete(Class, id)` | 先 `dao.load`/`queryForObject` 获取实体，再 `dao.delete(entity)` |
| `dao.update(entity)` 不设 modifyDate | 必须 `.modifyDate(SystemClock.nowDate())` |
| `dao.save(entity)` 不设 ID | 必须 `entity.setId(dao.getSequenceId(Entity.class))` |
| `dao.update` 部分字段不设 modifyDate | `new Entity().id(id).field(value)` 只更新非 null 字段，必须同时 `.modifyDate()` |
| `modifyDate` 在 `dao.save()` 时必填 | `dao.save()` 时可设 `modifyDate(null)` 或不设（框架不强制）；`dao.update()` 时必须设 |
| `if (result.isSuccess()) { ... } else { ... }` | `result.onSuccess(...)` / `.onNotSuccess(...)` — 禁止 if-else 判断 ResponseData 状态 |
| `var result = dao.load(...); return result;` | `return dao.load(...);` — 零中间变量直接返回 |
| `result.getData() != null` 判断成功 | `result.isSuccess()` 或直接用 `onSuccess` / `onNotSuccess` 链式处理 |
| `dao.beginBatchUpdate()` 不开事务 | **批量更新必须先 `beginTransaction()`**，否则 addBatch 数据被 autoCommit 立即提交，框架抛 TransactionException |
| 批量提交后不复位 batchSize | `submit()` 后 batchSize 重置为默认 100，连续批量提交需重新 `setBatchSize` |
| MySQL 批量无 `rewriteBatchedStatements=true` | 连接 URL 须加 `rewriteBatchedStatements=true` 才能真正合并批处理 |
| `dao.queryForValue` 用基本类型接收 | 查标量值用包装类型（`Long.class`），NULL 返回 null；禁止 list+size() 计数 |
| `@QueryMeta(expr="id in (?)")` IN 单值 | IN 查询占位符对 List/数组自动展开，单值也用 List/数组 |
| `GlobalResponseAdvice` 场景手动包裹 | Controller 直接 `return` 业务对象/`ResponseData<T>`，框架自动包裹；仅文件下载等用 `@ResponseAdviceIgnore` |
| Controller 内 try-catch 异常 | `GlobalExceptionAdvice` 自动转为错误响应，Controller 无需 try-catch |
| `@MscPermDeclare(user = {A, B})` 多类型 | `user()` 为单值，多类型访问需拆分接口 |
| PERM 校验带 `{id}` 路径变量 | 权限匹配不支持路径变量，带变量用 `AuthType.NONE/USER` 或拆固定路径 |

### Cache 模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `GlobalCache.delete()` | `GlobalCache.invalidate(cacheName, key)` — 方法名是 invalidate |
| `CacheDataLoader` 用 lambda | `new CacheDataLoader<K,V>(){@Override public V load(K key){...}}` — 是抽象类 |
| `GlobalCache.getIfPresent()` | `GlobalCache.get(cacheName, key, CacheDataLoader, expireMillis)` / `containsKey()` — 没有 getIfPresent |
| `FusionCache.size()` | `FusionCache.localCacheSize(Class)` — 没有 size 方法 |
| `FusionCache.invalidateAll()` | 逐个 `FusionCache.invalidate(Class, key)`；全量清空用 `invalidate(Class, null)` — 没有 invalidateAll |
| `FusionCache` 缓存 `DataList` | 缓存 `ArrayList<Entity>` — DataList 序列化异常 |
| FusionCache/GlobalCache 选错 | 单条实体详情用 FusionCache；列表、临时数据用 GlobalCache |
| `CacheDataLoader<V>` V 用接口类型 | V 必须是**具体实现类**（Kryo 反序列化要求），如 `ArrayList<Entity>` 而非 `List<Entity>` |
| `GlobalLocker` 不续期长任务 | 执行超过 lockTimeMillis 会自动过期被抢占；长任务必须 `keepLock` 续期 |
| `GlobalLocker` 自行加锁后 CAS | unlock/keepLock 已是 Lua CAS 原子操作（stamp 为 SnowflakeId 跨 JVM 唯一），无需自行加锁 |

### 任务框架模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `TaskCroner` / `TaskRunner` 不加 `@Component` | 必须加 `@Component`（是 Spring Bean，与 Helper 不同） |
| `TaskRunner` 使用非线程安全实例变量 | TaskRunner 是单例，多消费者线程并发调用，禁止非线程安全实例变量 |
| 复用同一 `taskData` 对象多次提交 | run 系列方法会写入运行期字段（id/queueDate/runType），每次必须新建 |
| `retryTimesByProgram` 配置 | **没有 `retryTimesByProgram`**，程序异常（STATE_FAIL_PROGRAM）不重试 |
| 第三方错误抛普通异常 | 抛 `TaskPartnerException` 才按 `retryTimesByPartner` 重试 |
| 数据错误抛普通异常 | 抛 `TaskDataException` 表示不重试 |
| `runTask` 在 Web 请求线程高频同步调用 | 远程 RPC 默认超时 180 秒，高频同步调用会耗尽 Tomcat 线程 |
| `runTaskLocal` 本机无 runner | 本机无 runner 时 `runTaskLocal` **抛异常**（不降级入队） |

### AI 集成模块

> 详细规范见上方「AI 集成规范」。跨模块编排（通知推送、跨 Helper 调用）与降级总原则见下方「外部集成模块」。

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| AI 调用退化为数据库查询 | `AiClientHelper.generate(AiChatGenerateParam)` — 标注 `[调用AI]` 的步骤必须真实调用 |
| `configId` 硬编码 | 优先 `configCode`（语义化、跨环境）；确需运行时决定则从 AiConfig 表读取 |
| `AiChatGenerateParam` 手动 `setSaasId` | `.bindAuthInfo()` 自动绑定认证四元组，禁止手动 set |
| `AiClientHelper.generate()` 不检查结果 | `if (aiResult.isNotSuccess()) { 降级处理 }`，禁止 `.getData()` 后直接用 |
| 流式对话用同步方式 | 对话/SSE 场景用 `chatGenerate(param)` 返回 `Flux<String>` |
| `translateList`/`translateMap` 当单 Map 用 | 返回**数组**（每目标语言一条），按语言索引取值并判空；`textMap` 用 `LinkedHashMap` |
| `generateImage` 用 `.userPrompt(...)` | Builder 用 `.prompt(...)` 设置图片提示词 |
| `AiTool` 不加 `@Component` | AI 工具类必须是 Spring Bean 并实现 `AiTool<P,R>` |
| `toolCode` 写 `toolName()` | `toolCode` 对应工具类的**类名**（`Class.getSimpleName()`），不是 toolName/toolVersion |
| `AiTool.toolVersion()` 不递增 | 升级时必须递增 version，框架据此判断是否需同步元数据 |

### Helper/通用模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| Helper 加 `@Component` / `@Service` | Helper 是纯静态工具类，禁止 Spring 注解，用 `DaoManager.getInstance()` 静态获取 |
| Helper 层使用 `AuthQueryParam` | AuthQueryParam 依赖 Spring Security 上下文，Helper 层用 `dao.list(Entity, sql, params)` |

### 枚举模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `entity.setState(0/1)` 硬编码 | `CommonState.ENABLED/DISABLED.getValue()` |
| `warnCode("USER_NOT_FOUND", "用户不存在")` 硬编码 | `warnCode(GuestResponseCode.USER_NOT_FOUND)` — 定义 ResponseCode 枚举 |
| 枚举类散落在 entity/service 包 | 统一放在 `{package}/constant/` 包下 |

### @Schema/DateUtils/DigestUtils 模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| `@Schema(description="xxx")` 缺 title | `@Schema(title="xxx", description="xxx")` — 必须同时设置 |
| `DateUtils.getDayStart/getDayEnd/addDays` | `DateUtils.beginOfToday(date)` / `endOfToday(date)` / `offsetDay(date, n)` |
| `DateUtils.format(date, fmt)` / `parse(str)` | `DateUtils.dateToString(date, fmt)` / `stringToDate(str)` — 方法名不同 |
| `DigestUtils.md5()` / `sha256()` | `DigestUtils.signHex(msg, DigestUtils.Algorithm.SHA_256)` — 无独立方法，用 Algorithm 枚举 |
| `new Date()` / `System.currentTimeMillis()` | `SystemClock.nowDate()` / `SystemClock.now()` — createDate/modifyDate 赋值必用 |
| 自写 IP 匹配/CIDR | `IpMatchUtils.sortList(ipList)` 后 `matches(sortedList, ip)` — 必须先 sortList 再二分查找 |
| 手写脱敏方法 `desensitize/hide` | `MaskUtils.maskXxx()` — 所有方法统一 mask 前缀，无 desensitize/hide |

### QueryParam 模块

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| 自定义 QueryParam 重新声明 `saasId/mchId/userId/userType` | 继承 `AuthQueryParam`/`AuthPageQueryParam` 后**禁止重新声明**这四个字段及其 @QueryMeta/getter/setter — 否则 WHERE 条件被注入两次且反射读到错误对象 |
| 新增权限维度用 `groupId`（已被占用） | 使用父类未占用的字段名；如需 `groupId`，确认父类未定义 |
| `QueryParam` 不限定可排序字段 | 覆写 `ALLOWED_SORT_PROPERTY()` 返回 `属性名→列名` 映射，防排序注入 |
| `@QueryMeta(expr="id in (?)")` 传单个值 | IN 查询传 List/数组自动展开占位符 |

### 外部集成模块

> Helper 的 Javadoc 步骤中标注了 `[调用AI]`、`[推送通知]`、`[外部调用]`、`[跨模块调用]`、`[降级处理]` 的步骤，**必须使用对应 SDK 实现，禁止替换为数据库操作或省略**。

| ❌ 错误写法 | ✅ 正确写法 |
|------------|-----------|
| 通知推送退化为数据库 INSERT | `dao.save(msgNotice)` 后必须调用 `NotifyClientHelper.pushNotify(new WebNotifyMsg(...))` |
| 跨模块调用被省略 | 如 `acceptAnswer()` 必须：PostQuestionHelper.resolveQuestion() + GuestPointHelper.earnPoint() + MsgNotifyHelper.sendNotice() |

> AI 调用的专属陷阱（退化/降级/bindAuthInfo/configCode/流式/工具扩展）见上方「AI 集成模块」。

**降级原则**：外部服务不可用时的降级是写降级逻辑（如返回提示信息），不是直接删除该步骤。

## 四条编码原则

> 被 210-java-uniweb-dev、211-java-uniweb-dev-review、620/621/720/721 共同引用。
> 本文件是编码规范的**唯一权威来源**，其他技能文件不再重复列举规则。

### 原则一：集中管理（Single Source of Truth）

一个配置在多个地方可能被使用，或属于字典/枚举/映射类数据，应集中管理。

| 集中到哪里 | 管什么 | 详见 |
|-----------|--------|------|
| `constant/` | 业务枚举（状态/类型/响应码） | 详见「枚举与响应码规范」 |
| `service/` | 业务逻辑（Helper） | 详见「Helper 设计规范」 |
| `vo/` | 视图对象（裁剪/聚合输出） | 详见「Vo/Ex 规范」 |
| `dto/` | 数据传输对象（代码生成器产出，仅裁剪） | 详见「代码生成与开发流程」 |

### 原则二：类型安全（No Escape Hatches）

如果 Java 编译器无法推断类型，说明代码有问题。

| 禁止 | 替代方案 | 详见 |
|------|---------|------|
| `ResponseData.warn/error("CODE","msg")` | `warnCode`/`errorCode` | 详见「AI Coding 禁忌清单」 |
| Lombok（`@Data`/`@Getter`/`@Setter`） | 手写 getter/setter/构造器 | 详见「Helper 设计规范」 |
| 方法参数使用包装类型 | 使用基本类型 | 详见「Controller 规范」 |
| `@Schema` 只设 `description` | 必须同时设置 `title` 和 `description` | 详见「Controller 规范」 |

### 原则三：项目一致性（Use What Exists）

编写任何代码前，先检查项目中是否已有相同或类似功能的实现。

| 场景 | 做法 | 详见 |
|------|------|------|
| 数据访问 / 分页查询 | DaoManager + AuthQueryParam | 详见「DAO 数据访问规范」 |
| 缓存 | FusionCache（实体）/ GlobalCache（列表） | 详见「缓存使用规范」 |
| 权限声明 / 响应格式 | @MscPermDeclare + ResponseData\<T\> | 详见「Controller 规范」 |
| Helper 依赖获取 / 方法风格 | 静态获取 + public static | 详见「Helper 设计规范」 |
| 枚举 / 响应码 | CommonState + ResponseCode | 详见「枚举与响应码规范」 |
| 定时/队列任务 | TaskCroner / TaskRunner + TaskFactory | 详见「任务框架规范」 |
| AI 对话/翻译/工具 | AiClientHelper + AiTool | 详见「AI 集成规范」 |

### 原则四：代码可读性（Self-Documenting Code）

一个新团队成员能否在不看注释的情况下理解代码意图。

| 禁止 | 替代方案 | 详见 |
|------|---------|------|
| 硬编码数字 `setState(0)` | `setState(CommonState.DISABLED.getValue())` | 详见「枚举与响应码规范」 |
| 硬编码字符串 `warnCode("CODE","msg")` | `warnCode(ResponseCode.XXX)` | 详见「枚举与响应码规范」 |
| 方法超过 50 行 / 圈复杂度超过 10 | 拆分子方法 / 简化条件 | — |
| 缺少 `@Schema` 注解 | 所有 Entity/Dto/Vo 有完整 @Schema | 详见「Controller 规范」 |

## 自动化验证

开发完成后，在 `backend/{project-name}-app/` 目录下依次执行以下检查：

```bash
# 1. Lombok 检查（应为 0）
grep -rn '@Data\|@Getter\|@Setter\|@RequiredArgsConstructor' src/main/java/ --include="*.java" | wc -l

# 2. 硬编码状态值检查（应为 0）
grep -rn 'setState(0)\|setState(1)\|setState(-1)' src/main/java/ --include="*.java" | wc -l

# 3. 硬编码响应码检查（应为 0）
grep -rn 'warnCode("\|errorCode("' src/main/java/ --include="*.java" | wc -l

# 4. ResponseData 泛型陷阱检查（应为 0）
grep -rn 'ResponseData\.warn("\|ResponseData\.error("' src/main/java/ --include="*.java" | wc -l

# 5. DAO 方法名错误检查（应为 0）
grep -rn 'dao\.executeCommand(\|GlobalCache\.delete(\|FusionCache\.invalidateAll(' src/main/java/ --include="*.java" | wc -l

# 6. Controller Mapping 路径命名检查（应为 0）
grep -rn '@.*Mapping.*".*[-_].*"' src/main/java/ --include="*.java" | wc -l

# 7. TODO 残留检查（应为 0）
grep -rn '// TODO:' src/main/java/ --include="*.java" | wc -l

# 8. @Schema 完整性检查（缺少 title 的行应为 0）
grep -rn '@Schema(description' src/main/java/ --include="*.java" | grep -v 'title' | wc -l

# 9. ResponseCode i18n 资源文件检查（每个 ResponseCode 枚举必须配套 ≥3 个语种文件）
for enum_file in $(grep -rln 'implements ResponseCode' src/main/java/ --include="*.java"); do
    enum_class=$(echo "$enum_file" | sed 's|src/main/java/||;s|\.java||;s|/|.|g')
    resource_dir="src/main/resources/$(echo $enum_class | tr '.' '/')"
    if [ ! -d "$resource_dir" ]; then
        echo "MISSING i18n: $resource_dir (for $enum_class)"
    else
        file_count=$(ls "$resource_dir"/messages*.properties 2>/dev/null | wc -l)
        if [ "$file_count" -lt 3 ]; then
            echo "INCOMPLETE i18n: $resource_dir has $file_count files (need ≥3)"
        fi
    fi
done

# 10. 任务类 Spring Bean 检查（TaskCroner/TaskRunner 必须加 @Component）
for task_file in $(grep -rln 'extends TaskCroner\|extends TaskRunner' src/main/java/ --include="*.java"); do
    if ! grep -q '@Component' "$task_file"; then
        echo "MISSING @Component: $task_file"
    fi
done

# 11. Helper 误加 Spring 注解检查（Helper 禁止 @Component/@Service）
for helper_file in $(find src/main/java -name '*Helper.java' -path '*/service/*'); do
    if grep -qE '@Component|@Service|@Autowired' "$helper_file"; then
        echo "HELPER WITH SPRING ANNOTATION: $helper_file"
    fi
done

# 12. 编译 + 全量测试
mvn compile && mvn test -Dspring.profiles.active=debug
```
