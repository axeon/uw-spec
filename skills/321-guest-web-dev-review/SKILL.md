---
name: 321-guest-web-dev-review
description: Web前端Vue3开发评审技能。当Vue3前端代码开发完成后触发：(1)检查TDD测试驱动开发实践, (2)评审Vue3组合式API规范, (3)检查Pinia状态管理设计, (4)评审组件代码质量和复用性, (5)检查前端性能优化和懒加载, (6)检查XSS等安全漏洞防护。请务必在用户提及Vue3代码评审、前端代码检查、Web开发评审、Element Plus评审时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Web前端开发评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 + 单元测试评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

> **职责边界**：前端单元测试质量由前端工程师负责，评审由 js-lead / system-architect 执行。测试工程师不参与单元测试评审，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [320-guest-web-dev/SKILL.md](../320-guest-web-dev/SKILL.md) | **必读全文**：开发规范、开发步骤、220→320 衔接 |
| [220-guest-web-design/SKILL.md](../220-guest-web-design/SKILL.md) | 设计阶段架构约定（开发阶段必须遵循） |
| [220-guest-web-design/references/web-dev-standards.md](../220-guest-web-design/references/web-dev-standards.md) | Vue3+TS+Vite 编码规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| Web前端代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/` | 开发完成的代码 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/**/*.spec.ts` | 单元测试（与源文件同目录） |
| 覆盖率报告 | `PROJECT_ROOT/frontend/{项目名}-guest-web/coverage/` | 测试覆盖率数据 |
| 前端设计 | `PROJECT_ROOT/frontend/{项目名}-guest-web/README.md` | 220设计阶段产出的设计规范 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| Web前端开发评审报告 | `PROJECT_ROOT/frontend/{项目名}-guest-web/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

> 每个维度的检查标准以 [320-guest-web-dev/SKILL.md](../320-guest-web-dev/SKILL.md) 和 [220-guest-web-design/SKILL.md](../220-guest-web-design/SKILL.md) 中的原始约定为准。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 |
|------|---------|-----------|
| TDD实践 | composables/Store/工具函数有测试、覆盖率≥70%、Vitest + Vue Test Utils | `320-guest-web-dev` → 开发步骤 |
| Vue3规范 | `<script setup>`、Composition API、TypeScript类型定义、禁用any | `220-guest-web-design` → references/web-dev-standards.md |
| 状态管理 | Store按模块划分、Actions职责单一、Pinia setup风格 | `320-guest-web-dev` → 开发步骤 |
| 代码质量 | 目录结构、命名规范、类型完整、字段一致性 | `220-guest-web-design` → references/web-dev-standards.md |
| 字段一致性 | 前端字段与后端DTO字段命名一致（camelCase）、表单绑定与API参数一致 | `220-guest-web-design` → Phase 2 Step 4 |
| 性能优化 | 路由懒加载、v-for key、computed缓存 | `320-guest-web-dev` → 开发步骤 |
| 安全性 | 禁止v-html未转义、路由守卫权限控制、Token安全存储 | `320-guest-web-dev` → 开发步骤 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，测试覆盖率 ≥ 70%，Vue3 规范 100%，TypeScript 覆盖率 ≥ 90%，无性能问题 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。报告中需包含"覆盖率统计"扩展统计节。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: TDD/Vue3/状态管理/代码质量/性能/安全性
**评审对象**: PROJECT_ROOT/frontend/{项目名}-guest-web/
**参与人员**: @js-lead @system-architect @js-developer


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `320-guest-web-dev`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 3 轮）
4. 仅在通过或轮次耗尽时输出结果

> 此流程全自动执行：中间不暂停、不询问、不汇报。
> 未收到通过确认前，禁止结束本技能任务。

