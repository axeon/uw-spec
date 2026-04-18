---
name: 331-test-case-dev-review
description: 测试脚本开发评审技能。当测试脚本开发完成后触发：(1)评审API测试脚本质量和断言准确性，(2)评审E2E测试脚本和Page Object规范性，(3)评审跨终端测试脚本和多BrowserContext管理，(4)评审JMeter压测脚本配置合理性，(5)评审安全扫描脚本覆盖完整性，(6)评审bin/执行脚本可用性。当用户提及测试脚本评审、Playwright评审、JMeter评审、自动化测试评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "3.0.0"
---

# 测试脚本开发评审

## 描述
对四类测试脚本（API/E2E/压测/安全）进行全面评审，验证脚本质量、覆盖完整性、可维护性。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 测试脚本开发完成后 | "评审测试脚本" |
| 检查脚本质量 | "检查测试代码质量" |
| JMeter脚本评审 | "评审JMeter脚本" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试评审 | `test-lead` |
| 协作 | 后端确认 | `java-developer` |
| 协作 | 前端确认 | `js-developer` |

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| API测试脚本 | `test/scripts/api/` | Playwright request API 脚本 |
| E2E测试脚本 | `test/scripts/e2e/` | Playwright browser 脚本 |
| JMeter脚本 | `test/scripts/load/` | .jmx 压测脚本 |
| 安全测试脚本 | `test/scripts/security/` | ZAP/Trivy/Playwright 脚本 |
| 执行脚本 | `test/scripts/bin/` | Bash 执行脚本 |
| 测试设计 | `test/design/` | 设计文档参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 测试脚本评审报告 | `test/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

## 流转关系
```
通过 → 进入 410-test-case-execution 阶段
不通过 → 返回 330-test-case-dev 修改
```

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

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 20分 |
| API脚本覆盖 | ≥90%设计用例有脚本 | 20分 |
| E2E脚本覆盖 | 核心流程100% | 20分 |
| JMeter脚本 | 4种场景齐全 | 15分 |
| 安全脚本 | OWASP Top 10覆盖 | 15分 |
| 执行脚本 | 全部可运行 | 10分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题
- Major问题 >3个

## 评审流程

### 1. 准备阶段
- 读取四类测试脚本
- 读取设计文档作为基准

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

**评审对象**: `test/scripts/`
**参与人员**: @test-lead @java-developer @js-developer
**流转方向**: 通过 → 410-test-case-execution；不通过 → 返回 330-test-case-dev

## 输出要求
**报告位置**: `test/reviews/REVIEW-CODE-YYMMDDHHMM.md`

**必须包含**:
- 评审信息（日期、人员、对象）
- 五类脚本各维度评审结果和得分
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项

## 参考
- [评审检查清单](references/checklist.md) - 五类脚本评审检查项
- [评审报告模版](../0-init/references/review-report-template.md) - 通用评审报告格式
- [测试脚本开发技能](../330-test-case-dev/SKILL.md) - 被评审的上游技能
