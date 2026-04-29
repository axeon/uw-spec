---
name: 711-bugfix-tech-design-review
description: Bug修复方案评审技能。当Bug修复方案设计完成后触发：(1)评审修复方案正确性和彻底性, (2)评估数据库变更影响, (3)评估接口变更影响, (4)检查向后兼容性, (5)评审回滚方案可行性, (6)输出评审结论和改进建议。请务必在用户提及修复方案评审、Bug设计评审、修复方案检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug修复方案评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 方案评审 | `system-architect` |
| 协作 | 变更评估 | `java-developer` / `js-developer` |
| 协作 | 测试评估 | `test-engineer` |
| 决策 | 方案确认 | `product-manager` ★ |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [710-bugfix-tech-design/SKILL.md](../710-bugfix-tech-design/SKILL.md) | **必读全文**：修复方案设计规范、交付物规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 修复方案文档 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| DDL文件（如有） | `PROJECT_ROOT/database/migrations/BUGFIX-{YYMMDD}-{简述}.sql` | 710阶段输出 |
| 相关代码 | 代码仓库 | 需要修改的代码 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复方案评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-DESIGN-{YYMMDDHHMM}.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 根因理解 | 修复方案针对根因, 非表面修补 |
| 修复正确性 | 修复逻辑正确, 边界条件处理完整 |
| 彻底性 | 同类问题是否一并修复, 无遗漏 |
| 数据库变更 | DDL语法正确, 向后兼容, 有回滚方案 |
| 接口变更 | 不破坏现有接口, 兼容性保证 |
| 影响范围 | 变更范围最小化, 无不必要修改 |
| 回滚方案 | 回滚步骤明确, 数据可恢复 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，修复针对根因，逻辑正确边界完整，同类问题已覆盖，无破坏性变更，回滚方案可行 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或未针对根因修复 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查, 记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

**维度**: 根因理解/修复正确性/彻底性/数据库变更/接口变更/影响范围/回滚方案
**评审对象**: PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-*
**参与人员**: @system-architect @java-developer @test-engineer

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `710-bugfix-tech-design`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 3 轮）
4. 仅在通过或轮次耗尽时输出结果

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

