---
name: 221-guest-uniapp-design-review
description: 移动端设计与原型评审技能。当移动端设计与原型完成后触发：(1)评审设计规范完整性, (2)按端类型H5/小程序/App分别评审, (3)检查移动端触摸交互特性, (4)评估用户体验和易用性, (5)确认技术可行性和实现难度, (6)输出评审结论和改进建议。请务必在用户提及移动端原型评审、UniApp原型检查、移动端设计评审、小程序设计评审时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 移动端原型评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 原型评审 | `prototype-reviewer` |
| 协作 | 需求符合度 | `product-manager` |
| 协作 | 技术可行性 | `system-architect` |
| 协作 | 移动端实现 | `js-developer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [220-guest-uniapp-design/SKILL.md](../220-guest-uniapp-design/SKILL.md) | **必读全文**：架构约定、设计流程、设计完成标准 |
| [220-guest-uniapp-design/references/md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md) | UniApp 移动端开发规范 |
| [220-guest-uniapp-design/references/md-design-spec.md](../220-guest-uniapp-design/references/md-design-spec.md) | 移动端设计规范 |
| [220-guest-uniapp-design/references/templates.md](../220-guest-uniapp-design/references/templates.md) | README 模板、页面模板 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 移动端原型 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/` | 原型设计文件 |
| 需求文档 | `PROJECT_ROOT/requirement/prds/*` | 功能需求参考 |
| 移动端设计 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/README.md` | 设计规范参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 移动端原型评审报告 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/reviews/REVIEW-DESIGN-YYMMDDHHMM.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

> 每个维度的检查标准以 [220-guest-uniapp-design/SKILL.md](../220-guest-uniapp-design/SKILL.md) 中的原始约定为准。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 |
|------|---------|-----------|
| 需求符合度 | 功能覆盖、业务流程完整 | 220 → Phase 0 需求确认 |
| 移动端体验 | 触摸操作、手势支持、适配 | 220 → references/md-design-spec.md |
| 视觉设计 | 风格统一、字体合适、间距合理 | 220 → references/md-design-spec.md |
| 技术可行性 | 跨平台兼容、性能优化 | 220 → Phase 2 Step 5 |
| 一致性 | 组件复用、交互统一、字段一致 | 220 → Phase 2 Step 4 |
| **源技能约定** | 页面路径角色化、pages.json配置、导入路径修正 | 220 → 架构约定 + Phase 2 |

## 移动端专项评审

### 触摸与手势
| 评审项 | 检查要点 |
|--------|---------|
| 点击区域 | 按钮≥44x44px，间距足够 |
| 手势操作 | 滑动、下拉刷新、左滑删除 |
| 反馈及时 | 点击态、加载态、操作反馈 |
| 防误触 | 重要操作二次确认 |

### 屏幕适配
| 评审项 | 检查要点 |
|--------|---------|
| 响应式 | 适配不同屏幕尺寸 |
| 安全区 | 适配刘海屏、圆角屏 |
| 横竖屏 | 是否支持横屏，布局适配 |
| 字体缩放 | 支持系统字体大小调整 |

### 性能体验
| 评审项 | 检查要点 |
|--------|---------|
| 页面加载 | 骨架屏、懒加载 |
| 图片优化 | 压缩、WebP、渐进加载 |
| 列表性能 | 虚拟列表、分页加载 |
| 内存管理 | 页面销毁、资源释放 |

### 原生能力
| 评审项 | 检查要点 |
|--------|---------|
| 相机/相册 | 图片选择、拍照、预览 |
| 地理位置 | 定位精度、权限处理 |
| 扫码功能 | 二维码扫描、识别 |
| 推送通知 | 本地推送、远程推送 |

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 30分 |
| 需求符合度 | ≥95%功能覆盖 | 20分 |
| 移动端体验 | 符合移动端规范 | 25分 |
| 技术可行性 | 跨平台可实现 | 25分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（如：功能缺失、严重体验问题）
- Major问题 >3个

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。报告中需包含"移动端专项检查"扩展统计节。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 需求符合度/移动端体验/视觉设计/技术可行性/一致性
**评审对象**: PROJECT_ROOT/frontend/{项目名}-guest-uniapp/
**参与人员**: @prototype-reviewer @product-manager @system-architect @js-developer


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `220-guest-uniapp-design` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

