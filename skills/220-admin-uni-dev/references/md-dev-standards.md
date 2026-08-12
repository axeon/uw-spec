# 管理端UniApp移动端开发规范（Vue3 + TypeScript）

> 面向管理人员和运营人员的移动端应用（移动审批、商户管理、运营后台小程序）。被 220-admin-uni-dev 引用。

## 技术栈

| 技术       | 版本                   | 用途                        |
| ---------- | ---------------------- | --------------------------- |
| UniApp     | 3.0.0-4040520250104002 | 跨平台框架                  |
| Vue 3      | 3.5.x                  | 前端框架（Composition API） |
| TypeScript | 5.8.x                  | 类型安全                    |
| Pinia      | 2.3.x                  | 状态管理                    |
| Vite       | 5.4.x                  | 构建工具                    |
| uni-ui     | 1.5.x                  | 官方 UI 组件库              |
| uv-ui      | 1.1.x                  | 第三方 UI 组件库            |
| vue-i18n   | 9.1.x                  | 国际化                      |

## 目录结构规范

> 以当前项目 `uw-uniapp-template-main` 实际目录为标准。

```
src/
├── api/                       # API 调用封装
│   ├── request/               # 网络请求封装
│   ├── type/                  # 通用 API 类型
│   └── *.ts                   # 按业务生成的扁平 API 文件
├── components/                # 组件（扁平组织，不区分 common/business）
├── config/                    # 配置文件
│   ├── appVersion.ts
│   ├── common.ts
│   ├── EnumConst.ts
│   ├── EnumLabelMap.json
│   ├── icon.ts
│   └── theme.ts
├── hooks/                     # 组合函数
│   ├── useAuthGuard.ts
│   ├── useCommonSelectTypes.ts
│   ├── useCountdown.ts
│   └── usePermission.ts
├── i18n/                      # 国际化
│   ├── zh-CN/
│   ├── en/
│   ├── ja/
│   ├── zh-TW/
│   ├── dayjsLocale.ts
│   ├── i18n.type.ts
│   ├── index.ts
│   └── instance.ts
├── packages/                  # 分包页面
│   ├── about/                 # 通用页面：关于
│   ├── resetPassword/         # 通用页面：重置密码
│   ├── userInfo/              # 通用页面：个人资料
│   └── {module}/              # 业务模块
│       └── {role}/            # admin / mch / saas
│           └── {Feature}/     # 功能模块
│               └── {Entity}/  # 实体/页面
│                   └── index.vue
├── pages/                     # 通用页面（不按角色分层）
│   ├── home/                  # 工作台首页
│   ├── login/                 # 登录
│   ├── mine/                  # 我的
│   └── tools/                 # 常用工具
├── static/                    # 静态资源
├── store/                     # Pinia 状态管理
│   ├── index.ts               # Pinia 实例
│   ├── main.ts                # 全局状态
│   └── user.ts                # 用户状态
├── styles/                    # 全局样式
├── types/                     # 类型声明
├── utils/                     # 工具函数
├── App.vue
├── main.ts
├── manifest.json
├── pages.json
└── uni.scss
```

### 目录约定说明

1. **`src/pages/` 为通用页面目录**：`home`、`login`、`mine`、`tools` 属于通用入口页面，保持现有结构，不按角色分层。
2. **`src/packages/` 下通用页面保留**：`about/`、`resetPassword/`、`userInfo/` 为通用页面，保持现有结构不变。
3. **新增业务模块按角色组织**：后续新增管理端业务模块，统一按 `src/packages/{module}/{role}/{Feature}/{Entity}/index.vue` 组织，其中 `{role}` 为 `admin`/`mch`/`saas`，`{Feature}` 为功能模块，`{Entity}` 为实体/页面。  
   示例：`src/packages/uwAuthCenter/root/account/group/index.vue`（`uwAuthCenter` 模块 → `root` 角色 → `account` 功能 → `group` 实体页面）。
4. **组件目录保持扁平**：组件统一放在 `src/components/`，不强制区分 `common/` 与 `business/` 子目录。
5. **组合函数目录为 `src/hooks/`**：组合函数统一放在 `src/hooks/`，命名保持 `useXxx.ts`。
6. **Store 文件以项目现有为准**：`src/store/main.ts`、`src/store/user.ts` 作为标准，不强制改为 `auth.ts`/`menu.ts`。
7. **API 文件保持扁平**：API 文件按当前项目实际存在形式为准，如 `authUwAuthCenterApiAuth.ts`、`uwAuthCenterRootApi.ts` 等，不强制按模块分子目录。
8. **工具函数以项目实际存在为准**：`src/utils/` 下工具函数按当前项目实际文件组织。

### 本项目特殊约定

> 基于当前项目 `uw-uniapp-template-main` 的实际情况，以下约定优先于通用规范。

1. **TabBar 固定 3 个入口**：当前项目底部导航固定为「管理菜单」、「常用工具」、「我的」，分别对应 `pages/home/index`、`pages/tools/index`、`pages/mine/index`。
2. **列表数据统一使用 `res.data?.list`**：请求层已在 `src/api/request/index.ts` 中完成 `results -> list` 兼容，新开发页面统一通过 `res.data?.list` 获取列表数据；`res.data?.results` 仍可兼容旧代码，但新项目不推荐继续使用。

## 页面布局模式

管理端移动端以**导航栈模式**为主，当前项目因存在固定 TabBar，故底部保留 3 个入口：

```
┌─────────────────────┐
│  ← 页面标题          │  ← navigationBar（顶部导航栏）
├─────────────────────┤
│                     │
│    内容区域          │  ← 页面主体
│                     │
├─────────────────────┤
│  [搜索筛选]          │  ← 列表页顶部搜索/筛选区（可选）
│                     │
│  ┌───────────────┐  │
│  │ 数据项 1      │  │  ← 列表/卡片
│  │ 状态标签 时间  │  │
│  └───────────────┘  │
│  ┌───────────────┐  │
│  │ 数据项 2      │  │
│  └───────────────┘  │
│                     │
│        [+ FAB]      │  ← 新增按钮（固定悬浮）
└─────────────────────┘
```

## 页面编码规范

### SFC 结构顺序

```vue
<template>...</template>
<script setup lang="ts">
...
</script>
<style scoped lang="scss">
...
</style>
```

### 命名规范

| 类型         | 规范                                          | 示例                                                     |
| ------------ | --------------------------------------------- | -------------------------------------------------------- |
| 页面文件     | `index.vue`（页面目录使用 kebab-case）        | `src/packages/uwAuthCenter/root/account/group/index.vue` |
| 组件文件     | PascalCase                                    | `GbNavbar/index.vue`                                     |
| hooks        | use 前缀                                      | `usePermission.ts`                                       |
| Store 文件   | kebab-case                                    | `main.ts`、`user.ts`                                     |
| API 文件     | kebab-case                                    | `uwAuthCenterRootApi.ts`                                 |
| 类型文件     | kebab-case                                    | `API_TYPE.ts`                                            |
| 业务页面路径 | `packages/{module}/{role}/{Feature}/{Entity}` | `packages/uwAuthCenter/root/account/group`               |

### `<script setup>` 规范

```typescript
// 1. 类型导入（从 api 层导入）
import type { ProductInfo, ProductQueryParam } from '@/api/uwAuthCenterRootApi'

// 2. 组件导入
import GbNavbar from '@/components/GbNavbar/index.vue'
import ZIcon from '@/components/ZIcon/index.vue'

// 3. API 导入
import { rootProductList } from '@/api/uwAuthCenterRootApi'

// 4. Store 导入
import { useMainStore } from '@/store/main'
import { useUserStore } from '@/store/user'

// 5. hooks
import { usePermission } from '@/hooks/usePermission'

// 6. 响应式状态
const listData = ref<ProductInfo[]>([])

// 7. 计算属性
const filteredList = computed(() => listData.value.filter(...))

// 8. 方法定义
const fetchList = async () => { ... }

// 9. 生命周期
onMounted(() => fetchList())
```

### Props & Emits 规范

```typescript
const props = defineProps<{
  product: ProductInfo;
  showStatus?: boolean;
}>();

const emit = defineEmits<{
  click: [product: ProductInfo];
  statusChange: [productId: number, status: string];
}>();
```

## 本项目专属实践规范

### i18n 使用

- 页面/组件中所有面向用户的文案必须使用 `vue-i18n` 提供的 `t()` 函数，**禁止硬编码中文/英文**。

#### 多语言文件组织

`src/i18n/` 为项目多语言文件存放目录，按语言划分子目录（`zh-CN/`、`en/`、`ja/`、`zh-TW/`）。每个语言目录下文件分为两类：

| 文件类型     | 说明                                                   | 示例                                               |
| ------------ | ------------------------------------------------------ | -------------------------------------------------- |
| 通用翻译文件 | 跨模块复用的通用文案，内部采用**扁平化 JSON 结构**组织 | `common.json`、`enumeration.json`、`errorMsg.json` |
| 模块专属文件 | 某业务模块的专属文案，按模块命名                       | `uwAuthCenter.json`、`uwGatewayCenter.json`        |

**三个通用翻译文件职责**：

| 文件               | 职责                                                                      | 示例键                                          |
| ------------------ | ------------------------------------------------------------------------- | ----------------------------------------------- |
| `common.json`      | 通用操作/状态/提示文案（确定、取消、加载中、启用/禁用、增删改成功失败等） | `confirm`、`cancel`、`loading`、`enableSuccess` |
| `enumeration.json` | 枚举值翻译（登录类型、上下架状态、支付方式等）                            | `onShelf`、`offShelf`、`WECHATPay`              |
| `errorMsg.json`    | 后端错误码/错误信息翻译                                                   | `AUTH-0011`、`AccountOrPwdError`                |

通用翻译文件采用扁平化结构，键名直接挂在顶层，通过顶层键直接使用：`t('confirm')`、`t('loading')`、`t('noData')`，**禁止**写成 `t('common.confirm')`。

```json
// src/i18n/zh-CN/common.json（扁平化结构）
{
  "confirm": "确定",
  "cancel": "取消",
  "search": "搜索",
  "loading": "加载中...",
  "noMore": "没有更多了",
  "loadMore": "上拉加载更多",
  "noData": "暂无数据",
  "add": "新增",
  "edit": "编辑",
  "delete": "删除",
  "enable": "启用",
  "disable": "禁用",
  "enableSuccess": "启用成功",
  "deleteSuccess": "删除成功",
  "submitting": "提交中...",
  "remark": "备注",
  "createDate": "创建时间"
}
```

#### 翻译复用流程（强制）

进行翻译工作时，必须按以下顺序，确保翻译内容的一致性和可维护性：

1. **先查通用翻译文件**：从 `common.json`、`enumeration.json`、`errorMsg.json` 中查找是否存在合适的翻译条目。若存在，**直接复用**现有翻译，禁止在模块文件中重复定义。
2. **再写模块专属文件**：若通用文件中不存在，则在对应模块的专属多语言文件中新增翻译条目。
3. **通用文案沉淀**：当某文案在多个模块中重复出现时，应将其提取到 `common.json` 等通用文件中，消除重复。

| ✅ 正确                                                                             | ❌ 错误                                |
| ----------------------------------------------------------------------------------- | -------------------------------------- |
| `t('confirm')` 复用 `common.json` 现有翻译                                          | 在模块文件中重复定义"确定"             |
| `t('loading')`、`t('noData')` 复用通用文案                                          | 每个模块各自定义"加载中..."/"暂无数据" |
| 模块特有文案 `t('/uwAuthCenter/root/account/group/title')` 写入 `uwAuthCenter.json` | 把模块专属文案塞进 `common.json`       |

#### 模块专属文案命名

- **i18n 命名空间统一使用页面路径风格 `/{module}/{role}/{feature}/{entity}/{key}`**（带斜杠、混合 kebab 与 camelCase，与 `usePermission` 权限路径一致）。新增文案按模块写入对应模块文件，例如 `src/i18n/zh-CN/uwAuthCenter.json`：

```json
{
  "uw-auth-center": "MSC管理中心",
  "/uwAuthCenter/root/account/group": "用户组管理",
  "/uwAuthCenter/root/account/group/title": "部门管理",
  "/uwAuthCenter/root/account/group/detailTitle": "部门详情",
  "/uwAuthCenter/root/account/group/addTitle": "新增部门",
  "/uwAuthCenter/root/account/group/editTitle": "编辑部门",
  "/uwAuthCenter/root/account/group/searchPlaceholder": "搜索部门名称",
  "/uwAuthCenter/root/account/group/addBtn": "新增部门",
  "/uwAuthCenter/root/account/group/loadError": "加载失败，请稍后重试",
  "/uwAuthCenter/root/account/group/groupNameLabel": "部门名称",
  "/uwAuthCenter/root/account/group/groupNamePlaceholder": "请输入部门名称",
  "/uwAuthCenter/root/account/group/groupNameRequired": "请输入部门名称",
  "/uwAuthCenter/root/account/group/statusLabel": "状态",
  "/uwAuthCenter/root/account/group/remarkLabel": "备注",
  "/uwAuthCenter/root/account/group/remarkPlaceholder": "请输入备注",
  "/uwAuthCenter/root/account/group/enableConfirm": "确认启用部门「{groupName}」？",
  "/uwAuthCenter/root/account/group/disableConfirm": "确认禁用部门「{groupName}」？",
  "/uwAuthCenter/root/account/group/deleteConfirm": "确认删除部门「{groupName}」？删除后不可恢复",
  "/uwAuthCenter/root/account/group/noData": "暂无部门数据"
}
```

> 通用文案（如 `enableSuccess`、`deleteSuccess`、`remark`、`remarkPlaceholder` 等）已沉淀在 `common.json`，应直接用 `t('enableSuccess')`、`t('remark')` 复用，**禁止**在模块文件重复定义；模块专属文件仅保留该模块独有的文案（如 `noData`:"暂无部门数据"）。

- 页面中使用：

```typescript
import { useI18n } from "vue-i18n";
const { t } = useI18n();

uni.showToast({
  title: t("/uwAuthCenter/root/account/group/loadError"),
  icon: "none",
});
```

> `noPermission` 用于页面级权限守卫提示（位于 `common.json`）。

### 组件复用

> **强制要求**：`src/packages/` 下的所有业务页面，文件顶部必须引入并使用 `GbNavbar` 自定义导航栏组件，标题使用 i18n 文案。

| 场景              | 推荐组件                   | 用法                                                                               |
| ----------------- | -------------------------- | ---------------------------------------------------------------------------------- |
| 顶部导航栏        | `GbNavbar`（业务页面强制） | `<GbNavbar :title="t('/uwAuthCenter/root/account/group/title')" @back="goBack" />` |
| 悬浮新增按钮      | `GbFabButton`              | `<GbFabButton v-if="usePermission('save')" @click="handleAdd" />`                  |
| 图标              | `uv-icon` / `ZIcon`        | 通用图标（箭头/勾选/关闭等）用 `uv-icon`；仅项目自定义 iconfont 才用 `ZIcon`       |
| 全屏加载          | `GbLoading`                | `<GbLoading v-if="loading" />`                                                     |
| 列表空状态        | 使用 `uv-empty`            | `<uv-empty v-if="!loading && !listData.length" mode="list" />`                     |
| 按钮              | `uv-button`                | 统一使用 `uv-button`，便于主题适配                                                 |
| 表单输入          | `uv-input` / `uv-textarea` | 配合 `uv-form` 和 `rules` 使用                                                     |
| 弹窗确认          | `uv-modal`                 | 二次确认、信息提示                                                                 |
| 下拉刷新/上拉加载 | `ZScrollView` / `uv-list`  | 列表页统一封装下拉刷新与上拉加载                                                   |

### 请求参数格式

- 列表/详情/表单等 API 调用直接展开参数对象，**不包装 `{ param: {...} }`**：

```typescript
const res = await rootAccountGroupList({
  $pg: pageNum.value,
  $rn: 20,
  keyword: keyword.value || undefined,
  state: currentFilter.value === "all" ? undefined : currentFilter.value,
});
```

- 创建/更新接口若后端需要 `{ data: form.value }`，则按 API Schema 传入：

```typescript
await rootAccountGroupCreate({ data: form.value });
```

### 表单校验

- **强制**：新项目一律使用 `uv-form` + `uv-form-item` + `rules`，禁止手动逐字段 `if / showToast` 校验。
- 仅当页面**没有表单场景**（如单个搜索框、单字段确认弹窗）时，可用 `uni.showToast` 单点提示。
- 校验规则命名与字段一致，放在页面内：

```typescript
const rules = {
  groupName: [
    {
      required: true,
      message: t("/uwAuthCenter/root/account/group/groupNameRequired"),
      trigger: "blur",
    },
    {
      max: 50,
      message: t("maxLength", { length: 50 }),
      trigger: "blur",
    },
  ],
};
```

- 提交时调用 `uvFormRef.value.validate()`，校验通过后再调用 API。

### 错误处理

- 列表请求 catch 必须展示 `uni.showToast`：

```typescript
} catch (error) {
  console.error('fetchList error', error)
  uni.showToast({ title: t('/uwAuthCenter/root/account/group/loadError'), icon: 'none' })
} finally {
  loading.value = false
}
```

- 表单提交 catch 展示业务错误信息：

```typescript
} catch (error: any) {
  uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
}
```

- 全局网络错误、401/403/498 已在 `src/api/request/index.ts` 统一处理，页面中不再重复拦截。

## 页面生命周期规范

UniApp 页面同时支持 Vue 生命周期和 UniApp 生命周期，需按场景选择：

| 场景                 | 使用钩子                  | 说明                             |
| -------------------- | ------------------------- | -------------------------------- |
| 页面初始化、获取参数 | `onLoad((options) => {})` | 仅页面级可用，options 为页面参数 |
| 页面每次显示         | `onShow(() => {})`        | 从其他页面返回或从后台切回时触发 |
| 页面隐藏             | `onHide(() => {})`        | 跳转其他页面或切后台             |
| DOM 首次就绪         | `onReady(() => {})`       | 仅触发一次，适合获取节点信息     |
| 页面卸载             | `onUnload(() => {})`      | 清理定时器、取消请求             |
| 组件初始化           | `onMounted(() => {})`     | Vue 标准钩子，组件内使用         |
| 监听返回按钮         | `onBackPress(() => {})`   | 返回 `true` 阻止默认返回         |

**使用原则**：

- 页面级逻辑（获取参数、每次刷新数据）用 UniApp 钩子（`onLoad`/`onShow`）
- 组件级逻辑（初始化、销毁）用 Vue 钩子（`onMounted`/`onUnmounted`）
- `onShow` 中避免重复初始化，用标志位或判断数据是否已加载

```typescript
import { onLoad, onShow } from "@dcloudio/uni-app";

onLoad((options) => {
  const id = options.id;
  loadDetail(id);
});

onShow(() => {
  if (needsRefresh.value) {
    fetchList();
    needsRefresh.value = false;
  }
});
```

## 权限管理规范（管理端特有）

### 菜单权限

后端返回当前用户的菜单权限列表，前端动态构建可访问页面。当前项目将菜单和权限统一存放在 `store/main.ts`：

```typescript
// store/main.ts
export const useMainStore = defineStore("main", () => {
  // 当前用户菜单信息
  const appMenu = ref<MscAppPermVo[]>([]);
  // 当前用户权限码列表
  const userPermissions = shallowRef<string[]>([]);

  const handleUserLoadAppMenu = async () => {
    const res = await userLoadAppMenu({ appId: 0 });
    if (res.state === "success" && res.data) {
      // 处理并设置权限列表
      await setAppMenu(res.data);
    }
  };

  const setAppMenu = (menuList?: MscAppPermVo[]) => {
    /* 解析菜单，提取 userPermissions */
  };

  return { appMenu, userPermissions, handleUserLoadAppMenu, setAppMenu };
});
```

### 按钮权限

在页面中根据权限码控制按钮显示：

```vue
<button v-if="usePermission('save')" @click="handleAdd">新增</button>
```

或基于当前菜单 path 判断：

```vue
<button
  v-if="usePermission('/uwAuthCenter/root/account/group/save')"
  @click="handleAdd"
>
  新增
</button>
```

**列表页操作按钮组示例**：

```vue
<view class="card-actions">
  <view
    v-if="item.state === 0 && usePermission('enable')"
    class="btn btn-success"
    @click="handleEnable(item)"
  >
    启用
  </view>
  <view
    v-if="item.state === 1 && usePermission('disable')"
    class="btn btn-info"
    @click="handleDisable(item)"
  >
    禁用
  </view>
  <view
    v-if="item.state === 1 && usePermission('update')"
    class="btn btn-primary"
    @click="handleEdit(item)"
  >
    编辑
  </view>
  <view
    v-if="item.state === 0 && usePermission('delete')"
    class="btn btn-danger"
    @click="handleDelete(item)"
  >
    删除
  </view>
</view>
```

**详情页操作按钮示例**：

```vue
<view class="action-bar">
  <uv-button
    v-if="usePermission('update')"
    type="primary"
    :text="t('edit')"
    @click="handleEdit"
  />
  <uv-button
    v-if="usePermission('delete')"
    type="error"
    :text="t('delete')"
    @click="handleDelete"
  />
</view>
```

**权限短码规则**：短码 = 后端接口 URL 路径最后一段。

示例：接口 `rootMscMscPermUpdate` → URL `/uw-auth-center/root/msc/mscPerm/update` → 短码 `update`。

**标准权限短码**（禁止自定义命名）：

- `save` - 新增（对应 `/xxx/save`）
- `update` - 编辑（对应 `/xxx/update`）
- `delete` - 删除（对应 `/xxx/delete`）
- `enable` - 启用（对应 `/xxx/enable`）
- `disable` - 禁用（对应 `/xxx/disable`）
- `list` - 列表查询（对应 `/xxx/list`）
- `load` - 详情查看（对应 `/xxx/load`）

> ❌ 禁止使用 `add` / `edit` / `remove` / `view` / `detail` 等自定义短码。

### 页面级权限守卫

```typescript
// 在页面 onLoad 中检查权限
import { usePermission } from "@/hooks/usePermission";

const { t } = useI18n();

onLoad(() => {
  if (!usePermission("/uwAuthCenter/root/account/group/list")) {
    uni.showToast({ title: t("noPermission"), icon: "none" });
    setTimeout(() => uni.navigateBack(), 1500);
  }
});
```

## 多端适配规范

### 表单校验规范

前端提交前必须校验，不等后端返回错误：

| 校验类型 | 实现方式        | 示例                                                     |
| -------- | --------------- | -------------------------------------------------------- |
| 必填     | `uv-form` rules | `{ required: true, message: t('...'), trigger: 'blur' }` |
| 格式     | 正则校验        | 手机号、邮箱、身份证                                     |
| 长度     | `uv-form` rules | `{ max: 50, message: t('...'), trigger: 'blur' }`        |
| 范围     | 数值范围判断    | 金额 > 0                                                 |
| 一致性   | 双字段比对      | 确认密码与密码一致                                       |

```typescript
const { t } = useI18n();

const rules = {
  groupName: [
    {
      required: true,
      message: t("/uwAuthCenter/root/account/group/groupNameRequired"),
      trigger: "blur",
    },
    {
      max: 50,
      message: t("maxLength", { length: 50 }),
      trigger: "blur",
    },
  ],
};

const handleSubmit = async () => {
  const valid = await uvFormRef.value?.validate?.();
  if (!valid) return;
  // ...
};
```

> ⚠️ 单字段/无表单场景下（如搜索框），才允许使用 `uni.showToast` 单点提示。有表单一律走 `uv-form` + `rules`。

### 条件编译

```typescript
// #ifdef MP-WEIXIN
// 微信小程序专用代码
// #endif

// #ifdef H5
// H5 专用代码
// #endif

// #ifdef APP-PLUS
// App 专用代码（Android/iOS）
// #endif

// #ifndef MP-WEIXIN
// 非微信小程序代码
// #endif
```

### 平台差异对照

| 能力     | 微信小程序                                         | H5                    | App (Android/iOS)              |
| -------- | -------------------------------------------------- | --------------------- | ------------------------------ |
| 登录     | `uni.login({ provider: 'weixin' })` → 后端 wxLogin | 账号密码登录          | `plus.oauth.getServices()`     |
| 扫码     | `uni.scanCode()`                                   | 不支持（需摄像头）    | `uni.scanCode()`               |
| 推送     | 微信模板消息                                       | WebSocket / SSE       | `plus.push.addEventListener()` |
| 存储     | `uni.setStorageSync` / `uni.getStorageSync`        | localStorage          | `plus.io`                      |
| 网络状态 | `uni.getNetworkType()`                             | `navigator.onLine`    | `uni.getNetworkType()`         |
| 文件选择 | `uni.chooseImage()`                                | `<input type="file">` | `plus.gallery.pick()`          |

### 样式框架（Tailwind First · 推荐）

项目已集成 `tailwindcss@^4.3` + `weapp-tailwindcss`，**全平台可用**（H5/微信小程序/App）。样式书写遵循 **Tailwind 优先**原则，**但不做硬性强制** — 以"哪种写法更清晰、更易维护"为最终判断标准。

| 场景                             | 推荐做法                                        | SCSS 依然合适的场景             |
| -------------------------------- | ----------------------------------------------- | ------------------------------- |
| 简单一次性布局（flex/间距/圆角） | Tailwind 原子类                                 | -                               |
| rpx 尺寸/字体                    | 任意值语法：`w-[112rpx] p-[24rpx] text-[26rpx]` | -                               |
| 主题色                           | CSS 变量任意值：`text-[var(--color-primary)]`   | -                               |
| 可复用业务组件（GbNavbar 等）    | -                                               | ✅ SCSS + BEM，外部只暴露 props |
| 5+ 原子类叠加同一元素、可读性差  | -                                               | ✅ 抽 SCSS 类                   |
| `:deep()` 深度选择器覆盖 uv-ui   | -                                               | ✅ 保留在 `<style scoped>` 内   |
| `env(safe-area-inset-bottom)`    | -                                               | ✅ 保留在 `<style scoped>` 内   |
| `@keyframes` 动画                | -                                               | ✅ 保留在 `<style scoped>` 内   |
| 伪元素 `::before` / `::after`    | -                                               | ✅ 保留在 `<style scoped>` 内   |

**不推荐**：

- 使用 Tailwind 内置调色板（`text-blue-500` 等），主题相关必须通过 CSS 变量支持主题切换
- 用 `@apply` 自造 utility 类（复用请抽组件）
- 小程序端使用 `space-x-*` / `divide-*`（后代通配符不稳），改用 `gap-*`

**取舍建议**：

- **简单页面 / 一次性布局** → 优先 Tailwind，减少 `<style>` 冗余
- **可复用业务组件**（如 `GbNavbar`、`GbFabButton`）→ 内部 SCSS，外部通过 props 控制主题，避免使用方需要传大量原子类
- **复杂交互样式**（弹层、动画、深色模式切换、伪元素）→ SCSS 更清晰
- **不必为了 Tailwind 而 Tailwind**：如果一行 SCSS 的语义更清楚，就用 SCSS

**示例**：

场景 A · 简单一次性布局 → **推荐 Tailwind**

```vue
<template>
  <view
    class="flex items-center px-[28rpx] py-[24rpx] bg-white rounded-[20rpx]"
  >
    <text class="text-[32rpx] font-semibold text-[#1a1a1a]">部门名称</text>
    <text class="ml-auto text-[24rpx] text-[var(--color-primary)]">正常</text>
  </view>
</template>
```

场景 B · 可复用业务组件 / 复杂样式 → **推荐 SCSS**

```vue
<template>
  <view class="fab-btn" @click="$emit('click')">
    <text class="fab-btn__icon">+</text>
  </view>
</template>
<style lang="scss" scoped>
.fab-btn {
  position: fixed;
  right: 32rpx;
  bottom: 120rpx;
  width: 112rpx;
  height: 112rpx;
  border-radius: 50%;
  background: linear-gradient(
    135deg,
    var(--color-primary),
    var(--color-primary-secondary)
  );
  box-shadow: 0 8rpx 24rpx rgba(60, 156, 255, 0.4);

  &__icon {
    font-size: 60rpx;
    color: #fff;
  }
}
</style>
```

### 样式单位

| 单位    | 使用场景         | 说明                                |
| ------- | ---------------- | ----------------------------------- |
| `rpx`   | 宽度、间距、字体 | 响应式单位，750rpx = 屏幕宽度       |
| `px`    | 边框、阴影       | 固定尺寸                            |
| `%`     | 弹性布局         | 相对父容器                          |
| `vh/vw` | 全屏布局         | 相对视口（仅 H5/App，小程序不支持） |

### 主题颜色变量

项目支持主题切换功能，**必须使用 CSS 变量定义颜色**，禁止硬编码颜色值。

| 变量名                           | 说明                   | 使用场景                     |
| -------------------------------- | ---------------------- | ---------------------------- |
| `var(--color-primary)`           | 当前主题主色           | 按钮、链接、标签、强调文字   |
| `var(--color-primary-secondary)` | 当前主题渐变辅色       | 渐变背景、装饰性元素         |
| `var(--color-primary-bg)`        | 当前主题色带透明度背景 | 卡片背景、悬停状态、选中状态 |

**正确示例：**

```scss
.tabs__item {
  &--active {
    color: var(--color-primary);
    font-weight: 600;

    &::after {
      background-color: var(--color-primary);
    }
  }
}

.btn--primary {
  background-color: var(--color-primary);
  color: #fff;
}

.card--selected {
  background-color: var(--color-primary-bg);
}
```

**错误示例：**

```scss
// ❌ 禁止硬编码
.tabs__item--active {
  color: #3c9cff;
}
```

### 安全区适配

```css
/* 底部安全区 */
padding-bottom: constant(safe-area-inset-bottom);
padding-bottom: env(safe-area-inset-bottom);

/* 顶部状态栏 */
.status-bar {
  height: var(--status-bar-height);
}
```

## 状态管理规范（Pinia）

使用 Pinia setup 风格：`defineStore('name', () => { ... })`，State 用 ref，Getters 用 computed，Actions 为普通函数。

```typescript
// 认证状态
export const useAuthStore = defineStore("auth", () => {
  const token = ref("");
  const userInfo = ref<UserInfo | null>(null);

  const isLogin = computed(() => !!token.value);

  const login = async (credentials: LoginParams) => {
    const res = await apiLogin(credentials);
    token.value = res.data.token;
    userInfo.value = res.data.userInfo;
    uni.setStorageSync("token", token.value);
  };

  return { token, userInfo, isLogin, login };
});
```

## 网络请求规范

> 网络请求已由 `src/api/request/` 封装，页面中禁止直接使用 `uni.request`。API 调用通过 `src/api/` 层的生成函数完成。

### ResponseData<T> 解析规范

```
列表 API 返回：ResponseData<DataList<T>>
  - 列表数据：res.data?.list       → 类型 T[]（新项目统一使用 list）
  - 兼容取值：res.data?.results    → 类型 T[]（旧代码兼容，不推荐继续使用）
  - 分页总数：res.data?.total

实体 API 返回：ResponseData<T>
  - 实体数据：res.data              → 类型 T

无返回值 API：ResponseData<void>
  - 检查状态：res.state === 'success'
```

### 标准用法

```typescript
import {
  rootAccountGroupList,
  rootAccountGroupLoad,
} from "@/api/uwAuthCenterRootApi";
import type { AccountGroupInfo } from "@/api/uwAuthCenterRootApi";

// 列表页
const res = await rootAccountGroupList({ $pg: 1, $rn: 20 });
const list: AccountGroupInfo[] = res.data?.list || [];

// 详情页
const res = await rootAccountGroupLoad({ id: groupId });
const detail: AccountGroupInfo = res.data!;
```

## 路由配置规范（pages.json）

| 规范     | 说明                                                                |
| -------- | ------------------------------------------------------------------- |
| 路径格式 | `packages/{module}/{role}/{Feature}/{Entity}`                       |
| 导航栏   | 每个页面配置 `navigationBarTitleText`                               |
| TabBar   | 当前项目固定 3 个入口：管理菜单、常用工具、我的；其余页面采用导航栈 |
| 分包     | 主包 ≤ 2MB，超出使用 `subPackages` 分包                             |
| 懒加载   | 非首屏页面使用分包懒加载                                            |

## 字段一致性原则

| 层级          | 命名规范                 | 示例           |
| ------------- | ------------------------ | -------------- |
| 数据库字段    | snake_case               | `product_name` |
| 后端 DTO 字段 | camelCase                | `productName`  |
| 前端表单字段  | camelCase（与 DTO 一致） | `productName`  |
| 前端列表字段  | camelCase（与 DTO 一致） | `productName`  |
| API 参数      | camelCase（与 DTO 一致） | `productName`  |

## 禁用规则

| 禁用项            | 说明                                        |
| ----------------- | ------------------------------------------- |
| `any` 类型        | 禁止使用 `any`，必须定义具体类型            |
| `v-html`          | 小程序不支持，使用 `rich-text` 组件         |
| DOM 操作          | 禁止直接操作 DOM，使用 Vue 响应式           |
| 全局事件总线      | 禁止 `uni.$emit`/`uni.$on`，使用 Pinia 替代 |
| `window/document` | 禁止直接使用，需条件编译包裹                |
| 同步存储          | 大数据使用 `uni.setStorage` 异步版本        |

## 性能规范

| 场景   | 规范                                                     |
| ------ | -------------------------------------------------------- |
| 图片   | 使用 `mode="aspectFill"` / `mode="widthFix"`，启用懒加载 |
| 列表   | 上拉加载分页，单页 ≤ 20 条                               |
| 缓存   | 列表页使用 `uni.setStorageSync` 缓存首屏数据             |
| 分包   | 主包 ≤ 2MB，总包 ≤ 20MB（微信小程序）                    |
| 请求   | 并行请求使用 `Promise.all`，避免瀑布式请求               |
| 页面栈 | 页面栈 ≤ 10 层，超层使用 `uni.reLaunch`                  |
