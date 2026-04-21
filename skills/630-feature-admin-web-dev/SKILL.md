---
name: 630-feature-admin-web-dev
description: 功能Web前端开发技能。当技术方案设计完成后触发：(1)基于技术方案生成Web前端代码, (2)实现Vue页面组件和交互逻辑, (3)对接后端API, (4)编写单元测试, (5)调用631-feature-admin-web-dev-review自动评审, (6)自动修复代码问题, (7)合并代码到主分支并更新CHANGELOG。请务必在用户提及Web前端开发、Vue开发、实现前端功能时使用此技能。适用于root/ops/saas/mch角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能Web前端开发

## 描述

基于610阶段的技术方案，使用AI驱动的方式生成Web前端代码（Vue3 + TypeScript + Element Plus），自动完成实现、评审、修复、测试的完整闭环。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Web前端开发 | "实现订单导出页面" |
| Vue组件开发 | "开发导出按钮组件" |
| 前端功能实现 | "实现前端导出功能" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码生成实现 + 单元测试 | `js-developer` |
| 协作 | 代码评审 | `js-lead` |
| 协作 | 接口对接 | `java-developer` |

> **职责边界**：前端单元测试（composables/Store/工具函数）由前端工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 640）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 技术方案 | `frontend/{项目名}-admin-web/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义 |
| 现有代码 | `frontend/{项目名}-admin-web/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 功能代码 | `frontend/{项目名}-admin-web/src/` | Vue前端代码 |
| 测试代码 | `frontend/{项目名}-admin-web/src/**/*.spec.ts` | 单元测试（与源文件同目录） |
| 开发文档 | `frontend/{项目名}-admin-web/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-admin-web.md` | 开发记录 |
| 评审报告 | `frontend/{项目名}-admin-web/reviews/` | AI评审报告 |
| 更新文档 | `frontend/{项目名}-admin-web/README.md` | 合并后的主文档 |
| 变更记录 | `frontend/{项目名}-admin-web/CHANGELOG.md` | 代码变更历史 |

## 610→630 衔接协议

610 设计完成后，630 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| Web前端方案 | `frontend/{项目名}-admin-web/issues/FEATURE-DESIGN-*-tech-design.md` | 页面结构、组件设计、交互流程 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义 |

**并行约束**：630 与 620/631/640 天然独立可并行。630 依赖后端 API 可访问。

## 执行流程

### 1. 代码生成

**生成内容**:

| 代码类型 | 生成位置 | 说明 |
|---------|---------|------|
| 页面组件 | `src/pages/` | Vue页面 |
| 业务组件 | `src/components/business/` | 可复用组件 |
| API模块 | `src/api/` | API调用封装 |
| Store | `src/stores/` | Pinia状态管理 |
| 类型定义 | `src/types/` | TypeScript类型 |
| 单元测试 | `src/**/*.spec.ts` | Vitest测试（与源文件同目录） |

**代码规范**:
- Vue3 + TypeScript + `<script setup>`
- Element Plus组件库
- Pinia状态管理
- Axios HTTP客户端

### 2. AI自动评审

**调用评审Skill**: `631-feature-admin-web-dev-review`

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 编码规范 | Vue规范、TS规范 |
| 组件设计 | 组件拆分合理性 |
| API对接 | 接口调用正确性 |
| 状态管理 | Store设计合理性 |
| 性能 | 渲染优化、懒加载 |
| 测试覆盖 | 单元测试完整性 |

### 3. 自动修复

## PLAN-REVIEW 循环（必须执行）

Web前端开发完成后，必须进入 PLAN-REVIEW 循环，确保代码质量达标。

调用技能 `631-feature-admin-web-dev-review`，评分 ≥ 95 通过，< 95 按下表修复后重新评审（最多5轮，round ≥ 6 强制退出）:

| 优先级 | 要求 |
|--------|------|
| Critical | 本轮必须全部修复，不可遗留 |
| Major | 本轮修复 ≥ 80%，剩余标注下轮计划 |
| Minor | 记录但不阻塞，按优先级排列 |

循环结束时，在评审报告末尾追加：
```markdown
## PLAN-REVIEW 循环结论
- 总轮次: {N}/5
- 最终评分: {N}分
- 状态: {通过 | 有条件通过 | 强制退出}
- 遗留问题: Critical {N}, Major {N}, Minor {N}
```

### 4. 测试执行

**测试类型**:

| 测试类型 | 命令 | 通过标准 |
|---------|------|---------|
| 单元测试 | `npm run test:unit` | 通过率100% |
| 类型检查 | `npx vue-tsc --noEmit` | 无类型错误 |
| 编译检查 | `npm run build` | 编译成功 |
| 代码规范 | `npm run lint` | 无规范错误 |
| API连通 | 手动验证 | 接口调用正常 |

### 5. 生成开发文档

**文件位置**: `frontend/{项目名}-admin-web/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-admin-web.md`

**文档内容**:
```markdown
# FEATURE-DESIGN-{YYMMDD}-{功能名称} - Web前端开发

## 实现概要
- 实现日期: {YYYY-MM-DD}
- 开发人员: AI Assistant

## 代码变更
### 新增文件
- `{文件路径}`: {说明}

### 修改文件
- `{文件路径}`: {说明}

## 页面清单
| 页面 | 路径 | 说明 |
|------|------|------|
| {name} | {path} | {desc} |

## 组件清单
| 组件 | 位置 | 说明 |
|------|------|------|
| {name} | {path} | {desc} |

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

### 6. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} FEATURE-{功能名称}
- 类型: Web前端开发
- 变更: {简述}
- 文档: FEATURE-DESIGN-{YYMMDD}-{简述}-admin-web.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**代码位置**: `frontend/{项目名}-admin-web/src/`

**开发文档**: `frontend/{项目名}-admin-web/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-admin-web.md`

**评审报告**: `frontend/{项目名}-admin-web/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 技术方案
    ↓
630-feature-admin-web-dev
    ↓
AI生成代码 → AI评审(321) → 自动修复 → 测试执行
    ↓
输出: Web前端代码
    ↓
等待: 650-feature-final-review（最终验收）
```

## 参考

- [Web前端评审技能](../631-feature-admin-web-dev-review/SKILL.md) - AI自动评审调用
- [Vue3速查](../0-init/references/frontend/vue3-cheatsheet.md)
- [Element Plus速查](../0-init/references/frontend/element-plus-cheatsheet.md)
