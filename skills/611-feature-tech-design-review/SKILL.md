---
name: 611-feature-tech-design-review
description: 功能技术方案评审技能。当功能技术方案设计完成后触发：(1)评审数据库设计合理性, (2)评审后端接口设计规范性, (3)评审前端页面设计可行性, (4)检查测试策略完整性, (5)评估方案与现有架构兼容性, (6)输出评审结论和改进建议。请务必在用户提及技术方案评审、功能设计评审、技术设计检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能技术方案评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 技术方案评审 | `system-architect` |
| 协作 | 数据库设计确认 | `java-developer` |
| 协作 | 前端设计确认 | `js-developer` |
| 协作 | 测试策略确认 | `test-engineer` |
| 决策 | 方案确认 | `product-manager` ★ |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [610-feature-tech-design/SKILL.md](../610-feature-tech-design/SKILL.md) | **必读全文**：技术方案设计规范、交付物规范 |
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | 后端架构约定（如涉及后端） |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能需求文档 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段输出 |
| 技术方案文档 | 各端 `issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| DDL文件 | `PROJECT_ROOT/database/migrations/FEATURE-{YYMMDD}-{简述}.sql` | 610阶段输出 |
| 现有架构文档 | 各端 README.md | 现有架构参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 技术方案评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-DESIGN-{YYMMDDHHMM}.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 数据库设计 | 表结构合理、命名规范、索引充分、DDL可执行、与现有设计兼容 |
| 后端接口设计 | 接口规范、权限注解、DTO设计、Helper签名、与现有代码一致 |
| 前端页面设计 | 页面结构、路由配置、组件设计、字段一致性、与现有页面风格统一 |
| 测试策略 | 测试类型覆盖、场景完整、数据准备方案 |
| 兼容性 | 不破坏现有功能、DDL向后兼容、接口向后兼容 |
| PRD覆盖度 | 技术方案覆盖所有功能需求点 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，数据库/后端/前端设计规范，PRD 覆盖 100%，无破坏性变更 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或 PRD 覆盖不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查, 记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

**维度**: 数据库/后端/前端/测试/兼容性/PRD覆盖度
**评审对象**: PROJECT_ROOT/issue/features/ + 各端 issues/
**参与人员**: @system-architect @java-developer @js-developer @test-engineer

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `610-feature-tech-design`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 5 轮）
4. 仅在通过或轮次耗尽时输出结果

> 此流程全自动执行：中间不暂停、不询问、不汇报。
> 未收到通过确认前，禁止结束本技能任务。

