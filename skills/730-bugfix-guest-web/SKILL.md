---
name: 730-bugfix-guest-web
description: Bug Web前端修复技能。当修复方案设计完成后触发：(1)基于修复方案修改Web前端代码, (2)编写回归测试, (3)调用731-bugfix-guest-web-review自动评审, (4)自动修复代码问题, (5)合并代码到主分支并更新CHANGELOG。请务必在用户提及Bug前端修复、Vue修复、Web问题修复时使用此技能。适用于guest（消费者）角色 ⚠️【强制】完成后必须调用 731-bugfix-guest-web-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug Web前端修复

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码修复 + 单元测试 | `js-developer` |
| 协作 | 代码评审 | `js-lead` |

> **职责边界**：前端单元测试（composables/Store/工具函数）由前端工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 740）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 现有代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/` | 修复后的代码 |
| 回归测试 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/**/*.spec.ts` | 回归测试代码（与源文件同目录） |
| 修复文档 | `PROJECT_ROOT/frontend/{项目名}-guest-web/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-guest-web.md` | 修复记录 |
| 评审报告 | `PROJECT_ROOT/frontend/{项目名}-guest-web/reviews/` | AI评审报告 |
| 变更记录 | `PROJECT_ROOT/frontend/{项目名}-guest-web/CHANGELOG.md` | 代码变更历史 |

## 710→730 衔接协议

710 设计完成后，730 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复思路、前端修改范围 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-*.md` | Bug复现步骤 |

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


## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `731-bugfix-guest-web-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

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

**修复代码**: `PROJECT_ROOT/frontend/{项目名}-guest-web/src/`

**修复文档**: `PROJECT_ROOT/frontend/{项目名}-guest-web/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-guest-web.md`

**评审报告**: `PROJECT_ROOT/frontend/{项目名}-guest-web/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 修复方案
    ↓
730-bugfix-guest-web
    ↓
AI修复代码 → AI评审(321) → 自动修复 → 测试执行
    ↓
输出: 修复后的代码
    ↓
等待: 750-bugfix-final-review
```

## 参考

- [Web前端评审技能](../731-bugfix-guest-web-review/SKILL.md)
