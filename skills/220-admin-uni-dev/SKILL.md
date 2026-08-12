---
name: 220-admin-uni-dev
description: 管理端UniApp开发（AI原生，逐页面完整交付）。当需要基于PRD进行管理端移动端页面开发时触发：(1)确认页面清单与角色权限, (2)编写架构蓝图README.md, (3)逐页面完整交付（创建页面+路由配置+API对接+平台适配+测试+编译验证）。当用户提及管理端App、移动审批、商户移动端、运营管理小程序时使用。适用于root/ops/saas/mch角色 ⚠️【强制】完成后必须调用 221-admin-uni-dev-review，未通过前禁止声称完成。
alwaysApply: false
author: 'axeon(23231269@qq.com)'
version: '3.0.0'
---

# 管理端UniApp开发（AI 原生）

## 项目环境检测

从当前目录向上查找 `project-info.md`，最多 3 层，找到后记为 `PROJECT_ROOT`。详见 [检测方法与前置检查](../0-init/references/project-env-check.md)。**未找到** → 提示用户先执行 `0-init`。

## 角色职责

| 角色 | 职责                                             | 智能体            |
| ---- | ------------------------------------------------ | ----------------- |
| 主导 | 页面开发 + API对接 + 平台适配 + 测试（一次完成） | `js-developer`    |
| 协作 | 业务需求确认                                     | `product-manager` |

## 输入

| 输入项   | 来源路径                                                  | 说明                                  |
| -------- | --------------------------------------------------------- | ------------------------------------- |
| PRD      | `PROJECT_ROOT/requirement/prds/*`                         | 产品需求文档，功能模块及页面需求      |
| API定义  | `PROJECT_ROOT/frontend/{project-name}-admin-uni/src/api/` | gencode 生成的 API 调用函数和类型定义 |
| 前端项目 | `PROJECT_ROOT/frontend/{project-name}-admin-uni/`         | init初始化 + gencode生成的代码        |

## 前置条件

| 前置技能                | 说明                        |
| ----------------------- | --------------------------- |
| `220-admin-uni-init`    | 移动端项目已通过模板初始化  |
| `220-admin-uni-gencode` | api/types已由代码生成器生成 |

**已生成代码**：代码生成器已产出 API 函数和 TypeScript 类型定义，在此基础上开发页面。

## 架构约定速查表

### 页面结构约定

| ✅ 正确                                                                                                                      | ❌ 错误                                                 |
| ---------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| 业务页面：`src/packages/{module}/{role}/{Feature}/{Entity}/index.vue`                                                        | 业务页面：`src/pages/{module}/{page}.vue`（无角色分层） |
| 通用页面：`src/pages/home/index.vue`、`src/pages/login/index.vue` 等                                                         | 通用页面按角色组织                                      |
| gencode 产出按角色权限映射移动到目标目录                                                                                     | 直接在 gencode 产出位置使用                             |
| 页面目录 kebab-case，组件文件 PascalCase                                                                                     | 页面用 PascalCase                                       |
| 列表页/详情页/表单页三种标准结构                                                                                             | 页面结构随意组织                                        |
| 通过 `src/api/` 层调用 API                                                                                                   | 直接 `uni.request`                                      |
| `pages.json` 中配置通用页面和 TabBar，`subPackages` 中配置业务页面，路径 `packages/{module}/{role}/{Feature}/{Entity}/index` | 路由路径缺少 `{role}/{Feature}/{Entity}` 层级           |
| 导航栈模式为主，当前项目固定 3 个 TabBar 入口（管理菜单/常用工具/我的），其余页面用 navigationBar + 返回按钮                 | 非 TabBar 页面错误使用 TabBar                           |
| 后端返回菜单权限列表，前端动态控制页面访问和按钮显示                                                                         | 前端硬编码权限                                          |

### 数据解析约定

| ✅ 正确                                                                | ❌ 错误                            |
| ---------------------------------------------------------------------- | ---------------------------------- |
| 列表 API：`res.data?.list` → `T[]`（`res.data?.results` 仅兼容旧代码） | `res.data?.rows`                   |
| 实体 API：`res.data` → `T`                                             | 直接解构 `res`                     |
| 无返回值：`res.state === 'success'`                                    | 忽略状态检查                       |
| 状态/类型字段使用 `number` 值                                          | 状态值用 `string`（如 `'通过'`）   |
| 表单字段名 = API Schema 字段名（camelCase）                            | 凭 PRD 描述猜测字段名              |
| 列表查询参数默认带 `stateGte: 0`（过滤已删除 `-1`，仅显示上线/下线）   | 不传状态过滤条件，误显示已删除数据 |

### 平台适配约定

| ✅ 正确                                                        | ❌ 错误                               |
| -------------------------------------------------------------- | ------------------------------------- |
| `#ifdef MP-WEIXIN` / `#ifdef H5` 条件编译                      | 运行时 `if (platform === 'xxx')` 判断 |
| 微信登录用 `uni.login({ provider: 'weixin' })`                 | 微信登录用账号密码                    |
| H5 登录用账号密码                                              | H5 调用 `uni.login`                   |
| 样式用 rpx（750rpx = 屏幕宽度），Tailwind 用任意值 `p-[24rpx]` | 微信小程序用 px 固定尺寸              |
| 安全区 `env(safe-area-inset-bottom)`                           | 底部固定定位不考虑安全区              |

### 样式约定（Tailwind First · 强制）

> **总体原则**：**强制使用 Tailwind**。页面内所有一次性、简单样式必须优先使用 Tailwind 原子类；仅在明确允许的场景下才可使用 SCSS。AI 生成代码必须遵守本约定，禁止为简单样式新建 SCSS 类。
>
> - **强制 Tailwind**：布局/尺寸/间距/圆角/字体色/背景/边框/阴影等一次性、简单样式必须使用 Tailwind 任意值原子类（如 `p-[24rpx] m-[16rpx] bg-[#fff] rounded-[20rpx]`）
> - **允许 SCSS**：仅限可复用业务组件内部、复杂弹层/动画、`:deep()` 穿透、`env()` 安全区、`@keyframes`、伪元素、以及 5 个以上原子类叠加导致可读性严重下降的场景
> - **禁止为 Tailwind 找借口**：禁止以"class 太长"为由把简单样式写成 SCSS；class 长时应优先检查是否过度内联，而非退回 SCSS

| ✅ 必须                                                  | ❌ 禁止                                                     |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| 简单一次性布局使用 Tailwind 原子类                       | 简单一行样式新建 SCSS 类                                    |
| rpx 单位使用任意值：`w-[112rpx] p-[24rpx] text-[26rpx]`  | 依赖 Tailwind 内置 `rem` 尺寸（会导致移动端布局失真）       |
| 主题色使用 CSS 变量任意值：`text-[var(--color-primary)]` | `text-blue-500` 等 Tailwind 内置调色板（无法响应主题切换）  |
| 复杂/可复用样式可用 `<style scoped>` + BEM 组织          | 使用 `@apply` 自造 utility 类                               |
| 小程序端使用 `gap-*` 控制间距                            | 使用 `space-x-*` / `divide-*`（依赖后代通配符，小程序不稳） |
| 每个页面 `<style>` 块以空或极少量必要 SCSS 为主          | 页面写大量自定义 SCSS 类                                    |

**样式评审红线**：

- 页面出现超过 3 个自定义 SCSS 类且无明确理由 → 必须改为 Tailwind
- 简单样式（如单个 padding/margin/color/background）使用 SCSS → 必须改为 Tailwind
- 同一样式在多处重复定义 → 必须复用 Tailwind 原子类

**允许使用 SCSS 的明确场景**：

- 可复用业务组件内部样式（如 `GbNavbar`、`GbFabButton`、`GbEchart`），避免使用方传大量原子类
- 5 个以上原子类叠加同一元素、可读性下降明显时
- 复杂弹层 / 动画 / 深色模式切换 / 伪元素 / `:deep()` / `env()` / `@keyframes`
- 项目已有大量 SCSS，保持一致性优先于强推 Tailwind（需在本次改动范围内逐步迁移）

**AI 生成样式红线（强制）**：

- 任何 AI 生成的页面/组件代码，布局、尺寸、间距、颜色、背景、边框、圆角、阴影等必须使用 Tailwind 原子类表达；禁止输出裸 `<style>` 块或内联 `style="..."` 来处理简单样式
- 若生成的代码中出现未使用 Tailwind 的简单样式（如手写 CSS 类、px 固定尺寸、非 CSS 变量的硬编码色值），必须回退并改写为 Tailwind

### 表单组件约定

| ✅ 正确                                                      | ❌ 错误                                          |
| ------------------------------------------------------------ | ------------------------------------------------ |
| 表单输入优先使用 `uv-ui` 组件：`uv-input`、`uv-textarea` 等  | 手写原生 `<input>` / `<textarea>` 并自行维护样式 |
| 选择器优先使用 `uni-data-select` 或 `uv-picker`              | 手写原生选择器或自己封装弹层                     |
| 表单校验、数字输入、禁用状态等复用组件内置能力               | 为单个表单手写步进器、校验逻辑、disabled 样式    |
| 复杂表单场景（如带 ± 步进器）仍优先以 `uv-ui` 组件为基础改造 | 从零手写完整的表单控件和样式                     |
| 表单必须使用 `uv-form` + `uv-form-item` 组织并声明校验规则   | 表单字段散落排版、手动 `if` 逐个校验             |

> 目标：统一表单体验，减少重复造轮子，确保表单样式与框架组件库一致。

### 页面级操作按钮约定

> 目标：统一页面级操作入口的布局方式，避免操作按钮散落在搜索栏或列表区域，保持页面简洁。

| 场景                                       | 布局方式                                              |
| ------------------------------------------ | ----------------------------------------------------- |
| 页面仅一个独立功能（如新增）               | 右下角 `GbFabButton` 直接触发                         |
| 页面有多个独立功能（如新增 + 测试 + 导出） | 右下角 `GbFabButton` + `uv-action-sheet` 二级菜单选择 |

**实现要点（多功能场景）**：

```vue
<!-- FAB + ActionSheet -->
<GbFabButton @click="fabActionSheetRef?.open()" />
<uv-action-sheet
  ref="fabActionSheetRef"
  :actions="fabActions"
  @select="handleFabAction"
></uv-action-sheet>
```

```ts
const fabActionSheetRef = ref()
const fabActions = [
  { name: '测试功能', id: 'test' },
  { name: '新增', id: 'add' }
]
const handleFabAction = (item: { id: string }) => {
  if (item.id === 'add') openAdd()
  else openTest()
}
```

| ✅ 正确                                     | ❌ 错误                    |
| ------------------------------------------- | -------------------------- |
| 页面级操作统一收口到 FAB 或 FAB+ActionSheet | 在搜索栏旁堆叠多个操作按钮 |
| 单功能直接 FAB 触发                         | 单功能也套 ActionSheet     |
| 多功能用 `uv-action-sheet` 原生组件         | 手写浮动子菜单/遮罩/动画   |

### 枚举复用约定

> 目标：避免页面重复定义枚举，统一维护在 `useCommonSelectTypes` 中，支持多语言切换。

| ✅ 正确                                                           | ❌ 错误                                              |
| ----------------------------------------------------------------- | ---------------------------------------------------- |
| 枚举优先从 `useCommonSelectTypes` 解构使用                        | 页面内重复定义已有的枚举（如 saasLevel、filterType） |
| hook 中不存在的枚举才在页面内新建                                 | 不检查 hook 直接新建                                 |
| `uni-data-select` 需要的 `{ text, value }` 格式通过 computed 转换 | 直接改 hook 的返回结构                               |
| label 显示统一用 `handleTypeForLabel(value, list)` 查找           | 页面内手写 `if (x === 1) return '白名单'`            |

**转换示例**：

```ts
const { saasLevel, handleTypeForLabel } = useCommonSelectTypes()

// hook 返回 { label, value }，uni-data-select 需要 { text, value }
const saasLevelOptions = computed(() =>
  saasLevel.value.map((item) => ({ value: item.value, text: item.label }))
)

// label 查找
const saasLevelLabel = (level?: number) =>
  handleTypeForLabel(level, saasLevel.value, '-')
```

### 交互规范约定

| ✅ 正确                                                             | ❌ 错误                        |
| ------------------------------------------------------------------- | ------------------------------ |
| `onPullDownRefresh` → 重置页码 → 请求 → `uni.stopPullDownRefresh()` | 下拉刷新只调接口不重置页码     |
| `onReachBottom` → 追加数据 → `hasMore` 控制                         | 上拉加载不判断是否有更多       |
| `loading` ref 控制骨架屏/加载动画                                   | 无加载状态反馈                 |
| 列表为空展示空状态组件                                              | 空白页面                       |
| `try/catch` + `uni.showToast` 错误提示                              | 无错误处理                     |
| 提交前逐项校验，`uni.showToast` 提示具体字段                        | 前端不校验直接提交             |
| 危险操作 `uni.showModal` 二次确认                                   | 删除操作无确认                 |
| Pinia Store 管理共享状态                                            | `uni.$emit`/`uni.$on` 全局事件 |
| 从 `@/api/` 导入类型                                                | 页面内重复定义类型             |

> 编码原则见 [coding-principles.md](references/coding-principles.md)
> 设计规范见 [md-design-spec.md](references/md-design-spec.md)
> 开发规范见 [md-dev-standards.md](references/md-dev-standards.md)

## 工作流程

### Phase 0: 需求确认

| 确认项                | 启发式问题                                      |
| --------------------- | ----------------------------------------------- |
| 页面清单 + 复杂度分类 | "根据PRD，识别到N个页面\[列出]，是否有遗漏？"   |
| 角色权限映射          | "各页面分别属于哪个角色（root/ops/saas/mch）？" |
| 平台适配              | "需要支持哪些平台？默认微信小程序+H5"           |
| 定制页面              | "除标准CRUD页面外，还有哪些定制页面？"          |

**页面分类决策**：

| 分类     | 条件                             | 代码策略                 |
| -------- | -------------------------------- | ------------------------ |
| 简单页面 | 仅标准CRUD（列表/详情/表单）     | gencode 产出，裁剪即可   |
| 复杂页面 | 含特殊交互、多表联动、自定义布局 | 基于生成器页面改造或新建 |

### Phase 1: 架构蓝图

**输出两个文件**：

| 文件        | 定位                   | 内容                                                                          |
| ----------- | ---------------------- | ----------------------------------------------------------------------------- |
| `README.md` | 架构蓝图（给人+AI 读） | 页面总览、角色权限映射、PRD功能点映射、路由设计、字段一致性检查、平台适配策略 |
| `TASKS.md`  | 进度清单（仅追踪）     | 页面清单（简单/复杂分类）、状态复选框                                         |

README.md 必须包含：

| 章节                                              | 必要性 |
| ------------------------------------------------- | ------ |
| 页面总览（页面清单、复杂度分类、涉及API）         | 必须   |
| 角色权限映射（root/ops/saas/mch × 页面访问矩阵）  | 必须   |
| PRD功能点映射（功能点 → 模块 → 页面）             | 必须   |
| 路由设计（pages.json 页面路由配置）               | 必须   |
| 字段一致性检查（表单字段 ↔ API Schema字段对照表） | 必须   |
| 平台适配（微信小程序/H5/App 适配策略）            | 必须   |
| 组件清单（复用组件列表）                          | 按需   |

TASKS.md 内容结构：页面分类（简单/复杂）+ 并行分组 + 进度复选框。模板见 [design-templates.md](references/design-templates.md)

### Phase 2: 逐页面完整交付

按 TASKS.md 顺序，**逐页面串行交付**。每个页面执行以下步骤：

#### Step 1: 加载上下文

| 操作          | 说明                                                            |
| ------------- | --------------------------------------------------------------- |
| 读 API Schema | 读取 `src/api/` 中对应的 TypeScript interface，记录字段名和类型 |
| 对照 PRD      | 确认 PRD 字段需求与 API 字段名一一对应                          |
| 关注类型      | 状态/类型字段是 `string` 还是 `number`                          |

| ❌ 常见错误                    | ✅ 正确做法                                             |
| ------------------------------ | ------------------------------------------------------- |
| 凭 PRD 描述猜测字段名          | 从 `export interface Xxx { }` 读取实际字段名            |
| 状态值用 string（如 `'通过'`） | 状态值用 number（如 `0`、`1`），对照 interface 确认类型 |

#### Step 2: 创建页面

| 策略              | 条件                               | 操作                                  |
| ----------------- | ---------------------------------- | ------------------------------------- |
| 裁剪 gencode 页面 | gencode 生成了对应页面且质量可接受 | 调整字段、样式、交互                  |
| 基于类型新建      | gencode 未生成页面，或质量差       | 导入 gencode API 类型和函数，从零创建 |

**页面结构**：列表页(搜索+筛选+数据列表+悬浮新增) / 详情页(信息展示+操作按钮) / 表单页(字段输入+底部提交)

#### Step 3: 按角色移动页面

| 操作         | 说明                                                                                        |
| ------------ | ------------------------------------------------------------------------------------------- |
| 角色移动     | 根据 README.md 角色权限映射，将 `{module}/` 从 `pages/` 移动到 `admin/`、`mch/`、`saas/` 等 |
| 修正导入路径 | 更新页面内的组件导入路径                                                                    |

#### Step 4: 配置路由

| 内容     | 说明                                                                                                                      |
| -------- | ------------------------------------------------------------------------------------------------------------------------- |
| 通用页面 | `pages/home/index`、`pages/login/index` 等，放入 `pages.json` 的 `pages` 节点                                             |
| 业务页面 | `packages/{module}/{role}/{Feature}/{Entity}/index`，放入 `pages.json` 的 `subPackages` 节点                              |
| 导航栏   | 业务页面顶部必须引入 `GbNavbar` 自定义导航栏，标题使用 i18n 文案；`pages.json` 中可配置 `navigationBarTitleText` 作为兜底 |
| TabBar   | 当前项目固定 3 个入口：管理菜单、常用工具、我的；其余页面不使用 TabBar                                                    |

#### Step 5: API 完整对接

> **一次写完，不建空壳**。页面脚本直接包含完整 API 调用逻辑，无 TODO 标记。

| 场景     | 代码模式                                                                                                   |
| -------- | ---------------------------------------------------------------------------------------------------------- |
| 列表页   | `const res = await root{Feature}{Entity}List({ stateGte: 0, $pg: 1, $rn: 20 })` → `res.data?.list \|\| []` |
| 详情页   | `const res = await root{Feature}{Entity}Load({ id })` → `res.data ?? null`                                 |
| 表单提交 | `await root{Feature}{Entity}Create({ data: form.value })` → `uni.showToast` + `uni.navigateBack`           |

> **列表查询默认状态过滤**：列表接口的 `queryParams` 默认带 `stateGte: 0`，过滤掉 `state = -1`（已删除）的数据，仅展示上线（1）与下线（0）。若业务需展示全部（含已删除），再显式覆盖。

| ❌ 错误写法        | ✅ 正确写法                                                |
| ------------------ | ---------------------------------------------------------- |
| 直接 `uni.request` | 通过 `@/api/` 层调用                                       |
| `res.data?.rows`   | `res.data?.list`（`res.data?.results` 仅兼容旧代码）       |
| 前端不校验直接提交 | 提交前校验，`if (!form.name.trim()) { showToast; return }` |

**权限交互**：

| 场景     | 实现                                                               |
| -------- | ------------------------------------------------------------------ |
| 按钮权限 | `v-if="usePermission('save')"`（短码基于当前页面路径自动拼接）     |
| 页面权限 | `onLoad` 中检查 `menuStore.hasMenu(path)`，无权限则 `navigateBack` |
| 数据权限 | API 自动按用户角色过滤，前端无需额外处理                           |

**权限短码规则**：`usePermission(shortCode)` 的短码 **= 接口 URL 路径最后一段**（后端权限码约定）。

- 示例：接口 `rootMscMscPermUpdate` → URL `/uw-auth-center/root/msc/mscPerm/update` → 最后一段 `update` → 短码使用 `usePermission('update')`
- 示例：接口 `rootAccountGroupSave` → URL `/uw-auth-center/root/account/group/save` → 短码 `save`
- 示例：接口 `rootAccountGroupList` → URL `/uw-auth-center/root/account/group/list` → 短码 `list`

**标准操作权限短码**（禁止自定义命名，严格与后端接口最后一段一致）：

| 短码      | 对应接口后缀               | 用法示例                                                       |
| --------- | -------------------------- | -------------------------------------------------------------- |
| `save`    | `/xxx/save`（新增/保存）   | `v-if="usePermission('save')"` 控制悬浮新增按钮                |
| `update`  | `/xxx/update`（编辑/修改） | `v-if="item.state === 1 && usePermission('update')"` 控制编辑  |
| `delete`  | `/xxx/delete`（删除）      | `v-if="item.state === 0 && usePermission('delete')"` 控制删除  |
| `enable`  | `/xxx/enable`（启用）      | `v-if="item.state === 0 && usePermission('enable')"` 控制启用  |
| `disable` | `/xxx/disable`（禁用）     | `v-if="item.state === 1 && usePermission('disable')"` 控制禁用 |
| `list`    | `/xxx/list`（列表查询）    | `v-if="usePermission('list')"` 控制入口/查询按钮               |
| `load`    | `/xxx/load`（详情查看）    | `v-if="usePermission('load')"` 控制查看按钮                    |

> ❌ 禁止使用：`add`（用 `save`）、`edit`（用 `update`）、`remove`（用 `delete`）、`view`/`detail`（用 `load`）等自定义短码。判断标准是"和后端接口路径最后一段完全一致"。

#### Step 6: 平台适配

| 适配项   | 微信小程序                          | H5           | App                           |
| -------- | ----------------------------------- | ------------ | ----------------------------- |
| 条件编译 | `#ifdef MP-WEIXIN`                  | `#ifdef H5`  | `#ifdef APP-PLUS`             |
| 登录     | `uni.login({ provider: 'weixin' })` | 账号密码登录 | `plus.oauth.getServices()`    |
| 扫码     | `uni.scanCode()`                    | 不支持       | `uni.scanCode()`              |
| 存储     | `uni.setStorageSync`                | localStorage | `plus.io`                     |
| 样式单位 | rpx（750rpx = 屏幕宽度）            | px/rpx 均可  | px/rpx 均可                   |
| 安全区   | `env(safe-area-inset-bottom)`       | 不需要       | `env(safe-area-inset-bottom)` |

> 完整平台适配规范见 [md-dev-standards.md](references/md-dev-standards.md)

#### Step 7: 业务组件 + 状态管理

| 组件类型 | 位置          | 示例                                 |
| -------- | ------------- | ------------------------------------ |
| 组件     | `components/` | GbNavbar, ZIcon, GbLoading, GbEchart |

**图表组件 `GbEchart`**：

- 位置：`src/components/GbEchart/index.vue`
- 基于 `lime-echart` + `echarts` 封装，支持 H5 / App / 微信小程序
- H5 / App 端使用 npm 包 `echarts` 初始化；小程序端依赖 `src/uni_modules/lime-echart`（需从 DCloud 插件市场导入）
- 使用方式：

```vue
<script setup>
import GbEchart from '@/components/GbEchart/index.vue'
const option = {
  xAxis: { type: 'category', data: ['Mon', 'Tue', 'Wed'] },
  yAxis: { type: 'value' },
  series: [{ data: [120, 200, 150], type: 'line' }]
}
</script>

<template>
  <GbEchart
    :option="option"
    width="100%"
    height="500rpx"
  />
</template>
```

- 注意：小程序端运行前必须手动导入 `lime-echart` 到 `src/uni_modules/lime-echart`，否则图表无法渲染

| Store           | 职责                            |
| --------------- | ------------------------------- |
| `store/main.ts` | 全局状态、菜单权限、按钮权限码  |
| `store/user.ts` | Token 管理、登录/登出、用户信息 |

Pinia 使用 setup 风格：`defineStore('name', () => { ... })`，State 用 ref，Getters 用 computed，Actions 为普通函数。

#### Step 8: 测试驱动开发（Red-Green 内部循环）

> **AI 原生 TDD**：内部自动执行 Red-Green 循环，用户只看到最终通过的结果。详见 [tdd-guide.md](../0-init/references/tdd-guide.md)。

**8.1 Red 阶段**：

- 为 composable/Store/工具函数编写测试代码
- 执行 `pnpm vitest run tests/hooks/useXxx.spec.ts` → **确认测试失败**
- ⚠️ 如果测试意外通过 → 说明断言不够严格，需加强

**8.2 Green 阶段**：

- 编写实现代码
- 执行 `pnpm vitest run tests/hooks/useXxx.spec.ts` → **确认测试通过**

**8.3 Refactor 阶段**（按需）：

- 优化代码结构
- 执行 `pnpm vitest run` → 确认仍然通过

| 测试对象 | 测试内容               | 文件位置                     |
| -------- | ---------------------- | ---------------------------- |
| hooks    | 业务逻辑计算、数据转换 | `tests/hooks/useXxx.spec.ts` |
| Store    | 状态变更、异步 action  | `tests/store/xxx.spec.ts`    |
| 工具函数 | 纯函数输入输出         | `tests/utils/xxx.spec.ts`    |

| ❌ 不测试    | ✅ 测试             |
| ------------ | ------------------- |
| 页面组件渲染 | composable 业务逻辑 |
| UI 样式      | Store 状态变更      |
| 框架 API     | 工具函数边界值      |

#### Step 9: 页面验证

```bash
pnpm build:h5 && pnpm build:mp-weixin
```

**检查项**：

| 检查项                     | 通过标准                                                                                |
| -------------------------- | --------------------------------------------------------------------------------------- |
| H5 编译                    | 通过                                                                                    |
| 微信小程序编译             | 通过                                                                                    |
| `res.data?.rows` 误用      | `grep -rn 'res\.data\?\.rows\b' src/ --include="*.vue" --include="*.ts"` 命中 0 行      |
| 旧代码 `res.data?.results` | 新项目应使用 `res.data?.list`；旧代码兼容保留，逐步迁移                                 |
| `ref` 双重调用             | `grep -rn 'ref<.*>(.*)(.*)' src/ --include="*.vue" --include="*.ts"` 命中 0 行          |
| 直接 `uni.request`         | `grep -rn 'uni\.request(' src/pages/ --include="*.vue"` 命中 0 行                       |
| Tailwind 强制使用          | 本次改动的新增/修改页面中，简单样式必须使用 Tailwind 原子类；禁止为简单样式新建 SCSS 类 |
| 页面可访问                 | 启动 dev 服务器验证                                                                     |

**全部通过后**，在 TASKS.md 中标记该页面为已完成，进入下一个页面。

> 代码模板见 [code-templates.md](references/code-templates.md)
> 开发示例见 [dev-examples.md](references/dev-examples.md)

## 完成标准

- [ ] README.md 覆盖所有页面和角色权限映射
- [ ] TASKS.md 包含所有页面，全部标记完成
- [ ] 所有 PRD 功能点都有对应页面
- [ ] 表单字段名与 API Schema 字段名一致
- [ ] 列表数据使用 `res.data?.list`（`res.data?.results` 仅兼容旧代码）
- [ ] 状态/类型字段使用 number 值
- [ ] 通用页面放在 `src/pages/`，业务页面按 `src/packages/{module}/{role}/{Feature}/{Entity}/index.vue` 组织
- [ ] pages.json 路由配置正确
- [ ] 无 `any` 类型（框架限制除外）
- [ ] 下拉刷新/上拉加载/空状态/错误处理完善
- [ ] 权限控制（按钮+页面级）正确
- [ ] 平台条件编译覆盖所有目标平台
- [ ] hooks/Store/工具函数单元测试全绿
- [ ] H5 和微信小程序编译通过
- [ ] 无 `ref<...>(null)(null)` 双重调用

## ⚠️ 完成验证（强制，全自动执行）

1. **强制执行编译验证**（全量检查项）
2. **强制调用** `221-admin-uni-dev-review`
3. 不通过（< 95）→ 自动修复 → 重新评审（最多 5 轮）
4. 通过（≥ 95）→ 向用户报告最终结果

> **全自动执行：中间不暂停、不询问、不汇报。未通过前禁止结束。**

## 参考

- [编码原则](references/coding-principles.md) - 四条核心原则 + 自动化验证
- [设计规范](references/md-design-spec.md) - 移动端设计规范
- [开发规范](references/md-dev-standards.md) - Vue3+TS 多端开发规范
- [设计模板](references/design-templates.md) - README + TASKS.md 模板
- [代码模板](references/code-templates.md) - 列表页/详情页/表单页模板
- [开发示例](references/dev-examples.md) - API 对接、平台适配示例
- [评审技能](../221-admin-uni-dev-review/SKILL.md)
