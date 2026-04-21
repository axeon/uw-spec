---
name: 631-feature-guest-uniapp-dev
description: 功能UniApp移动端开发技能。当技术方案设计完成后触发：(1)基于技术方案生成UniApp移动端代码, (2)实现跨平台页面组件（微信小程序/H5/App）, (3)对接后端API, (4)编写单元测试, (5)调用321-guest-uniapp-dev-review自动评审, (6)自动修复代码问题, (7)合并代码到主分支并更新CHANGELOG。请务必在用户提及UniApp开发、移动端开发、小程序开发时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能UniApp移动端开发

## 描述

基于610阶段的技术方案，使用AI驱动的方式生成UniApp移动端代码（Vue3 + TypeScript），支持微信小程序、H5、App三端，自动完成实现、评审、修复、测试的完整闭环。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| UniApp开发 | "实现订单导出移动端" |
| 小程序开发 | "开发微信小程序导出功能" |
| 移动端适配 | "适配移动端导出功能" |

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
| 技术方案 | `frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义 |
| 现有代码 | `frontend/{项目名}-guest-uniapp/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 功能代码 | `frontend/{项目名}-guest-uniapp/src/` | UniApp代码 |
| 测试代码 | `frontend/{项目名}-guest-uniapp/src/**/*.spec.ts` | 单元测试（与源文件同目录） |
| 开发文档 | `frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-guest-uniapp.md` | 开发记录 |
| 评审报告 | `frontend/{项目名}-guest-uniapp/reviews/` | AI评审报告 |
| 更新文档 | `frontend/{项目名}-guest-uniapp/README.md` | 合并后的主文档 |
| 变更记录 | `frontend/{项目名}-guest-uniapp/CHANGELOG.md` | 代码变更历史 |

## 610→631 衔接协议

610 设计完成后，631 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 移动端方案 | `frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-*-tech-design.md` | 页面结构、平台适配、交互流程 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义 |

**并行约束**：631 与 620/630/640 天然独立可并行。631 依赖后端 API 可访问。

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

### 3. AI自动评审

**调用评审Skill**: `321-guest-uniapp-dev-review`

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 编码规范 | Vue规范、TS规范、UniApp规范 |
| 平台适配 | 条件编译正确使用 |
| 性能 | 页面渲染优化、资源加载 |
| 体验 | 交互流畅度、加载状态 |
| API对接 | 接口调用正确性 |
| 测试覆盖 | 单元测试完整性 |

### 4. 自动修复

**PLAN-REVIEW循环（必须执行）**:

#### 循环规则

| 规则 | 说明 |
|------|------|
| 通过条件 | review 评分 ≥ 95分 |
| 强制退出 | 最多5轮review（round=1~5），round ≥ 6强制退出 |
| 每轮操作 | review → 修复问题 → 重新 review |

#### 循环流程

```
循环开始 (round=1)
  │
  ├─▶ 调用技能: 321-guest-uniapp-dev-review
  │
  ├─▶ 判断结果:
  │    ├─ 评分 ≥ 95 → 输出结论，循环结束 ✓
  │    └─ 评分 < 95 → 进入修复步骤
  │
  ├─▶ 修复问题:
  │    修复后 round++，回到循环开始
  │
  └─▶ round ≥ 6 → 强制退出，输出当前结论 ⚠️
```

### 5. 测试执行

**测试类型**:

| 测试类型 | 命令/方式 | 通过标准 |
|---------|----------|---------|
| 单元测试 | `npm run test:unit` | 通过率100% |
| 类型检查 | `npx vue-tsc --noEmit` | 无类型错误 |
| 编译检查 | `npm run build` | 编译成功 |
| H5预览 | 浏览器访问 | 功能正常 |
| 小程序预览 | 微信开发者工具 | 功能正常 |

### 6. 生成开发文档

**文件位置**: `frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-guest-uniapp.md`

**文档内容**:
```markdown
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
- 状态: {通过/有条件通过}
- 遗留问题: {数量}个
```

### 7. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} FEATURE-{功能名称}
- 类型: UniApp移动端开发
- 变更: {简述}
- 平台: 微信小程序/H5/App
- 文档: FEATURE-DESIGN-{YYMMDD}-{简述}-guest-uniapp.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**代码位置**: `frontend/{项目名}-guest-uniapp/src/`

**开发文档**: `frontend/{项目名}-guest-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-guest-uniapp.md`

**评审报告**: `frontend/{项目名}-guest-uniapp/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 技术方案
    ↓
631-feature-guest-uniapp-dev
    ↓
AI生成代码 → 平台适配 → AI评审(321) → 自动修复 → 测试执行
    ↓
输出: UniApp移动端代码
    ↓
等待: 650-feature-final-review（最终验收）
```

## 参考

- [UniApp开发评审技能](../321-guest-uniapp-dev-review/SKILL.md) - AI自动评审调用
- [Vue3速查](../0-init/references/frontend/vue3-cheatsheet.md)
- [UniApp文档](https://uniapp.dcloud.net.cn/)
