---
name: 221-admin-web-design-review
description: Web端设计与原型评审技能。当Web端设计与原型完成后触发：(1)评审设计规范完整性, (2)按前端类型Admin/Guest/Mch分别评审原型, (3)检查需求符合度和完整性, (4)评估用户体验和交互流畅性, (5)确认技术可行性和实现难度, (6)输出评审结论和改进建议。请务必在用户提及Web原型评审、Vue原型检查、前端设计评审、Web设计评审时使用此技能。适用于root/ops/saas/mch角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Web端原型评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 原型评审 | `prototype-reviewer` |
| 协作 | 需求符合度 | `product-manager` |
| 协作 | 技术可行性 | `system-architect` |
| 协作 | 实现难度 | `js-developer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [220-admin-web-design/SKILL.md](../220-admin-web-design/SKILL.md) | **必读全文**：架构约定、设计流程 Phase 0-2、设计完成标准 |
| [220-admin-web-design/references/web-dev-standards.md](../220-admin-web-design/references/web-dev-standards.md) | Vue3+TS+Vite+ElementPlus 开发规范 |
| [220-admin-web-design/references/web-design-spec.md](../220-admin-web-design/references/web-design-spec.md) | 色彩/字体/间距/响应式设计规范 |
| [220-admin-web-design/references/templates.md](../220-admin-web-design/references/templates.md) | README 模板、页面模板 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 前端原型项目 | `PROJECT_ROOT/frontend/{项目名}-admin-web/` | 220设计阶段产出的可运行原型 |
| README.md | `PROJECT_ROOT/frontend/{项目名}-admin-web/README.md` | 页面清单与路由设计文档 |
| 需求文档 | `PROJECT_ROOT/requirement/prds/*` | 功能需求参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| Web端原型评审报告 | `PROJECT_ROOT/frontend/{项目名}-admin-web/reviews/REVIEW-DESIGN-YYMMDDHHMM.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

> 每个维度的检查标准以 [220-admin-web-design/SKILL.md](../220-admin-web-design/SKILL.md) 中的原始约定为准，下表为概要。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 |
|------|---------|-----------|
| 需求符合度 | 功能覆盖、业务流程完整 | `220-admin-web-design` → Phase 0 需求确认 |
| 用户体验 | 布局合理、操作简洁、反馈明确 | `220-admin-web-design` → references/web-design-spec.md |
| 视觉设计 | 风格统一、配色合理、间距一致 | `220-admin-web-design` → references/web-design-spec.md |
| 技术可行性 | 交互可实现、性能可接受 | `220-admin-web-design` → Phase 2 Step 5 编译验证 |
| 一致性 | 组件复用、命名规范、状态完整、字段一致 | `220-admin-web-design` → Phase 2 Step 4 字段一致性 |
| **源技能约定** | 页面路径角色化、SFC结构顺序、导入路径修正、路由配置 | `220-admin-web-design` → 架构约定 + Phase 2 |

## 按前端类型评审

### 管理端（Admin）
| 评审项 | 检查要点 |
|--------|---------|
| 数据表格 | 筛选、排序、分页、批量操作 |
| 表单设计 | 字段分组、快捷编辑、验证反馈 |
| 权限控制 | 菜单权限、按钮权限、数据权限 |
| 数据可视化 | 图表直观、实时更新 |

### 游客端（Guest）
| 评审项 | 检查要点 |
|--------|---------|
| 首页吸引力 | 核心价值传达、CTA明确 |
| 功能路径 | 核心功能路径最短 |
| 新手引导 | 首次使用引导完善 |
| 空状态 | 无数据状态友好提示 |

### SaaS运营端（Saas）
| 评审项 | 检查要点 |
|--------|---------|
| 租户管理 | 租户列表、租户配置、套餐管理 |
| 数据隔离 | 租户间数据隔离、跨租户统计 |
| 计费管理 | 套餐订阅、用量统计、账单展示 |
| 系统配置 | 全局参数、通知模板、权限模板 |

### 商户端（Mch）
| 评审项 | 检查要点 |
|--------|---------|
| 工作台 | 数据概览、待办事项 |
| 商品管理 | 批量操作、快速编辑 |
| 订单管理 | 状态清晰、操作便捷 |
| 数据统计 | 多维度统计、趋势分析 |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，需求符合度 ≥ 95%，无重大体验问题，技术可行 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或需求符合度不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 需求符合度/用户体验/视觉设计/技术可行性/一致性
**评审对象**: PROJECT_ROOT/frontend/{项目名}-admin-web/
**参与人员**: @prototype-reviewer @product-manager @system-architect @js-developer


### 2. 评审结论

计算最终评分后，**必须逐项确认以下检查清单**：

**评分 ≥ 95（通过）：**
- [ ] 已标记评审状态为「通过」

**评分 < 95（不通过）：**
- [ ] 已调用 `220-admin-web-design` 技能，传入问题清单
- [ ] 修复完成后已重新执行本技能评审

> 以上检查项未全部勾选，禁止结束本技能任务。

