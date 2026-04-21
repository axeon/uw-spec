---
name: 730-bugfix-admin-web
description: Bug Web前端修复技能。当修复方案设计完成后触发：(1)基于修复方案修改Web前端代码, (2)编写回归测试, (3)调用731-bugfix-admin-web-review自动评审, (4)自动修复代码问题, (5)合并代码到主分支并更新CHANGELOG。请务必在用户提及Bug前端修复、Vue修复、Web问题修复时使用此技能。适用于root/ops/saas/mch角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug Web前端修复

## 描述

基于710阶段的修复方案，使用AI驱动的方式修复Web前端代码（Vue3），自动完成修复、评审、测试的完整闭环。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Bug前端修复 | "修复前端Bug" |
| Vue修复 | "修复Vue代码问题" |
| Web问题修复 | "修复页面问题" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码修复 + 单元测试 | `js-developer` |
| 协作 | 代码评审 | `js-lead` |

> **职责边界**：前端单元测试（composables/Store/工具函数）由前端工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 740）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复方案 | `issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 现有代码 | `frontend/{项目名}-admin-web/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复代码 | `frontend/{项目名}-admin-web/src/` | 修复后的代码 |
| 回归测试 | `frontend/{项目名}-admin-web/src/**/*.spec.ts` | 回归测试代码（与源文件同目录） |
| 修复文档 | `frontend/{项目名}-admin-web/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-admin-web.md` | 修复记录 |
| 评审报告 | `frontend/{项目名}-admin-web/reviews/` | AI评审报告 |
| 变更记录 | `frontend/{项目名}-admin-web/CHANGELOG.md` | 代码变更历史 |

## 710→730 衔接协议

710 设计完成后，730 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 修复方案 | `issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复思路、前端修改范围 |
| Bug分析报告 | `issue/bugs/BUGFIX-{YYMMDD}-*.md` | Bug复现步骤 |

**并行约束**：730 与 720/731/740 天然独立可并行。

## 执行流程

### 1. 代码修复

**修复内容**:

| 修复类型 | 说明 |
|---------|------|
| 逻辑修复 | 修复业务逻辑错误 |
| 渲染修复 | 修复渲染问题 |
| 事件修复 | 修复事件处理 |
| 样式修复 | 修复样式问题 |
| 兼容性修复 | 修复浏览器兼容性问题 |

### 2. 回归测试

**测试内容**:

| 测试类型 | 说明 |
|---------|------|
| Bug复现测试 | 验证Bug已修复 |
| 组件测试 | 测试组件功能 |
| 交互测试 | 测试用户交互 |
| 原有功能 | 确保未破坏原有功能 |

### 3. AI自动评审

**调用评审Skill**: `731-bugfix-admin-web-review`

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 修复正确性 | 是否正确修复Bug |
| 代码质量 | Vue规范、TS规范 |
| 测试覆盖 | 回归测试完整性 |
| 副作用 | 是否引入新问题 |

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
  ├─▶ 调用技能: 731-bugfix-admin-web-review
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

### 5. 生成修复文档

**文件位置**: `frontend/{项目名}-admin-web/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-admin-web.md`

**文档内容**:
```markdown
# BUGFIX-{YYMMDD}-{Bug名称} - Web前端修复

## 修复概要
- 修复日期: {YYYY-MM-DD}
- 修复人员: AI Assistant

## 代码变更
### 修改文件
| 文件 | 修改说明 |
|------|---------|
| {file} | {desc} |

## 回归测试
- 测试用例: {数量}个
- 通过率: {percentage}%

## 评审结果
- 评分: {分数}
- 状态: {通过/有条件通过}
```

### 6. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} BUG-{Bug名称}
- 类型: Web前端修复
- 变更: {简述}
- 文档: BUGFIX-{YYMMDD}-{简述}.md
- 评审: {评分}分
- 状态: 已修复
```

## 输出要求

**修复代码**: `frontend/{项目名}-admin-web/src/`

**修复文档**: `frontend/{项目名}-admin-web/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-admin-web.md`

**评审报告**: `frontend/{项目名}-admin-web/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 修复方案
    ↓
730-bugfix-admin-web
    ↓
AI修复代码 → AI评审(321) → 自动修复 → 测试执行
    ↓
输出: 修复后的代码
    ↓
等待: 750-bugfix-final-review
```

## 参考

- [Web前端评审技能](../731-bugfix-admin-web-review/SKILL.md)
