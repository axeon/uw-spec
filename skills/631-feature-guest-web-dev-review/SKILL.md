---
name: 631-feature-guest-web-dev-review
description: 功能Web前端开发评审技能（Guest）。当功能Web前端代码开发完成后触发：(1)检查代码与技术方案一致性, (2)评审TDD实践和测试覆盖, (3)评审Vue3规范遵循, (4)检查代码质量和安全性, (5)评估功能影响范围, (6)输出评审结论。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能Web前端开发评审（Guest）

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
| [630-feature-guest-web-dev/SKILL.md](../630-feature-guest-web-dev/SKILL.md) | **必读全文**：开发规范、完成标准 |

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

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

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

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，方案一致性 100%，测试覆盖率 ≥ 70%，Vue3 规范 100%，安全性达标，无不必要变更 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或覆盖率不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查, 记录问题。详见 [评审报告模板](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `630-feature-guest-web-dev`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 5 轮）
4. 仅在通过或轮次耗尽时输出结果

> 此流程全自动执行：中间不暂停、不询问、不汇报。
> 未收到通过确认前，禁止结束本技能任务。

