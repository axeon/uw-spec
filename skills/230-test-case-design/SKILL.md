---
name: 230-test-case-design
description: 测试用例设计技能。当需要设计测试方案时触发：(1)设计API接口测试用例, (2)设计E2E界面操作测试场景, (3)设计压力测试场景与SLA指标, (4)设计安全测试场景与检查项, (5)按角色×模块规划测试覆盖矩阵。当用户提及测试用例、测试设计、API测试、E2E测试、压测方案、安全测试时使用此技能 ⚠️【强制】完成后必须调用 231-test-case-design-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 测试用例设计

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试设计 | `test-engineer` |
| 协作 | 验收标准确认 | `product-manager` |
| 协作 | 实现细节确认 | `java-developer` |
| 协作 | 性能/安全指标确认 | `system-architect` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `PROJECT_ROOT/requirement/prds/*` | 产品需求文档，功能模块及验收标准 |
| 数据库设计文档 | `PROJECT_ROOT/database/database-design.md` | 表结构、字段定义、实体关系 |
| 后端设计文档 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 接口设计、模块划分、角色权限映射 |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}-app/` 启动后 | OpenAPI Spec（接口路径、参数类型、枚举值、必填标记） |
| 原型页面 | `{项目名}-{role}-{platform}/src/` | 页面组件结构、表单字段、操作流程 |

## 设计原则

| 原则 | 说明 |
|------|------|
| 人类可读 | 文档面向测试工程师和产品经理评审，语言清晰无歧义 |
| 可自动化指导 | 每个步骤必须包含足够的技术细节，指导后期脚本编写 |
| 独立性 | 每个用例独立可执行，通过 beforeEach 准备数据 |
| 可重复性 | 明确测试数据，多次执行结果一致 |
| 字段一致性 | 数据库字段名 = 后端参数名 = 前端表单字段名，全链路命名一致（见下方说明） |

### 字段一致性原则

本项目采用数据库驱动的全链路命名一致设计：

| 层级 | 命名规则 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `user_name`, `email`, `order_status` |
| 后端参数（JSON） | camelCase（自动映射） | `userName`, `email`, `orderStatus` |
| 前端表单字段 | 与后端参数一致 | `userName`, `email`, `orderStatus` |
| 前端表格列 | 与后端参数一致 | `userName`, `email`, `orderStatus` |
| 测试选择器 | 基于字段名或语义属性定位 | `getByLabel('用户名')`, `getByRole('button', { name: '提交' })` |

**核心结论**：由于全链路字段命名一致，E2E 测试无需额外添加 `data-testid`。330 阶段编写测试脚本时，直接使用元素自身属性（role、label、placeholder、text）定位即可。

## 四类测试设计

### 1. API 接口测试（基于后端）

| 项目 | 说明 |
|------|------|
| 设计依据 | Controller接口定义 + OpenAPI Spec |
| 测试工具 | Playwright request API |
| 测试范围 | 接口正确性、参数校验、状态码、异常处理、权限控制 |

### 2. E2E 界面操作测试（基于前端）

| 项目 | 说明 |
|------|------|
| 设计依据 | PRD验收标准 + 原型页面结构与操作流程 |
| 测试工具 | Playwright browser |
| 测试范围 | 页面导航、表单操作、列表交互、状态变更、权限控制 |
| 跨终端支持 | 多角色协作流程（如：guest下单→admin审核→mch发货） |
| 定位策略 | 基于字段一致性原则，使用 role/label/placeholder/text 定位 |

### 3. 压力测试（基于后端）

| 项目 | 说明 |
|------|------|
| 设计依据 | 后端接口设计 + 性能需求 + 用户规模估算 |
| 测试工具 | JMeter |
| 测试范围 | 基准测试、负载测试、压力测试、稳定性测试 |

> SLA指标定义和线程组配置详见 [压测SLA指标模板](references/load-metrics.md)

### 4. 安全测试（基于后端）

| 项目 | 说明 |
|------|------|
| 设计依据 | OWASP Top 10 + 接口安全需求 + 认证授权机制 |
| 测试工具 | OWASP ZAP + Trivy + Playwright request API |
| 测试范围 | SQL注入、XSS、越权访问、敏感数据泄露、CSRF、依赖漏洞 |

> 安全测试检查项详见 [安全测试检查清单](references/security-checklist.md)

## 测试类型与自动化映射

| 类型 | 自动化映射 | 目的 |
|------|-----------|------|
| API请求验证 | `request.post()` / `request.get()` | 验证接口正确性和异常处理 |
| 页面导航 | `page.goto()` | 验证路由和页面加载 |
| 表单操作 | `page.fill()` / `page.click()` | 验证CRUD表单提交 |
| 列表交互 | `page.locator()` | 验证列表查询、排序、分页 |
| 状态变更 | 断言元素状态变化 | 验证enable/disable等状态切换 |
| 权限控制 | 登录不同角色 | 验证角色可见性和操作权限 |
| 压测采样 | JMeter HTTP Request Sampler | 验证系统性能和容量 |
| 安全扫描 | ZAP Active Scan | 发现安全漏洞 |

> 所有 Playwright 选择器基于字段一致性原则，优先使用 `getByRole()` / `getByLabel()` / `getByPlaceholder()` / `getByText()`，无需额外 `data-testid`。

## 用例设计流程

### 1. 需求分析
- 读取 PRD 验收标准
- 读取 Controller 代码确认接口路径和参数
- 按 角色×模块 建立测试覆盖矩阵

### 2. 四类用例并行设计

| 测试类型 | 设计产出 | 产出路径 |
|---------|---------|---------|
| API测试 | 接口测试用例（请求参数、响应断言、数据驱动） | `PROJECT_ROOT/test/design/api/README.md` |
| E2E测试 | 页面操作用例（操作步骤、元素定位、断言规则） | `PROJECT_ROOT/test/design/e2e/README.md` |
| 压力测试 | 压测场景（线程组配置、SLA指标、参数化数据） | `PROJECT_ROOT/test/design/load/README.md` |
| 安全测试 | 安全场景（检查项、扫描策略、业务安全用例） | `PROJECT_ROOT/test/design/security/README.md` |

> 各类测试用例模板详见 [测试用例模板](references/templates.md)

### 3. 用例评审
- 覆盖度检查（角色×模块矩阵）
- 可自动化检查（每个步骤是否可映射为自动化操作）
- 优先级确认

## 用例规范

每个用例必须包含以下字段，确保人类可读且可供后期脚本开发参考：

| 字段 | 要求 | 脚本开发参考 |
|------|------|-------------|
| 用例ID | 唯一标识，如 `TC-{类型}-{模块}-{序号}` | `test.describe()` / `test()` |
| 前置条件 | 登录角色、初始数据 | `beforeEach()` 中准备 |
| 测试步骤 | 操作序列（导航→操作→验证） | Playwright / JMeter 操作 |
| 元素定位 | 目标元素描述（基于字段一致性原则，如"用户名输入框"、"提交按钮"） | 330阶段按实际代码选择最优选择器 |
| 预期结果 | 状态断言、数据变化断言 | `expect()` / Response Assertion |
| 测试数据 | 输入值、预期值 | 测试数据JSON或fixture |
| 优先级 | P0（冒烟）/ P1（核心）/ P2（扩展）/ P3（边界） | 测试分组标记 |

### 用例组织规则

| 规则 | 说明 |
|------|------|
| 按角色分组 | 每个角色（admin/saas/mch/guest）独立章节 |
| 按模块分组 | 每个模块独立小节 |
| 优先级标记 | P0 冒烟测试、P1 核心流程、P2 扩展场景、P3 边界异常 |
| 测试数据独立 | 单独列出测试数据表，不内嵌在步骤中 |
| 跨终端标注 | E2E用例涉及多终端协作时标注"涉及终端"和步骤中的"终端/角色"列 |

## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `231-test-case-design-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

## 输出要求

**产出目录**:
```
PROJECT_ROOT/test/design/
├── README.md                         # 四类测试设计总览
├── api/README.md                     # API测试用例设计
├── e2e/README.md                     # E2E测试用例设计
├── load/README.md                    # 压测场景与SLA指标设计
├── security/README.md                # 安全测试场景设计
└── reviews/                          # 评审报告（REVIEW评审产出）
    └── REVIEW-DESIGN-YYMMDDHHMM.md
```

## 参考

- [测试用例模板](references/templates.md) - API/E2E/压测/安全测试用例模板
- [设计方法](references/methods.md) - 等价类、边界值、场景法、安全测试方法
- [压测SLA指标模板](references/load-metrics.md) - 响应时间、容量、资源指标
- [安全测试检查清单](references/security-checklist.md) - OWASP Top 10 检查项
- [测试用例开发技能](../330-test-case-dev/SKILL.md) - 后期基于本文档编写自动化脚本
- [测试设计评审技能](../231-test-case-design-review/SKILL.md) - REVIEW评审技能
