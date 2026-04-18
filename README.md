# uw-spec 软件开发流程

一套完整的 AI 辅助软件开发流程 Skills，采用 **uw-spec（氛围编程）** + **TDD（测试驱动开发）** 理念，让 AI 成为开发伙伴，人机协作完成软件开发生命周期的各个阶段。

## 🎯 核心理念

### uw-spec（氛围编程）
- **人机协作**：AI 不是替代开发者，而是作为开发伙伴
- **轻松高效**：在轻松、高效的氛围中进行开发
- **迭代优化**：持续改进，快速迭代
- **知识共享**：AI 解释代码逻辑，开发者提供业务洞察

### TDD（测试驱动开发）
- **测试先行**：先写测试，再写实现
- **红-绿-重构**：测试失败 → 实现通过 → 重构优化
- **验收标准**：每个需求都要有可验证的验收标准
- **持续测试**：测试贯穿整个开发周期

## 🏗️ 固定技术栈

### 后端技术栈
- **基础架构**: [uw-base](https://github.com/your-org/uw-base) - 基于 Spring Boot + Spring Cloud 的企业级多子模块基础类库
  - 提供微服务架构的完整基础设施支持
  - 涵盖数据访问、缓存管理、认证授权、任务调度、日志收集等功能模块
  - 已实现 SaaS 基础技术支持，通过微服务提供 SaaS 基础设施
- **微服务框架**: Spring Boot 3.x + Spring Cloud 2023
- **服务注册与配置**: Nacos 2.3.2
- **ORM框架**: MyBatis-Plus
- **数据库**: MySQL 8.4
- **全局缓存**: Redis 8.2
- **消息队列**: RabbitMQ
- **日志系统**: Elasticsearch

### 前端 Web 技术栈
- **框架**: Vue 3 + TypeScript
- **UI组件库**: Element Plus
- **构建工具**: Vite 8
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **HTTP客户端**: Axios
- **基础功能**: 多角色登录、MFA鉴权已内置

### 前端移动端技术栈
- **框架**: UniApp + Vue 3 + TypeScript
- **跨平台支持**: H5、Android、iOS、微信小程序
- **状态管理**: Pinia
- **HTTP客户端**: uni.request / Axios

## 📁 项目结构

```
.
├── skills/                           # 技能文件目录（平铺结构）
│   ├── 0-init/                       #    主流程入口技能
│   ├── 100-rapid-idea-check/         #    快速想法验证
│   ├── 110-requirement-planning/     #    需求规划
│   ├── 111-requirement-review/       #    需求评审
│   ├── 200-database-design/          #    数据库设计（MySQL 8.4）
│   ├── 201-database-design-review/   #    数据库设计评审
│   ├── 210-java-uniweb-init/           #    Java后端项目初始化（uw-base）
│   ├── 210-java-uniweb-gencode/      #    Java代码生成
│   ├── 210-java-uniweb-design/         #    Java SaaS架构设计（uw-base）
│   ├── 211-java-uniweb-design-review/  #    架构设计评审
│   ├── 220-web-vue-init/             #    Web端项目初始化（Vue3+ElementPlus）
│   ├── 220-web-vue-gencode/          #    Web端代码生成
│   ├── 220-web-vue-design/           #    Web端设计与原型开发（Vue3+ElementPlus）
│   ├── 221-web-vue-design-review/    #    Web端设计评审
│   ├── 220-md-uniapp-init/           #    移动端项目初始化（UniApp+Vue3）
│   ├── 220-md-uniapp-gencode/        #    移动端代码生成
│   ├── 220-md-uniapp-design/         #    移动端设计与原型开发（UniApp+Vue3）
│   ├── 221-md-uniapp-design-review/  #    移动端设计评审
│   ├── 230-test-case-design/         #    测试用例设计（API/E2E/压测/安全）
│   ├── 231-test-case-design-review/  #    测试用例设计评审
│   ├── 300-database-ddl-execution/   #    数据库DDL执行与验证
│   ├── 301-database-ddl-execution-review/ # DDL执行评审
│   ├── 310-java-uniweb-dev/            #    Java SaaS开发（uw-base）
│   ├── 311-java-uniweb-dev-review/     #    Java SaaS开发评审
│   ├── 320-web-vue-dev/              #    Web前端开发（Vue3+ElementPlus）
│   ├── 321-web-vue-dev-review/       #    Web前端开发评审
│   ├── 320-md-uniapp-dev/            #    UniApp开发
│   ├── 321-md-uniapp-dev-review/     #    UniApp开发评审
│   ├── 330-test-case-dev/            #    测试脚本开发（API/E2E/压测/安全）
│   ├── 331-test-case-dev-review/     #    测试脚本开发评审
│   ├── 410-test-case-execution/      #    测试执行（API/E2E/压测/安全）
│   ├── 411-test-case-execution-review/ #  测试执行评审
│   ├── 500-devops-doc/               #    运维文档编写
│   ├── 501-ops-manual/               #    运维文档评审
│   ├── 510-requirement-doc/          #    需求文档整理
│   ├── 511-requirement-doc-review/   #    需求文档评审
│   ├── 520-user-manual/              #    用户使用手册编写
│   ├── 521-user-manual-review/       #    用户手册评审
│   ├── 600-feature-clarify/          #    功能需求澄清
│   ├── 601-feature-review/           #    功能评审
│   ├── 610-feature-tech-design/      #    功能技术方案设计
│   ├── 620-feature-java-uniweb-dev/    #    功能Java后端开发
│   ├── 630-feature-web-vue-dev/      #    功能Web前端开发
│   ├── 631-feature-md-uniapp-dev/    #    功能UniApp移动端开发
│   ├── 640-feature-test-dev/         #    功能测试脚本开发
│   ├── 650-feature-final-review/     #    功能最终验收评审
│   ├── 660-feature-update-doc/       #    功能5xx文档更新
│   ├── 700-bugfix-analysis/          #    Bug分析
│   ├── 701-bugfix-review/            #    Bug评审
│   ├── 710-bugfix-tech-design/       #    Bug修复方案设计
│   ├── 720-bugfix-java-uniweb/         #    Bug Java后端修复
│   ├── 730-bugfix-web-vue/           #    Bug Web前端修复
│   ├── 731-bugfix-md-uniapp/         #    Bug UniApp移动端修复
│   ├── 740-bugfix-test/              #    Bug回归测试
│   ├── 750-bugfix-final-review/      #    Bug修复最终验收
│   └── 760-bugfix-update-doc/        #    Bug修复5xx文档更新
│
├── agents/                           # 智能体配置目录
│   ├── product-manager.md            # 产品经理
│   ├── system-architect.md           # 系统架构师
│   ├── project-manager.md            # 项目经理
│   ├── java-developer.md             # Java后端工程师
│   ├── js-developer.md               # JS前端工程师
│   ├── test-engineer.md              # 测试工程师
│   ├── java-code-reviewer.md         # Java代码审计员
│   ├── js-code-reviewer.md           # JS代码审计员
│   ├── security-auditor.md           # 安全审计员
│   └── prototype-reviewer.md         # 原型评审员
```

## 👥 角色团队

| 角色 | 职责 | 对应智能体 | 主导阶段 |
|------|------|-----------|---------|
| **产品经理** | 需求分析、产品设计、需求文档整理 | `product-manager` | 需求规划、项目收尾 |
| **系统架构师** | 架构设计、技术选型 | `system-architect` | 设计阶段 |
| **项目经理** | 项目规划、进度管理、技术实施协调 | `project-manager` | 项目实施 |
| **Java后端工程师** | 后端服务开发（基于uw-base） | `java-developer` | 后端开发 |
| **JS前端工程师** | 前端应用开发（Vue3/UniApp） | `js-developer` | 原型开发、前端开发 |
| **测试工程师** | 测试设计、测试执行、质量保证、用户手册编写 | `test-engineer` | 测试阶段、项目收尾 |
| **Java代码审计员** | Java代码质量审计 | `java-code-reviewer` | 代码审计 |
| **JS代码审计员** | JavaScript代码质量审计 | `js-code-reviewer` | 代码审计 |
| **安全审计员** | 安全漏洞审计 | `security-auditor` | 安全审计 |
| **原型评审员** | 原型设计评审 | `prototype-reviewer` | 原型评审 |

## 🔄 软件开发流程

### 流程并行度说明

本流程采用**阶段间串行、阶段内并行**的执行策略：
- **阶段间串行**：必须完成前一阶段才能进入下一阶段
- **阶段内并行**：同一阶段内的多个独立流程可同时执行
- **最大并行度**：设计阶段和实施阶段可同时执行6+个流程

```
┌─────────────────────────────────────────────────────────────────┐
│                    AI 软件开发流程 (uw-spec)                  │
├─────────────────────────────────────────────────────────────────┤
│  0. 主流程控制  →  0-init 协调整个流程                            │
├─────────────────────────────────────────────────────────────────┤
│  1. 需求阶段 (串行)                                               │
│     100-rapid-idea-check → 110-requirement-planning               │
│                          → 111-requirement-review                 │
├─────────────────────────────────────────────────────────────────┤
│  2. 设计阶段 (分三波执行)                                         │
│  ┌─ 第一波：基础设计（高度并行 ⭐⭐⭐）─────────────────────────┐   │
│  │  ┌─ 架构设计组 ──────────────────────────────────────┐    │   │
│  │  │  200-database-design  ────────┐                   │    │   │
│  │  │  210-java-uniweb-design  ───────┼── 可并行执行        │    │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  │  ┌─ 初始化+代码生成+设计组 ──────────────────────────┐    │   │
│  │  │  210-java-uniweb-init → 210-java-uniweb-gencode     │    │   │
│  │  │  220-web-vue-init → 220-web-vue-gencode           │    │   │
│  │  │  220-md-uniapp-init → 220-md-uniapp-gencode       │    │   │
│  │  │  220-web-vue-design → 221-web-vue-design-review   │    │   │
│  │  │  220-md-uniapp-design → 221-md-uniapp-design-review│   │   │
│  │  └──────────────────────────────────────────────────┘    │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌─ 第二波：测试设计（依赖第一波）───────────────────────────┐   │
│  │  230-test-case-design → 231-test-case-design-review      │   │
│  │  依赖：后端设计文档 + 原型产出                              │   │
│  └──────────────────────────────────────────────────────────┘   │
│  ┌─ 第三波：数据库执行（依赖第一波 201 评审通过）────────────┐   │
│  │  300-database-ddl-execution → 301-database-ddl-execution-review│
│  │  执行DDL建表，验证Schema，为开发提供数据库环境               │   │
│  └──────────────────────────────────────────────────────────┘   │
├─────────────────────────────────────────────────────────────────┤
│  3. 实施阶段 (高度并行 ⭐⭐⭐，依赖 301 通过)                      │
│  ┌─ 开发组 ──────────────────────────────────────────────────┐   │
│  │  310-java-uniweb-dev  ──────────┐                           │   │
│  │  320-web-vue-dev    ──────────┼── 可并行执行                │   │
│  │  320-md-uniapp-dev  ──────────┤                           │   │
│  │  330-test-case-dev  ──────────┘                           │   │
│  └────────────────────────────────────────────────────────────┘   │
│  → 各组评审流程 (311/321/331...)                                  │
├─────────────────────────────────────────────────────────────────┤
│  4. 测试阶段 (串行)                                               │
│     410-test-case-execution → 411-test-case-execution-review      │
│     【四类测试：API + E2E + 压测 + 安全扫描】                        │
│     【部署发布】※ Git驱动自动化，不纳入AI流程管理                     │
├─────────────────────────────────────────────────────────────────┤
│  5. 文档交付阶段 (文档并行)                                        │
│  ┌─ 文档编写组 ──────────────────────────────────────────────┐   │
│  │  500-devops-doc      ─────────┐                           │   │
│  │  510-requirement-doc ─────────┼── 可并行执行                │   │
│  │  520-user-manual     ─────────┘                           │   │
│  └────────────────────────────────────────────────────────────┘   │
│  → 各文档评审流程 (501/511/521)                                    │
├─────────────────────────────────────────────────────────────────┤
│  6. 功能开发阶段 (按需执行)                                        │
│  600-feature-clarify → 610-feature-tech-design                    │
│  → 620/630/631/640并行开发 → 650-final-review → 660-update-doc    │
├─────────────────────────────────────────────────────────────────┤
│  7. Bug修复阶段 (按需执行)                                         │
│  700-bugfix-analysis → 710-bugfix-tech-design                     │
│  → 720/730/731/740并行修复 → 750-final-review → 760-update-doc    │
└─────────────────────────────────────────────────────────────────┘
```

### 并行执行策略

| 阶段 | 并行组 | 可并行流程 | 并行度 |
|------|--------|-----------|--------|
| **阶段2-第一波** | 架构设计组 | 200-database-design, 210-java-uniweb-design | 2 |
| **阶段2-第一波** | 初始化+代码生成+设计组 | 210-init+gencode, 220-init+gencode, 220-design+review | 多 |
| **阶段2-第二波** | 测试设计组 | 230-test-case-design（依赖第一波的后端设计+原型产出） | 1 |
| **阶段2-第三波** | 数据库执行组 | 300-database-ddl-execution（依赖第一波 201 评审通过） | 1 |
| **阶段3** | 开发组 | 310-java-uniweb-dev, 320-web-vue-dev, 320-md-uniapp-dev, 330-test-case-dev | 4+ |
| **阶段6** | 文档组 | 600-devops-doc, 610-requirement-doc, 620-user-manual | 3 |

### 特殊说明

**部署发布环节**：测试阶段之后的部署发布采用Git驱动的自动化执行机制（CI/CD流水线），不纳入AI流程管理范畴。该环节通过Git标签、分支合并等操作触发自动化部署流程。

## 🔢 技能编码规则

本项目的技能采用**三位数字编码规则**，具体规范如下：

| 位数 | 含义 | 说明 |
|------|------|------|
| **第一位** | 主流程编号 | 按项目阶段推进逻辑依次编排（1-6） |
| **第二位** | 子流程编号 | 同一主流程下的子流程按执行顺序编号，不同主流程下允许重复 |
| **第三位** | 流程类型 | `0` = 主流程，`1` = 审计/评审流程 |

### 编码示例

```
200-database-design           # 2=设计阶段, 0=数据库, 0=主流程
201-database-design-review    # 2=设计阶段, 0=数据库, 1=评审流程

310-java-uniweb-dev             # 3=实施阶段, 1=Java开发, 0=主流程
311-java-uniweb-dev-review      # 3=实施阶段, 1=Java开发, 1=评审流程

410-test-case-execution       # 4=测试阶段, 1=测试执行, 0=主流程
510-devops-doc                # 5=文档交付阶段, 1=运维文档, 0=主流程
```

### 主流程编号对照表

| 编号 | 阶段名称 | 说明 |
|------|----------|------|
| 0 | 主流程控制 | 项目初始化、流程协调 |
| 1 | 需求阶段 | 需求收集、分析、规划 |
| 2 | 设计阶段 | 架构设计、数据库设计、原型设计 |
| 3 | 实施阶段 | 编码开发、单元测试 |
| 4 | 测试阶段 | 测试执行、报告生成 |
| 5 | 文档交付阶段 | 运维文档、需求文档、用户手册 |
| 6 | 功能开发阶段 | 新功能开发（按需执行） |
| 7 | Bug修复阶段 | Bug修复（按需执行） |

## 📝 技能清单

### 阶段 0: 主流程控制
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 0 | 0-init | uw-spec 软件开发流程入口技能 |

### 阶段 1: 需求规划
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 100 | 100-rapid-idea-check | 快速判断产品想法是否值得落地 |
| 110 | 110-requirement-planning | 按终端类型规划需求 |
| 111 | 111-requirement-review | 需求评审 |

### 阶段 2: 设计阶段

#### 架构设计
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 200 | 200-database-design | 数据库设计（MySQL 8.4） |
| 201 | 201-database-design-review | 数据库设计评审 |
| 210 | 210-java-uniweb-design | Java SaaS架构设计（uw-base） |
| 211 | 211-java-uniweb-design-review | 架构设计评审 |
| 230 | 230-test-case-design | 测试用例设计（API/E2E/压测/安全，依赖后端设计+原型） |
| 231 | 231-test-case-design-review | 测试用例设计评审 |

#### 前端设计
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 220 | 220-web-vue-design | Web端设计与原型开发（Vue3+ElementPlus） |
| 221 | 221-web-vue-design-review | Web端设计评审 |
| 220 | 220-md-uniapp-design | 移动端设计与原型开发（UniApp+Vue3） |
| 221 | 221-md-uniapp-design-review | 移动端设计评审 |

#### 项目初始化与代码生成
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 210 | 210-java-uniweb-init | Java后端项目初始化（uw-base） |
| 210 | 210-java-uniweb-gencode | Java代码生成 |
| 220 | 220-web-vue-init | Web端项目初始化（Vue3+ElementPlus） |
| 220 | 220-web-vue-gencode | Web端代码生成 |
| 220 | 220-md-uniapp-init | 移动端项目初始化（UniApp+Vue3） |
| 220 | 220-md-uniapp-gencode | 移动端代码生成 |

### 阶段 2.5: 数据库执行阶段

| 编号 | 技能名 | 说明 |
|------|--------|------|
| 300 | 300-database-ddl-execution | 数据库DDL执行与验证 |
| 301 | 301-database-ddl-execution-review | DDL执行评审 |

**前置条件**: 阶段2-数据库设计组 201 评审通过
**产出**: 可用的数据库环境（表、索引、初始数据就绪）

### 阶段 3: 实施阶段

#### 开发
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 310 | 310-java-uniweb-dev | Java SaaS开发（uw-base） |
| 311 | 311-java-uniweb-dev-review | Java SaaS开发评审 |
| 320 | 320-web-vue-dev | Web前端开发（Vue3+ElementPlus） |
| 321 | 321-web-vue-dev-review | Web前端开发评审 |
| 320 | 320-md-uniapp-dev | UniApp开发（UniApp+Vue3） |
| 321 | 321-md-uniapp-dev-review | UniApp开发评审 |
| 330 | 330-test-case-dev | 测试脚本开发（API/E2E/压测/安全） |
| 331 | 331-test-case-dev-review | 测试脚本开发评审 |

### 阶段 4: 测试执行阶段

| 编号 | 技能名 | 说明 |
|------|--------|------|
| 410 | 410-test-case-execution | 测试执行（API/E2E/压测/安全） |
| 411 | 411-test-case-execution-review | 测试执行评审 |

**责任人**: 测试工程师
**交付物**: 测试报告（包含四类测试结果、缺陷统计、性能指标、安全扫描）
**执行内容**: 安全扫描 → API测试 → E2E单终端测试 → E2E跨终端测试 → 压力测试
**执行顺序**: 安全测试优先扫描漏洞，API测试验证接口，E2E验证界面，压测最后执行避免影响

**执行时机**:
- 版本发布前（必须执行）
- 重大功能上线前（建议执行）
- 性能优化后（验证执行）
- 定期性能巡检（按月/季度）

### 阶段 5: 文档交付阶段
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 500 | 500-devops-doc | 运维文档编写 |
| 501 | 501-ops-manual | 运维文档评审 |
| 510 | 510-requirement-doc | 需求文档整理 |
| 511 | 511-requirement-doc-review | 需求文档评审 |
| 520 | 520-user-manual | 用户使用手册编写 |
| 521 | 521-user-manual-review | 用户手册评审 |

### 阶段 6: 功能开发阶段（新功能）
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 600 | 600-feature-clarify | 功能需求澄清 |
| 601 | 601-feature-review | 功能评审 |
| 610 | 610-feature-tech-design | 功能技术方案设计 |
| 620 | 620-feature-java-uniweb-dev | 功能Java后端开发 |
| 630 | 630-feature-web-vue-dev | 功能Web前端开发 |
| 631 | 631-feature-md-uniapp-dev | 功能UniApp移动端开发 |
| 640 | 640-feature-test-dev | 功能测试脚本开发 |
| 650 | 650-feature-final-review | 功能最终验收评审 |
| 660 | 660-feature-update-doc | 功能5xx文档更新 |

### 阶段 7: Bug修复阶段
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 700 | 700-bugfix-analysis | Bug分析 |
| 701 | 701-bugfix-review | Bug评审 |
| 710 | 710-bugfix-tech-design | Bug修复方案设计 |
| 720 | 720-bugfix-java-uniweb | Bug Java后端修复 |
| 730 | 730-bugfix-web-vue | Bug Web前端修复 |
| 731 | 731-bugfix-md-uniapp | Bug UniApp移动端修复 |
| 740 | 740-bugfix-test | Bug回归测试 |
| 750 | 750-bugfix-final-review | Bug修复最终验收 |
| 760 | 760-bugfix-update-doc | Bug修复5xx文档更新 |

#### 阶段6/7评审机制

**触发条件**:
- **触发式评审**: 功能开发完成(24h内)、Bug修复完成(24h内)、热修复完成(4h内紧急)、代码合并请求(合并前)
- **定期评审**: 周度评审(每周五)、月度评审(每月最后一周)、季度评审(每季度末)

**参与人员**:
- 开发团队、架构师、项目经理、产品经理（视评审类型而定）

**评审内容**:
- 代码质量、功能完整性、性能影响、安全合规、文档更新

**整改与复核**:
- 严重问题必须修复后重新评审，一般问题限期整改，轻微问题记录备案

### 阶段 5: 文档交付阶段

#### 5.1 运维文档
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 500 | 500-devops-doc | 运维文档编写（运维工程师执行） |
| 501 | 501-ops-manual | 运维文档评审（运维负责人主导） |

**交付物**: 系统架构说明、部署手册、监控手册、故障处理手册、应急预案等

#### 5.2 需求文档
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 510 | 510-requirement-doc | 需求文档整理（产品经理执行） |
| 511 | 511-requirement-doc-review | 需求文档评审（产品经理主导） |

**交付物**: PRD、用户故事地图、业务流程图、功能清单、验收标准

#### 5.3 用户手册
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 520 | 520-user-manual | 用户使用手册编写（测试工程师执行） |
| 521 | 521-user-manual-review | 用户手册评审（测试负责人主导） |

**交付物**: 用户使用手册、快速入门指南、FAQ、故障排除指南

### 阶段 6: 功能开发阶段（新功能）

当需要开发新功能时，使用6xx系列技能：

| 编号 | 技能名 | 说明 |
|------|--------|------|
| 600 | 600-feature-clarify | 功能需求澄清（人工确认★） |
| 601 | 601-feature-review | 功能评审 |
| 610 | 610-feature-tech-design | 功能技术方案设计（含数据库，人工确认★） |
| 620 | 620-feature-java-uniweb-dev | 功能Java后端开发（AI自动） |
| 630 | 630-feature-web-vue-dev | 功能Web前端开发（AI自动） |
| 631 | 631-feature-md-uniapp-dev | 功能UniApp移动端开发（AI自动） |
| 640 | 640-feature-test-dev | 功能测试脚本开发（AI自动） |
| 650 | 650-feature-final-review | 功能最终验收（人工确认★） |
| 660 | 660-feature-update-doc | 功能5xx文档更新（AI自动） |

**流程**: 600 → 601 → 610 → (620/630/631/640并行) → 650 → 660

### 阶段 7: Bug修复阶段

当需要修复Bug时，使用7xx系列技能：

| 编号 | 技能名 | 说明 |
|------|--------|------|
| 700 | 700-bugfix-analysis | Bug分析（人工确认★） |
| 701 | 701-bugfix-review | Bug评审 |
| 710 | 710-bugfix-tech-design | Bug修复方案设计（人工确认★） |
| 720 | 720-bugfix-java-uniweb | Bug Java后端修复（AI自动） |
| 730 | 730-bugfix-web-vue | Bug Web前端修复（AI自动） |
| 731 | 731-bugfix-md-uniapp | Bug UniApp移动端修复（AI自动） |
| 740 | 740-bugfix-test | Bug回归测试（AI自动） |
| 750 | 750-bugfix-final-review | Bug修复最终验收（人工确认★） |
| 760 | 760-bugfix-update-doc | Bug修复5xx文档更新（AI自动） |

**流程**: 700 → 710 → (720/730/731/740并行) → 750 → 760

#### 收尾阶段交付物总览

| 类别 | 交付物 | 责任人 | 格式 | 存储位置 |
|------|--------|--------|------|----------|
| **运维文档** | 系统架构说明、部署手册、监控手册、故障处理手册、应急预案 | 运维工程师 | Markdown | `manual/ops-manual/` |
| **需求文档** | PRD、用户故事地图、业务流程图、功能清单、验收标准 | 产品经理 | Markdown | `requirement/` |
| **用户文档** | 用户使用手册、快速入门指南、FAQ、故障排除指南 | 测试工程师 | Markdown/PDF | `manual/user-manual/` |

#### 文档评审机制

| 文档类型 | 评审技能 | 主导角色 | 评审重点 |
|----------|----------|----------|----------|
| 运维文档 | 501-ops-manual | 运维负责人 | 可操作性、架构一致性、运维就绪 |
| 需求文档 | 511-requirement-doc-review | 产品经理 | 完整性、可追溯性、可测试性 |
| 用户手册 | 521-user-manual-review | 测试负责人 | 易懂性、可操作性、技术准确性 |

**通过标准**:
- 关键检查项100%通过
- 一般检查项通过率 ≥ 95%
- 遗留问题有明确整改计划

**评审流程**:
1. 自评检查 → 2. 交叉评审 → 3. 汇总确认 → 4. 归档入库

## 🚀 使用方式

1. **开始新项目**：调用 `0-init` 技能
2. **特定阶段**：直接调用对应阶段的技能
3. **技能触发**：AI 根据用户输入自动识别并调用合适的技能

## 📄 文档输出规范

所有设计文档统一输出到项目根目录：

```
project/
├── project-info.md           # 项目信息
├── requirement/              # 需求文档
│   ├── prds/
│   ├── interviews/
│   └── reviews/
├── backend/                  # 后端项目
│   └── {项目名}-app/
│       ├── database/
│       ├── issues/
│       ├── reviews/
│       └── src/
├── frontend/                 # 前端项目
│   └── {用户角色}-{终端类型}/
│       ├── issues/
│       ├── reviews/
│       └── src/
├── test/                     # 测试项目
│   ├── design/
│   ├── scripts/
│   ├── reports/
│   ├── issues/
│   └── reviews/
├── issue/                    # Bug修复文档
│   ├── features/
│   ├── bugs/
│   └── reviews/
└── manual/                   # 文档交付
    ├── ops-manual/
    ├── user-manual/
    └── reviews/
```

## 🎨 设计原则

### 技能设计原则
1. **单一职责**：每个技能只负责一个明确的任务
2. **可组合**：技能之间可以灵活组合
3. **可扩展**：易于添加新技能
4. **可测试**：每个技能都有明确的输入输出

### TDD 实践
1. **测试先行**：在编写实现代码前先写测试
2. **小步快跑**：小步提交，频繁验证
3. **重构优化**：测试通过后进行代码重构
4. **持续集成**：自动化测试贯穿始终

### 技术栈约束
1. **后端必须使用uw-base**：所有后端项目基于uw-base架构开发
2. **Web端必须使用Vue3+ElementPlus**：固定前端技术栈
3. **移动端必须使用UniApp+Vue3**：固定移动端技术栈
4. **数据库必须使用MySQL 8.4**：固定数据库版本
5. **配置必须使用Nacos 2.3.2**：统一配置中心

## 📚 相关文档

- [Trae Rules 文档](https://docs.trae.cn/ide/rules)
- [Trae Skills 文档](https://docs.trae.cn/ide/skills)
- [如何写好 Skill](https://docs.trae.cn/ide/best-practice-for-how-to-write-a-good-skill)

## 🤝 贡献指南

欢迎提交 Issue 和 PR 来完善这套 uw-spec 流程。

## 📄 License

MIT License
