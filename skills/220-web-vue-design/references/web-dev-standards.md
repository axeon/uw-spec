# Web 后台管理系统开发规范（Vue3 + TypeScript + Vite + Element Plus）

> 适用于 SaaS 后台管理系统开发。被 220-web-vue-design 和 320-web-vue-dev 引用。

## 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| Vue | 3.x | 前端框架（Composition API） |
| TypeScript | 5.x | 类型安全 |
| Vite | 8.x | 构建工具 |
| Element Plus | 2.x | UI 组件库 |
| Pinia | 2.x | 状态管理 |
| Vue Router | 4.x | 路由 |
| Axios | 1.x | HTTP 客户端 |

## 目录结构规范

```
src/
├── pages/                     # 页面（按角色组织）
│   ├── admin/{module}/
│   ├── mch/{module}/
│   ├── saas/{module}/
│   └── guest/{module}/
├── components/                # 组件
│   ├── business/              # 业务组件
│   └── common/                # 通用组件
├── api/                       # API 调用封装
│   └── {module}/index.ts
├── types/                     # TypeScript 类型定义
│   └── {module}.ts
├── stores/                    # Pinia 状态管理
│   └── {module}.ts
├── composables/               # 组合函数
│   └── use{Feature}.ts
├── utils/                     # 工具函数
│   ├── request.ts             # Axios 封装
│   └── auth.ts                # 登录/Token 管理
├── router/                    # 路由配置
│   └── index.ts
├── styles/                    # 全局样式
├── App.vue
└── main.ts
```

## 后台管理系统布局规范

| 布局区域 | 组件 | 说明 |
|---------|------|------|
| 侧边栏 | `el-menu` | 根据角色动态生成菜单，支持折叠模式 |
| 顶栏 | 自定义组件 | 面包屑导航 + 用户信息下拉 + 退出登录 |
| 内容区 | `<router-view>` | 路由出口，搭配 `<keep-alive>` 缓存页面状态 |
| 标签页 | `el-tabs`（可选） | 多页签模式，支持关闭和右键菜单 |

**布局结构**：
```
┌─────────────────────────────────────────┐
│  顶栏（面包屑 + 用户信息）               │
├──────────┬──────────────────────────────┤
│  侧边栏  │  内容区（<router-view>）      │
│  el-menu │  带 <keep-alive> 缓存        │
│  可折叠   │                              │
└──────────┴──────────────────────────────┘
```

**布局组件位置**：`src/components/layout/AdminLayout.vue`

**角色菜单**：侧边栏菜单根据用户角色从路由配置中过滤生成，路由 meta 中定义 `roles` 字段。

## 页面编码规范

### SFC 结构顺序

```vue
<template>...</template>
<script setup lang="ts">...</script>
<style scoped lang="scss">...</style>
```

### 命名规范

| 类型 | 规范 | 示例 |
|------|------|------|
| 页面文件 | PascalCase | `ProductList.vue` |
| 组件文件 | PascalCase | `ProductSelector.vue` |
| composables | use 前缀 | `usePagination.ts` |
| Store 文件 | kebab-case | `cart.ts` |
| API 文件 | kebab-case | `product.ts` |
| 类型文件 | kebab-case | `product.ts` |
| 路由路径 | `/{角色}/{模块}/{操作}` | `/admin/product/list` |

### `<script setup>` 规范

```typescript
// 1. 类型导入
import type { FormInstance, FormRules } from 'element-plus'
import type { ProductInfo, ProductQueryParam } from '@/types/product'

// 2. 组件导入
import ProductSelector from '@/components/business/ProductSelector.vue'

// 3. API 导入
import { listProduct, deleteProduct } from '@/api/product'

// 4. composables
const { loading, run } = useLoading()

// 5. 响应式状态
const tableData = ref<ProductInfo[]>([])
const formRef = ref<FormInstance>()

// 6. 方法定义
const fetchList = async () => { ... }

// 7. 生命周期
onMounted(fetchList)
```

### Props & Emits 规范

```typescript
const props = defineProps<{
  productId?: number
  visible: boolean
}>()

const emit = defineEmits<{
  update: [id: number]
  close: []
}>()

// v-model 使用 computed get/set
const modelValue = computed({
  get: () => props.visible,
  set: (val) => emit('update:visible', val)
})
```

## Element Plus 使用规范

### 表格（el-table）

| 规范 | 说明 |
|------|------|
| 列定义 | 使用 `prop` 绑定字段名（与后端 DTO 一致） |
| 自定义列 | 使用 `#default="{ row }"` 插槽 |
| 操作列 | 固定右侧 `fixed="right"` |
| 加载状态 | 使用 `v-loading` 指令 |
| 分页 | 使用 `el-pagination`，配合 `@current-change` |

### 表单（el-form）

| 规范 | 说明 |
|------|------|
| 校验规则 | 使用 `FormRules` 类型定义 |
| 字段名 | `prop` 与后端 DTO 字段名一致 |
| 提交 | `@Valid` 对应 `formRef.value.validate()` |
| 标签宽度 | 统一 `label-width="120px"` |

### 消息提示

| 场景 | 组件 |
|------|------|
| 操作成功 | `ElMessage.success('保存成功')` |
| 操作确认 | `ElMessageBox.confirm('确认删除？', '提示', { type: 'warning' })` |
| 表单校验 | Element Plus 内置校验提示 |

## 网络请求规范（Axios）

> **Token 安全说明**：Token 存储在 localStorage，适用于 SaaS 后台管理系统内网场景。如需更高安全性，考虑 HttpOnly Cookie 方案。

```typescript
// src/utils/request.ts
import axios from 'axios'
import { ElMessage } from 'element-plus'
import router from '@/router'
import type { ResponseData } from '@/types/common'

const request = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL,
  timeout: 30000
})

request.interceptors.request.use((config) => {
  const token = localStorage.getItem('token')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

request.interceptors.response.use(
  (response) => {
    const data = response.data as ResponseData<unknown>
    if (data.code === 200) {
      return data.data
    }
    ElMessage.error(data.message || '请求失败')
    return Promise.reject(new Error(data.message))
  },
  (error) => {
    if (error.response?.status === 401) {
      router.push('/login')
    }
    return Promise.reject(error)
  }
)
```

## 路由规范

| 规范 | 说明 |
|------|------|
| 路径格式 | `/{角色}/{模块}/{操作}` |
| 角色守卫 | 路由 meta 配置 `roles` 字段 |
| 懒加载 | 页面组件使用 `() => import()` |
| 导航 | 侧边栏根据角色动态生成 |

```typescript
// 路由配置
{
  path: '/admin/product',
  component: () => import('@/pages/admin/product/List.vue'),
  meta: { title: '商品列表', roles: ['admin'] }
}
```

## 状态管理规范（Pinia）

使用 Pinia setup 风格：`defineStore('name', () => { ... })`，State 用 ref，Getters 用 computed，Actions 为普通函数。

```typescript
export const useUserStore = defineStore('user', () => {
  const token = ref<string>('')
  const userInfo = ref<UserInfo | null>(null)

  const isLoggedIn = computed(() => !!token.value)

  const login = async (credentials: LoginForm) => {
    const res = await authApi.login(credentials)
    token.value = res.token
    userInfo.value = res.user
    localStorage.setItem('token', res.token)
  }

  const logout = () => {
    token.value = ''
    userInfo.value = null
    localStorage.removeItem('token')
  }

  return { token, userInfo, isLoggedIn, login, logout }
})
```

## 字段一致性原则

| 层级 | 命名规范 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `product_name` |
| 后端 DTO 字段 | camelCase | `productName` |
| 前端表单字段 | camelCase（与 DTO 一致） | `productName` |
| 前端表格列 | camelCase（与 DTO 一致） | `productName` |
| API 参数 | camelCase（与 DTO 一致） | `productName` |

## 禁用规则

| 禁用项 | 说明 |
|--------|------|
| `any` 类型 | 禁止使用，必须定义具体类型 |
| Options API | 统一使用 Composition API + `<script setup>` |
| jQuery | 禁止引入 |
| 直接 DOM 操作 | 使用 Vue 响应式 |
| `v-html` 渲染用户输入 | 禁止，存在 XSS 风险。使用 `v-text` 或 Element Plus 富文本组件 |

## 性能规范

| 场景 | 规范 |
|------|------|
| 表格数据 | 分页查询，单页 ≤ 50 条 |
| 组件懒加载 | 大组件使用 `defineAsyncComponent` |
| 图片 | 使用 CDN + 懒加载 |
| 首屏 | 路由懒加载，减少首屏体积 |
| 请求 | 并行请求使用 `Promise.all` |
