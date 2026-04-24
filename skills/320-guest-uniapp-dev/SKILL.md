---
name: 320-guest-uniapp-dev
description: UniApp移动端开发技能（基于220原型设计完善）。当需要基于220原型设计产出完善UniApp项目时触发：(1)对接真实后端API替换mock, (2)完善页面交互逻辑和业务组件, (3)实现平台适配（微信小程序/H5/App）, (4)实现前端状态管理, (5)编写单元测试, (6)指定模块开发或并行开发全部模块。当用户提及完善移动端、对接API、实现业务逻辑、平台适配、指定模块、并行开发时使用此技能。适用于guest（消费者）角色。
alwaysApply: false
author: "axeon(23231269@qq.com)"
version: "1.0.0"
---

# UniApp移动端开发（基于原型完善）

## 描述

基于 220-guest-uniapp-design 产出的可运行原型页面，完善业务逻辑实现。220 设计阶段已完成页面结构、路由配置、字段绑定，本阶段专注于对接真实后端 API、完善交互逻辑、平台适配、实现业务组件。

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 0-init。

## 使用场景

| 触发条件 | 示例 |
|---------|------|
| 完善移动端实现 | "完善移动端业务逻辑" |
| 对接后端API | "对接后端API" |
| 平台适配 | "适配微信小程序/H5/App" |
| 实现交互功能 | "实现页面交互" |
| 指定模块开发 | "开发product模块页面" |
| 并行开发 | "并行开发所有模块页面" |

## 核心理念

| 传统方式 | 本技能方式 |
|---------|-----------|
| 从零开发页面 | **基于220原型设计完善** |
| 先写页面再对接API | **页面已存在，专注对接API** |
| 设计稿→代码 | **可运行原型→完善实现** |

## 角色职责

| 角色 | 职责 | 智能体 |
|------|------|--------|
| 主导 | 移动端开发完善 | `js-developer` |
| 协作 | API接口确认 | `java-developer` |
| 协作 | 交互设计确认 | `product-manager` |

## 输入

| 输入项 | 来源路径 | 说明 |
|--------|----------|------|
| PRD | `PROJECT_ROOT/requirement/prds/*` | 产品需求文档，功能验收标准 |
| 原型README | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/README.md` | 220设计阶段产出的页面清单和路由设计 |
| 原型页面 | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/pages/` | 220设计阶段裁剪后的页面代码 |
| 后端Swagger | `PROJECT_ROOT/backend/{项目名}-app/` 启动后 | API接口定义 |
| API/types | `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/src/api/`, `src/types/` | 230-gencode生成的代码 |

## 前置条件

| 前置技能 | 说明 |
|---------|------|
| [220-guest-uniapp-design](../220-guest-uniapp-design/SKILL.md) | 原型页面已产出并通过评审 |
| [310-java-uniweb-dev](../310-java-uniweb-dev/SKILL.md) | 后端API已实现并可访问 |

## 开发范围

### 与240原型的分工

| 阶段 | 负责内容 | 产出状态 |
|------|---------|---------|
| **220-design** | 页面结构、路由、字段绑定、基础样式 | 可运行，使用mock数据 |
| **320-dev** | API对接、交互逻辑、平台适配、业务组件、状态管理 | 可运行，使用真实API |

### 本阶段核心工作

| 工作项 | 说明 | 示例 |
|--------|------|------|
| **API对接** | 替换mock为真实API调用 | `api.getList()` → `api.product.list()` |
| **交互完善** | 实现加载状态、下拉刷新、上拉加载 | 下拉刷新、加载更多 |
| **平台适配** | 处理微信小程序/H5/App差异 | 登录、支付、分享 |
| **业务组件** | 实现复杂的业务组件 | 商品卡片、订单流程 |
| **状态管理** | 实现跨页面状态共享 | 购物车、用户状态 |

## 技术栈

| 技术 | 用途 | 文档 |
|------|------|------|
| UniApp + Vue3 | 跨平台框架 | [UniApp速查](../0-init/references/frontend/uniapp-cheatsheet.md) |
| TypeScript | 类型安全 | - |
| Pinia | 状态管理 | [Pinia速查](../0-init/references/frontend/pinia-cheatsheet.md) |
| uni-ui | UI组件库 | [uni-ui文档](https://uniapp.dcloud.net.cn/component/uniui/uni-ui.html) |

## 开发模式

| 模式 | 触发方式 | 说明 |
|------|---------|------|
| 单模块开发 | "完善 {模块名} 页面" | 指定一个模块，完成 API对接+交互+平台适配+测试 |
| 并行多模块 | "并行完善所有页面"（**默认**） | 各模块页面独立，天然可并行 |
| 顺序全量 | "完善所有页面" | 按模块逐个完善 |

## 220→320 衔接协议

220 设计阶段完成后，320 从 README.md 提取以下信息：

| 提取项 | README.md 章节 | 用途 |
|--------|---------------|------|
| 页面清单 | "页面总览" | 确定开发范围 |
| 复杂度分类 | "页面总览" | 简单页面仅裁剪，复杂页面需交互完善 |
| 角色权限矩阵 | "角色权限映射" | 页面归属和权限控制 |
| PRD映射 | "PRD功能点映射" | 按模块索引对应需求 |
| 路由设计 | "路由设计" | pages.json 页面路由 |
| 平台适配 | "平台适配" | 微信小程序/H5/App 适配策略 |

## 并行开发约束

| 约束 | 说明 |
|------|------|
| 页面天然独立 | 各模块页面在不同文件，无文件级冲突 |
| 共享组件串行 | 公共组件（components/）修改需串行 |
| API层只读 | src/api/ 由代码生成器产出，并行时只读不改 |
| review按模块 | 每个模块开发完成后独立进入 PLAN-REVIEW |

## 开发流程

### 1. 环境准备

**确认输入完整性**：
- [ ] 240原型README.md存在且完整
- [ ] 原型页面可正常编译运行
- [ ] 后端Swagger可访问
- [ ] API/types代码已生成

**启动开发环境**：
```bash
cd PROJECT_ROOT/frontend/{项目名}-guest-uniapp
npm install
# 微信小程序
npm run dev:mp-weixin
# H5
npm run dev:h5
```

### 2. API对接

> API 调用封装规范见 [md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md)

| 步骤 | 操作 |
|------|------|
| 1 | 检查 `src/api/{module}/index.ts` 已生成 |
| 2 | 导入API，替换mock调用为真实API调用 |
| 3 | 统一处理 ResponseData 包装、try-catch 错误提示 |

**API调用示例**：参见 [references/examples.md](references/examples.md)

### 3. 交互逻辑完善

> 多端交互规范（下拉刷新、上拉加载、确认弹窗等）见 [references/examples.md](references/examples.md)

### 4. 平台适配

> 完整平台适配规范（登录/支付/分享的条件编译）见 [md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md)

### 5. 业务组件实现

**组件目录**：`src/components/business/`

| 组件类型 | 示例 | 说明 |
|---------|------|------|
| 商品卡片 | `ProductCard.vue` | 商品列表展示 |
| 订单项 | `OrderItem.vue` | 订单列表项 |
| 规格选择 | `SkuSelector.vue` | 商品规格选择弹窗 |

**组件规范**：使用 `defineProps<T>()` + `defineEmits<T>()` 泛型风格。

### 6. 状态管理实现

> Pinia 状态管理规范和示例见 [md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md)

**Store目录**：`src/stores/`

| Store | 用途 |
|-------|------|
| `user.ts` | 用户状态、登录信息、权限 |
| `cart.ts` | 购物车、跨页面状态 |
| `app.ts` | 系统信息、网络状态 |

### 7. 编译验证

| 检查项 | 命令/方式 |
|--------|----------|
| 编译通过 | `npm run build:mp-weixin` / `npm run build:h5` / `npm run build:app` |
| 无类型错误 | `npx vue-tsc --noEmit` |
| 代码规范 | `npm run lint` |
| 单元测试 | `npm run test:unit` |
| 真机调试 | 微信开发者工具真机调试 |
| API连通 | 验证所有页面可正常调用后端API |

## 字段一致性原则

> 完整字段映射规则见 [md-dev-standards.md](../220-guest-uniapp-design/references/md-dev-standards.md)

| 层级 | 命名规范 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `product_name` |
| 后端DTO字段 | camelCase | `productName` |
| 前端表单字段 | camelCase | `productName` |
| API参数 | camelCase | `productName` |

## 输出要求

**代码位置**: `PROJECT_ROOT/frontend/{项目名}-guest-uniapp/`

**包含内容**:
- 完善后的页面组件（对接真实API）
- 平台适配代码（微信小程序/H5/App）
- 业务组件实现
- Pinia状态管理
- 单元测试
- 编译验证通过

## PLAN-REVIEW 循环（必须执行）

UniApp移动端开发完成后，调用 `321-guest-uniapp-dev-review` 执行评审。

## 参考

- [UniApp 开发规范](../220-guest-uniapp-design/references/md-dev-standards.md) - Vue3+TS 多端开发规范
- [开发示例](references/examples.md) - API对接、平台适配、组件示例
- [220-guest-uniapp-design](../220-guest-uniapp-design/SKILL.md) - 上游原型设计阶段
- [321-guest-uniapp-dev-review](../321-guest-uniapp-dev-review/SKILL.md) - PLAN-REVIEW循环评审
- [开发示例](references/examples.md) - API对接、平台适配、组件开发示例
