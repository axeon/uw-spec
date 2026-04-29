# uw-spec 技能清单

## 流程并行度说明

本流程采用**阶段间串行、阶段内并行**的执行策略：

| 阶段 | 编号 | 执行方式 | 并行组 | 可并行流程数 |
|------|------|---------|--------|-------------|
| 阶段0 | 0-init | 串行 | - | 1 |
| 阶段1 | 1xx | 串行 | - | 1 |
| 阶段2 | 2xx | **高度并行** ⭐⭐⭐ | 数据库设计组、后端初始化与设计组、前端初始化与设计组、测试设计组 | 4组并行 |
| 阶段2.5 | 3xx（300/301） | 串行 | 数据库执行组 | 1 |
| 阶段3 | 3xx（310+） | **高度并行** ⭐⭐⭐ | 开发组 | 4+ |
| 阶段4 | 4xx | 串行 | - | 1 |
| 阶段5 | 5xx | 文档并行 | 文档组 | 3 |
| 阶段6 | 6xx | 按需执行 | 开发组 | 4+ |
| 阶段7 | 7xx | 按需执行 | 修复组 | 4+ |

**并行执行原则**：
- 阶段间串行：必须完成前一阶段才能进入下一阶段
- 阶段内并行：同一阶段内的多个独立流程可同时执行
- 评审流程串行：每个主流程完成后进行评审

## 阶段 0: 主流程控制

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 0 | 0-init | uw-spec 软件开发流程入口 | 全角色 |

## 阶段 1: 需求阶段

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 100 | 100-rapid-idea-check | 快速想法验证 | 产品经理 |
| 110 | 110-requirement-planning | 需求规划（PRD、流程图、线框图） | 产品经理 |
| 111 | 111-requirement-review | 需求评审 | 项目经理 |

## 阶段 2: 设计阶段（高度并行 ⭐⭐⭐）

### 数据库设计组
| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 200 | 200-database-design | 数据库设计 | 系统架构师 |
| 201 | 201-database-design-review | 数据库设计评审 | 系统架构师 |

### 后端初始化与设计组
| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 210 | 210-java-uniweb-init | Java后端项目初始化 | Java后端工程师 |
| 210 | 210-java-uniweb-gencode | Java代码生成 | Java后端工程师 |
| 210 | 210-java-uniweb-design | Java SaaS架构设计 | 系统架构师 |
| 211 | 211-java-uniweb-design-review | 架构设计评审 | 系统架构师 |

### 前端初始化与设计组

> **角色约束**：`admin-*` 技能用于 root/ops/saas/mch 角色，`guest-*` 技能用于 guest 角色。详见 [角色-平台矩阵](role-platform-matrix.md)

| 编号 | 技能名 | 说明 | 适用角色 | 主导角色 |
|------|--------|------|---------|---------|
| 220 | 220-admin-web-init | Admin Web端项目初始化 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-web-init | Guest Web端项目初始化 | guest | JS前端工程师 |
| 220 | 220-admin-uniapp-init | Admin UniApp项目初始化 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-uniapp-init | Guest UniApp项目初始化 | guest | JS前端工程师 |
| 220 | 220-admin-web-gencode | Admin Web端代码生成 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-web-gencode | Guest Web端代码生成 | guest | JS前端工程师 |
| 220 | 220-admin-uniapp-gencode | Admin UniApp代码生成 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-uniapp-gencode | Guest UniApp代码生成 | guest | JS前端工程师 |
| 220 | 220-admin-web-design | Admin Web端设计与原型开发 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-web-design | Guest Web端设计与原型开发 | guest | JS前端工程师 |
| 220 | 220-admin-uniapp-design | Admin UniApp设计与原型开发 | root/ops/saas/mch | JS前端工程师 |
| 220 | 220-guest-uniapp-design | Guest UniApp设计与原型开发 | guest | JS前端工程师 |
| 221 | 221-admin-web-design-review | Admin Web端设计与原型评审 | root/ops/saas/mch | 原型评审员 |
| 221 | 221-guest-web-design-review | Guest Web端设计与原型评审 | guest | 原型评审员 |
| 221 | 221-admin-uniapp-design-review | Admin UniApp设计与原型评审 | root/ops/saas/mch | 原型评审员 |
| 221 | 221-guest-uniapp-design-review | Guest UniApp设计与原型评审 | guest | 原型评审员 |

### 测试设计组
| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 230 | 230-test-case-design | 测试用例设计（API/E2E/压测/安全） | 测试工程师 |
| 231 | 231-test-case-design-review | 测试用例设计评审 | 测试工程师 |

## 阶段 2.5: 数据库执行阶段

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 300 | 300-database-ddl-execution | 数据库DDL执行与验证 | Java后端工程师 |
| 301 | 301-database-ddl-execution-review | DDL执行评审 | 系统架构师 |

**前置条件**: 阶段2-数据库设计组 201 评审通过
**产出**: 可用的数据库环境（表、索引、初始数据就绪）
**下游依赖**: 310-java-uniweb-dev 等开发阶段依赖 301 通过

## 阶段 3: 实施阶段（高度并行 ⭐⭐⭐）

### 开发组（可并行）
| 编号 | 技能名 | 说明 | 适用角色 | 主导角色 |
|------|--------|------|---------|---------|
| 310 | 310-java-uniweb-dev | Java SaaS开发 | - | Java后端工程师 |
| 311 | 311-java-uniweb-dev-review | Java SaaS开发评审 | - | Java代码审计员 |
| 320 | 320-admin-web-dev | Admin Web前端开发 | root/ops/saas/mch | JS前端工程师 |
| 320 | 320-guest-web-dev | Guest Web前端开发 | guest | JS前端工程师 |
| 320 | 320-admin-uniapp-dev | Admin UniApp开发 | root/ops/saas/mch | JS前端工程师 |
| 320 | 320-guest-uniapp-dev | Guest UniApp开发 | guest | JS前端工程师 |
| 321 | 321-admin-web-dev-review | Admin Web前端开发评审 | root/ops/saas/mch | JS代码审计员 |
| 321 | 321-guest-web-dev-review | Guest Web前端开发评审 | guest | JS代码审计员 |
| 321 | 321-admin-uniapp-dev-review | Admin UniApp开发评审 | root/ops/saas/mch | JS代码审计员 |
| 321 | 321-guest-uniapp-dev-review | Guest UniApp开发评审 | guest | JS代码审计员 |
| 330 | 330-test-case-dev | 测试脚本开发（API/E2E/压测/安全） | - | 测试工程师 |
| 331 | 331-test-case-dev-review | 测试脚本开发评审 | - | 测试工程师 |

## 阶段 4: 测试阶段

### 测试执行
| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 410 | 410-test-case-execution | 测试执行（API/E2E/压测/安全） | 测试工程师 |
| 411 | 411-test-case-execution-review | 测试执行评审 | 测试工程师 |

## 阶段 5: 文档交付阶段（文档并行）

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 500 | 500-ops-manual | 运维文档编写 | 运维工程师 |
| 501 | 501-ops-manual-review | 运维文档评审 | 运维负责人 |
| 510 | 510-requirement-doc | 需求文档整理 | 产品经理 |
| 511 | 511-requirement-doc-review | 需求文档评审 | 产品经理 |
| 520 | 520-user-manual | 用户手册编写 | 测试工程师 |
| 521 | 521-user-manual-review | 用户手册评审 | 测试负责人 |

## 阶段 6: 功能开发阶段（按需执行）

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 600 | 600-feature-clarify | 功能需求澄清 | 产品经理 |
| 601 | 601-feature-review | 功能评审 | 项目经理 |
| 610 | 610-feature-tech-design | 功能技术方案设计 | 系统架构师 |
| 620 | 620-feature-java-uniweb-dev | 功能Java后端开发 | Java后端工程师 |
| 630 | 630-feature-admin-web-dev | 功能Admin Web前端开发（root/ops/saas/mch） | JS前端工程师 |
| 630 | 630-feature-guest-web-dev | 功能Guest Web前端开发（guest） | JS前端工程师 |
| 630 | 630-feature-admin-uniapp-dev | 功能Admin UniApp移动端开发（root/ops/saas/mch） | JS前端工程师 |
| 630 | 630-feature-guest-uniapp-dev | 功能Guest UniApp移动端开发（guest） | JS前端工程师 |
| 640 | 640-feature-test-dev | 功能测试脚本开发 | 测试工程师 |
| 650 | 650-feature-final-review | 功能最终验收评审 | 产品经理 |
| 660 | 660-feature-update-doc | 功能5xx文档更新 | 技术写作 |

**流程**: 600 → 601 → 610 → (620/630×4/640并行) → 650 → 660

## 阶段 7: Bug修复阶段（按需执行）

| 编号 | 技能名 | 说明 | 主导角色 |
|------|--------|------|---------|
| 700 | 700-bugfix-analysis | Bug分析 | 开发工程师 |
| 701 | 701-bugfix-review | Bug评审 | 项目经理 |
| 710 | 710-bugfix-tech-design | Bug修复方案设计 | 系统架构师 |
| 720 | 720-bugfix-java-uniweb | Bug Java后端修复 | Java后端工程师 |
| 730 | 730-bugfix-admin-web | Bug Admin Web前端修复（root/ops/saas/mch） | JS前端工程师 |
| 730 | 730-bugfix-guest-web | Bug Guest Web前端修复（guest） | JS前端工程师 |
| 730 | 730-bugfix-admin-uniapp | Bug Admin UniApp移动端修复（root/ops/saas/mch） | JS前端工程师 |
| 730 | 730-bugfix-guest-uniapp | Bug Guest UniApp移动端修复（guest） | JS前端工程师 |
| 740 | 740-bugfix-test | Bug回归测试 | 测试工程师 |
| 750 | 750-bugfix-final-review | Bug修复最终验收 | 产品经理 |
| 760 | 760-bugfix-update-doc | Bug修复5xx文档更新 | 技术写作 |

**流程**: 700 → 701 → 710 → (720/730/740并行) → 750 → 760
