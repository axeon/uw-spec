---
name: 201-database-design-review
description: 数据库设计评审技能。当数据库设计文档完成后触发：(1)检查表结构合理性和规范性, (2)评审索引设计和查询优化, (3)验证数据一致性和完整性保障, (4)检查数据库性能和扩展性, (5)检查数据安全和敏感信息保护, (6)输出评审结论和改进建议。请务必在用户提及数据库评审、表结构评审、索引评审、SQL评审、数据模型评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 数据库设计评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 数据库评审 | `system-architect` |
| 协作 | ORM映射确认 | `java-developer` |
| 协作 | 测试数据需求 | `test-engineer` |
| 协作 | 业务数据确认 | `product-manager` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [200-database-design/SKILL.md](../200-database-design/SKILL.md) | **必读全文**：设计规范、命名约定、设计流程、设计完成标准 |
| [200-database-design/references/database-design-guide.md](../200-database-design/references/database-design-guide.md) | 数据库设计指南 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 数据库设计文档 | `PROJECT_ROOT/database/database-design.md` | 表结构、索引、关联关系 |
| 需求文档 | `PROJECT_ROOT/requirement/prds/*` | 数据需求参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 数据库设计评审报告 | `PROJECT_ROOT/database/reviews/REVIEW-DB-YYMMDDHHMM.md` | 按时间戳命名，24小时制 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| 命名规范 | 表名符合`{模块简称}_{实体简称}`两段式、字段snake_case、索引前缀idx_/uk_ |
| 表结构 | 字段类型选择、约束完整、通用字段齐全 |
| 索引设计 | 主键、唯一索引、普通索引合理性 |
| 关联关系 | 外键命名`{关联表简称}_{关联实体简称}_id`、关联表、级联策略 |
| 性能优化 | 分区策略、归档方案、查询优化 |
| 安全性 | 敏感数据加密、权限控制 |

详细的各维度检查项见 [checklist.md](references/checklist.md)。命名规范对照 [命名规范](../200-database-design/references/naming-convention.md)。

## 通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 30分 |
| 命名规范符合度 | 表名100%符合`{模块简称}_{实体简称}`，字段/索引命名符合规范 | 20分 |
| 索引合理性 | 所有查询都有合适索引 | 25分 |
| 数据完整性 | 主外键约束完整 | 25分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（如：无主键、循环依赖）
- Major问题 >3个

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 表结构/索引/关联关系/性能/安全
**评审对象**: PROJECT_ROOT/database/
**参与人员**: @system-architect @java-developer @test-engineer


### 2. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `200-database-design` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

