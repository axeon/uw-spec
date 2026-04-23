---
name: 631-feature-guest-web-dev-review
description: 功能Web前端开发评审技能（Guest）。当功能Web前端代码开发完成后触发：(1)检查代码与技术方案一致性, (2)评审TDD实践和测试覆盖, (3)评审Vue3规范遵循, (4)检查代码质量和安全性, (5)评估功能影响范围, (6)输出评审结论。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能Web前端开发评审（Guest）

## 描述
对功能开发阶段（630）产出的Guest Web前端代码进行评审, 验证代码实现与技术方案的一致性、TDD实践、Vue3规范和代码质量。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 功能前端开发完成后 | "评审功能前端代码" |
| 检查功能实现 | "检查Web功能实现" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/` | 功能开发产出的代码 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/**/*.spec.ts` | 单元测试 |
| 技术方案 | `PROJECT_ROOT/frontend/{项目名}-guest-web/issues/FEATURE-DESIGN-*-tech-design.md` | 610阶段方案 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段需求 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 评审报告 | `PROJECT_ROOT/issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md` | 评审结论和问题清单 |

## 流转关系
```
通过 → 合并代码, 可进入 650-feature-final-review
不通过 → 返回 630-feature-guest-web-dev 修改
```

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 方案一致性 | 代码实现与技术方案完全一致 |
| TDD实践 | composables/Store/工具函数有测试, 核心业务≥70%覆盖 |
| Vue3规范 | `<script setup>`, Composition API, TypeScript类型完整, 禁止any |
| 代码质量 | 目录结构、命名规范、字段一致性、组件拆分 |
| 性能优化 | 路由懒加载、v-for key、computed缓存 |
| 安全性 | 禁止v-html未转义、路由守卫权限控制、Token安全存储 |
| 影响范围 | 未修改无关模块, 变更范围可控 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 20分 |
| 方案一致性 | 100%实现技术方案 | 20分 |
| 测试覆盖率 | 核心业务≥70% | 15分 |
| Vue3规范 | 100%符合 | 15分 |
| 安全性 | 无XSS漏洞, 路由权限完整 | 15分 |
| 影响范围 | 无不必要变更 | 15分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划, 无Critical问题

### 不通过（<85分）
- 存在Critical问题, Major问题 >3个, 测试覆盖率 <50%

## 评审流程

### 1. 准备阶段
- 读取技术方案文档和功能需求
- 获取开发代码和测试代码

### 2. 执行评审
按维度检查, 记录问题。详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

## 输出要求
**报告位置**: `PROJECT_ROOT/issue/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`
