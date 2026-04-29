---
name: 730-bugfix-admin-uniapp
description: Bug UniApp移动端修复技能。当修复方案设计完成后触发：(1)基于修复方案修改UniApp移动端代码, (2)编写回归测试, (3)调用731-bugfix-admin-uniapp-review自动评审, (4)自动修复代码问题, (5)合并代码到主分支并更新CHANGELOG。请务必在用户提及Bug移动端修复、UniApp修复、小程序修复时使用此技能。适用于root/ops/saas/mch角色 ⚠️【强制】完成后必须调用 731-bugfix-admin-uniapp-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug UniApp移动端修复

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码修复 + 单元测试 | `js-developer` |
| 协作 | 代码评审 | `js-lead` |
| 协作 | 平台适配 | `js-developer` |

> **职责边界**：前端单元测试（composables/Store/工具函数）由前端工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 740）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 现有代码 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/` | 修复后的代码 |
| 回归测试 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/**/*.spec.ts` | 回归测试代码（与源文件同目录） |
| 修复文档 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-admin-uniapp.md` | 修复记录 |
| 评审报告 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/reviews/` | AI评审报告 |
| 变更记录 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/CHANGELOG.md` | 代码变更历史 |

## 710→730 衔接协议

710 设计完成后，730 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复思路、移动端修改范围 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-*.md` | Bug复现步骤 |

**并行约束**：730 与 720/730/740 天然独立可并行。

## 执行流程

### 1. 代码修复

**修复内容**:

| 修复类型 | 说明 |
|---------|------|
| 逻辑修复 | 修复业务逻辑错误 |
| 平台适配 | 修复平台差异问题 |
| API修复 | 修复API调用问题 |
| 样式修复 | 修复样式问题 |
| 性能修复 | 修复性能问题 |

### 2. 平台适配验证

**验证维度**:

| 平台 | 验证点 |
|------|--------|
| 微信小程序 | 微信SDK、分享、支付 |
| H5 | 浏览器兼容性、响应式 |
| App | 原生能力、推送 |


## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `731-bugfix-admin-uniapp-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

# BUGFIX-{YYMMDD}-{Bug名称} - UniApp移动端修复

## 修复概要
- 修复日期: {YYYY-MM-DD}
- 修复人员: AI Assistant
- 支持平台: 微信小程序 / H5 / App

## 代码变更
### 修改文件
| 文件 | 修改说明 |
|------|---------|
| {file} | {desc} |

## 平台适配
| 平台 | 修复点 |
|------|--------|
| 微信小程序 | {fix} |
| H5 | {fix} |
| App | {fix} |

## 回归测试
- 测试用例: {数量}个
- 通过率: {percentage}%

## 评审结果
- 评分: {分数}
- 状态: {通过/不通过}
```

### 6. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} BUG-{Bug名称}
- 类型: UniApp移动端修复
- 变更: {简述}
- 平台: 微信小程序/H5/App
- 文档: BUGFIX-{YYMMDD}-{简述}.md
- 评审: {评分}分
- 状态: 已修复
```

## 输出要求

**修复代码**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/src/`

**修复文档**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-admin-uniapp.md`

**评审报告**: `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 修复方案
    ↓
731-bugfix-admin-uniapp
    ↓
AI修复代码 → 平台适配 → AI评审(321) → 自动修复 → 测试执行
    ↓
输出: 修复后的代码
    ↓
等待: 750-bugfix-final-review
```

## 参考

- [UniApp开发评审技能](../731-bugfix-admin-uniapp-review/SKILL.md)
