---
name: 741-bugfix-test-review
description: Bug修复回归测试评审技能。当Bug回归测试脚本开发完成后触发：(1)检查回归测试覆盖Bug场景, (2)评审回归范围合理性, (3)评审测试代码质量, (4)输出评审结论。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug修复回归测试评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试评审 | `qa-lead` |
| 协作 | 方案确认 | `system-architect` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [740-bugfix-test/SKILL.md](../740-bugfix-test/SKILL.md) | **必读全文**：回归测试规范 |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 回归测试代码 | `PROJECT_ROOT/test/` | 回归测试脚本 |
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 710阶段方案 |
| Bug报告 | `PROJECT_ROOT/issue/bugs/BUG-{YYMMDD}-*.md` | 700阶段分析 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-TEST-{YYMMDDHHMM}.md` | 评审结论 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| Bug场景覆盖 | 有针对Bug根因的回归测试 |
| 回归范围 | 关联功能有回归, 无遗漏 |
| 回归深度 | 正常/异常/边界场景覆盖 |
| 代码质量 | 命名规范, 断言明确 |
| 防复发 | 测试可防止同类Bug再次出现 |

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 20分 |
| Bug场景覆盖 | Bug根因有回归测试 | 25分 |
| 回归范围 | 关联功能覆盖 | 20分 |
| 代码质量 | 命名规范, 断言明确 | 20分 |
| 防复发 | 可防止同类Bug | 15分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划, 无Critical问题

### 不通过（<85分）
- 存在Critical问题, Bug场景无回归测试

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `740-bugfix-test` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

