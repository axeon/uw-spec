---
name: 631-feature-admin-uniapp-dev-review
description: 功能UniApp移动端开发评审技能（Admin）。当功能UniApp代码开发完成后触发：(1)检查代码与技术方案一致性, (2)评审TDD实践, (3)评审UniApp规范和跨平台兼容, (4)检查代码质量和安全性, (5)评估功能影响范围, (6)输出评审结论。适用于root/ops/saas/mch角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能UniApp移动端开发评审（Admin）

## 描述
对功能开发阶段（631）产出的UniApp移动端代码进行评审, 验证代码实现与技术方案一致性、TDD实践、UniApp规范和跨平台兼容。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 功能移动端开发完成后 | "评审功能移动端代码" |
| 检查UniApp功能实现 | "检查UniApp实现" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能代码 | `frontend/{项目名}-admin-uniapp/src/` | 功能开发产出 |
| 测试代码 | `frontend/{项目名}-admin-uniapp/src/**/*.spec.ts` | 单元测试 |
| 技术方案 | `frontend/{项目名}-admin-uniapp/issues/FEATURE-DESIGN-*-tech-design.md` | 610阶段方案 |
| 功能需求 | `issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段需求 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md` | 评审结论 |

## 流转关系
```
通过 → 合并代码, 可进入 650-feature-final-review
不通过 → 返回 630-feature-admin-uniapp-dev 修改
```

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

## 量化通过标准

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

### 2. 执行评审
按维度检查。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

## 输出要求
**报告位置**: `issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`
