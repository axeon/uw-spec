---
name: 310-java-uniweb-dev
description: Java SaaS后端开发技能。当需要基于UniWeb+Saas技术栈开发Java后端服务时触发：(1)实现Helper业务逻辑，(2)开发RESTful API接口，(3)集成DaoManager/FusionCache，(4)对接AIP/AIS/saas-finance，(5)指定模块开发或并行开发全部模块，(6)TDD Green实现单元测试。当用户提及Java后端开发、uw-base、SaaS开发、API开发、Helper实现、指定模块、并行开发时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "3.0.0"
---

# Java SaaS开发

## 描述

基于 UniWeb + SaaS 技术栈（uw-base / saas-base / saas-finance）实现后端业务逻辑。220 设计阶段已产出 Controller 骨架和 Helper 签名，本阶段在此基础上填充实现代码。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Helper实现 | "实现商品Helper" |
| Controller开发 | "开发订单接口" |
| 指定模块开发 | "开发product模块" |
| 并行开发 | "并行开发所有模块" |
| 数据访问 | "查询商品列表" |
| 缓存集成 | "添加FusionCache" |
| SaaS服务对接 | "对接saas-finance支付" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 后端开发 + 单元测试实现 | `java-developer` |
| 协作 | 技术方案确认 | `system-architect` |

> **职责边界**：单元测试由 Java 开发工程师负责（TDD Green 阶段，从 220 设计阶段产出的 Red 骨架实现为 Green）。测试工程师不参与单元测试，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `requirement/prds/*` | 产品需求文档，功能模块及验收标准 |
| 数据库设计文档 | `database/database-design.md` | 表结构、实体关系 |
| 数据库DDL | `database/database-ddl.sql` | 建表语句 |
| 后端设计文档 | `backend/{项目名}-app/README.md` | 模块划分、接口设计、角色权限映射、缓存策略 |
| Controller代码 | `backend/{项目名}-app/src/main/java/{包路径}/controller/` | 220阶段裁剪后的Controller |
| Helper代码 | `backend/{项目名}-app/src/main/java/{包路径}/service/` | 220阶段新建的Helper方法签名 |
| DTO代码 | `backend/{项目名}-app/src/main/java/{包路径}/dto/` | 220阶段裁剪后的DTO |
| 测试骨架代码 | `backend/{项目名}-app/src/test/java/{包路径}/service/` | 220阶段产出的 Helper 单元测试骨架（TDD Red） |
| 测试用例设计 | `test/design/` | 测试场景和用例覆盖（API/E2E/压测/安全） |

## 技术栈

| 技术栈 | 文档入口 | 用途 |
|--------|----------|------|
| UniWeb 基础框架 | [backend/uniweb/README.md](../210-java-uniweb-design/references/backend/uniweb/README.md) | uw-base类库、微服务、开发规范 |
| SaaS 业务框架 | [backend/saas/README.md](../210-java-uniweb-design/references/backend/saas/README.md) | AIP/AIS、saas-base、saas-finance |

### 核心类库速查

| 场景 | 类库 | 文档 |
|------|------|------|
| 数据访问 | DaoManager, DataEntity, QueryParam | [uw-dao.md](../210-java-uniweb-design/references/backend/uniweb/uw-dao.md) |
| 缓存 | FusionCache | [uw-cache.md](../210-java-uniweb-design/references/backend/uniweb/uw-cache.md) |
| 权限声明 | @MscPermDeclare, $PackageInfo$ | [uw-auth-service.md](../210-java-uniweb-design/references/backend/uniweb/uw-auth-service.md) |
| 统一响应 | ResponseData, DataList | [uw-common.md](../210-java-uniweb-design/references/backend/uniweb/uw-common.md) |
| 权限查询 | AuthQueryParam | [uw-common-app.md](../210-java-uniweb-design/references/backend/uniweb/uw-common-app.md) |
| 异步/队列 | @TaskScheduler, @QueueWorker | [uw-task.md](../210-java-uniweb-design/references/backend/uniweb/uw-task.md) |
| HTTP调用 | HttpClientPool | [uw-httpclient.md](../210-java-uniweb-design/references/backend/uniweb/uw-httpclient.md) |
| 服务间调用 | authRestTemplate | [uw-auth-client.md](../210-java-uniweb-design/references/backend/uniweb/uw-auth-client.md) |
| AIP授权计费 | Vendor, License, AipHelper | [aip-module.md](../210-java-uniweb-design/references/backend/saas/aip-module.md) |
| AIS接口服务 | Linker, AisHelper | [ais-module.md](../210-java-uniweb-design/references/backend/saas/ais-module.md) |
| 财务服务 | FinBalanceHelper, FinPaymentHelper | [saas-finance-client.md](../210-java-uniweb-design/references/backend/saas/saas-finance-client.md) |

## 开发模式

| 模式 | 触发方式 | 说明 |
|------|---------|------|
| 单模块开发 | "开发 {模块名}" 或 "实现 {模块名}Helper" | 指定一个模块，执行完整 Helper+Controller+Test 开发 |
| 并行多模块 | "并行开发所有模块" 或 "并行开发模块A、B" | 解析依赖图，无依赖模块组并行开发，有依赖模块按拓扑序串行 |
| 顺序全量 | "开发所有模块" | 按依赖拓扑序逐个开发，不并行 |

> **默认模式**：未明确指定时，按"顺序全量"执行（安全）。

## 220→310 衔接协议

220 设计完成后，310 从 README.md 提取以下信息作为开发输入：

| 提取项 | README.md 章节 | 用途 |
|--------|---------------|------|
| 模块清单 | "模块总览" | 确定开发范围 |
| 复杂度分类 | "模块总览" | 简单模块使用代码生成器产出，无需新建Helper |
| 依赖图 | "模块依赖关系"（Mermaid graph） | 拓扑排序，确定并行/串行分组 |
| 角色权限矩阵 | "角色权限映射" | Controller 权限注解依据 |
| 缓存策略 | "全局缓存策略" | Helper 缓存实现依据 |
| PRD映射 | "PRD功能点映射" | 按模块索引对应PRD需求 |

## 开发步骤

### 1. 环境准备 + 模块分析

**前置条件**：300-database-ddl-execution 已完成且 301-database-ddl-execution-review 评审通过（数据库表已就绪）。

1. 读取 `backend/{项目名}-app/README.md` 提取模块清单、依赖图、复杂度分类
2. 读取 [dev-standards.md](../210-java-uniweb-design/references/backend/uniweb/dev-standards.md) 了解编码规范
3. 读取 `requirement/prds/*` 了解负责的Module需求
4. **依赖拓扑分析**：解析 README.md 中 Mermaid 依赖图，输出开发分组

**拓扑分析规则**：

| 规则 | 说明 |
|------|------|
| 无依赖模块 | 可并行开发（同一组） |
| 有依赖模块 | 被依赖方先完成，依赖方后开发 |
| 禁止循环依赖 | 发现循环依赖报错，回 210 修复 |
| 降级策略 | Mermaid 依赖图解析失败或缺失时，回退为"顺序全量"模式逐个开发 |

**降级判断**：若 README.md 中无"模块依赖关系"章节、Mermaid graph 为空、或解析后无法得到有效拓扑，输出警告"依赖图缺失/无法解析，回退顺序全量模式"，继续执行。

**拓扑分组示例**（依赖图: A→C, B→C, C→D）：
```
组1（并行）: [A, B]     ← 无依赖，可同时开发
组2（串行）: [C]        ← 依赖A、B，等组1完成
组3（串行）: [D]        ← 依赖C，等组2完成
```

**单模块开发范围**：一个Agent负责一个Module，Module包含多个Feature的API实现。

### 2. Helper实现

> Helper 是业务逻辑的核心载体，位于 service 包下。

**读取技术栈**：按需读取以下文档确认 API 用法。

| 场景 | 读取文档 |
|------|---------|
| 数据访问 | [uw-dao.md](../210-java-uniweb-design/references/backend/uniweb/uw-dao.md) |
| 缓存 | [uw-cache.md](../210-java-uniweb-design/references/backend/uniweb/uw-cache.md) |
| 异步/队列 | [uw-task.md](../210-java-uniweb-design/references/backend/uniweb/uw-task.md) |
| HTTP调用 | [uw-httpclient.md](../210-java-uniweb-design/references/backend/uniweb/uw-httpclient.md) |
| AIP授权 | [aip-module.md](../210-java-uniweb-design/references/backend/saas/aip-module.md) |
| AIS接口 | [ais-module.md](../210-java-uniweb-design/references/backend/saas/ais-module.md) |
| 财务服务 | [saas-finance-client.md](../210-java-uniweb-design/references/backend/saas/saas-finance-client.md) |

| 开发规范 | 说明 |
|---------|------|
| 类注解 | `@Component` |
| 依赖注入 | 构造器注入 DaoManager、其他 Helper |
| 数据访问 | 使用 `DaoManager`（ResponseData 链式 lambda 风格），不使用 DaoFactory |
| 查询参数 | 使用 QueryParam 系列（IdQueryParam、AuthQueryParam 等） |
| 批量操作 | 使用 `BatchUpdateManager` |
| 缓存 | 使用 `FusionCache`，Kryo 序列化用具体实现类（ArrayList/LinkedHashMap/HashSet） |
| 禁用 Lombok | 全局禁用，getter/setter/构造器均手写 |

### 3. Controller裁剪与权限声明

> Controller 由代码生成器产出，220 阶段已裁剪移动，本阶段补充业务调用。

**读取技术栈**：按需读取权限相关文档。

| 场景 | 读取文档 |
|------|---------|
| 权限注解 | [uw-auth-service.md](../210-java-uniweb-design/references/backend/uniweb/uw-auth-service.md) |
| 响应格式 | [uw-common.md](../210-java-uniweb-design/references/backend/uniweb/uw-common.md) |

| 开发规范 | 说明 |
|---------|------|
| 路径第一级 | 必须为用户角色（`/saas/`、`/mch/`、`/admin/`、`/root/`、`/ops/`、`/rpc/`） |
| 返回值 | 统一使用 `ResponseData` 包装 |
| 权限声明 | 每个方法添加 `@MscPermDeclare` |
| $PackageInfo$ | 每个角色包下的模块目录必须包含 package-info.java |
| 参数校验 | `@RequestBody` 参数加 `@Valid`，触发 DTO 校验注解 |

### 4. TDD Green — 实现单元测试

> 220 阶段已产出测试骨架（Red），本阶段实现 Helper 逻辑并让测试逐个变绿。

**TDD 详细指南**：参见 [TDD设计指南](../210-java-uniweb-design/references/tdd-design-guide.md)

| 步骤 | 操作 |
|------|------|
| 读取测试骨架 | 读取 220 产出的 `{Module}HelperTest.java`，了解测试意图 |
| 实现 Helper 方法 | 按 Javadoc 实现步骤填充方法体 |
| 补充 Mock 初始化 | 在测试类中补充 `@BeforeEach` 的 MockedStatic 设置（FusionCache/GlobalLocker/AuthServiceHelper） |
| 替换 fail() 为断言 | 将 `fail("TDD Red")` 替换为具体的 Mock + assert 断言 |
| 逐个变绿 | 每实现一个 Helper 方法，对应测试从红变绿 |

**Mock 模式**：

| 依赖 | Mock 方式 |
|------|----------|
| DaoManager | `@Mock` + `@InjectMocks`（构造器注入） |
| FusionCache | `mockStatic(FusionCache.class)` + try-with-resources |
| GlobalLocker | `mockStatic(GlobalLocker.class)` + try-with-resources |
| AuthServiceHelper | `mockStatic(AuthServiceHelper.class)` + try-with-resources |

### 5. Entity与DTO

| 类型 | 规范 | 操作 |
|------|------|------|
| Entity | 继承 `DataEntity`，使用 `@TableMeta` + `@ColumnMeta` 注解 | 保留不动（代码生成器产出） |
| DTO | 代码生成器产出，QueryParam 系列类 | 仅裁剪，不新建 |

### 6. 编译验证 + 测试验证

| 检查项 | 命令/方式 | 预期结果 |
|--------|----------|---------|
| 编译通过 | `mvn compile` | BUILD SUCCESS |
| 测试编译通过 | `mvn test-compile` | BUILD SUCCESS |
| 单元测试通过 | `mvn test` | 全绿（或覆盖率 ≥ 80%） |
| Swagger可用 | 启动后访问 Swagger UI | 所有接口可展示 |
| 无运行时错误 | 启动无 Bean 注入失败等错误 | 正常启动 |

## 并行开发约束

| 约束 | 说明 |
|------|------|
| 无依赖模块可并行 | 模块A、B互不依赖，可同时由不同Agent开发 |
| 有依赖模块需串行 | 模块C依赖A，需A完成review通过后才开发C |
| 共享文件只读 | entity/dto 由代码生成器产出，并行时只读不改 |
| Helper天然独立 | 各模块Helper在不同文件，无文件级冲突 |
| review按模块 | 每个模块开发完成后独立进入PLAN-REVIEW，不等全部完成 |

## 输出要求

- **代码位置**: `backend/{项目名}-app/src/main/java/{包路径}/`
- **测试位置**: `backend/{项目名}-app/src/test/java/{包路径}/service/`
- **包含内容**: Helper实现代码、Controller业务调用、单元测试全绿、编译验证通过

## 参考

- [TDD设计指南](../210-java-uniweb-design/references/tdd-design-guide.md) - TDD三阶段映射、Mock策略、测试模板
- [UniWeb开发规范](../210-java-uniweb-design/references/backend/uniweb/dev-standards.md) - 项目初始化、依赖、配置、编码规范
- [SaaS开发规范](../210-java-uniweb-design/references/backend/saas/dev-standards.md) - 命名规范、API路径规范、工具类速查
- [Java开发评审技能](../311-java-uniweb-dev-review/SKILL.md) - PLAN-REVIEW循环调用的评审技能

## PLAN-REVIEW 循环（必须执行）

Java后端开发完成后，必须进入 PLAN-REVIEW 循环，确保代码质量达标。

#### 单模块 vs 并行模式

| 模式 | review 范围 | 说明 |
|------|-----------|------|
| 单模块 | 该模块全部代码 | 一个Agent完成即review |
| 并行多模块 | 每个模块独立review | 模块A review通过后，依赖A的模块才可开始开发 |
| 顺序全量 | 每个模块独立review | 同并行，但串行执行 |

#### 循环规则
| 规则 | 说明 |
|------|------|
| 通过条件 | review 评分 ≥ 95分 |
| 强制退出 | 最多5轮review（round=1~5），round ≥ 6强制退出 |
| 每轮操作 | review → 修复问题 → 重新 review |

#### 循环流程
```
循环开始 (round=1)
  │
  ├─▶ 调用技能: 311-java-uniweb-dev-review
  │    对 backend/ 下代码执行评审
  │
  ├─▶ 获取评审报告: backend/{项目名}-app/reviews/REVIEW-CODE-YYMMDDHHMM.md
  │
  ├─▶ 判断结果:
  │    ├─ 评分 ≥ 95 → 输出结论，循环结束 ✓
  │    └─ 评分 < 95 → 进入修复步骤
  │
  ├─▶ 修复问题:
  │    按评审报告中的问题清单（Critical > Major > Minor）逐项修复
  │    修复后 round++，回到循环开始
  │
  └─▶ round ≥ 6 → 强制退出，输出当前结论 ⚠️
```

#### 每轮修复要求
| 优先级 | 要求 |
|--------|------|
| Critical | 本轮必须全部修复，不可遗留 |
| Major | 本轮修复 ≥ 80%，剩余标注下轮计划 |
| Minor | 记录但不阻塞，按优先级排列 |

#### 输出循环结论
每次循环结束（通过或强制退出），在评审报告末尾追加：
```markdown
## PLAN-REVIEW 循环结论
- 总轮次: X/3
- 最终评分: XX分
- 状态: 通过 / 有条件通过 / 强制退出
- 遗留问题: X个（Critical: X, Major: X, Minor: X）
```
