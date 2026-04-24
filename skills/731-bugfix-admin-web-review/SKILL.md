---
name: 731-bugfix-admin-web-review
description: Bug Web前端修复评审技能（Admin）。当Bug Web前端代码修复完成后触发：(1)检查修复正确性和彻底性, (2)评审回归测试覆盖, (3)评审Vue3规范遵循, (4)检查代码质量和安全性, (5)评估副作用范围, (6)输出评审结论。适用于root/ops/saas/mch角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug Web前端修复评审（Admin）

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [730-bugfix-admin-web/SKILL.md](../730-bugfix-admin-web/SKILL.md) | **必读全文**：Bug修复开发规范 |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/frontend/{项目名}-admin-web/src/` | Bug修复代码 |
| 回归测试 | `PROJECT_ROOT/frontend/{项目名}-admin-web/src/**/*.spec.ts` | 回归测试 |
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 710阶段方案 |
| Bug报告 | `PROJECT_ROOT/issue/bugs/BUG-{YYMMDD}-*.md` | 700阶段分析 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-CODE-{YYMMDDHHMM}.md` | 评审结论 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 修复正确性 | Bug根因已修复, 修复逻辑正确, 边界场景覆盖 |
| 彻底性 | 同类问题排查, 相关模块检查 |
| 回归测试 | 有回归测试, 覆盖Bug场景和关联功能 |
| Vue3规范 | `<script setup>`, Composition API, TypeScript类型 |
| 代码质量 | 命名规范, 字段一致性, 无临时方案 |
| 副作用评估 | 未引入新Bug, 变更范围可控 |
| 安全性 | 无XSS/注入等安全风险 |

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 15分 |
| 修复正确性 | 根因已修复, 逻辑正确 | 20分 |
| 彻底性 | 同类问题已排查 | 15分 |
| 回归测试 | 覆盖Bug场景 | 15分 |
| Vue3规范 | 100%符合 | 10分 |
| 副作用评估 | 无新Bug引入 | 15分 |
| 安全性 | 无安全风险 | 10分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划, 无Critical问题

### 不通过（<85分）
- 存在Critical问题, Major问题 >3个

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `730-bugfix-admin-web` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

