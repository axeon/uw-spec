---
name: 220-guest-web-design
description: Web端原型设计与开发（设计即代码）。当需要基于PRD进行Web端页面原型设计时触发：(1)确认页面清单与角色权限映射, (2)编写项目README.md页面清单, (3)裁剪230-gencode生成的页面代码, (4)配置路由与导航, (5)确保字段命名与数据库/后端一致, (6)输出可运行的原型项目。当用户提及Web原型、页面设计、Vue原型、前端设计时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# Web端原型设计与开发（设计即代码）

## 描述

基于 Vue3 + ElementPlus 技术栈，采用**设计即代码**模式：原型页面直接在前端项目中开发，设计文档作为项目 README.md。原型完成后项目可运行展示，页面代码可直接用于后续开发阶段。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| Web原型设计 | "设计Web端原型" |
| 页面原型开发 | "开发前端页面原型" |
| 原型裁剪 | "裁剪生成的页面代码" |

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

> **职责边界**：前端设计阶段产出原型页面（UI 壳子 + 路由 + 字段绑定），不含业务逻辑，无需单元测试。单元测试在 320 开发阶段由前端工程师编写（composables/Store/helper 函数）。测试工程师专职于测试脚本、API/E2E/压力/安全测试（阶段 330+）。

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `requirement/prds/*` | 产品需求文档，功能模块及页面需求 |
| 数据库设计 | `database/database-design.md` | 表结构、字段名（字段一致性参考） |
| 后端Swagger | `backend/{项目名}-app/` 启动后 | API接口定义 |
| 前端项目 | `frontend/{项目名}-guest-web/` | 230-init初始化 + 230-gencode生成的代码 |

## 前置条件

| 前置技能 | 说明 |
|---------|------|
| [220-guest-web-init](../220-guest-web-init/SKILL.md) | 前端项目已通过模板初始化 |
| [220-guest-web-gencode](../220-guest-web-gencode/SKILL.md) | api/types/pages已由代码生成器生成 |

**已生成代码**：代码生成器已产出 api调用、types定义、基础页面结构，原型阶段在此基础上裁剪和调整。

## 架构约定

| 约定 | 说明 |
|------|------|
| 页面路径第一级 | 必须为用户角色（`/saas/`、`/mch/`、`/admin/`、`/guest/`） |
| 页面初始位置 | 代码生成器产出在 `src/pages/{module}/`，按角色权限映射移动到目标目录 |
| 字段一致性 | 表单字段名必须与数据库字段名一致（snake_case → camelCase自动映射） |
| 组件命名 | 页面组件使用 PascalCase 命名规范 |

> 目录结构示例和完整开发规范见 [web-dev-standards.md](references/web-dev-standards.md)

## 设计流程

### Phase 0: 需求确认

确认聚焦业务层面。

| 确认项 | 目的 | 启发式问题 |
|--------|------|-----------|
| 页面清单 + 复杂度分类 | 决定哪些用生成器页面、哪些需要定制 | "根据PRD，识别到N个页面[列出]，是否有遗漏？" |
| 角色权限映射 | 确定页面归属角色 | "各页面分别属于哪个角色（guest/mch/admin/saas）？" |
| 定制页面 | 确定设计工作量 | "除标准CRUD页面外，还有哪些定制页面？" |

页面分类决策：

| 分类 | 条件 | 代码策略 |
|------|------|---------|
| 简单页面 | 仅标准CRUD（列表/详情/表单） | 代码生成器产出，裁剪即可 |
| 复杂页面 | 含特殊交互、多表联动、自定义布局 | 基于生成器页面改造或新建 |

**确认完成标准**：页面清单无遗漏、每个页面复杂度已分类、角色权限已映射。

### Phase 1: 编写 README.md

**输入**：PRD 文档 + Phase 0 确认结果

**输出**：前端项目根目录 `README.md`

**必须包含的章节**：

| 章节 | 内容 | 必要性 |
|------|------|--------|
| 页面总览 | 页面清单、复杂度分类、代码策略 | 必须 |
| 角色权限映射 | 角色（SAAS/MCH/ADMIN/GUEST）× 页面访问矩阵 | 必须 |
| PRD功能点映射 | 功能点 → 模块 → 页面的映射表 | 必须 |
| 路由设计 | 路由层级、导航结构 | 必须 |
| 字段一致性检查 | 表单字段 ↔ 数据库字段对照表 | 必须 |
| 组件清单 | 复用组件列表 | 按需 |

**README.md 模板**：参见 [references/templates.md](references/templates.md)

### Phase 2: 设计即代码

#### Step 1: 裁剪页面

> 页面由代码生成器自动生成，不新建文件，仅裁剪。

| 裁剪类型 | 操作 |
|---------|------|
| 删除不需要的页面 | 如资源不需要详情页，删除 Detail.vue |
| 调整表单字段 | 按业务需求增删表单项，确保字段名与数据库一致 |
| 调整列表列 | 按业务需求增删表格列 |
| 补充字段校验 | 添加 `rules` 校验规则 |

#### Step 2: 按角色移动页面

> 页面由代码生成器产出在 `src/pages/{module}/`，需按角色移动。

| 操作 | 说明 |
|------|------|
| 角色移动 | 根据 README.md 角色权限映射，将 `{module}/` 从 `pages/` 移动到目标角色目录（`admin/`、`mch/`、`saas/`、`guest/`） |
| 修正导入路径 | 更新页面内的组件导入路径 |
| 更新路由配置 | 在 `router/index.ts` 中注册新路由 |

**角色移动示例**：
```bash
# 将 product 模块从 pages 移动到 admin
mv src/pages/product/ src/pages/admin/product/
# 修正路由: /product/list → /admin/product/list
```

#### Step 3: 配置路由

> 在 `src/router/index.ts` 中配置角色级路由。

| 内容 | 说明 |
|------|------|
| 路由层级 | `/{角色}/{模块}/{页面}` |
| 权限守卫 | 使用路由守卫控制角色访问权限 |
| 导航菜单 | 在布局组件中配置侧边栏导航 |

#### Step 4: 字段一致性检查

> 确保表单字段、表格列名与数据库字段一致。

| 检查项 | 要求 |
|--------|------|
| 表单字段名 | 与后端DTO字段名一致（camelCase） |
| 表格列名 | 与后端返回字段名一致 |
| 搜索条件 | 与后端QueryParam字段一致 |
| 显示文本 | 使用 `label` 属性显示中文，字段名保持英文 |

**字段映射示例**：
```typescript
// 数据库字段: user_name (snake_case)
// 后端DTO字段: userName (camelCase)
// 前端表单字段: userName (camelCase)

<el-form-item label="用户名" prop="userName">
  <el-input v-model="form.userName" />
</el-form-item>
```

#### Step 5: 编译验证

| 检查项 | 命令/方式 |
|--------|----------|
| 编译通过 | `npm run build` |
| 无类型错误 | `npx vue-tsc --noEmit` |
| 页面可访问 | 启动dev服务器，验证各角色页面可正常访问 |
| 路由正确 | 验证路由跳转、权限控制正常 |

**代码模板**：参见 [references/templates.md](references/templates.md)

## PLAN-REVIEW 循环（必须执行）

设计完成后，必须进入 PLAN-REVIEW 循环，确保设计质量达标。

调用技能 `221-guest-web-design-review`，评分 ≥ 95 通过，< 95 按下表修复后重新评审（最多5轮，round ≥ 6 强制退出）:

| 优先级 | 要求 |
|--------|------|
| Critical | 本轮必须全部修复，不可遗留 |
| Major | 本轮修复 ≥ 80%，剩余标注下轮计划 |
| Minor | 记录但不阻塞，按优先级排列 |

循环结束时，在评审报告末尾追加：
```markdown
## PLAN-REVIEW 循环结论
- 总轮次: {N}/5
- 最终评分: {N}分
- 状态: {通过 | 有条件通过 | 强制退出}
- 遗留问题: Critical {N}, Major {N}, Minor {N}
```
## 产出结构
```
frontend/{项目名}-guest-web/
├── README.md                         # 页面清单与路由设计（Phase 1 产出）
├── src/
│   ├── pages/                        # 按角色组织的页面（Phase 2 产出）
│   │   ├── admin/                    # 裁剪后的页面
│   │   ├── mch/
│   │   ├── saas/
│   │   └── guest/
│   ├── components/                   # 业务组件
│   ├── router/                       # 路由配置
│   ├── api/                          # API调用（230-gencode生成）
│   └── types/                        # TypeScript定义（230-gencode生成）
└── package.json
```

## 设计完成标准

- [ ] README.md 覆盖所有页面和角色权限映射
- [ ] 所有PRD功能点都有对应页面
- [ ] 表单字段名与数据库字段名一致（字段一致性原则）
- [ ] 页面按角色正确组织到 `pages/{角色}/` 目录
- [ ] 路由配置正确，权限控制正常
- [ ] `npm run build` 编译通过
- [ ] 页面可正常访问展示
- [ ] 后端可基于Swagger开始联调

## 参考

- [Web 开发规范](references/web-dev-standards.md) - Vue3+TS+Vite+Element Plus 后台管理规范
- [README + 代码模板](references/templates.md)
- [Web端原型评审技能](../221-guest-web-design-review/SKILL.md)
