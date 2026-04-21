---
name: 640-feature-test-dev
description: 功能测试脚本开发技能。当技术方案设计完成后触发：(1)基于技术方案生成自动化测试脚本, (2)编写API接口测试（Playwright）, (3)编写E2E界面测试（Playwright）, (4)编写回归测试用例, (5)调用641-feature-test-dev-review自动评审, (6)自动修复测试脚本, (7)执行测试并生成报告。请务必在用户提及测试脚本开发、自动化测试、Playwright测试时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能测试脚本开发

## 描述

基于610阶段的技术方案，使用AI驱动的方式生成自动化测试脚本（Playwright），包括API测试、E2E测试，自动完成实现、评审、修复、执行的完整闭环。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 测试脚本开发 | "编写订单导出功能测试" |
| API测试 | "编写API自动化测试" |
| E2E测试 | "编写界面自动化测试" |
| 回归测试 | "编写回归测试用例" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试脚本开发 | `test-engineer` |
| 协作 | 接口确认 | `java-developer` |
| 协作 | 场景验证 | `product-manager` |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 技术方案 | `backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 功能需求 | `issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段输出 |
| 后端Swagger | `backend/{项目名}-app/` | API接口定义 |
| 前端代码 | `{项目名}-{role}-{platform}/src/` | 页面组件、表单字段（E2E测试参考） |
| 现有测试 | `test/scripts/` | 现有测试基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| API测试脚本 | `test/scripts/api/` | Playwright API测试 |
| E2E测试脚本 | `test/scripts/e2e/` | Playwright E2E测试 |
| 测试数据 | `test/scripts/data/` | 测试数据文件 |
| 测试文档 | `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md` | 测试记录 |
| 评审报告 | `test/reviews/` | AI评审报告 |
| 测试报告 | `test/reports/` | 测试执行报告 |
| 变更记录 | `test/scripts/CHANGELOG.md` | 测试变更历史 |

## 610→640 衔接协议

610 设计完成后，640 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 测试方案 | `test/issues/FEATURE-DESIGN-*-test-design.md` | 测试范围、场景、策略 |
| 功能需求 | `issue/features/FEATURE-{YYMMDD}-*.md` | 验收标准 |

**并行约束**：640 与 620/630/631 天然独立可并行。640 的 E2E 测试依赖前端页面可访问。

## 执行流程

### 1. 测试脚本生成

**选择器策略**：遵循字段一致性原则，E2E 测试优先使用 `getByRole()` / `getByPlaceholder()` / `locator('[name=""]')` 定位元素，无需额外添加 `data-testid`。详见 [330-test-case-dev](../330-test-case-dev/SKILL.md)。

**生成内容**:

| 测试类型 | 生成位置 | 说明 |
|---------|---------|------|
| API测试 | `test/scripts/api/{feature}.spec.ts` | 接口自动化测试 |
| E2E测试 | `test/scripts/e2e/{feature}.spec.ts` | 端到端测试 |
| 测试数据 | `test/scripts/data/{feature}.json` | 测试数据 |
| 测试工具 | `test/scripts/utils/` | 测试工具函数 |

**测试覆盖**:

| 覆盖维度 | 测试内容 |
|---------|---------|
| 正向流程 | 正常操作场景 |
| 异常流程 | 错误处理场景 |
| 边界条件 | 极限值测试 |
| 权限控制 | 角色权限测试 |
| 数据验证 | 数据正确性验证 |

### 2. AI自动评审

**调用评审Skill**: `641-feature-test-dev-review`

> **评审范围限定**：640 仅生成 API + E2E 脚本，调用 331 评审时仅检查 API 测试脚本和 E2E 测试脚本两个维度，跳过 JMeter/安全/bin 评审。

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 覆盖度 | 是否覆盖所有AC |
| 正确性 | 测试逻辑是否正确 |
| 可维护性 | 测试代码结构 |
| 稳定性 | 测试是否稳定 |
| 性能 | 测试执行效率 |

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
  ├─▶ 调用技能: 641-feature-test-dev-review
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
| 针对性测试 | 只跑相关用例 | 全部通过 |

### 5. 生成测试文档

**文件位置**: `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md`

**文档内容**:
```markdown
# FEATURE-{YYMMDD}-{功能名称} - 测试脚本开发

## 测试概要
- 开发日期: {YYYY-MM-DD}
- 测试人员: AI Assistant

## 测试覆盖
### API测试
| 接口 | 测试场景 | 状态 |
|------|---------|------|
| {api} | {scenario} | {status} |

### E2E测试
| 场景 | 测试步骤 | 状态 |
|------|---------|------|
| {scenario} | {steps} | {status} |

## 测试数据
- 数据文件: {file}
- 数据量: {count}

## 执行结果
- API测试: {pass}/{total} 通过
- E2E测试: {pass}/{total} 通过
- 通过率: {percentage}%

## 评审结果
- 评分: {分数}
- 状态: {通过/有条件通过}
- 遗留问题: {数量}个
```

### 6. 文档合并

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} FEATURE-{功能名称}
- 类型: 测试脚本开发
- API测试: {数量}个
- E2E测试: {数量}个
- 文档: FEATURE-DESIGN-{YYMMDD}-{简述}-test.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**测试脚本**: `test/scripts/api/` + `test/scripts/e2e/`

**测试文档**: `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md`

**评审报告**: `test/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

**测试报告**: `test/reports/report-YYMMDDHHMM.md`

## 流转关系

```
输入: 技术方案 + 功能需求
    ↓
640-feature-test-dev
    ↓
AI生成测试 → AI评审(331) → 自动修复 → 测试执行
    ↓
输出: 测试脚本 + 测试报告
    ↓
等待: 650-feature-final-review（最终验收）
```

## 参考

- [测试脚本评审技能](../641-feature-test-dev-review/SKILL.md) - AI自动评审调用
- [Playwright文档](https://playwright.dev/)
