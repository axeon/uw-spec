# 项目模板与规范

## 项目信息文件格式

**文件位置**: `project-info.md`

```markdown
---
project-name: project-name（限定英文小写字母+数字+下划线，下划线只能出现在中间，长度1-32个字符，例如：nova_app）
project-label: 项目标签（项目中文名，例如：Nova应用）
project-desc: 项目描述信息（例如：Nova应用是一个基于Java的Web应用，用于管理用户信息和订单。）
project-mode: uniweb（项目模式：uniweb=UniWeb基础项目，saas=SaaS多租户项目）
project-server: 127.0.0.1（开发服务器IP地址）
created-at: 2024-01-01T00:00:00+08:00（项目创建时间，ISO8601格式）
updated-at: 2024-01-01T00:00:00+08:00（最后更新时间，ISO8601格式）
current-stage: 0-init（当前流程阶段）
current-skill: 0-init（当前执行技能）
---

# 项目信息

## 项目概述
...

## 技术栈
...

## 项目结构
...
```

## 完整项目结构

```
project/
├── project-info.md           # 项目信息文件
├── requirement/              # 需求规划文档
│   ├── prds/                 # PRD产品需求文档
│   │   ├── README.md         # PRD主文档（合并所有变更）
│   │   ├── CHANGELOG.md      # PRD变更历史
│   │   ├── {用户角色}-{终端类型}/   # 按用户角色和终端类型分目录
│   │   │   ├── README.md
│   │   └── ...
│   ├── interviews/           # 需求访谈记录
│   │   └── INTERVIEW-{YYMMDDHHMM}-{主题}.md
│   └── reviews/              # 需求评审报告
│       └── REVIEW-PRD-{YYMMDDHHMM}.md
├── backend/                  # 后端项目（Java + uw-base）
│   └── {项目名}-app/
│       ├── README.md     # 后端架构文档
│       ├── CHANGELOG.md  # 代码变更历史
│       ├── database/         # 数据库设计文档
│       │   ├── database-design.md
│       │   ├── database-ddl.sql
│       │   └── migrations/   # 日期开头的DDL文件
│       │       ├── FEATURE-{YYMMDD}-{简述}.sql
│       │       └── BUGFIX-{YYMMDD}-{简述}.sql
│       ├── issues/     # 功能级技术文档（6xx/7xx阶段）
│       │    ├── FEATURE-DESIGN-{YYMMDD}-{简述}.md
│       │    └── BUGFIX-DESIGN-{YYMMDD}-{简述}.md
│       ├── reviews/      # 评审报告
│       │   ├── REVIEW-DB-{YYMMDDHHMM}.md
│       │   ├── REVIEW-DESIGN-{YYMMDDHHMM}.md
│       │   └── REVIEW-CODE-{YYMMDDHHMM}.md
│       └── src/              # 源代码
├── frontend/                 # 前端项目
│   └── {项目名}-{用户角色}-{终端类型}/         
│       ├── README.md
│       ├── CHANGELOG.md
│       ├── issues/     # 功能级技术文档（6xx/7xx阶段）
│       │    ├── FEATURE-DESIGN-{YYMMDD}-{简述}.md
│       │    └── BUGFIX-DESIGN-{YYMMDD}-{简述}.md
│       ├── reviews/      # 评审报告
│       │   ├── REVIEW-DESIGN-{YYMMDDHHMM}.md
│       │   └── REVIEW-CODE-{YYMMDDHHMM}.md
│       └── src/              # 源代码
├── test/                     # 测试项目
│   ├── README.md
│   ├── CHANGELOG.md
│   ├── design/               # 测试设计文档
│   │   ├── api/              # API测试设计
│   │   ├── e2e/              # E2E测试设计
│   │   ├── load/             # 压测设计
│   │   └── security/         # 安全测试设计
│   ├── scripts/              # 测试脚本
│   │   ├── api/              # API测试脚本（Playwright）
│   │   │   └── *.spec.ts
│   │   ├── e2e/              # E2E测试脚本（Playwright）
│   │   │   ├── {项目名}/      # 单终端E2E
│   │   │   ├── cross-case/   # 跨终端E2E
│   │   │   └── shared/       # 共享工具和Page Object
│   │   ├── load/             # 压测脚本（JMeter .jmx）
│   │   │   └── data/         # CSV测试数据
│   │   ├── security/         # 安全测试脚本
│   │   ├── utils/            # 共享工具函数
│   │   ├── fixtures/         # 测试固件
│   │   ├── data/             # 测试数据
│   │   └── bin/              # 执行脚本
│   ├── reports/              # 汇总测试报告
│   │   └── test-report-{YYMMDDHHMM}/
│   │       ├── api/
│   │       ├── e2e/
│   │       ├── load/
│   │       ├── security/
│   │       └── README.md
│   ├── issues/      # 功能级测试设计（6xx阶段）
│   │   ├── FEATURE-DESIGN-{YYMMDD}-{简述}.md
│   │   └── BUGFIX-DESIGN-{YYMMDD}-{简述}.md
│   └── reviews/
│       ├── REVIEW-DESIGN-{YYMMDDHHMM}.md
│       └── REVIEW-CODE-{YYMMDDHHMM}.md
├── issue/              # Bug修复文档（7xx阶段）
│   ├── features/                 # 功能修复方案
│   │   ├── FEATURE-{YYMMDD}-{简述}.md
│   ├── bugs/                 # Bug分析与修复方案
│   │   ├── BUGFIX-{YYMMDD}-{简述}.md
│   └── reviews/              # 验收评审报告
│       ├── REVIEW-FEATURE-{YYMMDD}-{简述}-{HHMM}.md
│       └── REVIEW-BUGFIX-{YYMMDD}-{简述}-{HHMM}.md
└── manual/                 # 文档交付阶段（5xx）
    ├── ops-manual/           # 运维文档
    │   ├── README.md         # 运维文档主文档
    │   ├── CHANGELOG.md
    ├── user-manual/          # 用户手册
    │   ├── README.md
    │   ├── CHANGELOG.md
    │   └── {用户角色}-{终端类型}/   # 按用户角色和终端类型分目录
    │        └── README.md
    └── reviews/              # 验收评审报告
        ├── REVIEW-OPS-MANUAL-{YYMMDDHHMM}.md
        └── REVIEW-USERS-MANUAL-{YYMMDDHHMM}.md
```

## 八阶段流转规则

### 流转原则
```
阶段间串行 → 必须完成前一阶段才能进入下一阶段
阶段内并行 → 同一阶段内的多个独立流程可同时执行
按需执行 → 阶段6/7根据需求触发，不强制流转
```

### 阶段流转条件

| 从阶段 | 到阶段 | 流转条件 | 说明 |
|--------|--------|----------|------|
| 阶段0 | 阶段1 | 项目信息确认 | 完成项目初始化 |
| 阶段1 | 阶段2 | 需求评审通过（≥95分） | 需求明确且可执行 |
| 阶段2 | 阶段2.5 | 设计评审全部通过（≥95分） | DDL执行就绪 |
| 阶段2.5 | 阶段3 | DDL执行评审通过（≥95分） | 数据库环境可用 |
| 阶段3 | 阶段4 | 开发评审全部通过（≥95分） | 代码开发完成 |
| 阶段4 | 阶段5 | 测试评审全部通过（≥95分） | 系统可发布 |
| 阶段5 | - | 文档整理完成 | 项目文档归档 |
| *按需* | 阶段6 | 新功能需求 | 触发600-feature-clarify |
| *按需* | 阶段7 | Bug报告 | 触发700-bugfix-analysis |

### 评审流转规则

**通过标准**:
- **通过（≥95分）**: 进入下一阶段
- **有条件通过（80-89分）**: 修复Major问题后进入下一阶段
- **不通过（<80分或有Critical）**: 返回当前阶段修改

**流转公式**:
```
通过 → 进入下一阶段
有条件通过 → 修复后进入下一阶段
不通过 → 返回当前阶段修改
```

### 阶段内并行规则

| 阶段 | 并行组 | 并行流程 | 合并点 |
|------|--------|----------|--------|
| 2-design | 数据库组 | 200, 201-review | 完成后进入评审 |
| 2-design | 后端组 | 210-init, 210-gencode, 210-design, 211-review | 完成后进入评审 |
| 2-design | 前端组 | 220-init, 220-gencode, 220-design, 221-review | 完成后进入评审 |
| 2-design | 测试组 | 230-design, 231-review | 完成后进入评审 |
| 2.5-ddl | 执行组 | 300-execution, 301-review | 完成后进入开发 |
| 3-project | 开发组 | 310, 320-web-vue, 320-md-uniapp, 330 | 全部完成后进入测试 |
| 6-feature | 开发组 | 620, 630, 631, 640 | 全部完成后进入验收 |
| 7-bugfix | 修复组 | 720, 730, 731, 740 | 全部完成后进入验收 |
| 5-manual | 文档组 | 500, 510, 520 | 全部完成后项目归档 |

### 阶段回退规则

| 回退场景 | 回退到 | 触发条件 |
|----------|--------|----------|
| 设计阶段发现需求问题 | 1-requirement | 需求评审不通过 |
| 开发阶段发现设计问题 | 2-design | 设计评审不通过 |
| 测试阶段发现代码问题 | 3-project | 开发评审不通过 |
| 测试阶段发现需求问题 | 1-requirement | 严重需求缺陷 |

### 特殊流转

**阶段6/7**:
- 功能开发: 600 → 601 → 610 → (620/630/631/640并行) → 650 → 660
- Bug修复: 700 → 701 → 710 → (720/730/731/740并行) → 750 → 760
- 阶段6/7可独立循环，不影响主线流程

**快速通道**:
- 紧急修复: 可跳过部分评审，但需事后补评审
- 文档更新: 可直接进入阶段5

## 项目信息更新规则

### 元数据更新时机

| 触发时机 | 更新字段 | 说明 |
|----------|----------|------|
| 任何技能执行完成 | `updated-at` | 更新为当前时间（ISO8601格式） |
| 任何技能执行完成 | `current-skill` | 更新为刚完成的技能ID |
| 阶段流转完成 | `current-stage` | 更新为新阶段ID |

### 项目结构更新规则

每个流程完成后，必须在 `project-info.md` 的「项目结构」章节更新已明确的目录位置：

| 完成流程 | 更新内容 | 示例 |
|----------|----------|------|
| 0-init | 确认项目根目录结构 | `project-name: my-shop` → `backend/my-shop-app/` |
| 110-requirement | 添加PRD文档路径 | `requirement/prds/guest-web/README.md` |
| 200-database-design | 添加数据库文档路径 | `backend/my-shop-app/database/database-design.md` |
| 210-java-uniweb-init | 确认后端项目路径 | `backend/my-shop-app/`（已初始化） |
| 220-web-vue-init | 确认前端项目路径 | `frontend/my-shop-web/`（已初始化） |
| 600-feature-clarify | 添加功能需求文档 | `requirement/prds/{角色}-{终端}/20240115-FEATURE-订单导出.md` |
| 610-feature-tech-design | 添加技术方案文档 | `backend/my-shop-app/issues/FEATURE-DESIGN-240115-订单导出-tech-design.md` |
| 620-feature-java-uniweb-dev | 添加后端开发文档 | `backend/my-shop-app/issues/FEATURE-DESIGN-240115-订单导出-java-uniweb.md` |
| 700-bugfix-analysis | 添加Bug分析报告 | `issue/bugs/BUGFIX-240115-登录失败.md` |

### 更新示例

**初始状态**（0-init完成后）：
```markdown
## 项目结构

```
my-shop/
├── project-info.md
├── requirement/
└── backend/
    └── my-shop-app/          # 待初始化
```
```

**需求阶段完成后**（110-requirement完成后）：
```markdown
## 项目结构

```
my-shop/
├── project-info.md
├── requirement/
│   ├── prds/
│   │   ├── README.md
│   │   ├── CHANGELOG.md
│   │   ├── guest-web/        # 游客-Web端
│   │   │   ├── README.md
│   │   │   └── 20240115-FEATURE-订单导出.md
│   │   ├── guest-uniapp/     # 游客-移动端
│   │   │   └── README.md
│   │   └── admin-web/        # 管理员-Web端
│   │       └── README.md
│   │   └── interviews/
│   │       └── INTERVIEW-2401011430-初始需求.md
│   └── reviews/
└── backend/
    └── my-shop-app/          # 待初始化
```
```

**设计阶段完成后**（200-database-design + 210-java-uniweb-init完成后）：
```markdown
## 项目结构

```
my-shop/
├── project-info.md
├── requirement/
│   └── prds/
│       └── ...
└── backend/
    └── my-shop-app/          # 已初始化
        ├── pom.xml
        ├── README.md
        ├── CHANGELOG.md
        ├── database/
        │   ├── database-design.md
        │   ├── database-ddl.sql
        │   └── migrations/
        ├── issues/
        └── reviews/
```
```

**功能开发完成后**（600→660完成后）：
```markdown
## 项目结构

```
my-shop/
├── project-info.md
├── requirement/
│   └── prds/
│       ├── guest-web/
│       │   ├── README.md
│       │   └── 20240115-FEATURE-订单导出.md
│       └── ...
├── backend/
│   └── my-shop-app/
│       ├── database/
│       │   └── migrations/
│       │       └── FEATURE-240115-订单导出.sql
│       ├── issues/
│       │   └── FEATURE-DESIGN-240115-订单导出-tech-design.md
│       ├── reviews/
│       │   └── REVIEW-CODE-2401151430.md
│       └── src/
├── frontend/
│   └── guest-web/
│       ├── issues/
│       │   └── FEATURE-DESIGN-240115-订单导出-web-vue.md
│       ├── reviews/
│       │   └── REVIEW-CODE-2401151430.md
│       └── src/
├── test/
│   ├── issues/
│   │   └── FEATURE-DESIGN-240115-订单导出-test.md
│   ├── reviews/
│   │   └── REVIEW-CODE-2401151430.md
│   └── scripts/
├── issue/
│   ├── features/
│   │   └── FEATURE-240115-订单导出.md
│   └── reviews/
│       └── REVIEW-FEATURE-240115-订单导出-1430.md
└── manual/
    ├── ops-manual/
    │   ├── README.md
    │   └── CHANGELOG.md
    └── user-manual/
        ├── README.md
        ├── CHANGELOG.md
        └── guest-web/
            └── README.md
```
```
