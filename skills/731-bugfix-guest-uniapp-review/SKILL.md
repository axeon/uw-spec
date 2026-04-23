---
name: 731-bugfix-guest-uniapp-review
description: Bug UniApp移动端修复评审技能（Guest）。当Bug UniApp代码修复完成后触发：(1)检查修复正确性和彻底性, (2)评审回归测试覆盖, (3)评审UniApp规范和跨平台兼容, (4)检查代码质量和安全性, (5)评估副作用范围, (6)输出评审结论。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug UniApp移动端修复评审（Guest）

## 描述
对Bug修复阶段（731）产出的Guest UniApp移动端代码进行评审, 验证修复正确性、回归测试、跨平台兼容和副作用评估。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| Bug移动端修复完成后 | "评审Bug修复代码" |
| 检查UniApp修复 | "检查移动端修复" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/` | Bug修复代码 |
| 回归测试 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/**/*.spec.ts` | 回归测试 |
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 710阶段方案 |
| Bug报告 | `PROJECT_ROOT/issue/bugs/BUG-{YYMMDD}-*.md` | 700阶段分析 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-CODE-{YYMMDDHHMM}.md` | 评审结论 |

## 流转关系
```
通过 → 合并代码, 可进入 750-bugfix-final-review
不通过 → 返回 730-bugfix-guest-uniapp 修改
```

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 修复正确性 | Bug根因已修复, 修复逻辑正确 |
| 彻底性 | 同类问题排查, 相关模块检查 |
| 回归测试 | 覆盖Bug场景和关联功能 |
| UniApp规范 | `<script setup>`, 条件编译, pages.json |
| 跨平台兼容 | 修复后多端正常, 无平台特定退化 |
| 代码质量 | 字段一致性, 命名规范 |
| 副作用评估 | 未引入新Bug, 变更范围可控 |
| 安全性 | Token安全, 输入校验未被破坏 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 10分 |
| 修复正确性 | 根因已修复, 逻辑正确 | 20分 |
| 彻底性 | 同类问题已排查 | 15分 |
| 回归测试 | 覆盖Bug场景 | 15分 |
| 跨平台兼容 | 多端正常 | 15分 |
| 副作用评估 | 无新Bug引入 | 15分 |
| 安全性 | 无安全风险 | 10分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划, 无Critical问题

### 不通过（<85分）
- 存在Critical问题, Major问题 >3个

## 评审流程

### 2. 执行评审
按维度检查。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

## 输出要求
**报告位置**: `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-CODE-{YYMMDDHHMM}.md`
