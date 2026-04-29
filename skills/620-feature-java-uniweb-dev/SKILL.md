---
name: 620-feature-java-uniweb-dev
description: 功能Java后端开发技能。当技术方案设计完成后触发：(1)基于技术方案生成Java后端代码, (2)实现Helper业务逻辑和Controller接口, (3)编写单元测试和集成测试, (4)调用621-feature-java-uniweb-dev-review自动评审, (5)自动修复代码问题, (6)合并代码到主分支并更新CHANGELOG。请务必在用户提及Java后端开发、实现后端功能、开发API接口时使用此技能 ⚠️【强制】完成后必须调用 621-feature-java-uniweb-dev-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能Java后端开发

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码生成实现 + 单元测试 | `java-developer` |
| 协作 | 代码评审 | `java-lead` |

> **职责边界**：Java 单元测试由 Java 开发工程师负责（Helper/Service 逻辑验证）。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 640）。

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 技术方案 | `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-tech-design.md` | 610阶段输出 |
| 现有代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 现有代码基线 |
| 技术规范 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 编码规范 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 功能代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | Java后端代码 |
| 测试代码 | `PROJECT_ROOT/backend/{项目名}-app/src/test/` | 单元测试和集成测试 |
| 开发文档 | `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-java-uniweb.md` | 开发记录 |
| 评审报告 | `PROJECT_ROOT/backend/{项目名}-app/reviews/` | AI评审报告 |
| 更新文档 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 合并后的主文档 |
| 变更记录 | `PROJECT_ROOT/backend/{项目名}-app/CHANGELOG.md` | 代码变更历史 |

## 610→620 衔接协议

610 设计完成后，620 从以下文件提取开发输入：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| 后端技术方案 | `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-*-tech-design.md` | 接口设计、数据模型、实现要点 |
| DDL文件 | `PROJECT_ROOT/database/migrations/FEATURE-*.sql` | 数据库变更，需先执行 |
| 现有架构 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 编码规范、模块结构 |

**并行约束**：620 与 630/631/640 天然独立可并行。DDL需先执行完毕后才能开始 620。

## 执行流程

### 1. 代码生成

**生成内容**:

| 代码类型 | 生成位置 | 说明 |
|---------|---------|------|
| Entity/DTO | `entity/`, `dto/` | 数据实体 |
| Helper | `service/` | 业务逻辑 |
| Controller | `controller/` | API接口 |
| Mapper | `mapper/` | 数据访问（如需要） |
| 单元测试 | `PROJECT_ROOT/test/` | JUnit测试 |

**代码规范**:
- 遵循UniWeb + SaaS技术栈规范
- 使用DaoManager进行数据访问
- 使用FusionCache进行缓存
- 使用ResponseData统一响应

### 2. AI自动评审

**REVIEW评审技能**: `621-feature-java-uniweb-dev-review`

**评审维度**:

| 维度 | 检查项 |
|------|--------|
| 编码规范 | 命名、格式、注释 |
| 业务逻辑 | 是否正确实现需求 |
| 异常处理 | 是否完善 |
| 性能 | SQL优化、缓存使用 |
| 安全 | 输入校验、权限控制 |
| 测试覆盖 | 单元测试完整性 |

### 3. 自动修复

**修复策略**:

| 问题级别 | 处理方式 |
|---------|---------|
| Critical | 必须修复，循环直到通过 |
| Major | 自动修复80%，剩余记录 |
| Minor | 记录但不阻塞 |

## ⚠️ 完成验证（强制）

> 在声称"任务完成"之前，必须重新读取本节，逐项确认。

Java后端开发完成后，调用 `621-feature-java-uniweb-dev-review`。

- [ ] 已重新读取本节全部内容
- [ ] 已调用 `621-feature-java-uniweb-dev-review`
- [ ] 已收到评审通过确认（评分 ≥ 95）

> 未收到通过确认前，禁止结束开发任务。
> 以上检查项未全部勾选，禁止向用户报告"已完成"。


### 4. 测试执行

**测试类型**:

| 测试类型 | 命令 | 通过标准 |
|---------|------|---------|
| 编译检查 | `mvn compile` | 无编译错误 |
| 单元测试 | `mvn test` | 通过率100%，全绿 |
| Swagger检查 | 启动验证 | 接口可展示 |

### 5. 生成开发文档

**文件位置**: `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-java-uniweb.md`

**文档内容**:
```markdown
# FEATURE-{YYMMDD}-{功能名称} - Java后端开发

## 实现概要
- 实现日期: {YYYY-MM-DD}
- 开发人员: AI Assistant

## 代码变更
### 新增文件
- `{文件路径}`: {说明}

### 修改文件
- `{文件路径}`: {说明}

## 接口清单
| 接口 | 路径 | 说明 |
|------|------|------|
| {name} | {path} | {desc} |

## 测试覆盖
- 单元测试: {数量}个
- 覆盖率: {百分比}%

## 评审结果
- 评分: {分数}
- 状态: {通过/有条件通过}
- 遗留问题: {数量}个
```

### 6. 文档合并

**合并内容**:
- 更新 `PROJECT_ROOT/backend/{项目名}-app/README.md`（接口文档、架构说明）
- 追加 `PROJECT_ROOT/backend/{项目名}-app/CHANGELOG.md`

**CHANGELOG格式**:
```markdown
## {YYYY-MM-DD} FEATURE-{功能名称}
- 类型: Java后端开发
- 变更: {简述}
- 文档: FEATURE-DESIGN-{YYMMDD}-{简述}-java-uniweb.md
- 评审: {评分}分
- 状态: 已完成
```

## 输出要求

**代码位置**: `PROJECT_ROOT/backend/{项目名}-app/src/`

**开发文档**: `PROJECT_ROOT/backend/{项目名}-app/issues/FEATURE-DESIGN-{YYMMDD}-{简述}-java-uniweb.md`

**评审报告**: `PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-CODE-{YYMMDDHHMM}.md`

## 流转关系

```
输入: 技术方案
    ↓
620-feature-java-uniweb-dev
    ↓
AI生成代码 → AI评审(311) → 自动修复 → 测试执行
    ↓
输出: Java后端代码 + 测试代码
    ↓
等待: 650-feature-final-review（最终验收）
```

## 参考

- [Java开发评审技能](../621-feature-java-uniweb-dev-review/SKILL.md) - REVIEW评审技能
- [UniWeb开发规范](../210-java-uniweb-design/references/backend/uniweb/dev-standards.md)
- [SaaS开发规范](../210-java-uniweb-design/references/backend/saas/README.md)
