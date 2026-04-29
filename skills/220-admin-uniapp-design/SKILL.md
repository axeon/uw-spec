---
name: 220-admin-uniapp-design
description: UniApp移动端原型设计与开发（设计即代码）。当需要基于PRD进行移动端页面原型设计时触发：(1)确认页面清单与角色权限映射, (2)编写项目README.md页面清单, (3)裁剪230-gencode生成的页面代码, (4)配置pages.json路由, (5)确保字段命名与数据库/后端一致, (6)输出可运行的原型项目。当用户提及移动端原型、UniApp设计、小程序原型、跨平台设计时使用此技能。适用于root/ops/saas/mch角色 ⚠️【强制】完成后必须调用 221-admin-uniapp-design-review，未通过前禁止声称完成。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# UniApp移动端原型设计与开发（设计即代码）

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

## 核心理念

| 传统方式 | 本技能方式 |
|---------|-----------|
| 独立设计稿 → 原型开发 → 正式开发 | README.md + 页面代码 → 直接可用 |
| 设计稿与代码可能不一致 | 代码即原型，天然一致 |
| 按设计维度组织文档 | 按角色/模块组织页面 |
| 设计和原型两个阶段 | 合并为一个阶段 |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 原型设计与开发 | `js-developer` |
| 协作 | 需求确认 | `product-manager` |
| 协作 | 技术可行性 | `system-architect` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `PROJECT_ROOT/requirement/prds/*` | 产品需求文档，功能模块及页面需求 |
| 数据库设计 | `PROJECT_ROOT/database/database-design.md` | 表结构、字段名（字段一致性参考） |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}-app/` 启动后 | API接口定义 |
| 前端项目 | `PROJECT_ROOT/frontend/{项目名}-admin-uniapp/` | 230-init初始化 + 230-gencode生成的代码 |

## 前置条件

| 前置技能 | 说明 |
|---------|------|
| [220-admin-uniapp-init](../220-admin-uniapp-init/SKILL.md) | 移动端项目已通过模板初始化 |
| [220-admin-uniapp-gencode](../220-admin-uniapp-gencode/SKILL.md) | api/types/pages已由代码生成器生成 |

**已生成代码**：代码生成器已产出 api调用、types定义、基础页面结构，原型阶段在此基础上裁剪和调整。

## 架构约定

| 约定 | 说明 |
|------|------|
| 页面路径 | `src/pages/{角色}/{模块}/{页面}.vue` |
| 页面初始位置 | 代码生成器产出在 `src/pages/{模块}/`，按角色权限映射移动到目标目录 |
| 字段一致性 | 表单字段名必须与数据库字段名一致（snake_case → camelCase自动映射） |
| 组件命名 | 页面组件使用 kebab-case 命名规范 |
| 路由配置 | 在 `pages.json` 中配置页面路由 |

> 目录结构示例和完整开发规范见 [md-dev-standards.md](references/md-dev-standards.md)

## 设计流程

### Phase 0: 需求确认

确认聚焦业务层面。

| 确认项 | 目的 | 启发式问题 |
|--------|------|-----------|
| 页面清单 + 复杂度分类 | 决定哪些用生成器页面、哪些需要定制 | "根据PRD，识别到N个页面[列出]，是否有遗漏？" |
| 角色权限映射 | 确定页面归属角色 | "各页面分别属于哪个角色（guest/mch/admin/saas）？" |
| 平台适配 | 确定需要支持的平台 | "需要支持哪些平台（微信小程序/H5/App）？" |
| 定制页面 | 确定设计工作量 | "除标准CRUD页面外，还有哪些定制页面？" |

页面分类决策：

| 分类 | 条件 | 代码策略 |
|------|------|---------|
| 简单页面 | 仅标准CRUD（列表/详情/表单） | 代码生成器产出，裁剪即可 |
| 复杂页面 | 含特殊交互、多表联动、自定义布局 | 基于生成器页面改造或新建 |

**确认完成标准**：页面清单无遗漏、每个页面复杂度已分类、角色权限已映射、平台适配已确认。

### Phase 1: 编写 README.md

**输入**：PRD 文档 + Phase 0 确认结果

**输出**：前端项目根目录 `README.md`

**必须包含的章节**：

| 章节 | 内容 | 必要性 |
|------|------|--------|
| 页面总览 | 页面清单、复杂度分类、代码策略 | 必须 |
| 角色权限映射 | 角色（SAAS/MCH/ADMIN/GUEST）× 页面访问矩阵 | 必须 |
| PRD功能点映射 | 功能点 → 模块 → 页面的映射表 | 必须 |
| 路由设计 | pages.json 页面路由配置 | 必须 |
| 字段一致性检查 | 表单字段 ↔ 数据库字段对照表 | 必须 |
| 平台适配 | 微信小程序/H5/App 适配策略 | 必须 |
| 组件清单 | 复用组件列表 | 按需 |

**README.md 模板**：参见 [references/templates.md](references/templates.md)

### Phase 2: 设计即代码

#### Step 1: 裁剪页面

> 页面由代码生成器自动生成，不新建文件，仅裁剪。

| 裁剪类型 | 操作 |
|---------|------|
| 删除不需要的页面 | 如资源不需要详情页，删除 detail.vue |
| 调整表单字段 | 按业务需求增删表单项，确保字段名与数据库一致 |
| 调整列表列 | 按业务需求增删列表项 |
| 补充字段校验 | 添加表单校验规则 |

#### Step 2: 按角色移动页面

> 页面由代码生成器产出在 `src/pages/{模块}/`，需按角色移动。

| 操作 | 说明 |
|------|------|
| 角色移动 | 根据 README.md 角色权限映射，将 `{模块}/` 从 `pages/` 移动到目标角色目录（`admin/`、`mch/`、`saas/`、`guest/`） |
| 修正导入路径 | 更新页面内的组件导入路径 |
| 更新 pages.json | 在 `pages.json` 中注册新页面路由 |

**角色移动示例**：
```bash
# 将 product 模块从 pages 移动到 admin
mv src/pages/product/ src/pages/admin/product/
# 更新 pages.json: pages/product/list → pages/admin/product/list
```

#### Step 3: 配置路由

> 在 `pages.json` 中配置页面路由。

| 内容 | 说明 |
|------|------|
| 页面路径 | `pages/{角色}/{模块}/{页面}` |
| 导航栏 | 配置 navigationBarTitleText |
| TabBar | 如需要，配置底部导航 |

#### Step 4: 字段一致性检查

> 确保表单字段、列表项名与数据库字段一致。字段映射规则和示例见 [md-dev-standards.md](references/md-dev-standards.md)

| 检查项 | 要求 |
|--------|------|
| 表单字段名 | 与后端DTO字段名一致（camelCase） |
| 列表项名 | 与后端返回字段名一致 |
| 显示文本 | 使用 label 显示中文，字段名保持英文 |

#### Step 5: 平台适配

> 多端适配规范详见 [md-dev-standards.md](references/md-dev-standards.md)

#### Step 6: 编译验证

| 检查项 | 命令/方式 |
|--------|----------|
| 编译通过 | `npm run build:h5` / `npm run build:mp-weixin` |
| 无类型错误 | `npx vue-tsc --noEmit` |
| 页面可访问 | 启动dev服务器，验证各角色页面可正常访问 |
| 路由正确 | 验证页面跳转、权限控制正常 |

**代码模板**：参见 [references/templates.md](references/templates.md)

## ⚠️ 完成验证（强制，全自动执行）

开发工作完成后，**立即按以下顺序自动执行**：

1. **强制调用** `221-admin-uniapp-design-review`
2. 如果评审不通过（< 95），自动修复问题，然后回到步骤 1（最多 5 轮）
3. 直到评审通过（≥ 95），**才向用户报告最终结果**

> **此流程全自动执行：中间不暂停、不询问、不汇报。**
> **未收到通过确认前，禁止结束本技能任务。**

## 产出结构

```
$PROJECT_ROOT/frontend/{项目名}-admin-uniapp/
├── README.md                         # 页面清单与路由设计（Phase 1 产出）
├── src/
│   ├── pages/                        # 按角色组织的页面（Phase 2 产出）
│   │   ├── admin/                    # 裁剪后的页面
│   │   ├── mch/
│   │   ├── saas/
│   │   └── guest/
│   ├── components/                   # 业务组件
│   ├── api/                          # API调用（230-gencode生成）
│   └── types/                        # TypeScript定义（230-gencode生成）
├── pages.json                        # 页面路由配置
└── package.json
```

## 设计完成标准

- [ ] README.md 覆盖所有页面和角色权限映射
- [ ] 所有PRD功能点都有对应页面
- [ ] 表单字段名与数据库字段名一致（字段一致性原则）
- [ ] 页面按角色正确组织到 `pages/{角色}/` 目录
- [ ] pages.json 路由配置正确
- [ ] 编译通过（H5/微信小程序/App）
- [ ] 页面可正常访问展示
- [ ] 后端可基于Swagger开始联调

## 下一步

设计（含 REVIEW评审）通过评审后，提示用户进入 **320-admin-uniapp-dev** 进行UniApp移动端业务开发。

**流程位置**：`220-admin-uniapp-init` → `220-admin-uniapp-gencode` → `220-admin-uniapp-design` (+ 221 review) → **320-admin-uniapp-dev**

## 参考

- [UniApp 开发规范](references/md-dev-standards.md) - Vue3+TS 多端开发规范
- [README + 代码模板](references/templates.md)
- [移动端原型评审技能](../221-admin-uniapp-design-review/SKILL.md)
