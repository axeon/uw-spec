---
name: 331-test-case-dev-review
description: 测试脚本开发评审技能。当测试脚本开发完成后触发：(1)评审API测试脚本质量和断言准确性, (2)评审E2E测试脚本和Page Object规范性, (3)评审跨终端测试脚本和多BrowserContext管理, (4)评审JMeter压测脚本配置合理性, (5)评审安全扫描脚本覆盖完整性, (6)评审bin/执行脚本可用性。当用户提及测试脚本评审、Playwright评审、JMeter评审、自动化测试评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 测试脚本开发评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试评审 | `test-lead` |
| 协作 | 后端确认 | `java-developer` |
| 协作 | 前端确认 | `js-developer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [330-test-case-dev/SKILL.md](../330-test-case-dev/SKILL.md) | **必读全文**：四类测试脚本开发规范、完成标准 |
| [330-test-case-dev/references/examples.md](../330-test-case-dev/references/examples.md) | 测试脚本示例 |
| [330-test-case-dev/references/jmeter-template.md](../330-test-case-dev/references/jmeter-template.md) | JMeter 模板 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| API测试脚本 | `PROJECT_ROOT/test/scripts/api/` | Playwright request API 脚本 |
| E2E测试脚本 | `PROJECT_ROOT/test/scripts/e2e/` | Playwright browser 脚本 |
| JMeter脚本 | `PROJECT_ROOT/test/scripts/load/` | .jmx 压测脚本 |
| 安全测试脚本 | `PROJECT_ROOT/test/scripts/security/` | ZAP/Trivy/Playwright 脚本 |
| 执行脚本 | `PROJECT_ROOT/test/scripts/bin/` | Bash 执行脚本 |
| 测试设计 | `PROJECT_ROOT/test/design/` | 设计文档参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 测试脚本评审报告 | `PROJECT_ROOT/test/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 评审维度

### API 测试脚本评审
| 维度 | 检查要点 |
|------|---------|
| 接口覆盖 | 设计文档中的API用例均有对应脚本 |
| 断言质量 | 状态码 + 业务code + 数据字段断言 |
| 数据驱动 | 参数化数据使用fixture或JSON文件 |
| 权限测试 | 每个角色均有认证测试 |
| 错误处理 | 异常场景（400/401/403/404/500）覆盖 |

### E2E 测试脚本评审
| 维度 | 检查要点 |
|------|---------|
| 页面覆盖 | 设计文档中的E2E用例均有对应脚本 |
| Page Object | 共享组件封装为 Page Object |
| 元素定位 | 基于字段一致性原则，优先使用 getByRole/getByPlaceholder/locator([name])（无需 data-testid） |
| 跨终端 | 多终端用例使用 MultiTerminalTest |
| 独立性 | beforeEach 准备数据，无测试间依赖 |

### JMeter 脚本评审
| 维度 | 检查要点 |
|------|---------|
| 场景覆盖 | 基准/负载/压力/稳定性均有 .jmx 文件 |
| 参数化 | 服务器地址使用 `${__P()}` |
| 数据文件 | CSV在 `load/data/` 目录，格式正确 |
| 断言 | 状态码 + 业务code + 响应时间断言 |
| 线程组 | 线程数、ramp-up、持续时间与设计一致 |

### 安全测试脚本评审
| 维度 | 检查要点 |
|------|---------|
| ZAP配置 | 扫描目标、排除路径、输出格式 |
| Trivy配置 | 扫描范围（后端/前端/镜像）、严重级别 |
| Playwright安全 | SQL注入、XSS、越权测试脚本 |
| 覆盖完整性 | OWASP Top 10 检查项覆盖 |

### bin/ 执行脚本评审
| 维度 | 检查要点 |
|------|---------|
| 脚本完整 | setup/run-api/run-e2e/run-load/run-security/run-all 齐全 |
| 参数化 | 环境变量引用正确 |
| 报告输出 | 报告路径和命名规范（YYMMDDHHMM后缀） |

## 通过标准

| 等级 | 评分 | 条件 |
|------|------|------|
| 通过 | ≥ 95 分 | 无 Critical 问题，Major ≤ 2，API 脚本覆盖 ≥ 90%，E2E 核心流程 100%，JMeter 4 场景齐全，安全 OWASP 覆盖，全部可运行 |
| 不通过 | < 95 分 | 存在 Critical 或 Major > 2 或覆盖率不达标 |

> 评分 < 95 进入修复循环，无"有条件通过"中间态。

## 评审流程

> 开始评审前，先按"源技能引用"读取源技能，按"输入"读取所有评审对象。

### 1. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模板](../0-init/references/review-report-template.md)。

**评审对象**: `PROJECT_ROOT/test/scripts/`
**参与人员**: @test-lead @java-developer @js-developer


### 2. 评审结论

计算最终评分后，按以下规则执行：

**评分 ≥ 95（通过）：**
1. 标记评审状态为「通过」
2. 输出评审报告，任务结束

**评分 < 95（不通过）→ 自动修复循环：**
1. 立即调用 `330-test-case-dev`，传入问题清单
2. 修复完成后立即重新执行本技能评审
3. 若仍 < 95，回到步骤 1（最多 3 轮）
4. 仅在通过或轮次耗尽时输出结果

> 此流程全自动执行：中间不暂停、不询问、不汇报。
> 未收到通过确认前，禁止结束本技能任务。

## 参考
- [评审检查清单](references/checklist.md) - 五类脚本评审检查项
- [评审报告模板](../0-init/references/review-report-template.md) - 通用评审报告格式
- [测试脚本开发技能](../330-test-case-dev/SKILL.md) - 被评审的上游技能
