---
name: 301-database-ddl-execution-review
description: 数据库DDL执行评审技能。当DDL执行完成后触发：(1)验证表结构与设计文档一致，(2)检查索引是否正确创建，(3)验证通用字段完整性，(4)检查初始数据正确性，(5)评估执行结果和异常处理，(6)输出评审结论。请务必在用户提及DDL执行评审、建表验证、数据库验证、Schema验证时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 数据库DDL执行评审

## 描述

对 300-database-ddl-execution 阶段的执行结果进行全面评审，验证数据库实际 Schema 与设计文档一致，确保后端开发环境可用。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| DDL执行后 | "评审建表结果" |
| Schema验证 | "验证数据库表结构" |
| 数据库检查 | "检查数据库是否正确" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | DDL执行评审 | `system-architect` |
| 协作 | ORM映射确认 | `java-developer` |
| 协作 | 业务数据确认 | `product-manager` |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 执行报告 | `database/DDL-EXECUTION-REPORT-*.md` | 300阶段产出 |
| DDL文件 | `database/database-ddl.sql` | 执行的DDL |
| 设计文档 | `database/database-design.md` | 设计参照 |
| 实际数据库 | 目标数据库 | 通过SQL查询验证 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| DDL执行评审报告 | `database/reviews/REVIEW-DDL-EXECUTION-YYMMDDHHMM.md` | 评审结论 |

## 流转关系

```
通过 → 进入阶段3-实施阶段（310-java-uniweb-dev 等）
不通过 → 返回 300-database-ddl-execution 修复
```

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

## 量化通过标准

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

### 1. 准备阶段
- 读取执行报告和DDL文件
- 读取设计文档
- 连接目标数据库

### 2. 执行评审
逐表验证，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

**维度**: 表完整性/字段一致性/索引正确性/通用字段/初始数据
**评审对象**: `database/` + 实际数据库
**参与人员**: @system-architect @java-developer
**流转方向**: 通过 → 进入实施阶段；不通过 → 返回DDL执行修复

## 输出要求

**报告位置**: `database/reviews/REVIEW-DDL-EXECUTION-YYMMDDHHMM.md`

**必须包含**:
- 评审信息（日期、人员、数据库地址）
- 逐表验证结果
- 问题清单（含严重程度、表名、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项

## 参考

- [评审报告模版](../0-init/references/review-report-template.md) - 通用评审报告格式
- [DDL执行技能](../300-database-ddl-execution/SKILL.md) - 被评审的上游技能
- [数据库设计评审技能](../201-database-design-review/SKILL.md) - 设计评审参考
