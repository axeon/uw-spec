---
name: 111-requirement-review
description: 需求评审技能。当需求规划文档完成后触发：(1)检查需求完整性和覆盖度, (2)评审技术可行性和资源可行性, (3)验证需求一致性和逻辑正确性, (4)评估需求可测试性和可验收性, (5)检查用户故事和验收标准, (6)输出评审结论和改进建议。请务必在用户提及需求评审、检查需求、PRD评审、需求验收、需求质量检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 需求评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 需求评审 | `project-manager` |
| 协作 | 需求解释 | `product-manager` |
| 协作 | 技术可行性 | `system-architect` |
| 协作 | 可测试性 | `test-engineer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [110-requirement-planning/SKILL.md](../110-requirement-planning/SKILL.md) | **必读全文**：需求规划规范、交付物规范 |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 需求文档 | `PROJECT_ROOT/requirement/prds/` | PRD、用户故事、验收标准 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 需求评审报告 | `PROJECT_ROOT/requirement/reviews/REVIEW-PRD-YYMMDDHHMM.md` | 时间戳24小时制 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 完整性 | 背景、目标、功能需求、非功能需求、验收标准 |
| 可行性 | 技术可行、资源可行、业务可行、法律合规 |
| 一致性 | 术语一致、逻辑一致、前后一致、标准一致 |
| 可测试性 | 可验证、可度量、独立性 |

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个且有解决方案 | 30分 |
| 需求完整度 | ≥95% | 25分 |
| 验收标准可测试 | 100% | 25分 |
| 风险可控 | 有应对措施 | 20分 |

### 有条件通过（85-94分）
- Major问题 ≤5个且有明确修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题
- Major问题 >5个
- 需求完整度 <80%

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 完整性/可行性/一致性/可测试性
**评审对象**: PROJECT_ROOT/requirement/
**参与人员**: @project-manager @product-manager @system-architect @test-engineer


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `110-requirement-planning` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

