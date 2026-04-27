---
name: 321-guest-uniapp-dev-review
description: 移动端UniApp开发评审技能。当UniApp移动端代码开发完成后触发：(1)检查TDD测试驱动开发实践, (2)评审UniApp跨平台规范遵循, (3)检查多端兼容性H5/小程序/App, (4)评审Vue3组合式API代码质量, (5)检查移动端性能优化和体验, (6)检查组件复用和代码规范。请务必在用户提及UniApp代码评审、移动端评审、小程序代码检查、跨平台开发评审时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 移动端开发评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

> **职责边界**：前端单元测试质量由前端工程师负责，评审由 js-lead / system-architect 执行。测试工程师不参与单元测试评审，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [320-guest-uniapp-dev/SKILL.md](../320-guest-uniapp-dev/SKILL.md) | **必读全文**：开发规范、开发步骤、220→320 衔接 |
| [220-guest-uniapp-design/SKILL.md](../220-guest-uniapp-design/SKILL.md) | 设计阶段架构约定（开发阶段必须遵循） |
| [220-guest-uniapp-design/references/md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md) | UniApp 移动端编码规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 移动端代码 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/` | 开发完成的代码 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/**/*.spec.ts` | 组件测试 |
| 覆盖率报告 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/coverage/` | 测试覆盖率数据 |
| 移动端设计 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/README.md` | 220设计阶段产出的设计规范 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 移动端开发评审报告 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

> 每个维度的检查标准以 [320-guest-uniapp-dev/SKILL.md](../320-guest-uniapp-dev/SKILL.md) 和 [220-guest-uniapp-design/SKILL.md](../220-guest-uniapp-design/SKILL.md) 中的原始约定为准。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 |
|------|---------|-----------|
| TDD实践 | 测试先行、覆盖率≥70%、composables/Store/工具函数有测试 | `320-guest-uniapp-dev` → 开发步骤 |
| UniApp规范 | 正确使用uni API、条件编译（#ifdef）、生命周期 | `220-guest-uniapp-design` → references/md-dev-standards.md |
| 跨平台兼容 | 样式适配（rpx）、API兼容、功能降级 | `320-guest-uniapp-dev` → 开发步骤 |
| 代码质量 | 目录结构、命名规范、TypeScript类型完整、defineProps泛型风格 | `220-guest-uniapp-design` → references/md-dev-standards.md |
| 字段一致性 | 前端字段与后端DTO字段命名一致（camelCase）、表单绑定与API参数一致 | `220-guest-uniapp-design` → Phase 2 Step 4 |
| 性能优化 | 分包加载、图片优化、数据缓存、下拉刷新/上拉加载 | `320-guest-uniapp-dev` → 开发步骤 |
| 安全性 | 存储安全、传输安全（HTTPS）、代码安全 | `320-guest-uniapp-dev` → 开发步骤 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，测试覆盖率 ≥ 70%，UniApp 规范 100%，跨平台兼容，无性能问题 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或覆盖率不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。报告中需包含"覆盖率统计"扩展统计节。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: TDD/UniApp/兼容性/代码质量/字段一致性/性能/安全性
**评审对象**: PROJECT_ROOT/frontend/{项目名}-guest-uniapp/
**参与人员**: @js-lead @system-architect @js-developer


### 2. 评审结论

计算最终评分后，**必须逐项确认以下检查清单**：

**评分 ≥ 95（通过）：**
- [ ] 已标记评审状态为「通过」

**评分 < 95（不通过）：**
- [ ] 已调用 `320-guest-uniapp-dev` 技能，传入问题清单
- [ ] 修复完成后已重新执行本技能评审

> 以上检查项未全部勾选，禁止结束本技能任务。

