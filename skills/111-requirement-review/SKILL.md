---
name: 111-requirement-review
description: 需求评审技能。当需求规划文档完成后触发：(1)检查需求完整性和覆盖度, (2)评审技术可行性和资源可行性, (3)验证需求一致性和逻辑正确性, (4)评估需求可测试性和可验收性, (5)检查用户故事和验收标准, (6)输出评审结论和改进建议。请务必在用户提及需求评审、检查需求、PRD评审、需求验收、需求质量检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 需求评审

## 描述
对需求规划阶段产出的需求文档进行全面评审，确保需求的完整性、可行性、一致性和可测试性。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 需求文档完成后 | "评审需求文档" |
| 检查需求质量 | "检查需求是否有问题" |
| 需求评审 | "需求评审" |

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

## 流转关系
```
通过 → 进入阶段2-设计阶段
不通过 → 返回 110-requirement-planning 修改
```

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 完整性 | 背景、目标、功能需求、非功能需求、验收标准 |
| 可行性 | 技术可行、资源可行、业务可行、法律合规 |
| 一致性 | 术语一致、逻辑一致、前后一致、标准一致 |
| 可测试性 | 可验证、可度量、独立性 |

## 量化通过标准

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

### 1. 准备阶段
- **读取源技能**：读取 [110-requirement-planning/SKILL.md](../110-requirement-planning/SKILL.md) 全文，提取需求规划规范和交付物要求，作为评审的权威依据
- 读取需求文档
- 确认评审参与人员
- 准备评审检查单

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 完整性/可行性/一致性/可测试性
**评审对象**: PROJECT_ROOT/requirement/
**参与人员**: @project-manager @product-manager @system-architect @test-engineer
**流转方向**: 通过 -> 进入设计阶段；不通过 -> 返回需求规划修改

## 输出要求
**报告位置**: `PROJECT_ROOT/requirement/reviews/REVIEW-PRD-YYMMDDHHMM.md`

**命名规则**: 时间戳使用当前系统时间，24小时制，例如 `REVIEW-PRD-2604021430.md`

**按终端分别评审**:
| 终端 | 评审目录 |
|------|---------|
| 主文件 | `PROJECT_ROOT/requirement/prds/README.md` |
| guest | `PROJECT_ROOT/requirement/prds/guest-*` |
| admin | `PROJECT_ROOT/requirement/prds/admin-*` |
| saas | `PROJECT_ROOT/requirement/prds/saas-*` |
| mch | `PROJECT_ROOT/requirement/prds/mch-*` |
| root | `PROJECT_ROOT/requirement/prds/root-*` |

**必须包含**:
- 评审信息（日期、人员、对象）
- 各维度评审结果和得分
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项
