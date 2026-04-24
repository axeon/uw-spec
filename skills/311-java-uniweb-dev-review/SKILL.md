---
name: 311-java-uniweb-dev-review
description: Java SaaS开发评审技能。当Java后端代码开发完成后触发：(1)检查TDD测试驱动开发实践, (2)评审uw-base架构规范遵循情况, (3)检查多租户数据隔离实现, (4)评审代码质量和设计模式, (5)检查安全漏洞和OWASP风险, (6)检查单元测试覆盖率和质量。请务必在用户提及Java代码评审、后端评审、TDD检查、代码质量评审、安全漏洞检查时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Java SaaS开发评审

## 描述
对Java后端代码进行全面评审，验证TDD实践、uw-base架构规范、多租户实现和代码质量。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| Java代码开发完成后 | "评审Java代码" |
| 检查代码质量 | "检查后端代码质量" |
| 评审TDD实践 | "检查单元测试覆盖" |
| 安全漏洞检查 | "检查Java安全问题" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 + 单元测试评审 | `java-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `java-developer` |

> **职责边界**：单元测试质量由 Java 开发工程师负责，评审由 java-lead / system-architect 执行。测试工程师不参与单元测试评审，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [310-java-uniweb-dev/SKILL.md](../310-java-uniweb-dev/SKILL.md) | **必读全文**：开发规范、TDD Green 规则、210→310 衔接协议 |
| [310-java-uniweb-dev/references/examples.md](../310-java-uniweb-dev/references/examples.md) | TDD 开发示例 |
| [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) | 架构约定（设计阶段定义的规范，开发阶段必须遵循） |
| [210-java-uniweb-design/references/backend/uniweb/README.md](../210-java-uniweb-design/references/backend/uniweb/README.md) | UniWeb 技术栈规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| Java后端代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 开发完成的代码 |
| 测试代码 | `PROJECT_ROOT/backend/{项目名}-app/src/PROJECT_ROOT/test/` | 单元测试和集成测试 |
| 覆盖率报告 | `PROJECT_ROOT/backend/{项目名}-app/target/site/jacoco/` | 测试覆盖率数据 |
| 架构设计 | `PROJECT_ROOT/backend/{项目名}-app/README.md` | 210阶段产出的总体设计文档 |
| 开发任务 | `PROJECT_ROOT/backend/{项目名}-app/TASKS.md` | 210阶段产出的任务卡片和联调验证清单 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| Java开发评审报告 | `PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

## 流转关系
```
通过 → 并行进入其他开发评审（321/331）
不通过 → 返回 310-java-uniweb-dev 修改
```

## 评审维度

> 每个维度的检查标准以 [310-java-uniweb-dev/SKILL.md](../310-java-uniweb-dev/SKILL.md) 和 [210-java-uniweb-design/SKILL.md](../210-java-uniweb-design/SKILL.md) 中的原始约定为准。详细检查清单见 [checklist.md](references/checklist.md)。

| 维度 | 检查要点 | 源技能章节 |
|------|---------|-----------|
| TDD实践 | 210 Red骨架→310 Green实现、覆盖率≥80%、@ExtendWith+MockedStatic、无fail()残留、无TODO残留 | 310 → 开发步骤 2.4 |
| UniWeb规范 | DaoManager/ResponseData/AuthQueryParam/FusionCache、禁用Lombok、static方法风格 | 310 → 开发步骤 2.3 + 210 → 架构约定 |
| 代码质量 | 单一职责、分层清晰、命名规范、复杂度 | 310 → 开发步骤 2.3 |
| 安全性 | @Valid校验、QueryParam参数化、@MscPermDeclare、租户隔离 | 310 → 开发步骤 2.3 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 20分 |
| 测试覆盖率 | 行≥80%，分支≥70% | 20分 |
| uw-base规范 | 100%符合 | 20分 |
| 代码质量 | 圈复杂度≤10 | 20分 |
| 安全性 | @Valid校验、@MscPermDeclare、租户隔离 | 20分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（如：安全漏洞、严重bug）
- Major问题 >3个
- 测试覆盖率 <60%

## 评审标准
| 检查项 | 通过标准 |
|--------|----------|
| 测试覆盖率 | 行覆盖≥80%，分支覆盖≥70% |
| uw-base规范 | 正确使用多租户、缓存、响应格式 |
| 代码复杂度 | 圈复杂度≤10，方法行数≤50 |
| 安全性 | 无SQL注入、XSS漏洞 |

## 评审流程

### 1. 准备阶段
- **读取源技能**：读取 [310-java-uniweb-dev/SKILL.md](../310-java-uniweb-dev/SKILL.md) 全文，提取所有开发规范和 TDD 规则，作为评审的权威依据
- 获取开发代码和测试代码
- 获取测试覆盖率报告
- 准备检查清单

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。报告中需包含"覆盖率统计"扩展统计节。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: TDD/uw-base/代码质量/安全性
**评审对象**: PROJECT_ROOT/backend/{项目名}-app/
**参与人员**: @java-lead @system-architect @java-developer
**流转方向**: 通过 -> 进入下一阶段评审；不通过 -> 返回开发修改

## 输出要求
**报告位置**: `PROJECT_ROOT/backend/{项目名}-app/reviews/REVIEW-CODE-YYMMDDHHMM.md`

**必须包含**:
- 评审信息（日期、人员、对象）
- 各维度评审结果和得分
- 覆盖率统计数据
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项
