---
name: 410-test-case-execution
description: 测试执行技能。当需要执行自动化测试时触发：(1)执行API接口测试，(2)执行E2E界面测试（单终端+跨终端），(3)执行JMeter压力测试，(4)执行安全扫描（ZAP+Trivy），(5)管理缺陷和Bug跟踪，(6)生成四类测试报告。当用户提及测试执行、运行测试、执行E2E测试、压力测试执行、安全扫描执行时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "3.0.0"
---

# 测试执行

## 描述

执行 330-test-case-dev 阶段产出的四类自动化测试脚本（API/E2E/压测/安全），收集测试结果，管理缺陷，生成测试报告。本阶段**不编写测试脚本**，仅执行和报告。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 执行测试 | "执行测试" |
| 运行API测试 | "运行API测试" |
| 运行E2E测试 | "运行E2E测试" |
| 执行压测 | "执行压力测试" |
| 执行安全扫描 | "执行安全扫描" |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试执行和报告 | `test-engineer` |
| 协作 | 后端缺陷修复 | `java-developer` |
| 协作 | 前端缺陷修复 | `js-developer` |
| 协作 | 缺陷优先级评估 | `project-manager` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| 测试脚本 | `test/scripts/` | 330阶段产出的四类自动化脚本 |
| API服务地址 | `API_BASE_URL` 环境变量 | 后端服务运行地址（如 `http://localhost:8080`） |
| Web前端地址 | `WEB_BASE_URL` 环境变量 | 前端服务运行地址（如 `http://localhost:5173`） |
| 测试设计文档 | `test/design/` | 250阶段产出的测试设计（覆盖率基准） |

## 测试前提

| 前提 | 验证方式 |
|------|---------|
| 330阶段脚本已完成 | `test/scripts/` 目录存在且通过评审 |
| 后端服务启动 | `curl {API_BASE_URL}/health` 返回200 |
| 前端服务启动 | `curl {WEB_BASE_URL}` 返回200 |
| JMeter已安装 | `jmeter --version` 可执行 |
| ZAP已启动（安全测试） | `zap-cli status` 可连接 |

## 执行顺序

| 顺序 | 测试类型 | 原因 |
|------|---------|------|
| 1 | 安全测试 | 先扫描漏洞，确保环境安全 |
| 2 | API测试 | 验证接口正确性，为E2E提供基础 |
| 3 | E2E单终端测试 | 验证各终端独立功能 |
| 4 | E2E跨终端测试 | 验证多终端协作流程 |
| 5 | 压力测试 | 最后执行，避免影响其他测试 |

## 执行命令

### 1. 安全测试
```bash
cd test/scripts
bash bin/run-security.sh
```

### 2. API测试
```bash
cd test/scripts
bash bin/run-api.sh
```

### 3. E2E单终端测试
```bash
cd test/scripts
bash bin/run-e2e.sh {项目名}
```

### 4. E2E跨终端测试
```bash
cd test/scripts
npx playwright test e2e/cross-case/ --project=e2e
```

### 5. 压力测试
```bash
cd test/scripts
bash bin/run-load.sh {scenario}
```

### 全部执行
```bash
cd test/scripts
bash bin/run-all.sh
```

> 各类执行命令的完整参数和输出详见 [执行示例](references/examples.md)

## 缺陷管理

| 字段 | 要求 |
|------|------|
| 缺陷ID | `BUG-{类型}-{序号}` |
| 严重程度 | 致命/严重/一般/轻微 |
| 关联测试 | 对应用例ID和脚本路径 |
| 复现步骤 | 自动截图 + 错误日志 |
| 状态跟踪 | 新建→确认→修复→验证→关闭 |

## 报告输出

```
test/reports/
├── api/
│   └── report-YYMMDDHHMM.html
├── e2e/
│   └── report-YYMMDDHHMM.html
├── load/
│   └── report-YYMMDDHHMM.html
├── security/
│   ├── zap-report-YYMMDDHHMM.html
│   └── trivy-report-YYMMDDHHMM.json
└── summary/
    └── summary-YYMMDDHHMM.md
```

## PLAN-REVIEW 循环（必须执行）

测试执行完成后，必须进入 PLAN-REVIEW 循环。

#### 循环规则
| 规则 | 说明 |
|------|------|
| 通过条件 | review 评分 ≥ 95分 |
| 强制退出 | 最多5轮review（round=1~5），round ≥ 6强制退出 |
| 每轮操作 | review → 修复缺陷 → 重新测试 |

#### 循环流程
```
循环开始 (round=1)
  │
  ├─▶ 调用技能: 411-test-case-execution-review
  │    对四类测试报告执行评审
  │
  ├─▶ 获取评审报告: test/reviews/REVIEW-EXECUTION-YYMMDDHHMM.md
  │
  ├─▶ 判断结果:
  │    ├─ 评分 ≥ 95 → 输出结论，循环结束 ✓
  │    └─ 评分 < 95 → 进入修复步骤
  │
  ├─▶ 修复缺陷 → round++
  │
  └─▶ round ≥ 6 → 强制退出，输出当前结论 ⚠️
```

## 参考

- [执行示例](references/examples.md) - 四类测试执行命令、报告模板、缺陷模板
- [测试脚本开发技能](../330-test-case-dev/SKILL.md) - 上游脚本开发阶段
- [测试执行评审技能](../411-test-case-execution-review/SKILL.md) - PLAN-REVIEW循环评审
