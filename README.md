# uw-spec 开发流程规范

uw-spec 是一套完整的 AI 辅助开发流程规范（agents + Skills），采用 **harness-engineering（工程约束）** + **TDD（测试驱动开发）** 理念，让 AI 作为开发伙伴，与人机协作完成软件开发生命周期的各个阶段。

## 📦 安装与使用

### 方式一：Claude Code Plugin 安装（推荐）

Claude Code 通过 Plugin 机制同时安装 Skills 和 Agents。

#### 从 GitHub Marketplace 安装

```bash
# 第一步：添加 Marketplace
/plugin marketplace add axeon/uw-spec

# 第二步：安装插件
/plugin install uw-spec@uw-spec
```

#### 本地临时加载（适合开发调试）

直接读取源目录，修改文件后下次加载自动生效：

```bash
claude --plugin-dir /path/to/uw-spec
```

**验证安装**：

```
/skills    # 查看已安装的 Skills
/agents    # 查看已安装的 Agents
```

**作用域选择**：

| 作用域 | 安装位置 | 说明 |
|--------|----------|------|
| User（默认） | `~/.claude/` | 所有项目生效 |
| Project | `.claude/` | 仅当前项目生效 |

**更新与卸载**：

| 操作 | 命令 | 说明 |
|------|------|------|
| 更新索引 | `/plugin marketplace update uw-spec` | 拉取最新版本信息 |
| 更新插件 | `/plugin install uw-spec@uw-spec` | 覆盖安装即可 |
| 禁用插件 | `/plugin disable uw-spec` | 暂停使用，不卸载 |
| 卸载插件 | `/plugin uninstall uw-spec` | 彻底移除 |
| 查看插件 | `/plugin list` | 查看已安装的插件 |

### 方式二：npx skills 安装（仅 Skills）

`npx skills` 只能安装 Skills，**不支持安装 Agents**。如需 Agents 请使用方式一或方式三。

```bash
# 安装所有 Skills
npx skills add axeon/uw-spec --all -y

# 安装指定 Skill
npx skills add axeon/uw-spec --skill 0-init -y

# 查看已安装的 Skills
npx skills list

# 更新已安装的 Skills
npx skills update -y
```

**指定安装目标**：

```bash
# 安装到 Claude Code
npx skills add axeon/uw-spec --all -a claude-code -y

# 安装到 Trae
npx skills add axeon/uw-spec --all -a trae -y

# 安装到多个工具
npx skills add axeon/uw-spec --all -a claude-code -a trae -y

# 全局安装（跨项目生效）
npx skills add axeon/uw-spec --all -g -y
```

### 方式三：手动安装

```bash
# 克隆仓库
git clone https://github.com/axeon/uw-spec.git

# 安装 Skills（复制或 symlink 到目标目录）
# Claude Code
mkdir -p .claude/skills
ln -s $(pwd)/uw-spec/skills/* .claude/skills/

# Trae
mkdir -p .trae/skills
ln -s $(pwd)/uw-spec/skills/* .trae/skills/

# 安装 Agents（仅 Claude Code 支持）
mkdir -p .claude/agents
ln -s $(pwd)/uw-spec/agents/* .claude/agents/
```

### 三种方式对比

| 特性 | Plugin（方式一） | npx skills（方式二） | 手动（方式三） |
|------|-----------------|--------------------|-------------|
| 安装 Skills | ✅ | ✅ | ✅ |
| 安装 Agents | ✅ | ❌ | ✅ |
| 一键安装 | ✅ | ✅ | ❌ |
| 自动更新 | ✅ | ✅ | ❌ |
| 多工具支持 | Claude Code | 41+ 工具 | 任意 |

### 快速开始

安装完成后，在对话中直接说：

> "我想开发一个 XXX 应用"

`0-init` 技能会自动触发，引导你完成项目初始化并进入 uw-spec 开发流程。

## 🎯 核心理念

### harness-engineering（工程约束）

harness-engineering 是一种**以人为本的 AI 协作开发范式**，强调通过合理的工程约束来引导 AI 发挥最大效能，同时保持开发者的主导权和创造力。

#### 核心原则

| 原则 | 说明 | 实践方式 |
|------|------|----------|
| **人机协作** | AI 不是替代开发者，而是作为开发伙伴 | 开发者负责业务洞察和架构决策，AI 负责代码实现和细节处理 |
| **约束引导** | 通过预设的工程规范约束 AI 行为 | 使用 Skills 定义清晰的开发流程、代码规范和评审标准 |
| **轻松高效** | 在轻松、高效的氛围中进行开发 | 自动化重复性工作，让开发者专注于创造性任务 |
| **迭代优化** | 持续改进，快速迭代 | 短周期开发循环，快速验证和反馈 |
| **知识共享** | AI 解释代码逻辑，开发者提供业务洞察 | 代码即文档，AI 辅助生成和更新文档 |

#### 优势详解

**1. 可控性优势**
- 通过 Skills 预设开发流程，确保每次 AI 输出都符合项目规范
- 评审机制（review skills）作为质量闸门，防止低质量代码进入主干
- 开发者始终掌握最终决策权，AI 作为执行助手

**2. 效率优势**
- 并行开发模式（设计阶段、实施阶段多流程并行）大幅缩短项目周期
- 代码生成 Skills 自动化 boilerplate 代码编写，开发者专注业务逻辑
- 自动化测试和文档生成减少重复劳动

**3. 质量优势**
- 每个阶段都有对应的评审 Skill，形成多层质量防护网
- 固定的技术栈（uw-base + Vue3/UniApp）确保技术一致性
- 测试驱动开发确保代码可测试性和功能正确性

**4. 可维护性优势**
- 标准化的项目结构和编码规范降低维护成本
- 完整的文档体系（需求文档、运维文档、用户手册）保障知识传承
- 功能开发和 Bug 修复都有独立的流程 Skills，确保变更可控

---

### TDD（测试驱动开发）

TDD（Test-Driven Development）是一种**先写测试、后写实现**的开发方法论，通过测试来驱动设计，确保代码质量和功能正确性。

#### TDD 循环（红-绿-重构）

```
┌─────────┐    ┌─────────┐    ┌─────────┐
│  编写测试  │ → │  运行失败  │ → │  编写实现  │
│  (Red)   │    │  (Red)   │    │  (Green) │
└─────────┘    └─────────┘    └────┬────┘
     ↑                               │
     └───────────────────────────────┘
              ┌─────────┐
              │  重构优化  │
              │(Refactor)│
              └─────────┘
```

| 阶段 | 目标 | 产出 |
|------|------|------|
| **红（Red）** | 编写失败的测试 | 明确的验收标准和测试用例 |
| **绿（Green）** | 编写最简单的实现让测试通过 | 可运行的功能代码 |
| **重构（Refactor）** | 优化代码结构，消除重复 | 高质量的整洁代码 |

#### 核心实践

| 实践 | 说明 | 在本流程中的体现 |
|------|------|------------------|
| **测试先行** | 先写测试，再写实现 | 230-test-case-design 在开发前完成测试设计 |
| **验收标准** | 每个需求都要有可验证的验收标准 | 需求规划阶段明确测试验收点 |
| **持续测试** | 测试贯穿整个开发周期 | 开发阶段编写单元测试，测试阶段执行 API/E2E/压测/安全扫描 |
| **自动化** | 测试自动运行，快速反馈 | 测试脚本开发（330）和测试执行（410）形成自动化流水线 |

#### 优势详解

**1. 设计优势**
- **驱动良好设计**：先写测试迫使开发者思考接口设计，产出更易用的 API
- **解耦模块化**：可测试的代码必然是松耦合、高内聚的模块化设计
- **明确边界**：测试定义了组件的契约边界，便于团队协作

**2. 质量优势**
- **缺陷早发现**：在编码阶段发现问题，修复成本远低于生产环境
- **回归保护**：完整的测试套件防止新代码破坏已有功能
- **信心保障**：重构时有测试保护，开发者敢于优化代码结构

**3. 文档优势**
- **活文档**：测试代码是准确的 API 使用文档，随代码同步更新
- **示例代码**：测试用例展示了组件的各种使用场景和边界情况
- **需求验证**：测试通过即证明需求已实现，消除歧义

**4. 在本流程中的具体实践**

| 阶段 | TDD 实践 | 对应 Skills |
|------|----------|-------------|
| 设计阶段 | 测试用例设计 | 230-test-case-design |
| 实施阶段 | 单元测试开发 | 310/320-java/web-dev（内置） |
| 测试阶段 | API/E2E/压测/安全测试 | 330-test-case-dev, 410-test-case-execution |
| 功能开发 | 功能测试先行 | 640-feature-test-dev |
| Bug 修复 | 回归测试验证 | 740-bugfix-test |

## 🏗️ 固定技术栈

### 后端技术栈

#### 基础框架
| 技术 | 版本 | 说明 |
|------|------|------|
| Java | 21+ | 运行环境 |
| Spring Boot | 3.5.x | 微服务基础框架 |
| Spring Cloud | 2025.0.0 | 微服务框架 |
| Spring Cloud Alibaba | 2023.0.1.2 | Nacos 注册/配置中心 |
| Maven | 3.8+ | 构建工具 |

#### 基础设施
| 技术 | 版本 | 用途 |
|------|------|------|
| MySQL | 8.4+ | 数据存储 |
| Redis | 8.2+ | 缓存/分布式锁/序列 |
| RabbitMQ | 3.10+ | 消息队列 |
| Elasticsearch | 8.x | 日志存储与搜索 |
| Nacos | 2.3.2+ | 服务注册与配置中心 |

#### 核心类库（uw-base）
| 模块 | 用途 |
|------|------|
| uw-dao | 数据访问层（DaoManager、DataEntity、QueryParam） |
| uw-cache | 缓存管理（FusionCache） |
| uw-auth-service | 认证服务端 |
| uw-auth-client | 认证客户端（Token自动管理） |
| uw-mfa | 多因素认证（TOTP） |
| uw-task | 分布式任务框架（@TaskScheduler、@QueueWorker） |
| uw-httpclient | HTTP客户端（连接池管理） |
| uw-log-es | Elasticsearch 日志客户端 |
| uw-logback-es | Logback ES Appender |
| uw-ai | AI集成模块（Spring AI、Ollama、RAG） |
| uw-oauth2-client | OAuth2客户端 |
| uw-gateway-client | 网关客户端 |
| uw-mydb-client | 数据库客户端（分库分表） |
| uw-notify-client | 通知客户端（SSE推送） |
| uw-tinyurl-client | 短链接客户端 |
| uw-webot | Web自动化框架 |
| uw-common | 通用工具类库（ResponseData、JsonUtils、AESUtils、SnowflakeIdGenerator） |
| uw-common-app | Web应用公共类库 |

#### 微服务平台
| 微服务 | 功能说明 |
|--------|----------|
| uw-gateway | API网关（SSL、ACL、限流、负载均衡、HTTP/2） |
| uw-gateway-center | 网关管理中心（灰度发布、监控、SSL证书） |
| uw-auth-center | 统一鉴权中心（Token分发、API权限、MFA认证） |
| uw-task-center | 任务管理中心 |
| uw-ops-center | 运维管理中心（Docker全自动部署） |
| uw-code-center | 代码生成服务 |
| uw-mydb-center | 数据库运维中心 |
| uw-mydb-proxy | MySQL分库分表代理（基于Netty） |
| uw-tinyurl-center | 短链接服务 |
| uw-notify-center | 实时通知推送中心（SSE） |
| uw-ai-center | AI服务中心（向量数据库、RAG） |

---

### SaaS 技术栈

SaaS 技术栈基于 UniWeb 框架构建的多租户 SaaS 架构体系，提供租户管理、产品授权计费、支付结算等 SaaS 核心能力。

#### SaaS 微服务
| 微服务 | 功能说明 |
|--------|----------|
| saas-base | SaaS 平台核心基础设施服务，负责租户管理、商户管理、产品授权计费（AIP）、应用接口服务（AIS）等 |
| saas-finance | SaaS 平台财务核心服务，负责支付通道管理、余额账户管理、对账管理、汇率管理等 |

#### SaaS 核心类库
| 类库 | Maven坐标 | 功能说明 |
|------|-----------|----------|
| saas-base-common | `saas:saas-base-common` | SaaS 基础公共模块，提供租户管理、商户管理、消息通知、对象存储等基础能力 |
| saas-finance-client | `saas:saas-finance-client` | SaaS 财务客户端，提供支付通道、余额管理、对账管理等财务功能 |

#### SaaS 核心模块
| 模块 | 核心概念 | 功能说明 |
|------|----------|----------|
| **AIP** (Application Infrastructure Provider) | Vendor, Product, Order, License, Balance | 应用基础设施与产品授权计费，支持 License/App/AppLicense/Task 四种计费模式 |
| **AIS** (Application Interface Service) | LinkerType, Linker, LinkerConfig | 应用接口服务框架，通过 Linker 机制实现不同服务提供商的统一接入（邮件/短信/支付） |

---

### 前端 Web 技术栈
| 技术 | 版本 | 用途 |
|------|------|------|
| Vue | 3.x | 前端框架 |
| TypeScript | - | 类型安全 |
| Element Plus | - | UI组件库 |
| Vite | 8.x | 构建工具 |
| Pinia | - | 状态管理 |
| Vue Router | 4.x | 路由管理 |
| Axios | - | HTTP客户端 |

**内置功能**: 多角色登录、MFA鉴权

---

### 前端移动端技术栈
| 技术 | 版本 | 用途 |
|------|------|------|
| UniApp | - | 跨平台框架 |
| Vue | 3.x | 前端框架 |
| TypeScript | - | 类型安全 |
| Pinia | - | 状态管理 |
| uni-ui | - | UI组件库 |

**支持平台**: H5、Android、iOS、微信小程序

**HTTP客户端**: uni.request / Axios

---

### 测试工具链
| 类型 | 工具 | 用途 |
|------|------|------|
| 单元测试 | JUnit 5 / Vitest | Java / TypeScript 单元测试 |
| 集成测试 | Spring Boot Test | 后端集成测试 |
| API测试 | Playwright (request API) | 接口自动化测试 |
| E2E测试 | Playwright (browser) | 端到端界面测试 |
| 跨终端E2E | Playwright (Multi-BrowserContext) | 多终端协作流程测试 |
| 性能测试 | JMeter | 压力/负载/稳定性测试 |
| 安全扫描 | OWASP ZAP / Trivy | Web漏洞/依赖漏洞扫描 |
| 覆盖率 | JaCoCo / Vitest Coverage | 代码覆盖率统计 |

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
│  │  ┌─ 初始化+代码生成+设计组 ───────────────────────┐    │   │
│  │  │  210-java-uniweb-init → 210-java-uniweb-gencode  │    │   │
│  │  │  220-admin/guest-web-init → admin/guest-gencode  │    │   │
│  │  │  220-admin/guest-uniapp-init → admin/guest-gencode│    │   │
│  │  │  220-admin/guest-web-design → admin/guest-review │    │   │
│  │  │  220-admin/guest-uniapp-design → admin/guest-review│    │   │
│  │  └─────────────────────────────────────────────────┘    │   │
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
│  ┌─ 开发组 ────────────────────────────────────────────┐   │
│  │  310-java-uniweb-dev       ─────────┐               │   │
│  │  320-admin/guest-web-dev   ─────────┼── 可并行执行  │   │
│  │  320-admin/guest-uniapp-dev ────────┤               │   │
│  │  330-test-case-dev         ─────────┘               │   │
│  └────────────────────────────────────────────────────┘   │
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
│  → 620/630/640并行开发 → 650-final-review → 660-update-doc    │
├─────────────────────────────────────────────────────────────────┤
│  7. Bug修复阶段 (按需执行)                                         │
│  700-bugfix-analysis → 710-bugfix-tech-design                     │
│  → 720/730/740并行修复 → 750-final-review → 760-update-doc    │
└─────────────────────────────────────────────────────────────────┘
```

### 并行执行策略

| 阶段 | 并行组 | 可并行流程 | 并行度 |
|------|--------|-----------|--------|
| **阶段2-第一波** | 架构设计组 | 200-database-design, 210-java-uniweb-design | 2 |
| **阶段2-第一波** | 初始化+代码生成+设计组 | 210-init+gencode, 220-init+gencode, 220-design+review | 多 |
| **阶段2-第二波** | 测试设计组 | 230-test-case-design（依赖第一波的后端设计+原型产出） | 1 |
| **阶段2-第三波** | 数据库执行组 | 300-database-ddl-execution（依赖第一波 201 评审通过） | 1 |
| **阶段3** | 开发组 | 310-java-uniweb-dev, 320-admin/guest-web-dev, 320-admin/guest-uniapp-dev, 330-test-case-dev | 4+ |
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
| 220 | 220-admin-web-design | Admin Web端设计与原型开发（Vue3+ElementPlus） |
| 220 | 220-guest-web-design | Guest Web端设计与原型开发（Vue3+ElementPlus） |
| 221 | 221-admin-web-design-review | Admin Web端设计评审 |
| 221 | 221-guest-web-design-review | Guest Web端设计评审 |
| 220 | 220-admin-uniapp-design | Admin UniApp设计与原型开发（UniApp+Vue3） |
| 220 | 220-guest-uniapp-design | Guest UniApp设计与原型开发（UniApp+Vue3） |
| 221 | 221-admin-uniapp-design-review | Admin UniApp设计评审 |
| 221 | 221-guest-uniapp-design-review | Guest UniApp设计评审 |

#### 项目初始化与代码生成
| 编号 | 技能名 | 说明 |
|------|--------|------|
| 210 | 210-java-uniweb-init | Java后端项目初始化（uw-base） |
| 210 | 210-java-uniweb-gencode | Java代码生成 |
| 220 | 220-admin-web-init | Admin Web端项目初始化（Vue3+ElementPlus） |
| 220 | 220-guest-web-init | Guest Web端项目初始化（Vue3+ElementPlus） |
| 220 | 220-admin-web-gencode | Admin Web端代码生成 |
| 220 | 220-guest-web-gencode | Guest Web端代码生成 |
| 220 | 220-admin-uniapp-init | Admin UniApp项目初始化（UniApp+Vue3） |
| 220 | 220-guest-uniapp-init | Guest UniApp项目初始化（UniApp+Vue3） |
| 220 | 220-admin-uniapp-gencode | Admin UniApp代码生成 |
| 220 | 220-guest-uniapp-gencode | Guest UniApp代码生成 |

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
| 320 | 320-admin-web-dev | Admin Web前端开发（Vue3+ElementPlus） |
| 320 | 320-guest-web-dev | Guest Web前端开发（Vue3+ElementPlus） |
| 321 | 321-admin-web-dev-review | Admin Web前端开发评审 |
| 321 | 321-guest-web-dev-review | Guest Web前端开发评审 |
| 320 | 320-admin-uniapp-dev | Admin UniApp开发（UniApp+Vue3） |
| 320 | 320-guest-uniapp-dev | Guest UniApp开发（UniApp+Vue3） |
| 321 | 321-admin-uniapp-dev-review | Admin UniApp开发评审 |
| 321 | 321-guest-uniapp-dev-review | Guest UniApp开发评审 |
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
| 630 | 630-feature-admin-web-dev | 功能Admin Web前端开发 |
| 630 | 630-feature-guest-web-dev | 功能Guest Web前端开发 |
| 630 | 630-feature-admin-uniapp-dev | 功能Admin UniApp移动端开发 |
| 630 | 630-feature-guest-uniapp-dev | 功能Guest UniApp移动端开发 |
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
| 730 | 730-bugfix-admin-web | Bug Admin Web前端修复 |
| 730 | 730-bugfix-guest-web | Bug Guest Web前端修复 |
| 730 | 730-bugfix-admin-uniapp | Bug Admin UniApp移动端修复 |
| 730 | 730-bugfix-guest-uniapp | Bug Guest UniApp移动端修复 |
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
| 630 | 630-feature-admin-web-dev | 功能Admin Web前端开发（AI自动） |
| 630 | 630-feature-guest-web-dev | 功能Guest Web前端开发（AI自动） |
| 630 | 630-feature-admin-uniapp-dev | 功能Admin UniApp移动端开发（AI自动） |
| 630 | 630-feature-guest-uniapp-dev | 功能Guest UniApp移动端开发（AI自动） |
| 640 | 640-feature-test-dev | 功能测试脚本开发（AI自动） |
| 650 | 650-feature-final-review | 功能最终验收（人工确认★） |
| 660 | 660-feature-update-doc | 功能5xx文档更新（AI自动） |

**流程**: 600 → 601 → 610 → (620/630/640并行) → 650 → 660

### 阶段 7: Bug修复阶段

当需要修复Bug时，使用7xx系列技能：

| 编号 | 技能名 | 说明 |
|------|--------|------|
| 700 | 700-bugfix-analysis | Bug分析（人工确认★） |
| 701 | 701-bugfix-review | Bug评审 |
| 710 | 710-bugfix-tech-design | Bug修复方案设计（人工确认★） |
| 720 | 720-bugfix-java-uniweb | Bug Java后端修复（AI自动） |
| 730 | 730-bugfix-admin-web | Bug Admin Web前端修复（AI自动） |
| 730 | 730-bugfix-guest-web | Bug Guest Web前端修复（AI自动） |
| 730 | 730-bugfix-admin-uniapp | Bug Admin UniApp移动端修复（AI自动） |
| 730 | 730-bugfix-guest-uniapp | Bug Guest UniApp移动端修复（AI自动） |
| 740 | 740-bugfix-test | Bug回归测试（AI自动） |
| 750 | 750-bugfix-final-review | Bug修复最终验收（人工确认★） |
| 760 | 760-bugfix-update-doc | Bug修复5xx文档更新（AI自动） |

**流程**: 700 → 710 → (720/730/740并行) → 750 → 760

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

## 🤝 贡献指南

欢迎提交 Issue 和 PR 来完善这套 uw-spec 流程。

## 📄 License

MIT License
