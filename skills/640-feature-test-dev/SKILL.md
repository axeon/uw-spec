---
name: 640-feature-test-dev
description: 功能测试脚本开发技能。当技术方案设计完成后触发：(1)基于技术方案生成自动化测试脚本, (2)编写API接口测试（Playwright）, (3)编写E2E界面测试（Playwright）, (4)编写回归测试用例, (5)调用641-feature-test-dev-review自动评审, (6)自动修复测试脚本, (7)执行测试并生成报告。请务必在用户提及测试脚本开发、自动化测试、Playwright测试时使用此技能 ⚠️【强制】完成后必须调用 641-feature-test-dev-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能测试脚本开发

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试脚本开发 | `test-engineer` |
| 协作 | 接口确认 | `java-developer` |
| 协作 | 场景验证 | `product-manager` |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 技术方案 | `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段输出 |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}-app/` | API接口定义 |
| 前端代码 | `{项目名}-{role}-{platform}/src/` | 页面组件、表单字段（E2E测试参考） |
| 现有测试 | `PROJECT_ROOT/test/scripts/` | 现有测试基线 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| API测试脚本 | `PROJECT_ROOT/test/scripts/api/` | Playwright API测试 |
| E2E测试脚本 | `PROJECT_ROOT/test/scripts/e2e/` | Playwright E2E测试 |
| 测试数据 | `PROJECT_ROOT/test/scripts/data/` | 测试数据文件 |
| 测试文档 | `PROJECT_ROOT/test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md` | 测试记录 |
| 评审报告 | `PROJECT_ROOT/test/reviews/` | AI评审报告 |
| 测试报告 | `PROJECT_ROOT/test/reports/` | 测试执行报告 |
| 变更记录 | `PROJECT_ROOT/test/scripts/CHANGELOG.md` | 测试变更历史 |

## 610→640 衔接协议

610 设计完成后，640 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 测试方案 | `PROJECT_ROOT/test/issues/FEATURE-DESIGN-*-test-design.md` | 测试范围、场景、策略 |
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-*.md` | 验收标准 |

**并行约束**：640 与 620/630/631 天然独立可并行。640 的 E2E 测试依赖前端页面可访问。

## 执行流程

### 1. 测试脚本生成

**选择器策略**：遵循字段一致性原则，E2E 测试优先使用 `getByRole()` / `getByPlaceholder()` / `locator('[name=""]')` 定位元素，无需额外添加 `data-testid`。详见 [330-test-case-dev](../330-test-case-dev/SKILL.md)。

**生成内容**:

| 测试类型 | 生成位置 | 说明 |
|---------|---------|------|
| API测试 | `PROJECT_ROOT/test/scripts/api/{feature}.spec.ts` | 接口自动化测试 |
| E2E测试 | `PROJECT_ROOT/test/scripts/e2e/{feature}.spec.ts` | 端到端测试 |
| 测试数据 | `PROJECT_ROOT/test/scripts/data/{feature}.json` | 测试数据 |
| 测试工具 | `PROJECT_ROOT/test/scripts/utils/` | 测试工具函数 |

**测试覆盖**:

| 覆盖维度 | 测试内容 |
|---------|---------|
| 正向流程 | 正常操作场景 |
| 异常流程 | 错误处理场景 |
| 边界条件 | 极限值测试 |
| 权限控制 | 角色权限测试 |
| 数据验证 | 数据正确性验证 |

### 2. AI自动评审

**REVIEW评审技能**: `641-feature-test-dev-review`

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

## ⚠️ 完成验证（强制）

> 在声称"任务完成"之前，必须重新读取本节，逐项确认。

测试脚本开发完成后，调用 `641-feature-test-dev-review`。

- [ ] 已重新读取本节全部内容
- [ ] 已调用 `641-feature-test-dev-review`
- [ ] 已收到评审通过确认（评分 ≥ 95）

> 未收到通过确认前，禁止结束测试脚本开发任务。
> 以上检查项未全部勾选，禁止向用户报告"已完成"。


### 4. 测试执行

**执行策略**:

| 测试类型 | 执行命令 | 通过标准 |
|---------|---------|---------|
| API测试 | `npx playwright test api/` | 通过率100%（失败需分析原因，非脚本问题可放行） |
| E2E测试 | `npx playwright test e2e/` | 通过率100%（失败需分析原因，非脚本问题可放行） |
| 针对性测试 | 只跑相关用例 | 全部通过 |

### 5. 生成测试文档

**文件位置**: `PROJECT_ROOT/test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md`

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

**测试脚本**: `PROJECT_ROOT/test/scripts/api/` + `PROJECT_ROOT/test/scripts/e2e/`

**测试文档**: `PROJECT_ROOT/test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test.md`

**评审报告**: `PROJECT_ROOT/test/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

**测试报告**: `PROJECT_ROOT/test/reports/report-YYMMDDHHMM.md`

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

- [测试脚本评审技能](../641-feature-test-dev-review/SKILL.md) - REVIEW评审技能
- [Playwright文档](https://playwright.dev/)
