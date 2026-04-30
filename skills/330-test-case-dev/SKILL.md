---
name: 330-test-case-dev
description: 测试脚本开发技能。当需要开发自动化测试脚本时触发：(1)编写API接口测试脚本（Playwright request）, (2)编写E2E界面测试脚本（Playwright browser）, (3)编写跨终端协作测试脚本（多BrowserContext）, (4)编写JMeter压测脚本（.jmx）, (5)编写安全测试脚本（ZAP+Trivy+Playwright）, (6)编写执行bash脚本（bin/）, (7)指定类型开发或并行开发全部类型。当用户提及测试脚本开发、Playwright脚本、JMeter脚本、自动化测试开发、指定类型、并行开发时使用此技能 ⚠️【强制】完成后必须调用 331-test-case-dev-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 测试脚本开发

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试脚本开发 | `test-engineer` |
| 协作 | 接口确认 | `java-developer` |
| 协作 | 前端元素确认 | `js-developer` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| API测试设计 | `PROJECT_ROOT/test/design/api/README.md` | API接口测试用例 |
| E2E测试设计 | `PROJECT_ROOT/test/design/e2e/README.md` | E2E界面测试用例 |
| 压测设计 | `PROJECT_ROOT/test/design/load/README.md` | 压测场景与SLA指标 |
| 安全测试设计 | `PROJECT_ROOT/test/design/security/README.md` | 安全测试场景 |
| Controller代码 | `PROJECT_ROOT/backend/{项目名}/src/main/java/{包路径}/controller/` | 接口路径、请求/响应格式 |
| 前端代码 | `{项目名}-{role}-{platform}/src/` | 页面组件、表单字段 |

## 技术栈

| 测试类型 | 工具 | 脚本格式 | 产出目录 |
|---------|------|---------|---------|
| API测试 | Playwright request API | `.spec.ts` | `PROJECT_ROOT/test/scripts/api/` |
| E2E测试 | Playwright browser | `.spec.ts` | `PROJECT_ROOT/test/scripts/e2e/{项目名}/` |
| 跨终端E2E | Playwright multi-context | `.spec.ts` | `PROJECT_ROOT/test/scripts/e2e/cross-case/` |
| 压力测试 | JMeter | `.jmx` | `PROJECT_ROOT/test/scripts/load/` |
| 安全测试 | ZAP + Trivy + Playwright | `.sh` / `.spec.ts` | `PROJECT_ROOT/test/scripts/security/` |
| 执行脚本 | Bash | `.sh` | `PROJECT_ROOT/test/scripts/bin/` |

## 目录结构

```
$PROJECT_ROOT/test/scripts/
├── playwright.config.ts          # Playwright配置（api + e2e 双项目）
├── package.json                  # 依赖管理
├── README.md                     # 脚本使用说明
├── api/                          # API测试脚本
│   └── {module}.spec.ts
├── e2e/                          # E2E测试脚本
│   ├── {项目名}/                 # 单终端测试（如 guest-uniapp/、admin-web/）
│   │   └── {module}.spec.ts
│   ├── cross-case/               # 跨终端协作测试
│   │   └── {flow}.spec.ts
│   └── shared/                   # 共享Page Objects和工具
│       ├── page-objects/
│       └── helpers/
├── load/                         # JMeter压测脚本
│   ├── {scenario}.jmx
│   └── data/                     # CSV测试数据
│       └── {data}.csv
├── security/                     # 安全测试脚本
│   ├── zap-scan.sh
│   ├── trivy-scan.sh
│   └── auth-bypass.spec.ts
├── utils/                        # 共享工具
│   ├── test-data.ts              # 测试数据生成
│   ├── auth.ts                   # 登录/Token管理
│   └── multi-context.ts          # 多终端BrowserContext管理
├── fixtures/                     # 测试固件
│   └── test-fixtures.ts
├── data/                         # 测试数据文件
│   └── {module}.json
└── bin/                          # 执行脚本
    ├── setup.sh
    ├── run-api.sh
    ├── run-e2e.sh
    ├── run-load.sh
    ├── run-security.sh
    └── run-all.sh
```

## 开发模式

| 模式 | 触发方式 | 说明 |
|------|---------|------|
| 单类型开发 | "只开发API测试" 或 "只开发E2E测试" | 指定一种测试类型开发 |
| 并行全类型 | "并行开发全部测试"（**默认**） | API/E2E/压测/安全四类天然可并行 |
| 顺序全量 | "开发全部测试脚本" | 按类型逐个开发 |

## 230→330 衔接协议

230 设计完成后，330 从 PROJECT_ROOT/test/design/ 提取以下信息：

| 提取项 | 来源文件 | 用途 |
|--------|---------|------|
| API测试用例 | `PROJECT_ROOT/test/design/api/README.md` | API测试脚本输入 |
| E2E测试用例 | `PROJECT_ROOT/test/design/e2e/README.md` | E2E测试脚本输入 |
| 压测场景 | `PROJECT_ROOT/test/design/load/README.md` | JMeter脚本输入 |
| 安全测试场景 | `PROJECT_ROOT/test/design/security/README.md` | 安全扫描脚本输入 |

## 并行开发约束

| 约束 | 说明 |
|------|------|
| 四类测试天然独立 | API/E2E/压测/安全脚本在不同目录，无冲突 |
| 共享工具串行 | utils/ 和 fixtures/ 修改需串行 |
| E2E依赖前端 | E2E测试需前端页面可访问 |
| API依赖后端 | API测试需后端接口可访问 |
| review按类型 | 每种测试类型完成后调用 `331-test-case-dev-review`，通过后才继续下一类型 |

## 开发流程

### 1. 前提检查
- 读取 `PROJECT_ROOT/test/design/` 下四类测试设计文档
- 确认API接口可正常访问
- 读取 Controller 代码确认接口路径和参数

### 2. 环境准备

```bash
cd PROJECT_ROOT/test/scripts
npm init -y
npm install @playwright/test @faker-js/faker
npx playwright install
```

### 3. Playwright 配置（API + E2E 双项目）

```typescript
export default defineConfig({
  projects: [
    {
      name: 'api',
      testDir: './api',
      use: { baseURL: process.env.API_BASE_URL },
    },
    {
      name: 'e2e',
      testDir: './e2e',
      use: {
        baseURL: process.env.WEB_BASE_URL,
        ...devices['Desktop Chrome'],
      },
    },
  ],
});
```

### 4. 四类脚本并行开发

#### E2E 选择器策略（字段一致性原则）

本项目采用数据库驱动的全链路命名一致设计（数据库字段 = 后端参数 = 前端字段名），因此 E2E 测试**不需要额外添加 `data-testid`**。选择器优先级：

| 优先级 | 策略 | 示例 | 适用场景 |
|--------|------|------|---------|
| 1 | `getByRole()` | `page.getByRole('button', { name: '提交' })` | 按钮、链接、复选框 |
| 2 | `getByPlaceholder()` | `page.getByPlaceholder('请输入用户名')` | 带占位符的输入框 |
| 3 | `locator()` | `page.locator('[name="userName"]')` | 表单字段（name属性与数据库字段一致） |
| 4 | `getByText()` | `page.getByText('确认删除')` | 纯文本内容、弹窗确认 |
| 5 | `locator()` | `page.locator('.class-name')` | 以上均不适用的兜底方案 |

**核心策略**：使用 `name` 属性定位表单元素，name值与数据库字段名（camelCase）完全一致：

```typescript
// 数据库字段: user_name
// 前端表单: <input name="userName" />
// E2E选择器: page.locator('[name="userName"]')

// 填写表单
await page.locator('[name="userName"]').fill('testuser')
await page.locator('[name="email"]').fill('test@example.com')
await page.locator('[name="phone"]').fill('13800138000')
```

| 测试类型 | 脚本规范 | 关键模式 |
|---------|---------|---------|
| API | `api/{module}.spec.ts` | `request.post/get()` + 响应断言 |
| E2E单终端 | `e2e/{项目名}/{module}.spec.ts` | Page Object + 字段一致性选择器 |
| E2E跨终端 | `e2e/cross-case/{flow}.spec.ts` | MultiTerminalTest 多BrowserContext |
| JMeter | `load/{scenario}.jmx` | 线程组 + HTTP采样器 + CSV数据 |
| 安全 | `security/*.sh` + `security/*.spec.ts` | ZAP扫描 + Trivy扫描 + Playwright注入 |

> 各类脚本完整示例见 [脚本示例](references/examples.md)

### 5. JMeter 脚本开发

> JMeter .jmx 完整模板见 [JMeter脚本模板](references/jmeter-template.md)

**脚本规范**:
| 规范 | 要求 |
|------|------|
| 文件命名 | `{module}-{scenario-type}.jmx` |
| 参数化 | 服务器地址、端口使用 `${__P()}` 属性 |
| 数据文件 | CSV放在 `load/data/` 目录 |
| 断言 | HTTP状态码 + 业务code + 响应时间 |

### 6. 安全测试脚本开发

> 安全扫描配置和脚本详见 [安全扫描脚本](references/security-scan.md)

**脚本规范**:
| 脚本 | 用途 |
|------|------|
| `zap-scan.sh` | ZAP Active/Passive 扫描 |
| `trivy-scan.sh` | 依赖和镜像漏洞扫描 |
| `auth-bypass.spec.ts` | Playwright 认证绕过测试 |

### 7. 执行脚本开发

**bin/ 目录脚本**:
| 脚本 | 功能 |
|------|------|
| `setup.sh` | 安装依赖、初始化环境 |
| `run-api.sh` | 执行API测试 |
| `run-e2e.sh` | 执行E2E测试（支持指定项目名） |
| `run-load.sh` | 执行JMeter压测 |
| `run-security.sh` | 执行安全扫描 |
| `run-all.sh` | 按顺序执行全部测试 |

## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `331-test-case-dev-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

## 参考

- [脚本示例](references/examples.md) - API/E2E/跨终端 Playwright 脚本示例
- [JMeter脚本模板](references/jmeter-template.md) - JMeter .jmx 完整模板
- [安全扫描脚本](references/security-scan.md) - ZAP/Trivy/Playwright安全脚本
- [测试用例设计技能](../230-test-case-design/SKILL.md) - 上游设计阶段
- [测试脚本评审技能](../331-test-case-dev-review/SKILL.md) - REVIEW评审技能
