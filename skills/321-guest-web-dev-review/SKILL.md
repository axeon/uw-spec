---
name: 321-guest-web-dev-review
description: Web前端Vue3开发评审技能。当Vue3前端代码开发完成后触发：(1)检查TDD测试驱动开发实践, (2)评审Vue3组合式API规范, (3)检查Pinia状态管理设计, (4)评审组件代码质量和复用性, (5)检查前端性能优化和懒加载, (6)检查XSS等安全漏洞防护。请务必在用户提及Vue3代码评审、前端代码检查、Web开发评审、Element Plus评审时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Web前端开发评审

## 描述
对Vue3前端代码进行全面评审，验证TDD实践、组件设计规范、状态管理、代码质量和安全性。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景
| 触发条件 | 示例 |
|---------|------|
| 前端代码开发完成后 | "评审前端代码" |
| 检查Vue规范 | "检查Vue3代码质量" |
| 评审TDD实践 | "检查前端测试覆盖" |
| 安全漏洞检查 | "检查前端安全问题" |

## 角色职责
| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 代码评审 + 单元测试评审 | `js-lead` |
| 协作 | 架构规范 | `system-architect` |
| 协作 | 代码修改 | `js-developer` |

> **职责边界**：前端单元测试质量由前端工程师负责，评审由 js-lead / system-architect 执行。测试工程师不参与单元测试评审，专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 输入
| 输入项 | 来源 | 说明 |
|--------|------|------|
| Web前端代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/` | 开发完成的代码 |
| 测试代码 | `PROJECT_ROOT/frontend/{项目名}-guest-web/src/**/*.spec.ts` | 单元测试（与源文件同目录） |
| 覆盖率报告 | `PROJECT_ROOT/frontend/{项目名}-guest-web/coverage/` | 测试覆盖率数据 |
| 前端设计 | `PROJECT_ROOT/frontend/{项目名}-guest-web/README.md` | 220设计阶段产出的设计规范 |

## 输出
| 输出项 | 位置 | 说明 |
|--------|------|------|
| Web前端开发评审报告 | `PROJECT_ROOT/frontend/{项目名}-guest-web/reviews/REVIEW-CODE-YYMMDDHHMM.md` | 评审结论和问题清单 |

## 流转关系
```
通过 → 并行进入其他开发评审（311/331）
不通过 → 返回 320-guest-web-dev 修改
```

## 评审维度
| 维度 | 检查要点 |
|------|---------|
| TDD实践 | composables/Store/工具函数有测试、覆盖率≥70%、Vitest + Vue Test Utils |
| Vue3规范 | `<script setup>`、Composition API、TypeScript类型定义、禁用any |
| 状态管理 | Store按模块划分、Actions职责单一、Pinia setup风格 |
| 代码质量 | 目录结构、命名规范、类型完整、字段一致性 |
| 性能优化 | 路由懒加载、v-for key、computed缓存 |
| 安全性 | 禁止v-html未转义、路由守卫权限控制、Token安全存储 |

## 量化通过标准

### 通过（≥95分）
| 检查项 | 标准 | 分值 |
|--------|------|------|
| 无Critical问题 | 0个 | 必须 |
| Major问题 | ≤2个 | 25分 |
| 测试覆盖率 | 核心业务≥70% | 20分 |
| Vue3规范 | 100%符合 | 20分 |
| 类型定义 | TypeScript覆盖率≥90% | 15分 |
| 性能优化 | 无性能问题 | 20分 |

### 有条件通过（85-94分）
- Major问题 ≤3个且有修复计划
- 无Critical问题

### 不通过（<85分）
- 存在Critical问题（如：安全漏洞、严重bug）
- Major问题 >3个
- 测试覆盖率 <50%

## 评审标准
| 检查项 | 通过标准 |
|--------|----------|
| 测试覆盖率 | 核心业务≥70% |
| Vue3语法 | 必须使用`<script setup>` |
| 类型定义 | 禁止any |
| Store设计 | 按模块划分，职责单一 |
| 性能 | 路由懒加载、合理使用computed |
| 安全 | 无XSS漏洞、路由权限控制 |

## 评审流程

### 1. 准备阶段
- 获取前端代码和测试代码
- 获取测试覆盖率报告
- 准备检查清单

### 2. 执行评审
按维度检查，记录问题。评审发现记录格式和评审报告结构详见 [评审报告模版](../0-init/references/review-report-template.md)。报告中需包含"覆盖率统计"扩展统计节。

详细的评审检查清单见 [checklist.md](references/checklist.md)。

**维度**: TDD/Vue3/状态管理/代码质量/性能/安全性
**评审对象**: PROJECT_ROOT/frontend/{项目名}-guest-web/
**参与人员**: @js-lead @system-architect @js-developer
**流转方向**: 通过 -> 进入下一阶段评审；不通过 -> 返回开发修改

## 输出要求
**报告位置**: `PROJECT_ROOT/frontend/{项目名}-guest-web/reviews/REVIEW-CODE-YYMMDDHHMM.md`

**必须包含**:
- 评审信息（日期、人员、对象）
- 各维度评审结果和得分
- 覆盖率统计数据
- 问题清单（含严重程度、责任人、状态）
- 量化评审结论
- 流转方向说明
- 下一步行动项
