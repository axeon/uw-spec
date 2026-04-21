---
name: 210-java-uniweb-init
description: Java后端项目初始化技能。当需要创建Java后端API项目时触发：(1)从uw-app-template.zip模板解压项目结构, (2)全文替换模板关键字为实际项目名, (3)验证项目结构完整性。请务必在用户提及Java项目初始化、后端项目创建、Spring Boot项目、API项目搭建时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 后端项目初始化

## 描述

从 `assets/` 目录下的模板 zip 解压创建后端API项目，全文替换模板关键字为实际项目名，生成可用的项目脚手架。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 初始化后端项目 | "初始化后端项目" |
| 创建Java项目 | "创建Java项目" |
| 新建API项目 | "新建API项目" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 项目初始化 | `java-developer` |
| 协作 | 架构确认 | `system-architect` |

## 前置检查

检查 `project-info.md` 是否存在且 5 个参数完整，不通过则引导用户执行 0-init。
详见 [项目信息前置检查](../0-init/references/project-init-check.md)。

## 模板选择

| 模板文件 | 适用场景 | 说明 |
|---------|---------|------|
| `saas-app-template.zip` | SaaS多租户项目 | 基于 saas-base 框架，含租户/商户管理、AIP/AIS |
| `uw-app-template.zip` | UniWeb基础项目 | 基于 uw-base 框架，通用API应用 |

**选择时机**：在 `0-init` 阶段通过 `project-info.md` 的 `project-mode` 字段（`uniweb`/`saas`）确定，本脚本自动读取。

## 初始化流程

**执行前必须询问用户确认**。

### 步骤1: 展示后端项目目录

**目标目录**：直接使用当前项目根目录（包含 `project-info.md` 的目录）。

检查并展示后端目录状态：

```bash
ls -la backend/ 2>/dev/null || echo "backend/ 目录不存在，将自动创建"
```

**后端项目将初始化在**：`backend/{project-name}-app/`

### 步骤2: 询问用户参数

使用 `AskUserQuestion` 工具询问用户以下信息：

| # | header | question | 参考选项 | multiSelect |
|---|--------|----------|----------|-------------|
| 1 | 覆盖模式 | 如果目录已存在，如何处理？(force:强制覆盖, skip:跳过已存在文件) | force, skip | false |

### 步骤3: 用户确认

**必须**向用户展示以下信息并等待确认：

```
即将执行 Java 后端项目初始化：
- 项目名: {project-name}
- 项目模式: {project-mode}（uniweb/saas）
- 输出目录: backend/{project-name}-app/
- 覆盖模式: {用户选择的模式}

确认执行吗？
```

用户确认后，再执行脚本。

### 步骤4: 执行脚本

用户确认后，执行命令（确保在技能目录下执行）：
```bash
bash scripts/init.sh [目标目录] [模式]
```

**参数说明**：

| 参数 | 说明 | 是否必填 | 默认值 |
|------|------|----------|--------|
| 目标目录 | 项目根目录（包含 `project-info.md`） | 是 | 当前目录 |
| 模式 | `force`(强制覆盖), `skip`(跳过已存在文件) | 否 | force |

**执行示例**：

```bash
# 强制覆盖模式（默认）
bash scripts/init.sh /Users/user/project

# 强制覆盖模式（显式指定）
bash scripts/init.sh /Users/user/project force

# 跳过已存在文件
bash scripts/init.sh /Users/user/project skip
```

**脚本输出说明**：
- `✓` 表示通过
- `⚠` 表示警告
- `ERROR` 表示错误，会终止执行

## 输出要求

**输出位置**: `backend/{项目名}-app/` 目录

**包含内容**: 从模板解压并替换后的完整项目结构

**目录结构**:
```
项目根目录/
├── project-info.md
└── backend/
    └── {项目名}-app/
        ├── src/
        └── pom.xml
```

## 参考

- [初始化脚本](scripts/init.sh) - 自动化初始化脚本
- [SaaS模板](assets/saas-app-template.zip) - SaaS后端项目模板
- [UniWeb模板](assets/uw-app-template.zip) - UniWeb后端项目模板
