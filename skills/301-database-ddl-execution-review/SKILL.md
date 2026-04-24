---
name: 301-database-ddl-execution-review
description: 数据库DDL执行评审技能。当DDL执行完成后触发：(1)验证表结构与设计文档一致, (2)检查索引是否正确创建, (3)验证通用字段完整性, (4)检查初始数据正确性, (5)评估执行结果和异常处理, (6)输出评审结论。请务必在用户提及DDL执行评审、建表验证、数据库验证、Schema验证时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 数据库DDL执行评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | DDL执行评审 | `system-architect` |
| 协作 | ORM映射确认 | `java-developer` |
| 协作 | 业务数据确认 | `product-manager` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [300-database-ddl-execution/SKILL.md](../300-database-ddl-execution/SKILL.md) | **必读全文**：DDL执行流程、验证标准 |
| [200-database-design/SKILL.md](../200-database-design/SKILL.md) | 设计阶段命名约定和设计完成标准 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 执行报告 | `PROJECT_ROOT/database/DDL-EXECUTION-REPORT-*.md` | 300阶段产出 |
| DDL文件 | `PROJECT_ROOT/database/database-ddl.sql` | 执行的DDL |
| 设计文档 | `PROJECT_ROOT/database/database-design.md` | 设计参照 |
| 实际数据库 | 目标数据库 | 通过SQL查询验证 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| DDL执行评审报告 | `PROJECT_ROOT/database/reviews/REVIEW-DDL-EXECUTION-YYMMDDHHMM.md` | 评审结论 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

| 维度 | 检查要点 |
|------|---------|
| 表完整性 | 设计文档中的所有表都已创建 |
| 字段一致性 | 字段名、类型、约束与DDL一致 |
| 索引正确性 | 主键、唯一索引、普通索引均已创建 |
| 通用字段 | id/saas_id/state/create_date/modify_date 齐全 |
| 初始数据 | 枚举表、配置表数据正确 |
| 无多余对象 | 无遗留的测试表或临时表 |

**验证方式**：对每张表执行 `SHOW CREATE TABLE` 和 `SHOW INDEX`，与 `database-ddl.sql` 逐项比对。

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| 表完整率 | 100% | 25分 |
| 字段一致率 | ≥99% | 25分 |
| 索引完整率 | ≥95% | 25分 |
| 通用字段齐全 | 100% | 25分 |

### 有条件通过（85-94分）
- 字段一致率 ≥ 95%
- 索引完整率 ≥ 90%
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（表缺失、核心字段缺失）
- 表完整率 < 95%

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
逐表验证，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 表完整性/字段一致性/索引正确性/通用字段/初始数据
**评审对象**: `PROJECT_ROOT/database/` + 实际数据库
**参与人员**: @system-architect @java-developer


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `300-database-ddl-execution` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

## 参考

- [评审报告模板](../0-init/references/review-report-template.md) - 通用评审报告格式
- [评审检查清单](references/checklist.md) - DDL执行评审检查项
- [DDL执行技能](../300-database-ddl-execution/SKILL.md) - 被评审的上游技能
- [数据库设计评审技能](../201-database-design-review/SKILL.md) - 设计评审参考
