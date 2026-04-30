---
name: 630-feature-admin-uniapp-dev
description: 功能UniApp移动端开发技能。当技术方案设计完成后触发：(1)基于技术方案生成UniApp移动端代码, (2)实现跨平台页面组件（微信小程序/H5/App）, (3)对接后端API, (4)编写单元测试, (5)调用631-feature-admin-uniapp-dev-review自动评审, (6)自动修复代码问题, (7)合并代码到主分支并更新CHANGELOG ⚠️【强制】完成后必须调用 631-feature-admin-uniapp-dev-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能UniApp移动端开发

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码生成实现 + 单元测试 | `js-developer` |
| 协作 | 代码评审 | `js-lead` |
| 协作 | 平台适配 | `js-developer` |
| 协作 | 接口对接 | `java-developer` |

> **职责边界**：前端单元测试（composables/Store/工具函数）由前端工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 640）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 技术方案 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}/` | API接口定义 |
| 现有代码 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 功能代码 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/` | UniApp代码 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/**/*.spec.ts` | 单元测试（与源文件同目录） |
| 开发文档 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-admin-uniapp.md` | 开发记录 |
| 评审报告 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/reviews/` | AI评审报告 |
| 更新文档 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/README.md` | 合并后的主文档 |
| 变更记录 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/CHANGELOG.md` | 代码变更历史 |

## `610-feature-tech-design` → `630-feature-admin-uniapp-dev` 衔接协议

`610-feature-tech-design` 设计完成后，`630-feature-admin-uniapp-dev` 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 移动端方案 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/FEATURE-DESIGN-*-tech-design.md` | 页面结构、平台适配、交互流程 |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}/` | API接口定义 |

**并行约束**：`630-feature-admin-uniapp-dev` 与 `620-feature-java-uniweb-dev`/`630-feature-admin-web-dev`/`630-feature-guest-web-dev`/`630-feature-guest-uniapp-dev`/`640-feature-test-dev` 天然独立可并行。依赖后端 API 可访问。

## 执行流程

### 1. 代码生成

**生成内容**:

| 代码类型 | 生成位置 | 说明 |
|---------|---------|------|
| 页面组件 | `src/pages/` | UniApp页面 |
| 业务组件 | `src/components/` | 可复用组件 |
| API模块 | `src/api/` | API调用封装 |
| Store | `src/stores/` | Pinia状态管理 |
| 平台适配 | `src/platform/` | 平台差异化代码 |
| 单元测试 | `src/**/*.spec.ts` | Vitest测试（与源文件同目录） |

**代码规范**:
- Vue3 + TypeScript + `<script setup>`
- UniApp组件规范
- 条件编译处理平台差异（`#ifdef MP-WEIXIN`）
- uview-plus组件库

### 2. 平台适配

**适配维度**:

| 平台 | 适配点 | 处理方式 |
|------|--------|---------|
| 微信小程序 | 登录、支付、分享 | 微信SDK调用 |
| H5 | 浏览器兼容性、响应式 | CSS适配 |
| App | 原生能力、推送 | Plus API调用 |


## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `631-feature-admin-uniapp-dev-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

# FEATURE-DESIGN-{YYMMDD}-{功能名称} - UniApp移动端开发

## 实现概要
- 实现日期: {YYYY-MM-DD}
- 开发人员: AI Assistant
- 支持平台: 微信小程序 / H5 / App

## 代码变更
### 新增文件
- `{文件路径}`: {说明}

### 修改文件
- `{文件路径}`: {说明}

## 页面清单
| 页面 | 路径 | 平台支持 |
|------|------|---------|
| {name} | {path} | {platforms} |

## 平台适配
| 适配点 | 微信小程序 | H5 | App |
|--------|-----------|-----|-----|
| {point} | {impl} | {impl} | {impl} |

## API对接
| API | 用途 | 状态 |
|-----|------|------|
| {api} | {usage} | {status} |

## 测试覆盖
- 单元测试: {数量}个
- 覆盖率: {百分比}%

## 评审结果
- 评分: {分数}
- 状态: {通过/不通过}
- 遗留问题: {数量}个
```

### 7. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} FEATURE-{功能名称}
- 类型: UniApp移动端开发
- 变更: {简述}
- 平台: 微信小程序/H5/App
- 文档: FEATURE-DESIGN-{YYMMDD}-{简述}-admin-uniapp.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**代码位置**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/`

**开发文档**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-admin-uniapp.md`

**评审报告**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 技术方案
    ↓
630-feature-admin-uniapp-dev
    ↓
AI生成代码 → 平台适配 → AI评审(`631-feature-admin-uniapp-dev-review`) → 自动修复 → 测试执行
    ↓
输出: UniApp移动端代码
    ↓
等待: 650-feature-final-review（最终验收）
```

## 参考

- [UniApp开发评审技能](../631-feature-admin-uniapp-dev-review/SKILL.md) - REVIEW评审技能
- [UniApp文档](https://uniapp.dcloud.net.cn/)
