---
name: 740-bugfix-test
description: Bug回归测试技能。当修复方案设计完成后触发：(1)基于修复方案生成回归测试脚本，(2)编写Bug复现测试，(3)编写边界条件测试，(4)调用331-test-case-dev-review自动评审，(5)执行测试并生成报告。请务必在用户提及Bug回归测试、修复验证、测试脚本开发时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "2.0.0"
---

# Bug回归测试

## 描述

基于710阶段的修复方案，生成并执行回归测试，验证Bug已修复且未引入新问题。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Bug回归测试 | "编写Bug回归测试" |
| 修复验证 | "验证Bug修复效果" |
| 测试脚本开发 | "开发回归测试脚本" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试开发 | `test-engineer` |
| 协作 | 场景验证 | `product-manager` |
| 协作 | 接口确认 | `java-developer` |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 修复方案 | `issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| Bug分析报告 | `issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义（API测试参考） |
| 前端代码 | `{项目名}-{role}-{platform}/src/` | 页面组件、表单字段（E2E测试参考） |
| 现有测试 | `test/scripts/` | 现有测试基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 回归测试脚本（API） | `test/scripts/api/regression-{bug}.spec.ts` | 回归API测试代码 |
| 回归测试脚本（E2E） | `test/scripts/e2e/{项目名}/regression-{bug}.spec.ts` | 回归E2E测试代码 |
| 测试报告 | `test/reports/` | 测试执行报告 |
| 测试文档 | `test/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-test.md` | 测试记录 |
| 评审报告 | `test/reviews/` | AI评审报告 |
| 变更记录 | `test/scripts/CHANGELOG.md` | 测试变更历史 |

## 710→740 衔接协议

710 设计完成后，740 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 修复方案 | `issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 测试策略、回归范围 |
| Bug分析报告 | `issue/bugs/BUGFIX-{YYMMDD}-*.md` | Bug复现步骤、预期行为 |

**并行约束**：740 与 720/730/731 天然独立可并行。E2E 测试依赖前端页面可访问。

## 执行流程

### 1. 回归测试生成

**选择器策略**：遵循字段一致性原则，E2E 测试优先使用 `getByRole()` / `getByPlaceholder()` / `locator('[name=""]')` 定位元素，无需额外添加 `data-testid`。详见 [330-test-case-dev](../330-test-case-dev/SKILL.md)。

**测试类型**:

| 测试类型 | 说明 |
|---------|------|
| Bug复现测试 | 验证Bug已修复 |
| 边界测试 | 测试边界条件 |
| 异常测试 | 测试异常场景 |
| 原有功能测试 | 确保未破坏原有功能 |

### 2. AI自动评审

**调用评审Skill**: `331-test-case-dev-review`

> **评审范围限定**：740 仅生成回归测试脚本（API + E2E），调用 331 评审时仅检查 API 测试脚本和 E2E 测试脚本两个维度，跳过 JMeter/安全/bin 评审。

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 覆盖度 | 是否覆盖Bug场景 |
| 正确性 | 测试逻辑是否正确 |
| 完整性 | 是否包含边界和异常 |
| 可维护性 | 测试代码结构 |

### 3. 自动修复

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
  ├─▶ 调用技能: 331-test-case-dev-review
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

### 4. 测试执行

**执行策略**:

| 测试类型 | 执行命令 | 通过标准 |
|---------|---------|---------|
| API测试 | `npx playwright test api/` | 通过率100%（失败需分析原因，非脚本问题可放行） |
| E2E测试 | `npx playwright test e2e/` | 通过率100%（失败需分析原因，非脚本问题可放行） |
| 回归测试 | 只跑相关用例 | 全部通过 |

### 5. 生成测试文档

**文件位置**: `test/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-test.md`

**文档内容**:
```markdown
# BUGFIX-{YYMMDD}-{Bug名称} - 回归测试

## 测试概要
- 开发日期: {YYYY-MM-DD}
- 测试人员: AI Assistant

## 测试覆盖
### Bug复现测试
| 场景 | 测试步骤 | 状态 |
|------|---------|------|
| {scenario} | {steps} | {status} |

### 边界测试
| 边界条件 | 测试结果 |
|---------|---------|
| {condition} | {result} |

## 执行结果
- 回归测试: {pass}/{total} 通过
- 通过率: {percentage}%

## 评审结果
- 评分: {分数}
- 状态: {通过/有条件通过}
```

### 6. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} BUGFIX-{Bug名称}
- 类型: 回归测试
- 测试: {数量}个
- 文档: BUGFIX-DESIGN-{YYMMDD}-{简述}-test.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**回归测试**: `test/scripts/api/regression-*.spec.ts` + `test/scripts/e2e/{项目名}/regression-*.spec.ts`

**测试文档**: `test/issues/BUGFIX-DESIGN-{YYMMDD}-{简述}-test.md`

**测试报告**: `test/reports/report-{YYYYMMDD-HHMM}.md`

## 流转关系

```
输入: 修复方案
    ↓
740-bugfix-test
    ↓
AI生成测试 → AI评审(331) → 自动修复 → 测试执行
    ↓
输出: 回归测试 + 测试报告
    ↓
等待: 750-bugfix-final-review
```

## 参考

- [测试脚本评审技能](../331-test-case-dev-review/SKILL.md)
