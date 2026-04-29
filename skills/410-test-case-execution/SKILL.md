---
name: 410-test-case-execution
description: 测试执行技能。当需要执行自动化测试时触发：(1)执行API接口测试, (2)执行E2E界面测试（单终端+跨终端）, (3)执行JMeter压力测试, (4)执行安全扫描（ZAP+Trivy）, (5)管理缺陷和Bug跟踪, (6)生成四类测试报告。当用户提及测试执行、运行测试、执行E2E测试、压力测试执行、安全扫描执行时使用此技能 ⚠️【强制】完成后必须调用 411-test-case-execution-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 测试执行

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

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
| 测试脚本 | `PROJECT_ROOT/test/scripts/` | 330阶段产出的四类自动化脚本 |
| API服务地址 | `API_BASE_URL` 环境变量 | 后端服务运行地址（如 `http://localhost:8080`） |
| Web前端地址 | `WEB_BASE_URL` 环境变量 | 前端服务运行地址（如 `http://localhost:5173`） |
| 测试设计文档 | `PROJECT_ROOT/test/design/` | `230-test-case-design` 阶段产出的测试设计（覆盖率基准） |

## 测试前提

| 前提 | 验证方式 |
|------|---------|
| `330-test-case-dev` 阶段脚本已完成 | `PROJECT_ROOT/test/scripts/` 目录存在且通过评审 |
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
cd PROJECT_ROOT/test/scripts
bash bin/run-security.sh
```

### 2. API测试
```bash
cd $PROJECT_ROOT/test/scripts
bash bin/run-api.sh
```

### 3. E2E单终端测试
```bash
cd PROJECT_ROOT/test/scripts
bash bin/run-e2e.sh {项目名}
```

### 4. E2E跨终端测试
```bash
cd $PROJECT_ROOT/test/scripts
npx playwright test e2e/cross-case/ --project=e2e
```

### 5. 压力测试
```bash
cd PROJECT_ROOT/test/scripts
bash bin/run-load.sh {scenario}
```

### 全部执行
```bash
cd $PROJECT_ROOT/test/scripts
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
$PROJECT_ROOT/test/reports/
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

## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `411-test-case-execution-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

## 参考

- [执行示例](references/examples.md) - 四类测试执行命令、报告模板、缺陷模板
- [测试脚本开发技能](../330-test-case-dev/SKILL.md) - 上游脚本开发阶段
- [测试执行评审技能](../411-test-case-execution-review/SKILL.md) - REVIEW评审技能
