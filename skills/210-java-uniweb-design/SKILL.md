---
name: 210-java-uniweb-design
description: SaaS后端技术设计（TDD驱动，设计即代码）。当需要基于PRD和数据库设计进行后端技术设计时触发：(1)确认模块划分与定制API，(2)编写项目README.md总体设计（含测试策略），(3)裁剪DTO和Controller，(4)新建Helper方法签名与缓存定义，(5)编写Helper单元测试骨架（TDD Red阶段），(6)确保编译通过且测试可运行。当用户提及后端设计、接口设计、技术设计、生成脚手架时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "3.0.0"
---

# SaaS 后端技术设计（TDD 驱动，设计即代码）

## 描述

基于 UniWeb + SaaS 技术栈，采用**TDD 驱动 + 设计即代码**模式：设计文档融入代码 Javadoc，总体设计作为项目 README.md。设计阶段同步产出 Helper 单元测试骨架（TDD Red 阶段），确保每个 Helper 方法都有可执行的测试契约。设计完成后项目可编译通过、测试可运行（全红为预期）、Swagger 可用。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 后端设计 | "开始后端技术设计" |
| 接口设计 | "设计API接口" |
| 脚手架生成 | "生成后端脚手架" |
| 设计裁剪 | "裁剪DTO和Controller" |

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
| PRD | `requirement/prds/*` | 产品需求文档，功能模块及各终端详细需求 |
| 数据库设计文档 | `database/database-design.md` | 表结构设计、实体关系、索引策略 |
| 数据库DDL | `database/database-ddl.sql` | DDL建表语句，用于对照表名和字段 |
| 项目目录 | `backend/{项目名}-app/` | 210阶段项目代码以及产出的entity/dto/controller |

## 前置条件

| 前置技能 | 说明 |
|---------|------|
| [200-database-design](../200-database-design/SKILL.md) | 数据库设计已完成并通过评审 |
| [210-java-uniweb-init](../210-java-uniweb-init/SKILL.md) | 项目已通过模板初始化 |
| [210-java-uniweb-gencode](../210-java-uniweb-gencode/SKILL.md) | entity/dto/controller 全量curd功能已由代码生成器生成在admin角色下，需定制裁剪 |

**已生成代码**：代码生成器已产出 entity 实体类、dto 数据传输对象和标准 CRUD Controller，设计阶段在此基础上裁剪和补充。

## 架构约定

| 约定 | 说明 |
|------|------|
| Controller 路径第一级 | 必须为用户角色（`/saas/`、`/mch/`、`/admin/`、`/root/`、`/ops/`、`/rpc/`） |
| Controller 初始位置 | 代码生成器产出在 `controller/admin/{module}/`，按角色权限映射移动到目标角色目录 |
| `$PackageInfo$` | 每个 controller 角色包下的 `{module}/` 目录必须包含 package-info.java |
| Helper 命名 | service 包内组件统一命名为 `{Module}Helper` |
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
| 定制 API | 确定设计工作量 | "除标准CRUD+enable+disable外，各模块还有哪些接口？" |
| 外部依赖 | 识别技术风险 | "需要对接哪些外部系统？saas-finance？第三方API？异步任务？" |
| 测试策略 | 确定测试覆盖范围 | "哪些Helper方法需要Mock DaoManager/FusionCache？哪些需要并发测试？" |

模块分类决策：

| 分类 | 条件 | 代码策略 |
|------|------|---------|
| 简单模块 | 仅标准CRUD+enable+disable | 代码生成器产出，裁剪即可 |
| 复杂模块 | 含业务流程、状态机、多表联动 | 新建 Helper 封装核心逻辑 |

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
| 搜索条件 | 在 QueryParam DTO 中删除不需要的 `@QueryMeta` 搜索字段 |
| 排序字段 | 从 ALLOWED_SORT_PROPERTY 中移除不需要的排序字段 |
| 校验注解 | 按设计补充 `@NotNull`、`@Size` 等约束 |
| Swagger注解 | 补充 `@Schema(description=...)` 说明 |

#### Step 2: 裁剪 Controller

> Controller 由代码生成器产出在 `controller/admin/{module}/`，需按角色移动并裁剪。

| 操作 | 说明 |
|------|------|
| 角色移动 | 根据 README.md 角色权限映射，将 `{module}/` 从 `admin/` 移动到目标角色目录（`saas/`、`mch/` 等），并修正 package 声明和 import |
| 删除多余方法 | 如资源不允许删除，删除 delete() 方法 |
| 补充权限注解 | 每个方法添加 `@MscPermDeclare` 权限声明 |
| 补充 Javadoc | 每个方法添加完整设计说明（设计思路 + 实现步骤） |
| 补充 package-info.java | 每个 `{角色}/{模块}/` 目录下新建 package-info.java，声明角色级权限 |

**角色移动示例**：
```bash
# 将 product 模块从 admin 移动到 saas
mv controller/admin/product/ controller/saas/product/
# 修正包名: my.shop.controller.admin.product → my.shop.controller.saas.product
```

#### Step 3: 新建 Helper

> service 包下新建 `{Module}Helper.java`，仅包含方法签名和完整 Javadoc，方法返回默认值。

| 内容 | 说明 |
|------|------|
| 类注解 | `@Component` |
| 依赖注入 | 构造器注入 DaoManager、其他 Helper |
| 方法签名 | 入参、出参明确定义，方法体返回默认值 |
| 缓存常量 | FusionCache.Config 参数、缓存 Key 常量 |
| Javadoc | 设计思路 + 实现步骤 + 缓存策略 + 事务策略 |

#### Step 3.5: 编写测试骨架（TDD Red）

> 每个 Helper 对应一个 `{Module}HelperTest.java`，这是 TDD 的 Red 阶段。测试方法有签名和断言结构，因 Helper 返回 null 故测试全红（预期行为）。

**TDD 详细指南**：参见 [references/tdd-design-guide.md](references/tdd-design-guide.md)

| 内容 | 说明 |
|------|------|
| 测试类位置 | `src/test/java/{包路径}/service/{Module}HelperTest.java` |
| 测试类注解 | `@ExtendWith(MockitoExtension.class)` + `@Mock DaoManager` + `@InjectMocks Helper` |
| 测试方法数 | 每个 Helper 方法 ≥ 2 个测试方法（正常 + 边界/异常） |
| 命名规范 | `test{Method}_{Scenario}_{ExpectedResult}` |
| 注解 | `@Test` + `@DisplayName("...")` |
| Javadoc | 测试意图 + 准备数据 + 预期结果 |
| 方法体 | `fail("TDD Red: Helper 方法尚未实现")` 标记 Red 状态 |
| MockedStatic | FusionCache/GlobalLocker/AuthServiceHelper 的静态 Mock 在开发阶段补充 |

**测试模板**：参见 [references/templates.md](references/templates.md) 第6节 Helper 单元测试模板

**快速示例**：
```java
// {Module}HelperTest.java - 每个Helper方法 ≥ 2个测试方法
@Test @DisplayName("查询详情 - 正常返回")
void testGet{Entity}_Found_ReturnEntity() { fail("TDD Red"); }

@Test @DisplayName("查询详情 - 不存在返回warn")
void testGet{Entity}_NotFound_ReturnWarn() { fail("TDD Red"); }
```

#### Step 4: 新建 VO（如需）

> 仅在需要聚合多表数据时创建 VO 类。

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

### Phase 3: PLAN-REVIEW 循环

设计完成后，必须进入 PLAN-REVIEW 循环。

#### 循环规则

| 规则 | 说明 |
|------|------|
| 通过条件 | review 评分 ≥ 95分 |
| 强制退出 | 最多5轮review（round=1~5），round ≥ 6强制退出 |
| 每轮操作 | review → 修复问题 → 重新 review |

#### 循环流程

1. 调用技能 `211-java-uniweb-design-review`，评审 README.md + 代码质量 + 测试骨架
2. 评分 ≥ 95 → 输出结论，循环结束
3. 评分 < 95 → 按优先级修复（Critical 必须全修、Major ≥ 80%、Minor 记录），round++，回到步骤 1
4. round ≥ 6 → 强制退出，输出当前结论 ⚠️

#### 输出循环结论

每轮在评审报告末尾追加：总轮次 X/5、最终评分、状态（通过/有条件通过/强制退出）、遗留问题数。

## 产出结构

```
{后端项目根目录}/
├── README.md                         # 总体设计文档（Phase 1 产出）
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
- [ ] 所有 Controller 方法有 Javadoc + @MscPermDeclare
- [ ] 所有 Helper 方法有 Javadoc（设计思路 + 实现步骤 + 缓存策略）
- [ ] 所有 Helper 有对应测试类 `{Module}HelperTest.java`
- [ ] 每个 Helper 方法有 ≥ 2 个测试方法（正常 + 边界/异常）
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
