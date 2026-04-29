---
name: 710-bugfix-tech-design
description: Bug修复方案设计技能。当Bug分析完成后触发：(1)设计修复方案, (2)评估数据库变更需求, (3)评估接口变更影响, (4)生成修复方案文档, (5)AI自动评审方案, (6)人工确认修复方案。请务必在用户提及修复方案设计、Bug修复方案、技术方案设计时使用此技能 ⚠️【强制】完成后必须调用 711-bugfix-tech-design-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Bug修复方案设计

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 方案设计 | `system-architect` |
| 协作 | 变更评估 | `java-developer` |
| 决策 | 方案确认 | `product-manager` ★ |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| Bug分析报告 | `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-{简述}.md` | 700阶段输出 |
| 相关代码 | 代码仓库 | 需要修改的代码 |
| 现有架构 | 各端README.md | 现有技术架构 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 修复方案文档 | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md` | 修复方案 |
| DDL文件（如有） | `PROJECT_ROOT/database/migrations/BUGFIX-{YYMMDD}-{简述}.sql` | 数据库变更 |

## 执行流程

### 1. 修复方案设计

设计修复方案：

| 方案要素 | 内容 |
|---------|------|
| 修复思路 | 如何解决Bug |
| 修改范围 | 涉及哪些文件 |
| 数据库变更 | 是否需要DDL |
| 接口变更 | 是否影响API |
| 兼容性 | 是否向后兼容 |

### 2. 变更影响评估

评估变更影响：

| 评估项 | 内容 |
|--------|------|
| 代码影响 | 修改的文件和行数 |
| 数据影响 | 是否需要数据修复 |
| 接口影响 | 是否影响其他模块 |
| 测试影响 | 需要补充的测试 |
| 回滚方案 | 如何回滚 |

### 3. 生成修复方案文档

**文件命名**: `BUGFIX-DESIGN-{YYMMDD}-{简述}.md`

**文档内容**:
```markdown
# BUGFIX-{YYMMDD}-{Bug名称} - 修复方案

## 修复思路
{修复思路描述}

## 修改范围
### 代码修改
| 文件 | 修改类型 | 说明 |
|------|---------|------|
| {file} | {type} | {desc} |

### 数据库变更（如有）
```sql
-- DDL语句
```

## 影响评估
| 评估项 | 结果 |
|--------|------|
| 代码影响 | {影响} |
| 数据影响 | {影响} |
| 接口影响 | {影响} |
| 兼容性 | {兼容情况} |

## 测试策略
- {测试点1}
- {测试点2}

## 回滚方案
{回滚步骤}

## 风险评估
| 风险 | 等级 | 缓解措施 |
|------|------|---------|
| {risk} | {level} | {mitigation} |
```


## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `711-bugfix-tech-design-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

## 人工检查点 ★

**必须人工确认**:
- [ ] 修复方案是否正确
- [ ] 变更影响是否评估完整
- [ ] 风险是否可接受

**确认后流转**: 并行进入 `720-bugfix-java-uniweb`/`730-bugfix-admin-web`/`730-bugfix-guest-web`/`730-bugfix-admin-uniapp`/`730-bugfix-guest-uniapp`/`740-bugfix-test`

## `710-bugfix-tech-design` → 下游衔接协议

`710-bugfix-tech-design` 设计完成后，各下游技能从以下文件提取开发输入：

| 下游技能 | 提取文件 | 用途 |
|---------|---------|------|
| `720-bugfix-java-uniweb` | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` + DDL文件（如有） | 修复方案 + 数据库变更 |
| `730-bugfix-admin-web` / `730-bugfix-guest-web` | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复方案（前端部分） |
| `730-bugfix-admin-uniapp` / `730-bugfix-guest-uniapp` | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` | 修复方案（移动端部分） |
| `740-bugfix-test` | `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-*.md` + `PROJECT_ROOT/issue/bugs/BUGFIX-{YYMMDD}-*.md` | 修复方案 + Bug分析（回归测试） |

**并行约束**：四端天然独立，可同时由不同Agent修复。DDL需先执行，后端修复依赖DDL。

## 输出要求

**修复方案**: `PROJECT_ROOT/issue/bugs/BUGFIX-DESIGN-{YYMMDD}-{简述}.md`

**DDL文件（如有）**: `PROJECT_ROOT/database/migrations/BUGFIX-{YYMMDD}-{简述}.sql`

## 流转关系

```
输入: Bug分析报告
    ↓
710-bugfix-tech-design
    ↓
AI自动评审
    ↓
人工确认 ★
    ↓
输出: 修复方案
    ↓
并行进入: `720-bugfix-java-uniweb`/`730-bugfix-admin-web`/`730-bugfix-guest-web`/`730-bugfix-admin-uniapp`/`730-bugfix-guest-uniapp`/`740-bugfix-test`
```
