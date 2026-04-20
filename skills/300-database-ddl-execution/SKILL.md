---
name: 300-database-ddl-execution
description: 数据库DDL执行与验证技能。当用户需要执行数据库初始化或迁移时触发：(1)执行DDL建表语句，(2)执行增量迁移脚本，(3)验证表结构一致性，(4)执行初始数据脚本。支持从配置文件读取MySQL连接参数。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.1.0"
---

# 数据库DDL执行与验证

## 描述

将 200-database-design 阶段产出的 DDL 文件在目标数据库上执行，验证执行结果与设计文档一致，为后端开发提供可用的数据库环境。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 执行DDL | "执行建表语句" |
| 数据库初始化 | "初始化数据库" |
| 执行迁移 | "执行迁移脚本" |
| 验证表结构 | "检查数据库表结构" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | DDL执行与验证 | `java-developer` |
| 协作 | 架构确认 | `system-architect` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| DDL文件 | `database/database-ddl.sql` | 200阶段产出的全量建表语句 |
| 迁移脚本 | `database/migrations/*.sql` | 增量DDL变更文件 |
| 设计文档 | `database/database-design.md` | 表结构设计文档（验证参照） |
| 评审报告 | `database/reviews/REVIEW-DB-*.md` | 201评审通过的报告 |

### 前置条件

- [ ] 201-database-design-review 评审通过（评分 ≥ 95分）
- [ ] 目标数据库可连接
- [ ] DDL文件存在且非空

## 执行流程

### 1. 环境检查

| 检查项 | 方式 | 预期 |
|--------|------|------|
| 数据库连接 | `mysql -h {host} -u {user} -p{pass} -e "SELECT 1"` | 连接成功 |
| 数据库存在 | `SHOW DATABASES LIKE '{db_name}'` | 已存在或可创建 |
| DDL文件存在 | 检查文件路径 | 文件存在且非空 |
| 迁移目录存在 | 检查目录 | 目录存在（可为空） |

### 2. 获取数据库连接信息

#### 2.1 读取配置文件

尝试从 `.uniweb/project-server.config` 读取 MySQL 连接参数：

| 参数 | 配置项 | 说明 |
|------|--------|------|
| 主机 | `MYSQL_HOST` | 数据库主机地址 |
| 端口 | `MYSQL_PORT` | 数据库端口（默认3306） |
| 用户名 | `MYSQL_ROOT_USERNAME` | 管理员用户名 |
| 密码 | `MYSQL_ROOT_PASSWORD` | 管理员密码 |

**读取逻辑**：
```
if (配置文件存在) {
  读取参数
  向用户确认（密码显示为 ***）
} else {
  提示用户手动输入
}
```

#### 2.2 确认连接信息

使用 `AskUserQuestion` 确认或补充参数：

| # | header | question |
|---|--------|----------|
| 1 | 数据库地址 | 确认 MySQL 连接地址（当前: {host}:{port}）？ |
| 2 | 数据库名 | 确认数据库名称（当前: {project_name}）？ |
| 3 | 用户名 | 确认数据库用户名（当前: {user}）？ |
| 4 | 密码 | 确认数据库密码（已隐藏）？ |
| 5 | 执行模式 | 首次建库还是增量迁移？ |

### 3. DDL执行

**数据库名生成规则**：
```
db_name = project_name.replace('-', '_')
```

**首次全量执行**：

```bash
mysql -h {host} -P {port} -u {user} -p{pass} {db_name} < database/database-ddl.sql
```

**增量迁移执行**：

1. 列出 `migrations/` 目录下最新的5个 SQL 文件（按修改时间倒序）
2. 使用 `AskUserQuestion` 让用户选择或手动输入：

| # | header | question | 选项 |
|---|--------|----------|------|
| 1 | 选择迁移文件 | 请选择要执行的迁移文件？ | 最新5个文件 + "手动输入" |

3. 执行选中的 SQL 文件：

```bash
mysql -h {host} -P {port} -u {user} -p{pass} {db_name} < "{selected_file}"
```

**执行原则**：

| 原则 | 说明 |
|------|------|
| 先建库再建表 | 确保数据库存在后再执行DDL |
| 按顺序执行 | 迁移文件按文件名时间戳排序 |
| 失败即停 | 任一SQL执行失败，停止后续执行并报告 |
| 不自动删库 | 禁止 `DROP DATABASE`，需人工确认 |

### 4. Schema验证

| 验证项 | SQL | 预期 |
|--------|-----|------|
| 表数量 | `SHOW TABLES` | 与设计文档一致 |
| 表结构 | `SHOW CREATE TABLE {table}` | 字段、类型、索引与DDL一致 |
| 字段数量 | `DESCRIBE {table}` | 每张表字段数与设计文档匹配 |
| 索引 | `SHOW INDEX FROM {table}` | 索引存在且类型正确 |
| 通用字段 | 检查每张表 | id/saas_id/state/create_date/modify_date 齐全 |

### 5. 初始数据（如有）

检查 `database/` 下是否有初始数据脚本（如 `init-data.sql`），有则执行。

### 6. 生成执行报告

**报告位置**：`database/log/DDL-EXECUTION-REPORT-{YYMMDDHHMM}.md`

```markdown
# DDL执行报告

## 执行信息
- 执行日期: {YYYY-MM-DD HH:mm}
- 目标数据库: {host}/{db_name}
- 执行模式: 全量/增量

## 执行结果
| 文件 | 状态 | 耗时 | 说明 |
|------|------|------|------|
| database-ddl.sql | ✅/❌ | {ms} | {说明} |

## Schema验证
| 验证项 | 结果 | 详情 |
|--------|------|------|
| 表数量 | ✅/❌ | 预期{N}张，实际{M}张 |
| 字段一致性 | ✅/❌ | {不一致的表} |
| 索引完整性 | ✅/❌ | {缺失的索引} |

## 执行统计
- 成功SQL: {N}条
- 失败SQL: {N}条
- 警告: {N}条
```

## PLAN-REVIEW 循环（必须执行）

DDL执行完成后，必须进入 PLAN-REVIEW 循环。

#### 循环规则
| 规则 | 说明 |
|------|------|
| 通过条件 | review 评分 ≥ 95分 |
| 强制退出 | 最多5轮review（round=1~5），round ≥ 6强制退出 |
| 每轮操作 | review → 修复 → 重新执行验证 |

#### 循环流程
```
循环开始 (round=1)
  │
  ├─▶ 调用技能: 301-database-ddl-execution-review
  │
  ├─▶ 判断结果:
  │    ├─ 评分 ≥ 95 → 输出结论，循环结束 ✓
  │    └─ 评分 < 95 → 修复并重新执行验证
  │
  └─▶ round ≥ 6 → 强制退出，输出当前结论 ⚠️
```

## 输出要求

**执行报告**: `database/DDL-EXECUTION-REPORT-{YYMMDDHHMM}.md`

**评审报告**: `database/reviews/REVIEW-DDL-EXECUTION-YYMMDDHHMM.md`

## 流转关系

```
输入: 200设计产出 + 201评审通过
    ↓
300-database-ddl-execution
    ↓
301-database-ddl-execution-review
    ↓
通过 → 进入阶段3（310-java-uniweb-dev 等）
不通过 → 修复DDL并重新执行
```

## 参考

- [数据库设计技能](../200-database-design/SKILL.md) - 上游设计阶段
- [数据库设计评审技能](../201-database-design-review/SKILL.md) - 上游评审阶段
- [DDL执行评审技能](../301-database-ddl-execution-review/SKILL.md) - PLAN-REVIEW循环评审
