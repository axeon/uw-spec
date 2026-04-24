---
name: 0-init
description: uw-spec流程入口技能。当用户想要开始软件开发项目时触发：(1)确认并保存项目信息到project-info.md, (2)协调需求规划阶段, (3)管理设计阶段任务, (4)指导项目实施开发, (5)管理测试阶段执行, (6)管理文档交付阶段, (7)管理功能开发阶段, (8)管理Bug修复阶段。请务必在用户提及想开发应用、开始新项目、软件开发、uw-spec、创建系统、做一个小程序时使用此技能, 即使用户没有明确说"项目初始化"。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 项目初始化 - uw-spec流程入口

## 触发检查

**第一层：检查用户输入**

| 条件 | 操作 |
|------|------|
| 用户已描述具体项目/产品 | 进入第二层检查 |
| 用户仅触发技能，未提供任何信息 | 使用 `AskUserQuestion` 提示用户输入项目描述 |

**提示话术**（无输入时）：

```
AskUserQuestion({
  questions: [{
    question: "请描述你想开发的产品或项目。例如：一个SaaS多租户的电商后台管理系统、面向餐饮商家的点餐小程序等",
    header: "项目描述",
    type: "text",
    options: [
      { label: "提交描述", description: "使用上方输入框中的内容继续" },
      { label: "先看示例", description: "查看一些项目示例帮助我思考" }
    ],
    multiSelect: false
  }]
})
```

用户回答后，将描述内容作为 `project-desc` 进入第二层检查。

**第二层：检查 `PROJECT_ROOT/project-info.md` 是否存在**

| 条件 | 操作 |
|------|------|
| `PROJECT_ROOT/project-info.md` 存在 | 展示项目信息，询问用户从哪个阶段开始 |
| `PROJECT_ROOT/project-info.md` 不存在 | 进入「项目信息初始化」流程，使用用户输入的描述填充第3步 |

**存储规则**：
- **≤300字**：直接存入 YAML 头部的 `project-desc`
- **>300字**：`project-desc` 存储摘要（前100字+"..."），完整内容存入正文「项目概述」章节

**从 100/110 跳转而来的场景**：
当从 100-rapid-idea-check 或 110-requirement-planning 跳转而来时，0-init 完成初始化后**自动触发回到来源技能**继续执行。

| 来源 | 初始化完成后 |
|------|-------------|
| 100-rapid-idea-check | 自动回到 100 执行五维分析 |
| 110-requirement-planning | 自动回到 110 执行 Phase 1 需求访谈 |

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](references/project-env-check.md)。**未找到** → 使用当前工作目录作为 `PROJECT_ROOT`（新项目场景）。

## 八阶段模型

| 阶段 | 编号 | 说明 |
|------|------|------|
| 阶段0 | 0-init | 主流程控制 |
| 阶段1 | 1-requirement | 需求阶段 - 明确做什么 |
| 阶段2 | 2-design | 设计阶段 - 明确怎么做 + 项目准备 |
| 阶段3 | 3-project | 实施阶段 - 按计划执行开发 |
| 阶段4 | 4-test | 测试阶段 - 执行自动化测试 |
| 阶段5 | 5-manual | 文档交付阶段 - 文档整理与项目归档 |
| 阶段6 | 6-feature | 功能开发阶段 - 新功能开发（按需） |
| 阶段7 | 7-bugfix | Bug修复阶段 - Bug修复（按需） |

## 角色团队

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 产品经理 | 需求分析、产品设计 | `product-manager` |
| 系统架构师 | 架构设计、技术选型 | `system-architect` |
| 项目经理 | 项目规划、进度管理 | `project-manager` |
| Java后端工程师 | 基于uw-base的后端开发 | `java-developer` |
| Java技术主管 | 后端代码评审、架构把控 | `java-lead` |
| JS前端工程师 | Vue3/UniApp前端开发 | `js-developer` |
| JS前端主管 | 前端代码评审、规范把控 | `js-lead` |
| 测试工程师 | 测试设计、质量保证 | `test-engineer` |
| 测试主管 | 测试评审、测试质量把控 | `test-lead` |
| 运维工程师 | 运维文档、部署、监控 | `devops-engineer` |
| 运维主管 | 运维评审、发布把控 | `devops-lead` |
| UI设计师 | 界面设计、交互规范 | `ui-designer` |
| 原型评审员 | 原型质量评审 | `prototype-reviewer` |

## 技术栈

### 后端
- **基础架构**: UniWeb + SaaS技术栈 (Spring Boot + Spring Cloud)
- **数据库**: MySQL 8.4
- **缓存**: Redis 8.2
- **消息队列**: RabbitMQ

### 前端Web
- **框架**: Vue 3 + TypeScript
- **UI组件库**: Element Plus
- **构建工具**: Vite 8
- **状态管理**: Pinia

### 前端移动端
- **框架**: UniApp + Vue 3
- **跨平台**: H5、Android、iOS、微信小程序
- **UI组件库**: uni-ui

## 指令

### 启动流程

1. **项目信息初始化**（最先执行）

   **逐个询问，一问一答**：

   | 步骤 | 操作 |
   |------|------|
   | 1 | 检查 `PROJECT_ROOT/project-info.md` 是否存在 |
   | 2 | **如不存在**：使用 `AskUserQuestion` 逐个询问以下4项 |
   | 3 | **如已存在**：展示现有信息，询问是否继续使用 |

   **项目信息五问（逐个提问）**：

   使用 `AskUserQuestion` 工具逐一询问，每问一题等待用户回答：

   | # | header | question | 参考选项 | multiSelect |
   |---|--------|----------|----------|-------------|
   | 1 | 项目名称 | 请设置项目名称（英文小写+数字+下划线） | 根据用户输入动态生成建议 | false |
   | 2 | 项目标签 | 项目中文名称是什么？ | 根据项目名称自动生成建议 | false |
   | 3 | 项目描述 | 请简要描述这个项目 | 让用户自由录入 | false |
   | 4 | 项目模式 | 项目使用哪种技术模式？ | uniweb（UniWeb基础项目）、saas（SaaS多租户项目） | false |
   | 5 | 开发服务器 | 开发服务器IP地址是什么？ | 127.0.0.1、192.168.x.x、其他（请填写） | false |

   **执行规则**：
   - 必须逐个提问，等待用户回答后再问下一个
   - 禁止使用纯文本列出所有问题让用户批量回答
   - 每个问题提供参考选项，同时支持用户自由录入

   **创建项目信息文件**：
   - 5个问题全部回答后，创建 `PROJECT_ROOT/project-info.md`
   - 使用YAML头部格式保存所有信息
   - 告知用户项目信息已保存

   **初始化完成后的跳转**：

   | 触发来源 | 完成后的操作 |
   |----------|-------------|
   | 用户直接触发 0-init | 询问用户从哪个阶段开始 |
   | 从 100-rapid-idea-check 跳转 | **自动回到 100** 执行五维分析 |
   | 从 110-requirement-planning 跳转 | **自动回到 110** 执行 Phase 1 需求访谈 |

2. **欢迎与确认**
   - 介绍uw-spec流程和TDD理念
   - 确认项目意向和基本信息
   - 询问从哪个阶段开始

3. **阶段执行策略**

   **并行执行原则**：阶段间串行，阶段内并行
   - 阶段间串行：必须完成前一阶段才能进入下一阶段
   - 阶段内并行：同一阶段内的多个独立流程可同时执行

4. **阶段执行**

   **阶段1 - 需求阶段**（产品经理主导，串行）
   - 100-rapid-idea-check - 快速想法验证（读取 project-info.md）
   - 110-requirement-planning - 需求规划（读取 project-info.md）
   - 111-requirement-review - 需求评审

   **阶段2 - 设计阶段**（多角色协作，分两波执行）

   **第一波：基础设计（高度并行 ⭐⭐⭐）**

   | 并行组 | 可并行流程（组内串行） |
   |--------|-----------|
   | 数据库设计组 | 200-database-design → 201-database-design-review |
   | 后端初始化与设计组 | 210-java-uniweb-init → 210-java-uniweb-gencode → 210-java-uniweb-design → 211-java-uniweb-design-review |
   | 前端初始化与设计组 | 220-admin-web-init, 220-guest-web-init, 220-admin-uniapp-init, 220-guest-uniapp-init → 220-admin-web-gencode, 220-guest-web-gencode, 220-admin-uniapp-gencode, 220-guest-uniapp-gencode → 220-admin-web-design, 220-guest-web-design, 220-admin-uniapp-design, 220-guest-uniapp-design → 221-admin-web-design-review, 221-guest-web-design-review, 221-admin-uniapp-design-review, 221-guest-uniapp-design-review |

   **第二波：测试设计（依赖第一波产出）**

   | 串行流程 | 说明 |
   |--------|------|
   | 230-test-case-design → 231-test-case-design-review | 依赖后端设计文档（API接口定义）+ 原型产出（页面结构与操作流程） |

   **第三波：数据库执行（依赖第一波 201 评审通过）**

   | 串行流程 | 说明 |
   |--------|------|
   | 300-database-ddl-execution → 301-database-ddl-execution-review | 执行DDL建表，验证Schema，为后端开发提供数据库环境 |

   **阶段3 - 实施阶段**（项目经理主导，高度并行 ⭐⭐⭐，依赖 301 通过）

   | 并行组 | 可并行流程 |
   |--------|-----------|
   | 开发组 | 310-java-uniweb-dev, 320-admin-web-dev, 320-guest-web-dev, 320-admin-uniapp-dev, 320-guest-uniapp-dev, 330-test-case-dev |

   评审流程（串行）：311-java-uniweb-dev-review, 321-admin-web-dev-review, 321-guest-web-dev-review, 321-admin-uniapp-dev-review, 321-guest-uniapp-dev-review, 331-test-case-dev-review...

   **阶段4 - 测试阶段**（串行）
   - 410-test-case-execution → 411-test-case-execution-review

   **阶段5 - 文档交付阶段**（文档并行）

   | 并行组 | 可并行流程 |
   |--------|-----------|
   | 文档组 | 500-ops-manual, 510-requirement-doc, 520-user-manual |

   评审流程（串行）：501-ops-manual-review, 511-requirement-doc-review, 521-user-manual-review

   **阶段6 - 功能开发阶段**（按需执行）
   - 600-feature-clarify → 610-feature-tech-design
   - → 620/630/640并行开发 → 650-final-review → 660-update-doc

   **阶段7 - Bug修复阶段**（按需执行）
   - 700-bugfix-analysis → 710-bugfix-tech-design
   - → 720/730/740并行修复 → 750-final-review → 760-update-doc

## 八阶段流转规则

详见 [八阶段流转规则](references/project-templates.md#八阶段流转规则)

5. **阶段流转**
   - 每个阶段完成后确认是否进入下一阶段
   - 允许跳转到特定阶段
   - 支持返回修改之前的阶段

## 项目信息文件格式

详见 [项目信息文件格式](references/project-templates.md#项目信息文件格式)

## 项目命名规范

详见 [项目命名规范](references/project-templates.md#项目命名规范)

## TDD驱动开发原则

1. **测试先行**：先写测试，再写实现
2. **红-绿-重构**：测试失败 → 实现通过 → 重构优化
3. **验收标准**：每个需求都要有可验证的验收标准
4. **持续测试**：测试贯穿整个开发周期

## 项目结构

详见 [项目结构](references/project-templates.md#项目结构)

## 参考

- [技能清单](references/skills-reference.md) - 完整技能列表和评审流程
- [项目模板与规范](references/project-templates.md) - 项目信息格式、命名规范、项目结构、八阶段流转规则
