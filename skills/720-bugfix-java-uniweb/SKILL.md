---
name: 720-bugfix-java-uniweb
description: Bug Java后端修复技能。当修复方案设计完成后触发：(1)基于修复方案修改Java后端代码, (2)编写回归测试, (3)调用721-bugfix-java-uniweb-dev-review自动评审, (4)自动修复代码问题, (5)合并代码到主分支并更新CHANGELOG。请务必在用户提及Bug后端修复、Java修复、后端问题修复时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug Java后端修复

## 描述

基于710阶段的修复方案，使用AI驱动的方式修复Java后端代码，自动完成修复、评审、测试的完整闭环。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Bug后端修复 | "修复后端Bug" |
| Java修复 | "修复Java代码问题" |
| 后端问题修复 | "修复API问题" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码修复 + 单元测试 | `java-developer` |
| 协作 | 代码评审 | `java-lead` |

> **职责边界**：Java 单元测试由 Java 开发工程师负责。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 740）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 现有代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 现有代码基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 修复后的代码 |
| 回归测试 | `PROJECT_ROOT/backend/{项目名}-app/src/PROJECT_ROOT/test/` | 回归测试代码 |
| 修复文档 | `PROJECT_ROOT/backend/{项目名}-app/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-java-uniweb.md` | 修复记录 |
| 评审报告 | `PROJECT_ROOT/backend/{项目名}-app/reviews/` | AI评审报告 |
| 变更记录 | `PROJECT_ROOT/backend/{项目名}-app/CHANGELOG.md` | 代码变更历史 |

## 710→720 衔接协议

710 设计完成后，720 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复思路、修改范围、影响评估 |
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-*.md` | Bug复现步骤、根因分析 |
| DDL文件（如有） | `PROJECT_ROOT/database/migrations/BUGFIX-*.sql` | 数据库变更，需先执行 |

**并行约束**：720 与 730/731/740 天然独立可并行。DDL需先执行完毕后才能开始 720。

## 执行流程

### 1. 代码修复

**修复内容**:

| 修复类型 | 说明 |
|---------|------|
| 逻辑修复 | 修复业务逻辑错误 |
| 空值处理 | 添加空值判断 |
| 边界处理 | 修复边界条件 |
| 异常处理 | 完善异常处理 |
| 性能优化 | 优化性能问题 |

### 2. 回归测试

**测试内容**:

| 测试类型 | 说明 |
|---------|------|
| Bug复现测试 | 验证Bug已修复 |
| 边界测试 | 测试边界条件 |
| 异常测试 | 测试异常场景 |
| 原有功能 | 确保未破坏原有功能 |

### 3. AI自动评审

**调用评审Skill**: `721-bugfix-java-uniweb-dev-review`

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 修复正确性 | 是否正确修复Bug |
| 代码质量 | 代码规范、可读性 |
| 测试覆盖 | 回归测试完整性 |
| 副作用 | 是否引入新问题 |

### 4. 自动修复

## PLAN-REVIEW 循环（必须执行）

Java后端修复完成后，必须进入 PLAN-REVIEW 循环，确保修复质量达标。

调用技能 `721-bugfix-java-uniweb-dev-review`，评分 ≥ 95 通过，< 95 按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行修复循环。

### 5. 生成修复文档

**文件位置**: `PROJECT_ROOT/backend/{项目名}-app/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-java-uniweb.md`

**文档内容**:
```markdown
# BUGFIX-{YYMMDD}-{Bug名称} - Java后端修复

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
- 类型: Java后端修复
- 变更: {简述}
- 文档: BUGFIX-{YYMMDD}-{简述}.md
- 评审: {评分}分
- 状态: 已修复
```

## 输出要求

**修复代码**: `PROJECT_ROOT/backend/{项目名}-app/src/`

**修复文档**: `PROJECT_ROOT/backend/{项目名}-app/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-java-uniweb.md`

**评审报告**: `PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 修复方案
    ↓
720-bugfix-java-uniweb
    ↓
AI修复代码 → AI评审(311) → 自动修复 → 测试执行
    ↓
输出: 修复后的代码
    ↓
等待: 750-bugfix-final-review
```

## 参考

- [Java开发评审技能](../721-bugfix-java-uniweb-dev-review/SKILL.md)
