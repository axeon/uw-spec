---
name: 641-feature-test-dev-review
description: 功能测试脚本开发评审技能。当功能测试脚本开发完成后触发：(1)检查测试覆盖与技术方案一致性, (2)评审测试用例完整性, (3)评审测试代码质量, (4)检查测试数据合理性, (5)输出评审结论。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能测试脚本开发评审

## 描述
对功能开发阶段（640）产出的测试脚本进行评审, 验证测试覆盖与技术方案的一致性、用例完整性和代码质量。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 功能测试脚本开发完成后 | "评审功能测试代码" |
| 检查测试覆盖 | "检查测试完整性" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试评审 | `qa-lead` |
| 协作 | 方案确认 | `system-architect` |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 测试代码 | `PROJECT_ROOT/test/` | 测试脚本 |
| 技术方案 | `PROJECT_ROOT/test/issues/FEATURE-DESIGN-*-test-design.md` | 610阶段测试方案 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段需求 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-TEST-{YYMMDDHHMM}.md` | 评审结论 |

## 流转关系
```
通过 → 可进入 650-feature-final-review
不通过 → 返回 640-feature-test-dev 修改
```

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 方案一致性 | 测试用例覆盖技术方案中所有测试点 |
| 用例完整性 | 正常/异常/边界场景覆盖, ≥90%需求点 |
| 测试数据 | Mock数据合理, 无硬编码敏感信息 |
| 代码质量 | 命名规范, 断言明确, 无冗余 |
| 自动化质量 | 测试独立可重复, 无执行顺序依赖 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 20分 |
| 方案一致性 | 100%覆盖技术方案测试点 | 20分 |
| 用例完整性 | ≥90%需求点有测试 | 20分 |
| 测试数据 | Mock合理, 无敏感信息 | 20分 |
| 代码质量 | 命名规范, 断言明确 | 20分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划, 无Critical问题

### 不通过（<85分）
- 存在Critical问题, 需求覆盖率 <70%

## 评审流程

### 2. 执行评审
按维度检查。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

## 输出要求
**报告位置**: `PROJECT_ROOT/issue/reviews/REVIEW-TEST-{YYMMDDHHMM}.md`
