---
name: 621-feature-java-uniweb-dev-review
description: 功能Java后端开发评审技能。当功能Java后端代码开发完成后触发：(1)检查代码实现与技术方案一致性, (2)评审TDD实践和单元测试覆盖, (3)检查uw-base架构规范遵循, (4)验证多租户数据隔离, (5)评审代码质量和安全性, (6)输出评审结论和改进建议。请务必在用户提及功能后端评审、Feature Java评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能Java后端开发评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `java-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `java-developer` |

> **职责边界**: 单元测试质量由 Java 开发工程师负责, 评审由 java-lead / system-architect 执行。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 640）。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [620-feature-java-uniweb-dev/SKILL.md](../620-feature-java-uniweb-dev/SKILL.md) | **必读全文**：开发规范、TDD规则 |
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | 架构约定（开发阶段必须遵循） |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 功能开发产出的代码 |
| 测试代码 | `PROJECT_ROOT/backend/{项目名}-app/src/test/` | 单元测试 |
| 覆盖率报告 | `PROJECT_ROOT/backend/{项目名}-app/target/site/jacoco/` | 测试覆盖率数据 |
| 技术方案 | `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-*-tech-design.md` | 610阶段技术方案 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段需求文档 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 功能后端评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 方案一致性 | 代码实现与技术方案完全一致, 接口/DTO/Helper无偏差 |
| TDD实践 | 210 Red骨架→620 Green实现, 覆盖率≥80%, 无fail()残留 |
| UniWeb规范 | DaoManager/ResponseData/AuthQueryParam/FusionCache, 禁用Lombok |
| 代码质量 | 单一职责, 分层清晰, 命名规范, 圈复杂度≤10 |
| 安全性 | @Valid校验, QueryParam参数化, @MscPermDeclare, 租户隔离 |
| 影响范围 | 未修改无关模块, 变更范围可控 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，方案一致性 100%，测试覆盖率行 ≥ 80%/分支 ≥ 70%，uw-base 规范 100%，安全性达标 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或覆盖率不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查, 记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

**维度**: 方案一致性/TDD/uw-base/代码质量/安全性/影响范围
**评审对象**: PROJECT_ROOT/backend/{项目名}-app/src/（仅功能变更部分）
**参与人员**: @java-lead @system-architect

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `620-feature-java-uniweb-dev`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 3 轮）
4. 仅在通过或轮次耗尽时输出结果

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

