---
name: 610-feature-tech-design
description: 功能技术方案设计技能。当需求澄清完成后触发：(1)设计数据库表结构和DDL, (2)设计后端接口和架构, (3)设计前端页面和交互, (4)设计测试策略和用例, (5)生成日期开头的DDL文件, (6)AI自动评审技术方案, (7)合并到各端主文档。请务必在用户提及技术方案设计、架构设计、数据库设计、接口设计时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能技术方案设计

## 描述

基于已澄清的需求，设计完整的技术方案，包括数据库设计、后端架构、前端实现方案、测试策略，并生成可执行的DDL文件。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 技术方案设计 | "设计订单导出功能的技术方案" |
| 数据库设计 | "设计数据库表结构" |
| 接口设计 | "设计API接口" |
| 架构设计 | "设计前后端架构" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 技术方案设计 | `system-architect` |
| 协作 | 数据库设计 | `java-developer` |
| 协作 | 接口设计 | `java-developer` |
| 协作 | 前端设计 | `js-developer` |
| 协作 | 测试策略 | `test-engineer` |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能需求文档 | `issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段输出 |
| 现有数据库设计 | `database/database-design.md` | 现有DB设计 |
| 现有后端架构 | `backend/{项目名}-app/README.md` | 现有架构文档 |
| 现有前端架构 | `frontend/{项目名}-{用户角色}-{终端类型}/README.md` | 现有Web架构 |
| 现有移动端架构 | `frontend/{项目名}-{用户角色}-{终端类型}/README.md` | 现有移动端架构 |
| 现有测试设计 | `test/design/` | 现有测试设计文档 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| DDL文件 | `database/migrations/FEATURE-{YYMMDD}-{简述}.sql` | 数据库变更DDL |
| 后端技术方案 | `backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | Java后端方案 |
| Web前端方案 | `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | Web前端方案 |
| 移动端方案 | `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 移动端方案 |
| 测试方案 | `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test-design.md` | 本功能的测试策略（供 640 参考，非 230 全量测试设计） |
| 更新主文档 | 各端README.md + database-design.md | 合并后的主文档 |
| 变更记录 | 各端CHANGELOG.md | 技术变更历史 |

## 执行流程

### 1. 数据库设计

**分析需求**:
- 识别新增/修改的数据实体
- 定义表结构和字段
- 设计索引和约束

**生成DDL文件**:

**文件命名**: `database/migrations/FEATURE-{YYMMDD}-{简述}.sql`

**DDL内容**:
```sql
-- FEATURE-{YYMMDD}-{功能名称}
-- 功能描述: {简述}
-- 创建时间: {YYYY-MM-DD}

-- 新增表
CREATE TABLE {table_name} (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    -- 字段定义
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 新增字段
ALTER TABLE {existing_table} ADD COLUMN {column_name} {type} {constraint};

-- 新增索引
CREATE INDEX idx_{name} ON {table_name}({column_list});
```

**更新数据库设计文档**:
- 合并到 `database-design.md`
- 追加到 `database-ddl.sql`
- 记录 `database/CHANGELOG.md`

### 2. 后端技术方案设计

**设计内容**:

| 设计项 | 内容 |
|--------|------|
| 模块划分 | 涉及的后端模块 |
| 接口设计 | API路径、请求/响应格式 |
| 数据流 | 数据流转过程 |
| 权限设计 | 权限点和角色映射 |
| 缓存策略 | 缓存方案和过期策略 |
| 异常处理 | 异常场景和处理方式 |

**生成后端技术方案文档**:

**文件位置**: `backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

**文档内容**:
```markdown
# FEATURE-DESIGN-{YYMMDD}-{功能名称} - 后端技术方案

## 接口设计

### API列表
| 接口 | 路径 | 方法 | 说明 |
|------|------|------|------|
| {api_name} | {path} | {method} | {description} |

### 请求/响应
{schema定义}

## 数据模型
### 实体变更
- {表名}: {变更说明}

## 实现要点
- {实现要点1}
- {实现要点2}

## 依赖服务
- {依赖服务列表}
```

### 3. Web前端技术方案设计

**设计内容**:

| 设计项 | 内容 |
|--------|------|
| 页面结构 | 页面布局和路由 |
| 组件设计 | 新增/复用组件 |
| 状态管理 | Pinia store设计 |
| API对接 | 后端接口调用 |
| 交互流程 | 用户操作流程 |

**生成Web前端技术方案文档**:

**文件位置**: `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

### 4. 移动端技术方案设计

**设计内容**: 同Web前端，增加平台适配考虑

**生成移动端技术方案文档**:

**文件位置**: `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

### 5. 测试方案设计

**设计内容**:

| 设计项 | 内容 |
|--------|------|
| 测试范围 | 需要测试的功能点和接口 |
| 测试类型 | API测试、E2E测试、边界测试 |
| 测试策略 | 正向/逆向/边界/异常 |
| 测试数据 | 需要准备的测试数据 |
| 回归范围 | 需要回归的关联功能 |

**生成测试方案文档**:

**文件位置**: `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test-design.md`

**文档内容**:
```markdown
# FEATURE-{YYMMDD}-{功能名称} - 测试方案

## 测试范围
### API测试
| 接口 | 测试场景 | 优先级 |
|------|---------|--------|
| {api} | {scenario} | {priority} |

### E2E测试
| 场景 | 操作步骤 | 预期结果 |
|------|---------|---------|
| {scenario} | {steps} | {expected} |

### 边界测试
| 边界条件 | 测试方法 | 预期结果 |
|---------|---------|---------|
| {condition} | {method} | {expected} |

### 异常测试
| 异常场景 | 触发方式 | 预期处理 |
|---------|---------|---------|
| {scenario} | {trigger} | {handling} |

## 测试数据
- {数据需求1}
- {数据需求2}

## 回归范围
- {需要回归的关联功能}
```

### 6. AI自动评审

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 完整性 | 是否覆盖所有需求点 |
| 一致性 | 前后端接口是否对齐 |
| 规范性 | 是否符合技术规范 |
| 可扩展性 | 是否考虑未来扩展 |
| 性能 | 是否考虑性能优化 |
| 可测试性 | 测试方案是否可执行 |

**评审结果处理**:
- Critical问题：必须修复
- Major问题：建议修复
- Minor问题：记录即可

### 7. 文档合并

**合并规则**:

| 文档 | 合并位置 | 方式 |
|------|---------|------|
| DDL | database-design.md + database-ddl.sql | 追加 |
| 后端方案 | backend/{项目名}-app/README.md | 更新架构章节 |
| Web方案 | frontend/{项目名}-{用户角色}-{终端类型}/README.md | 更新架构章节 |
| 移动端方案 | frontend/{项目名}-{用户角色}-{终端类型}/README.md | 更新架构章节 |
| 测试方案 | test/CHANGELOG.md | 追加 |

**CHANGELOG更新**：追加到各端 CHANGELOG.md，记录技术方案设计状态。

## PLAN-REVIEW 循环（必须执行）

技术方案设计完成后，必须进入 PLAN-REVIEW 循环，确保方案质量达标。

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
  ├─▶ 调用技能: 611-feature-tech-design-review
  │    对技术方案执行评审
  │
  ├─▶ 获取评审报告: issue/reviews/REVIEW-DESIGN-YYMMDDHHMM.md
  │
  ├─▶ 判断结果:
  │    ├─ 评分 ≥ 95 → 输出结论，循环结束 ✓
  │    └─ 评分 < 95 → 进入修复步骤
  │
  ├─▶ 修复问题:
  │    按评审报告中的问题清单（Critical > Major > Minor）逐项修复
  │    修复后 round++，回到循环开始
  │
  └─▶ round ≥ 6 → 强制退出，输出当前结论 ⚠️
```

#### 每轮修复要求
| 优先级 | 要求 |
|--------|------|
| Critical | 本轮必须全部修复，不可遗留 |
| Major | 本轮修复 ≥ 80%，剩余标注下轮计划 |
| Minor | 记录但不阻塞，按优先级排列 |

## 人工检查点 ★

**必须人工确认**:
- [ ] 数据库设计是否合理（索引、字段类型）
- [ ] 接口设计是否完整（参数、响应）
- [ ] 前后端接口是否对齐
- [ ] 测试方案是否覆盖所有需求点
- [ ] 技术方案是否可实施

**确认后流转**: 并行进入 620/630/640

## 610→下游衔接协议

610 设计完成后，各下游技能从以下文件提取开发输入：

| 下游技能 | 提取文件 | 用途 |
|---------|---------|------|
| 620-feature-java-uniweb-dev | `backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-*-tech-design.md` | 后端技术方案 |
| 630-feature-admin-web-dev / 630-feature-guest-web-dev | `frontend/{项目名}-{用户角色}-web/issues/FEATURE-DESIGN-{YYMMDD}-*-tech-design.md` | Web前端方案 |
| 630-feature-admin-uniapp-dev / 630-feature-guest-uniapp-dev | `frontend/{项目名}-{用户角色}-uniapp/issues/FEATURE-DESIGN-{YYMMDD}-*-tech-design.md` | 移动端方案 |
| 640-feature-test-dev | `test/issues/FEATURE-DESIGN-{YYMMDD}-*-test-design.md` | 测试方案 |

**并行约束**：四端天然独立，可同时由不同Agent开发。DDL需先执行，后端开发依赖DDL。

## 输出要求

**DDL文件**: `database/migrations/FEATURE-{YYMMDD}-{简述}.sql`

**后端方案**: `backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

**Web方案**: `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

**移动端方案**: `frontend/{项目名}-{用户角色}-{终端类型}/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md`

**测试方案**: `test/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-test-design.md`

## 流转关系

```
输入: 功能需求文档
    ↓
610-feature-tech-design
    ↓
AI自动评审
    ↓
人工确认 ★
    ↓
并行输出: DDL + 各端技术方案 + 测试方案
    ↓
并行进入: 620/630/631/640
```
