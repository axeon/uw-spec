---
name: 231-test-case-design-review
description: 测试用例设计评审技能。当测试用例设计完成后触发：(1)评审API接口测试设计完整性, (2)评审E2E界面测试场景覆盖, (3)评审压力测试场景与SLA指标合理性, (4)评审安全测试场景与检查项覆盖, (5)验证角色×模块覆盖矩阵完整性, (6)输出评审结论和改进建议。当用户提及测试设计评审、测试用例评审、测试方案评审时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 测试设计评审

## 描述
对四类测试设计文档（API/E2E/压测/安全）进行全面评审，验证测试覆盖率、可自动化性、技术细节完整性。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 测试设计完成后 | "评审测试用例" |
| 评审测试质量 | "评审测试设计质量" |
| 检查测试覆盖 | "检查测试覆盖率" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 测试评审 | `test-lead` |
| 协作 | 架构可行性 | `system-architect` |
| 协作 | 后端可测试性 | `java-developer` |
| 协作 | 前端可测试性 | `js-developer` |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [230-test-case-design/SKILL.md](../230-test-case-design/SKILL.md) | **必读全文**：四类测试设计规范、模板、完成标准 |
| [230-test-case-design/references/methods.md](../230-test-case-design/references/methods.md) | 测试方法论 |
| [230-test-case-design/references/templates.md](../230-test-case-design/references/templates.md) | 测试设计模板 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| API测试设计 | `PROJECT_ROOT/test/design/api/README.md` | API接口测试用例 |
| E2E测试设计 | `PROJECT_ROOT/test/design/e2e/README.md` | E2E界面测试用例 |
| 压测设计 | `PROJECT_ROOT/test/design/load/README.md` | 压测场景与SLA指标 |
| 安全测试设计 | `PROJECT_ROOT/test/design/security/README.md` | 安全测试场景 |
| PRD | `PROJECT_ROOT/requirement/prds/*` | 需求覆盖参考 |
| 后端设计文档 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 接口设计参考 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| 测试设计评审报告 | `PROJECT_ROOT/test/reviews/REVIEW-DESIGN-YYMMDDHHMM.md` | 评审结论和问题清单 |

## 流转关系
```
通过 → 进入 330-test-case-dev 阶段
不通过 → 返回 230-test-case-design 修改
```

## 评审维度

### API 测试评审
| 维度 | 检查要点 |
|------|---------|
| 接口覆盖 | 所有Controller接口均有对应用例 |
| 参数校验 | 必填/选填、类型、长度、格式均有验证 |
| 状态码覆盖 | 200/400/401/403/404/500 均有覆盖 |
| 权限控制 | 每个角色×接口的权限矩阵 |
| 异常处理 | 空值、超长、特殊字符、并发 |
| 数据驱动 | 参数化测试数据独立列出 |

### E2E 测试评审
| 维度 | 检查要点 |
|------|---------|
| 页面覆盖 | 所有页面/路由均有导航测试 |
| 操作覆盖 | CRUD操作、列表交互、状态变更 |
| 元素定位 | 基于字段一致性原则，优先使用 role/label/placeholder/name 属性定位（无需 data-testid） |
| 跨终端流程 | 多角色协作流程有独立用例 |
| 角色覆盖 | 每个角色的可见性和操作权限 |
| 优先级 | P0冒烟、P1核心、P2扩展、P3边界 |

### 压力测试评审
| 维度 | 检查要点 |
|------|---------|
| 场景类型 | 基准/负载/压力/稳定性四种场景齐全 |
| SLA指标 | P95/P99/错误率/TPS目标值合理 |
| 线程组配置 | 线程数、ramp-up、持续时间参数合理 |
| 参数化数据 | Token提取、CSV数据文件定义清晰 |
| 响应断言 | HTTP状态码 + 业务code + 响应时间 |

### 安全测试评审
| 维度 | 检查要点 |
|------|---------|
| OWASP覆盖 | Top 10类别均有对应检查项 |
| 注入测试 | SQL注入、XSS、命令注入 |
| 越权测试 | 水平越权、垂直越权、IDOR |
| 认证授权 | Token验证、暴力破解防护、Session管理 |
| 数据安全 | 敏感数据脱敏、加密传输、日志脱敏 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 25分 |
| API测试覆盖 | ≥90%接口有对应用例 | 20分 |
| E2E测试覆盖 | 核心业务流程100%覆盖 | 20分 |
| 压测场景完整 | 4种场景齐全，SLA合理 | 15分 |
| 安全测试覆盖 | OWASP Top 10映射完整 | 20分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（如：测试无法执行、覆盖率严重不足）
- Major问题 >3个

## 评审流程

### 1. 准备阶段
- **读取源技能**：读取 [230-test-case-design/SKILL.md](../230-test-case-design/SKILL.md) 全文，提取所有测试设计规范和模板要求，作为评审的权威依据
- 读取四类测试设计文档
- 确定评审范围
- 准备检查清单

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。

**评审对象**: `PROJECT_ROOT/test/design/`
**参与人员**: @test-lead @system-architect @java-developer @js-developer
**流转方向**: 通过 → 330-test-case-dev；不通过 → 返回 230-test-case-design

## 输出要求
**报告位置**: `PROJECT_ROOT/test/reviews/REVIEW-DESIGN-YYMMDDHHMM.md`

**必须包含**:
- 评审信息（日期、人员、对象）
- 四类测试各维度评审结果和得分
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项

## 参考
- [评审检查清单](references/checklist.md) - 四类测试评审检查项
- [评审报告模版](../0-init/references/review-report-template.md) - 通用评审报告格式
- [测试用例设计技能](../230-test-case-design/SKILL.md) - 被评审的上游技能
