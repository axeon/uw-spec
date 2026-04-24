---
name: 201-database-design-review
description: 数据库设计评审技能。当数据库设计文档完成后触发：(1)检查表结构合理性和规范性, (2)评审索引设计和查询优化, (3)验证数据一致性和完整性保障, (4)检查数据库性能和扩展性, (5)检查数据安全和敏感信息保护, (6)输出评审结论和改进建议。请务必在用户提及数据库评审、表结构评审、索引评审、SQL评审、数据模型评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 数据库设计评审

## 描述
对数据库设计文档进行全面评审，验证数据库设计的合理性、完整性、性能和可维护性。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 数据库设计完成后 | "评审数据库设计" |
| 检查数据库问题 | "检查数据库设计是否有问题" |
| 数据库评审 | "数据库设计评审" |

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

## 流转关系
```
通过 → 并行进入其他设计评审（211/221/231）
不通过 → 返回 200-database-design 修改
```

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

## 量化通过标准

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

### 1. 准备阶段
- **读取源技能**：读取 [200-database-design/SKILL.md](../200-database-design/SKILL.md) 全文，提取所有设计规范和命名约定，作为评审的权威依据
- 读取数据库设计文档
- 确认评审范围
- 准备检查清单

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: 表结构/索引/关联关系/性能/安全
**评审对象**: PROJECT_ROOT/database/
**参与人员**: @system-architect @java-developer @test-engineer
**流转方向**: 通过 -> 进入下一阶段评审；不通过 -> 返回数据库设计修改

## 输出要求
**报告位置**: `PROJECT_ROOT/database/reviews/REVIEW-DB-YYMMDDHHMM.md`

**命名规则**: 时间戳使用当前系统时间，24小时制，例如 `REVIEW-DB-2604021430.md`

**必须包含**:
- 评审信息（日期、人员、对象）
- 各维度评审结果和得分
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项
