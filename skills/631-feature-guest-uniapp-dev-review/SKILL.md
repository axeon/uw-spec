---
name: 631-feature-guest-uniapp-dev-review
description: 功能UniApp移动端开发评审技能（Guest）。当功能UniApp代码开发完成后触发：(1)检查代码与技术方案一致性, (2)评审TDD实践, (3)评审UniApp规范和跨平台兼容, (4)检查代码质量和安全性, (5)评估功能影响范围, (6)输出评审结论。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能UniApp移动端开发评审（Guest）

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
| [630-feature-guest-uniapp-dev/SKILL.md](../630-feature-guest-uniapp-dev/SKILL.md) | **必读全文**：开发规范、完成标准 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能代码 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/` | 功能开发产出 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/**/*.spec.ts` | 单元测试 |
| 技术方案 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-*-tech-design.md` | 610阶段方案 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段需求 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md` | 评审结论 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 方案一致性 | 代码实现与技术方案完全一致 |
| TDD实践 | composables/Store/工具函数有测试, 核心业务≥70%覆盖 |
| UniApp规范 | `<script setup>`, 条件编译, pages.json配置正确 |
| 跨平台兼容 | 样式适配rpx, 无平台API硬编码, 功能降级方案 |
| 代码质量 | 目录结构、命名规范、字段一致性 |
| 性能优化 | 分包加载、图片优化、列表渲染 |
| 安全性 | Token安全存储、输入校验、敏感数据保护 |
| 影响范围 | 未修改无关模块 |

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 15分 |
| 方案一致性 | 100%实现技术方案 | 15分 |
| 测试覆盖率 | 核心业务≥70% | 15分 |
| UniApp规范 | 100%符合 | 15分 |
| 跨平台兼容 | 多端正常运行 | 15分 |
| 安全性 | Token安全, 输入校验 | 15分 |
| 影响范围 | 无不必要变更 | 10分 |

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

评分 < 95 → **不通过**，调用 `630-feature-guest-uniapp-dev` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

