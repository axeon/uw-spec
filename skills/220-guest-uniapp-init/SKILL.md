---
name: 220-guest-uniapp-init
description: 移动端UniApp项目初始化技能。当需要创建UniApp跨平台项目时触发：(1)从uw-uniapp-template.zip模板解压项目结构, (2)全文替换模板关键字为实际项目名, (3)验证项目结构完整性。请务必在用户提及UniApp项目初始化、小程序项目创建、移动端项目搭建、跨平台项目初始化时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 移动端项目初始化

## 描述

从 `assets/` 目录下的模板 zip 解压创建移动端 UniApp 项目，全文替换模板关键字为实际项目名，生成可用的项目脚手架。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 初始化移动端项目 | "初始化UniApp项目" |
| 创建小程序项目 | "创建微信小程序项目" |
| 跨平台移动开发 | "创建移动端项目" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 项目初始化 | `js-developer` |
| 协作 | 架构确认 | `system-architect` |


## 模板选择

| 模板文件 | 适用场景 | 说明 |
|---------|---------|------|
| `saas-uniapp-template.zip` | SaaS多租户项目 | 基于 saas-base 框架，含租户/商户管理 |
| `uw-uniapp-template.zip` | UniWeb基础项目 | 基于 uw-base 框架，通用移动应用 |

**选择时机**：在 `0-init` 阶段通过 `PROJECT_ROOT/project-info.md` 的 `project-mode` 字段（`uniweb`/`saas`）确定，本脚本自动读取。

## 初始化流程

**执行前必须询问用户确认**。

### 步骤1: 询问用户参数

使用 `AskUserQuestion` 工具询问用户以下信息：

| # | header | question | 参考选项 | multiSelect |
|---|--------|----------|----------|-------------|
| 1 | 用户角色 | 这个UniApp端面向哪个用户角色？( admin:总后台, saas:SAAS管理, mch:SAAS商户, guest:消费者) | admin, saas, mch, guest | false |
| 2 | 覆盖模式 | 如果目录已存在，如何处理？(force:强制覆盖, skip:跳过已存在文件) | force, skip | false |

### 步骤2: 用户确认

**必须**向用户展示以下信息并等待确认：

```
即将执行 UniApp 移动端项目初始化：
- 项目名: {project-name}
- 前端项目名: {project-name}-{角色}-uniapp
- 角色: {用户选择的角色}
- 输出目录: $PROJECT_ROOT/frontend/{project-name}-{角色}-uniapp/

确认执行吗？
```

用户确认后，再执行脚本。

### 步骤3: 执行脚本

用户确认后，执行命令（确保在技能目录下执行）：
```bash
bash scripts/init.sh [目标目录] [模式] [角色]
```

**参数说明**：

| 参数 | 说明 | 是否必填 | 默认值 |
|------|------|----------|--------|
| 目标目录 | 项目根目录（包含 `project-info.md`） | 是 | 当前目录 |
| 模式 | `force`(强制覆盖), `skip`(跳过已存在文件) | 否 | force |
| 角色 | 用户角色（guest/mch/saas/admin/root） | 否 | guest |

**执行示例**：

```bash
# 初始化 guest-uniapp（默认角色）
bash scripts/init.sh /Users/user/project force

# 初始化 mch-uniapp
bash scripts/init.sh /Users/user/project force mch

# 初始化 guest-uniapp，跳过已存在文件
bash scripts/init.sh /Users/user/project skip guest
```

**脚本输出说明**：
- `✓` 表示通过
- `⚠` 表示警告
- `ERROR` 表示错误，会终止执行

## 输出要求

**输出位置**: `PROJECT_ROOT/frontend/{项目名}-{角色}-uniapp/` 目录

**包含内容**: 从模板解压并替换后的完整项目结构

**目录结构**:
```
项目根目录/
├── project-info.md
└── $PROJECT_ROOT/frontend/
    └── {项目名}-{角色}-uniapp/
        ├── src/
        ├── package.json
        ├── tsconfig.json
        └── vite.config.ts
```

**命名示例**（假设项目名为 `my-shop`）：
| 角色 | 前端项目名 | 目录 |
|------|-----------|------|
| root | my-shop-root-uniapp | PROJECT_ROOT/frontend/my-shop-root-uniapp/ |
| ops | my-shop-ops-uniapp | PROJECT_ROOT/frontend/my-shop-ops-uniapp/ |
| admin | my-shop-admin-uniapp | PROJECT_ROOT/frontend/my-shop-admin-uniapp/ |
| saas | my-shop-saas-uniapp | PROJECT_ROOT/frontend/my-shop-saas-uniapp/ |
| mch | my-shop-mch-uniapp | PROJECT_ROOT/frontend/my-shop-mch-uniapp/ |
| guest | my-shop-guest-uniapp | PROJECT_ROOT/frontend/my-shop-guest-uniapp/ |

## 下一步

初始化完成后，提示用户进入 **220-guest-uniapp-gencode** 进行移动端代码生成（从后端Swagger生成api/router/page/i18n）。

**流程位置**：`220-guest-uniapp-init` → **220-guest-uniapp-gencode** → `220-guest-uniapp-design` (+ 221 review) → `320-guest-uniapp-dev`

## 参考

- [初始化脚本](scripts/init.sh) - 自动化初始化脚本
- [SaaS模板](assets/saas-uniapp-template.zip) - SaaS UniApp模板
- [UniWeb模板](assets/uw-uniapp-template.zip) - UniWeb UniApp模板
