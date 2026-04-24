---
name: 750-bugfix-final-review
description: Bug修复最终验收技能。当所有Bug修复阶段完成后触发：(1)执行回归测试验证Bug已修复, (2)执行核心流程回归测试确保无新Bug, (3)生成修复验收报告, (4)人工确认修复完成。请务必在用户提及Bug验收、修复确认、Bug完成时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug修复最终验收

## 描述

在所有Bug修复阶段（720/730/731/740）完成后，执行全面的回归测试，验证Bug已修复且未引入新问题，由人工进行最终的修复确认。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Bug验收 | "验收Bug修复" |
| 修复确认 | "确认Bug已修复" |
| Bug完成 | "完成Bug修复" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 回归测试执行 | `test-engineer` |
| 协作 | 风险评估 | `project-manager` |
| 决策 | 修复确认 | `product-manager` ★ |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [700-bugfix-analysis/SKILL.md](../700-bugfix-analysis/SKILL.md) | Bug分析规范 |
| [710-bugfix-tech-design/SKILL.md](../710-bugfix-tech-design/SKILL.md) | 修复方案规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 修复方案 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 710阶段输出 |
| 修复代码 | 各端src/ | 720/730/731阶段输出 |
| 回归测试 | `PROJECT_ROOT/test/scripts/` | 740阶段输出 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复验收报告 | `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-{YYMMDDHHMM}.md` | 最终验收报告 |
| 测试报告 | `PROJECT_ROOT/test/reports/summary/final-YYMMDDHHMM.md` | 测试执行报告 |

## 执行流程

> **前置步骤**：先读取 [700-bugfix-analysis/SKILL.md](../700-bugfix-analysis/SKILL.md) 和 [710-bugfix-tech-design/SKILL.md](../710-bugfix-tech-design/SKILL.md)，提取 Bug 分析规范和修复方案规范，作为验收的权威依据。

详细的验收检查清单见 [checklist.md](references/checklist.md)。

### 1. Bug修复验证

**验证内容**:

| 验证项 | 说明 |
|--------|------|
| Bug复现 | 原Bug场景是否已修复 |
| 边界条件 | 边界场景是否正常 |
| 异常处理 | 异常场景是否正常 |

**通过标准**: Bug场景100%修复

### 2. 核心流程回归测试

**回归范围**:

| 流程 | 说明 |
|------|------|
| 主业务流程 | 确保核心功能未破坏 |
| 关联功能 | 与Bug相关的其他功能 |
| 权限流程 | 登录、权限校验等 |

**通过标准**: 核心流程100%通过

### 3. 生成修复验收报告

**文件位置**: `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-{YYMMDDHHMM}.md`

**报告内容**:
```markdown
# REVIEW-BUGFIX-{YYMMDD}-{Bug名称} - 修复验收报告

## 验收概要
- 验收日期: {YYYY-MM-DD}
- Bug名称: {Bug名称}
- 验收人员: AI Assistant + 产品经理

## Bug修复验证
| 验证项 | 结果 | 说明 |
|--------|------|------|
| Bug复现 | {status} | {note} |
| 边界条件 | {status} | {note} |
| 异常处理 | {status} | {note} |

## 回归测试结果
| 流程 | 状态 | 说明 |
|------|------|------|
| {flow} | {status} | {note} |

## 修复统计
- 修改文件: {数量}个
- 新增代码: {行数}行
- 删除代码: {行数}行
- 测试用例: {数量}个

## 风险评估
| 风险项 | 等级 | 说明 |
|--------|------|------|
| {risk} | {level} | {desc} |

## 验收结论
- Bug修复: {status}
- 回归测试: {status}
- 风险评估: {status}
- **最终状态: {通过/有条件通过/不通过}**

## 人工确认 ★
- [ ] Bug已修复确认
- [ ] 无新问题确认
- [ ] 修复完成确认

## 下一步
- 通过: 进入 760-bugfix-update-doc
- 不通过: 返回对应修复阶段
```

## 人工检查点 ★

**必须人工确认**:
- [ ] Bug是否已完全修复
- [ ] 是否引入新问题
- [ ] 是否可以关闭Bug

**确认后流转**:
- 通过 → 进入 760-bugfix-update-doc
- 不通过 → 返回对应修复阶段

## 输出要求

**修复验收报告**: `PROJECT_ROOT/issue/reviews/REVIEW-BUGFIX-{YYMMDDHHMM}.md`

**测试报告**: `PROJECT_ROOT/test/reports/summary/final-YYMMDDHHMM.md`

## 流转关系

```
输入: 所有修复阶段输出
    ↓
750-bugfix-final-review
    ↓
Bug修复验证 → 回归测试 → 生成报告
    ↓
人工确认 ★
    ↓
├─ 通过 → 进入 760-bugfix-update-doc
└─ 不通过 → 返回对应修复阶段
```

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| Bug修复验证 | Bug场景100%修复 | 30分 |
| 回归测试通过率 | 核心流程100%通过 | 25分 |
| 无新问题 | 未引入新的缺陷 | 20分 |
| 人工确认 | 产品经理确认修复完成 | 15分 |
| 文档更新 | 修复文档记录完整 | 10分 |

### 有条件通过（85-94分）
- Bug已修复但边界场景有小问题
- 回归测试有≤2个非核心用例失败
- 无新Critical/Major问题

### 不通过（<85分）
- Bug未完全修复
- 回归测试核心流程失败
- 引入了新的Critical问题

## 通过标准

| 检查项 | 标准 |
|--------|------|
| Bug修复 | 100%修复 |
| 回归测试 | 100%通过 |
| 人工确认 | 通过 |
