---
name: 210-java-uniweb-design
description: UniWeb后端技术设计（TDD驱动, 设计即代码）。当需要基于PRD和数据库设计进行后端技术设计时触发：(1)确认模块划分与定制API, (2)编写项目README.md总体设计（含测试策略）, (3)裁剪DTO和Controller, (4)新建Helper方法签名与缓存定义, (5)编写Helper单元测试骨架（TDD Red阶段）, (6)确保编译通过且测试可运行。当用户提及后端设计、接口设计、技术设计、生成脚手架时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# UniWeb后端技术设计（TDD 驱动，设计即代码）

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 核心理念

| 传统方式 | 本技能方式 |
|---------|-----------|
| 独立设计文档 → 脚手架生成 → 开发 | README.md + 代码Javadoc → 直接开发 |
| 文档与代码可能不一致 | Javadoc即代码，天然一致 |
| 按设计维度组织文档 | 按模块组织，AI开发时一个模块一个文件 |
| 设计和脚手架两个阶段 | 合并为一个阶段 |
| 测试留到开发阶段 | 设计阶段产出测试骨架（TDD Red），开发从 Green 开始 |

## 技术栈来源

| 技术栈 | 文档入口 | 用途 |
|--------|----------|------|
| UniWeb 基础框架 | [backend/uniweb/README.md](references/backend/uniweb/README.md) | uw-base类库、微服务、开发规范 |
| SaaS 业务框架 | [backend/saas/README.md](references/backend/saas/README.md) | AIP/AIS、saas-base、saas-finance |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 技术设计 + 单元测试骨架 | `system-architect` |
| 协作 | 业务需求 | `product-manager` |

> **职责边界**：单元测试（Helper 单元测试骨架 + 后续实现）由架构师/开发工程师负责。测试工程师不参与此阶段，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `PROJECT_ROOT/requirement/prds/*` | 产品需求文档，功能模块及各终端详细需求 |
| 数据库设计文档 | `PROJECT_ROOT/database/database-design.md` | 表结构设计、实体关系、索引策略 |
| 项目目录 | `PROJECT_ROOT/backend/{项目名}-app/` | 210阶段项目代码以及产出的entity/dto/controller |

## 前置条件

| 前置技能 | 说明 |
|---------|------|
| 200-database-design | 数据库设计已完成并通过评审 |
| 210-java-uniweb-init | 项目已通过模板初始化 |
| 210-java-uniweb-gencode | entity/dto/controller 全量curd功能已由代码生成器生成在admin角色下，需定制裁剪 |

**已生成代码**：代码生成器已产出 entity 实体类、dto 数据传输对象和标准 CRUD Controller，设计阶段在此基础上裁剪和补充。

## 架构约定

| 约定 | 说明 |
|------|------|
| Controller 路径第一级 | 必须为用户角色（`/saas/`、`/mch/`、`/admin/`、`/root/`、`/ops/`、`/rpc/`） |
| Controller 初始位置 | 代码生成器产出在 `controller/admin/{module}/`，按角色权限映射移动到目标角色目录 |
| `$PackageInfo$.java` | 仅标准角色（saas/mch/admin/root/ops/rpc）的 controller 包下 `{module}/` 目录必须包含。**guest 角色不适用** |
| Helper 命名 | service 包内组件统一命名为 `{Module}Helper` |
| Helper 存在前提 | **三条件满足其一才创建**：(1)逻辑非常复杂（状态机/多步流程/计算），(2)功能性（缓存/分布式锁/事务），(3)多处调用（2个以上调用方）。简单CRUD直接在Controller调DaoManager，不建Helper |
| Helper 方法风格 | **全部 static 方法**。DaoManager 通过 `DaoManager.getInstance()` 获取，FusionCache/GlobalCache/GlobalLocker 本身就是静态API |
| Entity | 代码生成器产出，保留不修改 |
| DTO | 代码生成器产出，仅裁剪不新建 |
| 禁用 Lombok | 全局禁用，getter/setter/构造器均手写 |

### Controller 目录结构

代码生成器产出全部在 `controller/admin/{module}/`，设计阶段按角色权限映射移动到目标角色目录（`saas/`、`mch/` 等），修正 package 声明和 import。

详细目录示例见 [references/templates.md](references/templates.md) 角色路径对照表。
## 设计流程

### Phase 0: 需求确认

技术栈（UniWeb + SaaS）已限定，确认聚焦业务层面。

| 确认项 | 目的 | 启发式问题 |
|--------|------|-----------|
| 模块清单 + 复杂度分类 | 决定哪些用生成器代码、哪些需要 Helper | "根据PRD和数据库，识别到N个模块[列出]，是否有遗漏？" |
| **横切关注点识别** | **识别跨模块的公共业务规则** | "PRD中哪些业务规则影响多个模块？（如敏感词检测、通知发送、权限校验、数据脱敏等）" |
| 定制 API | 确定设计工作量 | "除标准CRUD+enable+disable外，各模块还有哪些接口？" |
| 外部依赖 | 识别技术风险 | "需要对接哪些外部系统？saas-finance？第三方API？异步任务？" |
| 测试策略 | 确定测试覆盖范围 | "哪些Helper方法需要Mock DaoManager/FusionCache？哪些需要并发测试？" |

**横切关注点识别（重要）**：

模块级 Helper 按"一个模块一个 Helper"识别，但**横切关注点 Helper 按"一个业务规则一个 Helper"识别**。横切关注点天然满足 Helper 三条件中的"多处调用"。

| 横切关注点 | 影响的模块 | Helper 方法 | 说明 |
|-----------|-----------|------------|------|
| 敏感词检测 | PostQuestion, PostAnswer, PostArticle, CmsComment | `SensitiveWordHelper.checkSensitiveWord(text)` | 内容发布前检测，决定审核状态 |
| 通知发送 | PostQuestion, PostAnswer, PostArticle, GuestUser | `MsgNotifyHelper.sendNotify(entity)` | 多种业务事件触发通知 |
| 其他通用规则 | 按PRD分析 | `{Xxx}Helper.{method}()` | 按需识别 |

**识别方法**：通读 PRD，找出"在多个模块中出现相同描述"的业务规则（如"敏感词检测"、"发送通知"、"权限校验"），每个都应抽象为独立 Helper。

模块分类决策：

| 分类 | 条件 | 代码策略 |
|------|------|---------|
| 简单模块 | 仅标准CRUD+enable+disable，无复杂逻辑 | 代码生成器产出，Controller 直接调 `DaoManager.getInstance()`，**不建 Helper** |
| 复杂模块 | 含业务流程、状态机、多表联动、缓存、锁 | 仅将复杂方法提取到 Helper（static），简单 CRUD 仍在 Controller |

**确认完成标准**：模块清单无遗漏、每个模块复杂度已分类、定制 API 已列出、外部依赖已识别、测试策略已明确。
### Phase 1: 编写 README.md

**输入**：PRD 文档 + 数据库设计文档 + Phase 0 确认结果

**读取技术栈**：先读取 [backend/README.md](references/backend/README.md) 建立全局认知。
**输出**：后端项目根目录 `README.md`

**必须包含的章节**：

| 章节 | 内容 | 必要性 |
|------|------|--------|
| 模块总览 | 模块清单、复杂度分类、代码策略 | 必须 |
| 模块依赖关系 | Mermaid graph，禁止循环依赖 | 必须 |
| PRD 功能点映射 | 功能点 → 模块 → 接口的映射表 | 必须 |
| 角色权限映射 | 角色（SAAS/MCH/ADMIN/ROOT/OPS）× 模块 R/W 矩阵 | 必须 |
| 状态机设计 | 有状态实体的 Mermaid stateDiagram | 按需 |
| 全局缓存策略 | FusionCache/GlobalCache 选型、Key规范、TTL分级、更新机制 | 必须 |
| 性能预算 | 数据量估算（3月/1年/3年）、索引策略、RT/QPS目标 | 必须 |
| 外部依赖 | saas-finance、第三方接口、异步任务清单 | 按需 |
| 全局错误码 | 业务错误码体系 | 按需 |
| 测试策略 | 测试分层（单元/集成）、覆盖率目标、Mock策略、关键测试场景 | 必须 |

**README.md 模板**：参见 [references/templates.md](references/templates.md)

### Phase 1.5: 编写 TASKS.md

**输入**：Phase 1 的模块总览 + 依赖图 + Phase 0 的确认结果

**输出**：后端项目根目录 `TASKS.md`（独立于 README.md，专供 310 开发消费）

**为什么独立**：README.md 是设计文档（全体角色阅读），TASKS.md 是执行计划（仅 310 Sub Agent 消费）。分离后 Sub Agent 只加载轻量的 TASKS.md，不加载重量级 README.md。

**必须包含的章节**：

| 章节 | 内容 |
|------|------|
| 并行分组 | 按依赖图拓扑排序，无依赖同组并行，有依赖串行 |
| 任务卡片 | 每个复杂模块一张卡片（含 PRD/文件/方法/依赖路径） |
| 联调验证清单 | 自动化检查项（grep + mvn test + 启动检查） |

**任务卡片编写规则**：

每个复杂模块（需建 Helper）生成一张任务卡片，简单模块（仅 CRUD）不列入。卡片中的任务 ID 与代码中 `// TODO: [Tn]` 标记一一对应。

卡片必须包含以下字段：

| 字段 | 说明 | 示例 |
|------|------|------|
| PRD | 关联的 PRD 文件路径 | `PROJECT_ROOT/requirement/prds/product.md` |
| 数据库 | 关联的数据库表（含文档位置） | `PROJECT_ROOT/database/database-design.md` § Product 表 |
| 待实现 | Helper.java 文件的完整路径 | `src/main/java/.../service/ProductHelper.java` |
| 测试骨架 | HelperTest.java 文件的完整路径 | `src/PROJECT_ROOT/test/java/.../service/ProductHelperTest.java` |
| 技术栈 | 该模块需要读取的技术栈文档 | `uw-dao.md`、`uw-cache.md` |
| 待实现方法 | 所有带 `// TODO: [Tn]` 标记的方法签名 | `getProduct()`、`saveProduct()` |
| 依赖任务 | 前置任务 ID | 无 / T1 |

并行分组规则：按 Mermaid 依赖图拓扑排序，无依赖模块同组可并行，有依赖模块串行。

末尾必须包含联调验证清单（自动化检查项），供 310 联调阶段直接执行。

**TASKS.md 模板**：参见 [references/templates.md](references/templates.md)

### Phase 2: 设计即代码

**读取技术栈**：按需读取以下文档确认 API 签名。

| 场景 | 读取文档 |
|------|---------|
| Controller 权限注解 | [uw-auth-service.md](references/backend/uniweb/uw-auth-service.md) |
| 响应格式、QueryParam | [uw-common.md](references/backend/uniweb/uw-common.md)、[uw-common-app.md](references/backend/uniweb/uw-common-app.md) |
| DaoManager 数据访问 | [uw-dao.md](references/backend/uniweb/uw-dao.md) |
| 缓存 FusionCache/GlobalCache | [uw-cache.md](references/backend/uniweb/uw-cache.md) |
| 异步/队列任务 | [uw-task.md](references/backend/uniweb/uw-task.md) |
| 外部HTTP调用 | [uw-httpclient.md](references/backend/uniweb/uw-httpclient.md) |

#### Step 1: 裁剪 DTO

> DTO 由代码生成器自动生成，不新建文件，仅裁剪。

| 裁剪类型 | 操作 |
|---------|------|
| 搜索字段 | 删除不需要的 `@QueryMeta` 字段（含注释+注解+声明+getter+setter+链式调用，共6部分） |
| 排序字段 | **不裁剪** ALLOWED_SORT_PROPERTY，保留无害，裁剪风险大于收益 |
| 校验注解 | 按设计补充 `@NotNull`、`@Size` 等约束（开发阶段补充即可） |

**裁剪方法（重要）**：

**Edit 工具逐块删除** 用 old_string 精确匹配完整代码块，每次删除后确认结构完整
**禁用sed批量删除和Python re.sub 多行正则** **禁止** 正则只删签名不删方法体，必留孤儿代码导致编译失败
**sed 批量删除** **禁止** 多行块边界匹配不可靠

**裁剪后必须**：`mvn compile` 验证。文件损坏时用代码生成器重新生成 dto 再裁剪，不手动修补。

#### Step 2: 裁剪 Controller

> Controller 由代码生成器产出在 `controller/admin/{module}/`，需按角色移动并裁剪。

| 操作 | 说明 |
|------|------|
| 角色移动 | 根据 README.md 角色权限映射，将 `{module}/` 从 `admin/` 移动到目标角色目录（`saas/`、`mch/` 等），并修正 package 声明和 import |
| 删除多余方法 | 如资源不允许删除，删除 delete() 方法 |
| **修正权限注解** | **按角色修正 `@MscPermDeclare` 的 `user` 和 `auth` 参数**（见下方映射表），代码生成器全部产出为 `UserType.ADMIN + AuthType.PERM`，移动到其他角色后必须修正 |
| **方法体调用** | **必须确定完整调用链路，方法体不能留空**。复杂逻辑：`return {Module}Helper.{method}(...)`，简单 CRUD：`return DaoManager.getInstance().{method}(...)` |
| 补充 Javadoc | 每个方法添加完整设计说明（设计思路 + 实现步骤） |
| 补充 $PackageInfo$.java | 仅标准角色（saas/mch/admin/root/ops/rpc）的 `{角色}/{模块}/` 目录下新建 $PackageInfo$.java，声明角色级权限。**guest 角色不适用** |

**@MscPermDeclare 按角色映射表**（参考 [uw-auth-service.md](references/backend/uniweb/uw-auth-service.md)）：

| 角色 | user | auth | 说明 |
|------|------|------|------|
| SAAS | `UserType.SAAS` | `AuthType.PERM` | 需验证权限，注册为菜单 |
| MCH | `UserType.MCH` | `AuthType.PERM` | 需验证权限，注册为菜单 |
| ADMIN | `UserType.ADMIN` | `AuthType.PERM` | 需验证权限，注册为菜单 |
| ROOT | `UserType.ROOT` | `AuthType.PERM` | 需验证权限，注册为菜单 |
| OPS | `UserType.OPS` | `AuthType.PERM` | 需验证权限，注册为菜单 |
| RPC | `UserType.RPC` | `AuthType.NONE` | 内部调用无需鉴权 |
| **GUEST** | `UserType.GUEST` | `AuthType.USER` | **仅验证用户类型，不验证权限** |

> **重要**：Guest 控制器使用 `AuthType.USER`（仅验证登录），不使用 `AuthType.PERM`（会注册为菜单权限，但 Guest 无后台菜单系统）。

**Controller 方法体模板**：
```java
// 简单 CRUD - 直接调 DaoManager
@GetMapping("/list")
@MscPermDeclare(name = "列表", auth = AuthType.PERM, log = ActionLog.REQUEST)
public ResponseData<DataList<Xxx>> list(AuthQueryParam param) {
    return DaoManager.getInstance().list(Xxx.class, param);
}

// 复杂逻辑 - 调用 Helper 静态方法
@PostMapping("/save")
@MscPermDeclare(name = "新增", auth = AuthType.PERM, log = ActionLog.ALL)
public ResponseData<Xxx> save(@RequestBody Xxx entity) {
    return XxxHelper.saveXxx(entity);
}
```

**角色移动示例**：
```bash
# 将 product 模块从 admin 移动到 saas
mv controller/admin/product/ controller/saas/product/
# 修正包名: my.shop.controller.admin.product → my.shop.controller.saas.product
# 修正权限注解: UserType.ADMIN → UserType.SAAS
```

#### Step 3: 新建 Helper

> service 包下新建 `{Module}Helper.java`，仅包含 static 方法签名和完整 Javadoc，方法返回默认值。

**创建判断**：Helper 仅在以下条件满足**至少一项**时才创建：

| 条件 | 说明 | 示例 |
|------|------|------|
| 逻辑复杂 | 状态机、多步流程、计算、复杂校验 | 内容审核（多状态流转）、回答采纳（锁+状态+积分） |
| 功能性 | 缓存、分布式锁、事务等横切关注点 | 详情缓存（FusionCache）、并发操作（GlobalLocker） |
| 多处调用 | 2个以上 Controller 或其他 Helper 调用 | 发送通知（多处触发）、用户信息查询（多处引用） |

**不建 Helper 的场景**：简单 CRUD（list/get/save/update/delete/enable/disable）直接在 Controller 中调 `DaoManager.getInstance()` 即可。

**Helper 两种类型**：

| 类型 | 识别维度 | 示例 |
|------|---------|------|
| **模块级 Helper** | 按数据库表/模块识别，封装该模块的复杂业务 | PostQuestionHelper、PostAnswerHelper |
| **横切 Helper** | 按 PRD 中跨模块的公共业务规则识别，不绑定单表 | SensitiveWordHelper（敏感词检测）、MsgNotifyHelper（通知发送） |

> **横切 Helper 识别方法**：通读 PRD，找出"在多个模块中出现相同描述"的业务规则。例如 PRD 中 PostQuestion/PostAnswer/PostArticle/CmsComment 都提到"敏感词检测"，这就是一个横切关注点，需要独立 Helper。Phase 0 的"横切关注点识别"步骤会提前列出这些。

| 内容 | 说明 |
|------|------|
| 类注解 | 无（纯静态工具类，不加 `@Component`） |
| 依赖获取 | `DaoManager.getInstance()` 静态获取，FusionCache/GlobalCache/GlobalLocker 本身是静态API |
| 方法签名 | `public static`，入参、出参明确定义，方法体返回默认值 |
| **TODO 标记** | **每个方法体上方添加 `// TODO: [Tn] implement {methodName}`，Tn 为 README.md 任务卡片中的任务 ID** |
| 缓存常量 | `private static final` FusionCache.Config 参数、缓存 Key 常量 |
| **缓存初始化** | **设计阶段必须完成**：所有使用 FusionCache 的 Helper 必须在 `static {}` 块中调用 `FusionCache.config(new FusionCache.Config(...), new CacheDataLoader<>() {...})` 完成初始化，包括 Config 参数和 CacheDataLoader 数据加载器。缓存参数（localCacheMaxNum、cacheExpireMillis）在设计阶段就确定，开发阶段仅实现 CacheDataLoader.load() 内部的查询逻辑 |
| Javadoc | 设计思路 + 实现步骤 + 缓存策略 + 事务策略 |

**FusionCache 初始化模板**（设计阶段必须产出）：

```java
public class XxxHelper {
    private static final DaoManager daoManager = DaoManager.getInstance();
    private static final int CACHE_MAX_NUM = 500;
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    static {
        FusionCache.config(new FusionCache.Config(
            XxxEntity.class,
            CACHE_MAX_NUM,
            CACHE_EXPIRE_MILLIS
        ), new CacheDataLoader<Long, XxxEntity>() {
            @Override
            public XxxEntity load(Long key) {
                return daoManager.load(XxxEntity.class, key).getData();
            }
        });
    }

    // TODO: [Tn] implement getXxx with FusionCache
    public static ResponseData<XxxEntity> getXxx(Long id) {
        return ResponseData.success(null);
    }

    // TODO: [Tn] implement saveXxx
    public static ResponseData<XxxEntity> saveXxx(XxxEntity entity) {
        return ResponseData.success(null);
    }
}
```

> **重要**：FusionCache 的 Config（缓存容量、过期时间）和 CacheDataLoader（数据加载器签名）在设计阶段确定。GlobalCache 不需要 static 初始化（直接用 `GlobalCache.get(...)` 带行内 CacheDataLoader），但 FusionCache 必须在类加载时完成 config。

#### Step 3.5: 编写测试骨架（TDD Red）

> 每个 Helper 和 Controller 都对应一个测试类，这是 TDD 的 Red 阶段。测试方法有签名和断言结构，因方法体返回 null 故测试全红（预期行为）。

**TDD 详细指南**：参见 [references/tdd-design-guide.md](references/tdd-design-guide.md)

##### Helper 单元测试

| 内容 | 说明 |
|------|------|
| 测试类位置 | `src/test/java/{包路径}/service/{Module}HelperTest.java` |
| 测试类注解 | `@ExtendWith(MockitoExtension.class)` + `@MockedStatic DaoManager` + `@MockedStatic FusionCache/GlobalLocker`（静态方法测试用 MockedStatic） |
| 测试方法数 | 每个 Helper 方法 ≥ 2 个测试方法（正常 + 边界/异常） |
| 命名规范 | `test{Method}_{Scenario}_{ExpectedResult}` |
| 注解 | `@Test` + `@DisplayName("...")` |
| Javadoc | 测试意图 + 准备数据 + 预期结果 |
| 方法体 | `fail("TDD Red: [Tn] Helper 方法尚未实现")` 标记 Red 状态，Tn 为任务 ID |
| MockedStatic | DaoManager.getInstance()、FusionCache、GlobalLocker、AuthServiceHelper 的静态 Mock 在开发阶段补充 |

**测试模板**：参见 [references/templates.md](references/templates.md) 第6节 Helper 单元测试模板

**快速示例**：
```java
// {Module}HelperTest.java - 每个Helper方法 ≥ 2个测试方法
@Test @DisplayName("查询详情 - 正常返回")
void testGet{Entity}_Found_ReturnEntity() { fail("TDD Red: [Tn]"); }

@Test @DisplayName("查询详情 - 不存在返回warn")
void testGet{Entity}_NotFound_ReturnWarn() { fail("TDD Red: [Tn]"); }
```

##### Controller 单元测试

> Controller 测试验证 API 契约：HTTP 方法、路径、参数绑定、权限注解、响应格式、调用链路。

| 内容 | 说明 |
|------|------|
| 测试类位置 | `src/PROJECT_ROOT/test/java/{包路径}/controller/{角色}/{模块}/{Module}ControllerTest.java`（与源码目录结构对齐） |
| 测试类注解 | `@ExtendWith(MockitoExtension.class)` |
| 测试方法数 | 每个 Controller 方法 ≥ 1 个测试方法 |
| 命名规范 | `test{Method}_{Scenario}_{ExpectedResult}` |
| 注解 | `@Test` + `@DisplayName("...")` |
| Javadoc | 测试意图 + API 调用方式 + 预期响应 |
| 方法体 | `fail("TDD Red: Controller 测试尚未实现")` 标记 Red 状态 |

**快速示例**：
```java
// {Module}ControllerTest.java - 每个Controller方法 ≥ 1个测试方法
@Test @DisplayName("列表查询 - 正常返回")
void testList_Normal_ReturnSuccess() { fail("TDD Red"); }

@Test @DisplayName("新增 - 正常返回")
void testSave_Normal_ReturnSuccess() { fail("TDD Red"); }
```

#### Step 4: 新建 VO（如需）

>在需要聚合多表数据或裁剪输出的时创建 VO 类。

#### Step 5: 编译验证 + 测试验证

| 检查项 | 命令/方式 | 预期结果 |
|--------|----------|---------|
| 编译通过 | `mvn compile` | BUILD SUCCESS |
| 测试编译通过 | `mvn test-compile` | BUILD SUCCESS |
| 测试可运行 | `mvn test -pl {模块}` | 全红（Helper方法返回null），无编译/运行时错误 |
| Swagger 可用 | 启动后访问 Swagger UI | 所有接口可展示 |
| 无运行时错误 | 启动无 Bean 注入失败等错误 | 正常启动 |

**测试全红是预期行为**：设计阶段 Helper 方法体返回 `null`，测试断言必然失败。设计阶段的验收标准是"测试骨架可编译 + 可运行"，而非"测试通过"。

**代码模板**：参见 [references/templates.md](references/templates.md)

## PLAN-REVIEW 循环（必须执行）

设计完成后，调用 `211-java-uniweb-design-review` 执行评审。

## 产出结构

```
{后端项目根目录}/
├── README.md                         # 总体设计文档（Phase 1 产出）
├── TASKS.md                          # 开发任务分工（Phase 1.5 产出，310 消费）
├── src/main/java/{包路径}/
│   ├── controller/
│   │   ├── saas/                     # 裁剪 + Javadoc + $PackageInfo$
│   │   ├── mch/
│   │   ├── admin/
│   │   └── rpc/
│   ├── service/                      # 新建 Helper
│   │   ├── {ModuleA}Helper.java
│   │   └── {ModuleB}Helper.java
│   ├── entity/                       # 保留不动
│   ├── dto/                          # 裁剪后
│   └── vo/                           # 新建（如需）
├── src/test/java/{包路径}/
│   └── service/                      # TDD Red 阶段测试骨架
│       ├── {ModuleA}HelperTest.java
│       └── {ModuleB}HelperTest.java
└── pom.xml
```

## 设计完成标准

- [ ] README.md 覆盖所有模块和全局策略（含测试策略章节）
- [ ] TASKS.md 包含并行分组、任务卡片和联调验证清单
- [ ] 所有 Controller 方法有 Javadoc + @MscPermDeclare
- [ ] 所有 Helper 方法有 Javadoc（设计思路 + 实现步骤 + 缓存策略）
- [ ] 所有 Helper 方法体上方有 `// TODO: [Tn]` 标记，与 TASKS.md 任务卡片 ID 一一对应
- [ ] 所有使用 FusionCache 的 Helper 有 `static { FusionCache.config(...) }` 初始化块
- [ ] 所有 Helper 有对应测试类 `{Module}HelperTest.java`
- [ ] 每个 Helper 方法有 ≥ 2 个测试方法（正常 + 边界/异常）
- [ ] 所有测试方法体有 `fail("TDD Red: [Tn]")` 标记，与任务 ID 一致
- [ ] 所有 Controller 有对应测试类 `{Module}ControllerTest.java`
- [ ] 每个 Controller 方法有 ≥ 1 个测试方法
- [ ] 测试方法有 `@DisplayName` + Javadoc
- [ ] DTO 搜索字段和排序字段已按业务需求裁剪
- [ ] `mvn compile` 编译通过
- [ ] `mvn test-compile` 测试编译通过
- [ ] `mvn test` 可运行（全红为预期）
- [ ] Swagger UI 可展示所有接口
- [ ] 前端可基于 Swagger 开始联调

## 参考

- [README + 代码模板 + 测试模板](references/templates.md)
- [TDD设计指南](references/tdd-design-guide.md)
- [UniWeb 技术栈](references/backend/uniweb/README.md)
- [SaaS 技术栈](references/backend/saas/README.md)
