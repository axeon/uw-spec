---
name: 650-feature-final-review
description: 功能最终验收评审技能。当所有开发阶段完成后触发：(1)执行针对性自动化测试, (2)执行核心流程回归测试, (3)进行代码安全扫描和性能扫描, (4)生成完整验收报告, (5)人工确认上线决策。请务必在用户提及功能验收、最终评审、上线确认、功能完成时使用此技能。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# 功能最终验收评审

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 验收测试执行 | `test-engineer` |
| 协作 | 代码扫描 | `system-architect` |
| 协作 | 风险评估 | `project-manager` |
| 决策 | 上线确认 | `product-manager` ★ |

## 源技能引用

评审**必须先读取源技能文件**获取原始约定，再基于约定进行评审，禁止仅凭模型自身知识评审。

| 源技能文件 | 评审时读取的内容 |
|-----------|----------------|
| [600-feature-clarify/SKILL.md](../600-feature-clarify/SKILL.md) | 功能需求规范 |
| [610-feature-tech-design/SKILL.md](../610-feature-tech-design/SKILL.md) | 技术方案规范 |

## 输入

| 输入项 | 来源 | 说明 |
|--------|------|------|
| 功能需求 | `PROJECT_ROOT/issue/features/FEATURE-{YYMMDD}-{简述}.md` | 600阶段输出 |
| Java后端代码 | `PROJECT_ROOT/backend/{项目名}-app/src/` | 620阶段输出 |
| Web前端代码 | `PROJECT_ROOT/frontend/{项目名}-{用户角色}-{终端类型}/src/` | 630阶段输出 |
| 移动端代码 | `PROJECT_ROOT/frontend/{项目名}-{用户角色}-{终端类型}/src/` | 631阶段输出 |
| 测试脚本 | `PROJECT_ROOT/test/scripts/` | 640阶段输出 |
| 开发文档 | 各端issues/ | 各阶段开发文档 |

## 输出

| 输出项 | 位置 | 说明 |
|--------|------|------|
| 验收报告 | `PROJECT_ROOT/issue/reviews/REVIEW-FEATURE-{YYMMDDHHMM}.md` | 最终验收报告 |
| 测试报告 | `PROJECT_ROOT/test/reports/summary/final-YYMMDDHHMM.md` | 测试执行报告 |
| 扫描报告 | `PROJECT_ROOT/issue/` | 安全扫描报告 |
| 风险评估 | `PROJECT_ROOT/issue/reviews/` | 风险评估报告 |

报告格式详见 [评审报告模板](../0-init/references/review-report-template.md)。

## 执行流程

> 按上方"源技能引用"读取源技能，按"输入"读取所有评审对象。

详细的验收检查清单见 [checklist.md](references/checklist.md)。

### 1. 针对性自动化测试

**测试范围**:

| 测试类型 | 执行内容 | 通过标准 |
|---------|---------|---------|
| API测试 | 本功能相关API | 100%通过 |
| E2E测试 | 本功能相关流程 | 100%通过 |
| 单元测试 | 新增代码覆盖 | 100%通过 |

**执行命令**:
```bash
# 只执行与本功能相关的测试
npx playwright test --grep "FEATURE-{简述}"
```

### 2. 核心流程回归测试

**回归范围**:

| 流程 | 说明 |
|------|------|
| 主业务流程 | 确保核心功能未破坏 |
| 关联功能 | 与本功能相关的其他功能 |
| 权限流程 | 登录、权限校验等 |

**通过标准**: 核心流程100%通过

### 3. 代码安全扫描

**扫描维度**:

| 扫描类型 | 工具 | 检查内容 |
|---------|------|---------|
| 安全漏洞 | CodeQL/SonarQube | SQL注入、XSS等 |
| 依赖漏洞 | Trivy | 第三方依赖安全 |
| 敏感信息 | GitLeaks | 密钥、密码泄露 |
| 代码规范 | Checkstyle/ESLint | 规范符合度 |

### 4. 性能扫描

**扫描维度**:

| 扫描类型 | 检查内容 |
|---------|---------|
| SQL性能 | 慢查询、索引使用 |
| API性能 | 响应时间、并发能力 |
| 前端性能 | 资源加载、渲染性能 |

### 5. 生成验收报告

**文件位置**: `PROJECT_ROOT/issue/reviews/REVIEW-FEATURE-{YYMMDDHHMM}.md`

**报告内容**:
```markdown
# REVIEW-FEATURE-{YYMMDD}-{功能名称} - 最终验收报告

## 验收概要
- 验收日期: {YYYY-MM-DD}
- 功能名称: {功能名称}
- 验收人员: AI Assistant + 产品经理

## 测试执行结果
### 针对性测试
| 测试类型 | 用例数 | 通过 | 失败 | 通过率 |
|---------|-------|------|------|--------|
| API测试 | {total} | {pass} | {fail} | {rate} |
| E2E测试 | {total} | {pass} | {fail} | {rate} |
| 单元测试 | {total} | {pass} | {fail} | {rate} |

### 回归测试
| 流程 | 状态 | 说明 |
|------|------|------|
| {flow} | {status} | {note} |

## 代码扫描结果
### 安全扫描
| 扫描项 | 严重 | 高危 | 中危 | 低危 |
|--------|------|------|------|------|
| 安全漏洞 | {critical} | {high} | {medium} | {low} |
| 依赖漏洞 | {critical} | {high} | {medium} | {low} |

### 性能扫描
| 检查项 | 状态 | 说明 |
|--------|------|------|
| SQL性能 | {status} | {note} |
| API性能 | {status} | {note} |
| 前端性能 | {status} | {note} |

## 风险评估
| 风险项 | 等级 | 说明 | 缓解措施 |
|--------|------|------|---------|
| {risk} | {level} | {desc} | {mitigation} |

## 验收结论
- 测试通过率: {percentage}%
- 代码扫描: {status}
- 风险评估: {status}
- **最终状态: {通过/有条件通过/不通过}**

## 人工确认 ★
- [ ] 功能完整性确认
- [ ] 质量达标确认
- [ ] 上线决策确认
```

## 人工检查点 ★

**必须人工确认**:
- [ ] 功能是否完整实现需求
- [ ] 测试通过率是否达标（≥95%）
- [ ] 是否存在Critical安全漏洞
- [ ] 是否接受已知风险
- [ ] **是否允许上线**

**确认后流转**:
- 通过 → 进入 660-feature-update-doc
- 不通过 → 调用源技能修复，修复后重新评审（最多5轮）


### 6. 评审结论与修复循环

评分 ≥ 95 → **通过**，输出报告，按流转关系进入下一阶段。

评分 < 95 → **不通过**，调用 `600-feature-clarify` 修复，按 [REVIEW-FIX 循环规范](../0-init/references/review-fix-loop.md) 执行。

## 通过标准

| 检查项 | 标准 |
|--------|------|
| 测试通过率 | ≥ 95% |
| Critical漏洞 | 0个 |
| 核心流程回归 | 100%通过 |
| 人工确认 | 通过 |
