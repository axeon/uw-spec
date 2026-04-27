---
name: 721-bugfix-java-uniweb-dev-review
description: Bug Java后端修复评审技能。当Bug Java后端修复代码完成后触发：(1)检查修复正确性和彻底性, (2)评审回归测试覆盖, (3)检查uw-base架构规范遵循, (4)评估修复副作用和影响范围, (5)评审代码质量和安全性, (6)输出评审结论和改进建议。请务必在用户提及Bug后端修复评审、Java修复检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug Java后端修复评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `java-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `java-developer` |

> **职责边界**: 回归测试质量由 Java 开发工程师负责, 评审由 java-lead / system-architect 执行。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 740）。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [720-bugfix-java-uniweb/SKILL.md](../720-bugfix-java-uniweb/SKILL.md) | **必读全文**：Bug修复开发规范 |
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | 架构约定（修复代码必须遵循） |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 修复后的代码 |
| 回归测试代码 | `PROJECT_ROOT/backend/{项目名}-app/src/test/` | 回归测试 |
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| Bug修复评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-CODE-{YYMMDDHHMM}.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 修复正确性 | Bug已修复, 根因已解决, 边界条件处理 |
| 彻底性 | 同类问题已一并修复, 无遗漏 |
| 回归测试 | Bug复现测试, 边界测试, 原有功能未破坏 |
| UniWeb规范 | DaoManager/ResponseData/FusionCache, 禁用Lombok |
| 代码质量 | 修复简洁, 无冗余代码, 命名规范 |
| 副作用评估 | 未引入新问题, 变更范围可控 |
| 安全性 | 修复未引入安全漏洞 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，Bug 场景 100% 修复，有回归测试，uw-base 规范 100%，无副作用 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或无回归测试 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查, 记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

**维度**: 修复正确性/彻底性/回归测试/uw-base/代码质量/副作用/安全性
**评审对象**: PROJECT_ROOT/backend/{项目名}-app/src/（仅修复变更部分）
**参与人员**: @java-lead @system-architect

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论

计算最终评分后，**必须逐项确认以下检查清单**：

**评分 ≥ 95（通过）：**
- [ ] 已标记评审状态为「通过」

**评分 < 95（不通过）：**
- [ ] 已调用 `720-bugfix-java-uniweb` 技能，传入问题清单
- [ ] 修复完成后已重新执行本技能评审

> 以上检查项未全部勾选，禁止结束本技能任务。

