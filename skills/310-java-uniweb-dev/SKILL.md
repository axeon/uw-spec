---
name: 310-java-uniweb-dev
description: Java SaaS后端开发技能。当需要基于UniWeb+Saas技术栈开发Java后端服务时触发：(1)实现Helper业务逻辑, (2)开发RESTful API接口, (3)集成DaoManager/FusionCache, (4)对接AIP/AIS/saas-finance, (5)指定模块开发或并行开发全部模块, (6)TDD Green实现全链路测试。当用户提及Java后端开发、uw-base、SaaS开发、API开发、Helper实现、指定模块、并行开发时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.1.0"
---

# Java SaaS开发

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 后端开发 + 全链路测试实现 | `java-developer` |
| 协作 | 技术方案确认 | `system-architect` |

> **职责边界**：全链路测试由 Java 开发工程师负责（TDD Green 阶段，从 `210-java-uniweb-design` 阶段产出的 Red 骨架实现为 Green）。测试工程师不参与单元测试，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `PROJECT_ROOT/requirement/prds/*` | 产品需求文档，功能模块及验收标准 |
| 数据库设计文档 | `PROJECT_ROOT/database/database-design.md` | 表结构、实体关系 |
| 后端设计文档 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 模块划分、接口设计、角色权限映射、缓存策略 |
| Controller代码 | `PROJECT_ROOT/backend/{项目名}-app/src/main/java/{包路径}/controller/` | `210-java-uniweb-design` 裁剪后的Controller |
| Helper代码 | `PROJECT_ROOT/backend/{项目名}-app/src/main/java/{包路径}/service/` | `210-java-uniweb-design` 新建的Helper方法签名 |
| DTO代码 | `PROJECT_ROOT/backend/{项目名}-app/src/main/java/{包路径}/dto/` | `210-java-uniweb-design` 裁剪后的DTO |
| 测试骨架代码 | `PROJECT_ROOT/backend/{项目名}-app/src/test/java/{包路径}/service/` | `210-java-uniweb-design` 产出的 Helper 测试骨架（TDD Red） |
| 测试用例设计 | `PROJECT_ROOT/test/design/` | 测试场景和用例覆盖（API/E2E/压测/安全） |

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

## `210-java-uniweb-design` → `310-java-uniweb-dev` 衔接协议

`210-java-uniweb-design` 设计完成后产出三样东西，`310-java-uniweb-dev` 消费关系如下：

| `210-java-uniweb-design` 产出物 | `310-java-uniweb-dev` 消费方式 | 用途 |
|-----------|------------|------|
| TASKS.md | **直接读取** | 获取任务卡片（PRD/文件/方法/依赖路径） |
| Helper 代码中的 `// TODO: [Tn]` | `grep "// TODO:" src/main/java/` | 定位待实现方法，确认边界 |
| Test 代码中的 `fail("TDD Red: [Tn]")` | `grep "TDD Red" src/test/java/` | 定位待变绿测试 |

### 任务卡片结构（`210-java-uniweb-design` 产出，`310-java-uniweb-dev` 直接使用）

每张卡片是一个 Sub Agent 的完整工作订单，包含所有需要的上下文路径：

| 字段 | 说明 |
|------|------|
| PRD | 关联的产品需求文档路径 |
| 数据库 | 关联的表结构文档位置 |
| 待实现 | Helper.java 文件路径 |
| 测试骨架 | HelperTest.java 文件路径 |
| 技术栈 | 需要读取的技术栈文档列表 |
| 待实现方法 | `// TODO: [Tn]` 标记的方法签名列表 |
| 依赖任务 | 前置任务 ID（无则可并行） |

### Sub Agent 执行流程

```
1. 读取 TASKS.md 任务卡片 → 加载卡片中列出的 PRD + 技术栈文档（4-6 个文件）
2. grep "// TODO: [Tn]" → 确认本任务的待实现方法列表
3. 实现 Helper 方法体 → 删除对应 // TODO 行
4. 替换 fail("TDD Red: [Tn]") 为真实断言 → 测试变绿
5. mvn test -pl {模块} → 全绿
6. 调用 `311-java-uniweb-dev-review` → 通过 → 进入下一模块
```

## 开发步骤

### 1. 环境准备 + 任务解析

**前置条件**：`300-database-ddl-execution` 已完成且 `301-database-ddl-execution-review` 评审通过（数据库表已就绪）。

1. 读取 `PROJECT_ROOT/backend/{项目名}-app/TASKS.md` 提取并行分组表和任务卡片列表
3. 读取 [dev-standards.md](../210-java-uniweb-design/references/backend/uniweb/dev-standards.md) 了解编码规范
4. **依赖拓扑确认**：按分组表确定执行顺序，禁止循环依赖

**降级策略**：若 TASKS.md 不存在，回退为"顺序全量"模式（扫描所有 `// TODO:` 标记，逐个实现）。输出警告后继续执行。

### 2. 模块开发（按任务卡片执行）

每个 Sub Agent 拿到一张任务卡片，执行以下步骤：

#### 2.1 加载上下文

**仅加载卡片中列出的文件**，不读取无关模块：

| 读取项 | 来源 |
|--------|------|
| PRD | 卡片中的 PRD 路径 |
| 数据库表结构 | 卡片中的数据库路径 |
| 技术栈文档 | 卡片中列出的技术栈文档 |
| Helper 代码 | 卡片中的待实现文件路径 |
| 测试骨架 | 卡片中的测试文件路径 |

#### 2.2 TODO 扫描确认

验证卡片中的"待实现方法"与代码中 `// TODO: [Tn]` 标记一致：

```bash
grep "// TODO: \[Tn\]" src/main/java/**/service/{Module}Helper.java
```

若标记数量不匹配，报告异常并停止。

#### 2.3 Helper 实现

> Helper 是业务逻辑的核心载体，位于 service 包下，`210-java-uniweb-design` 已创建签名和 Javadoc。

| 开发规范 | 说明 |
|---------|------|
| 类注解 | 无（纯静态工具类，不加 `@Component`） |
| 依赖获取 | `DaoManager.getInstance()` 静态获取，FusionCache/GlobalCache/GlobalLocker 本身是静态API |
| 方法风格 | **全部 static 方法**，`210-java-uniweb-design` 阶段已定义签名 |
| 数据访问 | 使用 `DaoManager.getInstance()`（ResponseData 链式 lambda 风格），不使用 DaoFactory |
| 查询参数 | 使用 QueryParam 系列（IdQueryParam、AuthQueryParam 等） |
| 批量操作 | 使用 `BatchUpdateManager` |
| 缓存 | 使用 `FusionCache`，Kryo 序列化用具体实现类（ArrayList/LinkedHashMap/HashSet） |
| 禁用 Lombok | 全局禁用，getter/setter/构造器均手写 |

**实现每个方法后**：删除对应的 `// TODO: [Tn]` 行。

#### 2.4 TDD Green — 实现全链路测试

> `210-java-uniweb-design` 阶段已产出测试骨架（Red），本阶段实现 Helper 逻辑并让测试逐个变绿。

**TDD 详细指南**：参见 [TDD设计指南](../210-java-uniweb-design/references/tdd-design-guide.md)

| 步骤 | 操作 |
|------|------|
| 读取测试骨架 | 读取卡片中的测试文件，了解测试意图 |
| 实现 Helper 方法 | 按 Javadoc 实现步骤填充方法体，删除 `// TODO` 行 |
| 实现测试断言 | 将 `fail("TDD Red: [Tn]")` 替换为真实数据库操作 + assert 断言 |
| 数据清理 | 在 `cleanTestData()` 中实现本测试方法的数据清理逻辑 |
| 逐个变绿 | 每实现一个 Helper 方法，对应测试从红变绿 |

**测试原则**：
- 使用 `@SpringBootTest` 启动完整 Spring 上下文，测试真实数据库交互
- 所有测试继承 `BaseIntegrationTest`，共享 Spring Context
- 使用 `TestAuthUtil.setTestUser()` 设置测试用户（saasId=666）
- 使用测试数据前缀 + `@AfterEach` 手动清理，不用事务回滚
- 按 `saas_id = 666` 清理测试数据
- DaoManager/FusionCache/GlobalLocker 均使用真实实例

#### 2.5 模块验证

| 检查项 | 命令 | 通过标准 |
|--------|------|---------|
| 无残留 TODO | `grep "// TODO: \[Tn\]" {Helper文件}` | 0 行 |
| 无残留 TDD Red | `grep "TDD Red: \[Tn\]" {Test文件}` | 0 行 |
| 模块测试通过 | `mvn test -pl {模块}` | 全绿 |

#### 2.6 REVIEW评审

模块开发完成后，调用 `311-java-uniweb-dev-review`，传入当前模块名。

- [ ] 已调用 `311-java-uniweb-dev-review`
- [ ] 已收到评审通过确认（评分 ≥ 95）

> 未收到通过确认前，禁止进入下一模块。

### 3. 联调验证（所有模块完成后）

> 所有模块开发完成后，执行全量联调验证。

执行 TASKS.md "联调验证"章节中列出的检查项：

| 检查项 | 命令 | 通过标准 |
|--------|------|---------|
| 无残留 TODO | `grep -r "// TODO:" src/main/java/` | 0 行 |
| 无残留 TDD Red | `grep -r "TDD Red" src/test/java/` | 0 行 |
| 全量编译 | `mvn compile` | BUILD SUCCESS |
| 全量测试通过 | `mvn test` | 全绿 |
| 应用启动正常 | 启动 + Swagger 可访问 | 无异常 |

**失败处理**：任一项不通过，定位具体模块修复后重新验证。

### 4. 全量 REVIEW评审

联调验证通过后，调用 `311-java-uniweb-dev-review`（不传模块名，触发全量评审）。

- [ ] 已调用 `311-java-uniweb-dev-review`（全量模式）
- [ ] 已收到评审通过确认（评分 ≥ 95）

> 未收到通过确认前，禁止结束开发任务。

## 并行开发约束

| 约束 | 说明 |
|------|------|
| 无依赖模块可并行 | 模块A、B互不依赖，可同时由不同Agent开发 |
| 有依赖模块需串行 | 模块C依赖A，需A完成review通过后才开发C |
| 共享文件只读 | entity/dto 由代码生成器产出，并行时只读不改 |
| Helper天然独立 | 各模块Helper在不同文件，无文件级冲突 |
| review按模块 | 每个模块开发完成后调用 `311-java-uniweb-dev-review`，通过后才继续下一模块 |

## 输出要求

- **代码位置**: `PROJECT_ROOT/backend/{项目名}-app/src/main/java/{包路径}/`
- **测试位置**: `PROJECT_ROOT/backend/{项目名}-app/src/test/java/{包路径}/service/`
- **包含内容**: Helper实现代码、Controller业务调用、全链路测试全绿、编译验证通过

## 参考

- [TDD设计指南](../210-java-uniweb-design/references/tdd-design-guide.md) - TDD三阶段映射、测试策略、基类模板
- [UniWeb开发规范](../210-java-uniweb-design/references/backend/uniweb/dev-standards.md) - 项目初始化、依赖、配置、编码规范
- [SaaS开发规范](../210-java-uniweb-design/references/backend/saas/README.md) - SaaS框架总览、模块速查、命名规范
- [Java开发评审技能](../311-java-uniweb-dev-review/SKILL.md) - REVIEW评审技能
