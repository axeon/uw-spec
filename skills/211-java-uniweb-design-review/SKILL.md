---
name: 211-java-uniweb-design-review
description: SaaS后端设计评审技能（TDD驱动, 设计即代码模式）。当Java后端设计完成后触发：(1)评审README.md总体设计的完整性与合理性, (2)检查Controller/Helper/DTO代码质量与Javadoc完整性, (3)评审测试骨架质量（TDD Red阶段）, (4)验证缓存策略与uw-cache API使用正确性, (5)验证权限注解与角色映射一致性, (6)检查编译通过且测试可运行。当用户提及设计评审、架构评审、代码评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# SaaS 后端设计评审

## 描述

对基于 UniWeb + SaaS 技术栈的后端设计进行全面评审。评审对象为 README.md 总体设计文档 + 代码质量（Controller/Helper/DTO 的 Javadoc 和签名）+ 测试骨架质量（TDD Red 阶段）。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 设计完成后 | "评审后端设计" |
| Review 循环 | "执行设计评审" |
| 代码质量检查 | "检查Javadoc完整性" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 架构评审 | `system-architect` |
| 协作 | 代码质量 | `java-developer` |
| 协作 | 业务确认 | `product-manager` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | **必读全文**：架构约定、设计流程 Phase 0-2.5、设计完成标准 |
| [210-java-uniweb-design/references/templates.md](../210-java-uniweb-design/references/templates.md) | README 模板、代码模板、测试模板 |
| [210-java-uniweb-design/references/tdd-design-guide.md](../210-java-uniweb-design/references/tdd-design-guide.md) | TDD 设计阶段规则 |
| [210-java-uniweb-design/references/backend/uniweb/README.md](../210-java-uniweb-design/references/backend/uniweb/README.md) | UniWeb 技术栈规范 |

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

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-DESIGN-YYMMDDHHMM.md` | 评审结论和问题清单 |

## 流转关系

```
通过 → 进入开发阶段（310-java-uniweb-dev）
不通过 → 返回 210-java-uniweb-design 修改
```

## 评审维度

> 每个维度的检查标准以 [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) 中的原始约定为准，下表为概要。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 | 分值 |
|------|---------|-----------|------|
| README.md 完整性 | 模块总览、依赖关系、PRD映射、权限映射、缓存策略、性能预算、测试策略 | 210 → Phase 1 必须章节 | 20分 |
| TASKS.md 完整性 | 并行分组、任务卡片（含 Tn ID 和文件路径）、联调验证清单 | 210 → Phase 1.5 | 10分 |
| 代码可编译性 | mvn compile 通过、mvn test-compile 通过、mvn test 可运行（全红为预期）、Swagger 可用 | 210 → Phase 2 Step 5 | 必须 |
| Controller 质量 | 权限注解完整（含 guest 特殊规则）、Javadoc 完整、方法体有完整调用链路、URL 规范 | 210 → Phase 2 Step 2 | 15分 |
| Helper 质量 | Helper 三条件合理性、方法签名合理、Javadoc 含实现步骤、FusionCache 初始化完成、static 方法风格 | 210 → Phase 2 Step 3 | 15分 |
| 测试骨架质量 | 每个 Helper 有测试类、每个方法 ≥2 测试、@ExtendWith+MockedStatic、@DisplayName、fail("TDD Red: [Tn]") 与 TASKS.md ID 对应 | 210 → Phase 2 Step 3.5 | 15分 |
| DTO/VO 质量 | 搜索字段裁剪合理、校验注解完整 | 210 → Phase 2 Step 1 | 10分 |
| PRD 覆盖度 | 所有 PRD 功能点都有对应接口 | 210 → Phase 1 PRD功能点映射 | 15分 |

## 量化通过标准

### 通过（≥95分）

| 检查项 | 标准 |
|--------|------|
| 编译通过 | `mvn compile` 无错误 |
| 测试编译通过 | `mvn test-compile` 无错误 |
| 测试可运行 | `mvn test` 可运行（全红为预期，无编译/运行时错误） |
| Swagger 可用 | 所有接口可展示 |
| 无 Critical 问题 | 0个 |
| Major 问题 | ≤1个 |
| README.md | 所有必须章节完整（含测试策略） |
| 测试骨架 | 每个 Helper 有测试类，每个方法 ≥2 个测试 |
| PRD 覆盖度 | 100% 功能点有对应接口 |

### 有条件通过（85-94分）
- Major 问题 ≤3个且有修复计划
- 无 Critical 问题

### 不通过（<85分）
- 存在 Critical 问题
- 编译不通过
- Major 问题 >3个

## 评审流程

### 1. 准备阶段
- **读取源技能**：读取 [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) 全文，提取所有架构约定和设计完成标准，作为评审的权威依据
- 读取 README.md
- 读取 TASKS.md
- 读取 PRD 文档和数据库设计
- 列出所有 Controller/Helper/DTO 文件清单

### 2. 编译验证
- 执行 `mvn compile`，确认编译通过
- 执行 `mvn test-compile`，确认测试编译通过
- 执行 `mvn test`，确认测试可运行（全红为预期）
- 检查 Swagger 是否可用

### 3. 执行评审
按维度检查，记录问题。评审检查清单详见 [checklist.md](references/checklist.md)。

**评审对象**：README.md + Controller/Helper/DTO 代码 + 测试骨架代码
**参与人员**：@system-architect @java-developer @product-manager
**流转方向**：通过 → 进入开发阶段；不通过 → 返回设计修改

## 输出要求

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
