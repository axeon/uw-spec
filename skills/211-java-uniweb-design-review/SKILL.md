---
name: 211-java-uniweb-design-review
description: UniWeb后端设计评审技能（TDD驱动, 设计即代码模式）。当Java后端设计完成后触发：(1)评审README.md总体设计的完整性与合理性, (2)检查Controller/Helper/DTO代码质量与Javadoc完整性, (3)评审测试骨架质量（TDD Red阶段）, (4)验证缓存策略与uw-cache API使用正确性, (5)验证权限注解与角色映射一致性, (6)检查编译通过且测试可运行。当用户提及设计评审、架构评审、代码评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# UniWeb 后端设计评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | **必读全文**：架构约定、设计流程 Phase 0-2.5、设计完成标准 |
| [210-java-uniweb-design/references/templates.md](../210-java-uniweb-design/references/templates.md) | README 模板、代码模板、测试模板 |
| [210-java-uniweb-design/references/tdd-design-guide.md](../210-java-uniweb-design/references/tdd-design-guide.md) | TDD 设计阶段规则 |
| [210-java-uniweb-design/references/backend/uniweb/README.md](../210-java-uniweb-design/references/backend/uniweb/README.md) | UniWeb 技术栈规范 |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 架构评审 | `system-architect` |
| 协作 | 代码质量 | `java-developer` |
| 协作 | 业务确认 | `product-manager` |

## 输入

| 输入项 | 位置 | 说明 |
|--------|------|------|
| README.md | 后端项目根目录 | 总体设计文档 |
| TASKS.md | 后端项目根目录 | 开发任务分工 |
| Controller 代码 | `src/main/java/{包}/controller/` | 接口声明 + Javadoc |
| Helper 代码 | `src/main/java/{包}/service/` | 方法签名 + Javadoc + 缓存定义 |
| DTO 代码 | `src/main/java/{包}/dto/` | 裁剪后的 QueryParam |
| VO 代码 | `src/main/java/{包}/vo/` | 聚合数据（如有） |
| 测试骨架代码 | `src/PROJECT_ROOT/test/java/{包}/service/` | Helper 单元测试骨架（TDD Red） |
| PRD 文档 | `PROJECT_ROOT/requirement/` | 功能需求参考 |
| 数据库设计 | `PROJECT_ROOT/database/` | 数据模型参考 |

## 评审维度

> 每个维度的检查标准以 [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) 中的原始约定为准，下表为概要。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 | 分值 |
|------|---------|-----------|------|
| README.md 完整性 | 模块总览、依赖关系、PRD映射、权限映射、缓存策略、性能预算、测试策略 | `210-java-uniweb-design` → Phase 1 必须章节 | 20分 |
| TASKS.md 完整性 | 并行分组、任务卡片（含 Tn ID 和文件路径）、联调验证清单 | `210-java-uniweb-design` → Phase 1.5 | 10分 |
| 代码可编译性 | mvn compile 通过、mvn test-compile 通过、mvn test 可运行（全红为预期）、Swagger 可用 | `210-java-uniweb-design` → Phase 2 Step 5 | 必须 |
| Controller 质量 | 权限注解完整（含 guest 特殊规则）、Javadoc 完整、方法体有完整调用链路、URL 规范 | `210-java-uniweb-design` → Phase 2 Step 2 | 15分 |
| Helper 质量 | Helper 三条件合理性、方法签名合理、Javadoc 含实现步骤、FusionCache 初始化完成、static 方法风格 | `210-java-uniweb-design` → Phase 2 Step 3 | 15分 |
| 测试骨架质量 | 每个 Helper 有测试类、每个方法 ≥2 测试、继承 BaseIntegrationTest + @SpringBootTest、@DisplayName、fail("TDD Red: [Tn]") 与 TASKS.md ID 对应 | `210-java-uniweb-design` → Phase 2 Step 3.5 | 15分 |
| DTO/VO 质量 | 搜索字段裁剪合理、校验注解完整 | `210-java-uniweb-design` → Phase 2 Step 1 | 10分 |
| PRD 覆盖度 | 所有 PRD 功能点都有对应接口 | `210-java-uniweb-design` → Phase 1 PRD功能点映射 | 15分 |

## 通过标准

| 等级 | 分值 | Critical | Major | 编译 |
|------|------|----------|-------|------|
| 通过 | ≥ 95 | 0 | ≤1 | 必须通过 |
| 不通过 | < 95 | 存在 或 编译不通过 或 Major >1 | — | — |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 编译验证
- 执行 `mvn compile`，确认编译通过
- 执行 `mvn test-compile`，确认测试编译通过
- 执行 `mvn test`，确认测试可运行（全红为预期）
- 检查 Swagger 是否可用

### 2. 按维度评审
按上方评审维度逐项检查，记录问题。详细检查清单见 [checklist.md](references/checklist.md)。

### 3. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `210-java-uniweb-design`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 5 轮）
4. 仅在通过或轮次耗尽时输出结果

> 此流程全自动执行：中间不暂停、不询问、不汇报。
> 未收到通过确认前，禁止结束本技能任务。

## 输出

**报告位置**：`PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-DESIGN-YYMMDDHHMM.md`

**必须包含**：
- 评审信息（日期、人员、对象）
- 各维度评审结果和得分
- 问题清单（含严重程度、位置、状态）
- 量化评审结论
- 流转方向说明

## 参考

- [评审检查清单](references/checklist.md)
- [评审报告模板](../0-init/references/review-report-template.md)
- [UniWeb 技术栈](../210-java-uniweb-design/references/backend/uniweb/README.md)
