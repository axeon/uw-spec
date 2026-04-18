# UniApp 移动端开发规范（Vue3 + TypeScript）

> 适用于 UniApp 多端开发，支持 H5 / Android / iOS / 微信小程序。被 220-md-uniapp-design 和 320-md-uniapp-dev 引用。

## 技术栈

| 技术 | 版本 | 用途 |
|------|------|------|
| UniApp | 最新 | 跨平台框架 |
| Vue 3 | 3.x | 前端框架（Composition API） |
| TypeScript | 5.x | 类型安全 |
| Pinia | 2.x | 状态管理 |
| Vite | 8.x | 构建工具 |
| uni-ui | 最新 | UI 组件库 |

## 目录结构规范

```
src/
├── pages/                     # 页面（按角色组织）
│   ├── admin/{module}/
│   ├── mch/{module}/
│   ├── saas/{module}/
│   └── guest/{module}/
├── components/                # 组件
│   ├── common/                # 通用组件
│   └── business/              # 业务组件
├── api/                       # API 调用封装
│   └── {module}/index.ts
├── types/                     # TypeScript 类型定义
│   └── {module}.ts
├── stores/                    # Pinia 状态管理
│   └── {module}.ts
├── composables/               # 组合函数
│   └── use{Feature}.ts
├── utils/                     # 工具函数
│   ├── request.ts             # 网络请求封装
│   └── auth.ts                # 登录/Token 管理
├── static/                    # 静态资源
├── platform/                  # 平台差异化代码
├── App.vue
├── main.ts
├── manifest.json
├── pages.json
└── uni.scss
```

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
| 页面文件 | kebab-case | `product-list.vue` |
| 组件文件 | PascalCase | `ProductCard.vue` |
| composables | use 前缀 | `usePagination.ts` |
| Store 文件 | kebab-case | `cart.ts` |
| API 文件 | kebab-case | `product.ts` |
| 类型文件 | kebab-case | `product.ts` |
| 页面路径 | `pages/{角色}/{模块}/{页面}` | `pages/guest/product/list` |

### `<script setup>` 规范

```typescript
// 1. 类型导入
import type { ProductInfo, ProductQueryParam } from '@/types/product'

// 2. 组件导入
import ProductCard from '@/components/business/ProductCard.vue'

// 3. API 导入
import { listProduct } from '@/api/product'

// 4. composables
const { loading, run } = useLoading()

// 5. 响应式状态
const listData = ref<ProductInfo[]>([])

// 6. 计算属性
const filteredList = computed(() => listData.value.filter(...))

// 7. 方法定义
const fetchList = async () => { ... }

// 8. 生命周期
onMounted(() => fetchList())
```

### Props & Emits 规范

```typescript
const props = defineProps<{
  product: ProductInfo
  showPrice?: boolean
}>()

const emit = defineEmits<{
  click: [product: ProductInfo]
  addCart: [productId: number, quantity: number]
}>()
```

## 页面生命周期规范

UniApp 页面同时支持 Vue 生命周期和 UniApp 生命周期，需按场景选择：

| 场景 | 使用钩子 | 说明 |
|------|---------|------|
| 页面初始化、获取参数 | `onLoad((options) => {})` | 仅页面级可用，options 为页面参数 |
| 页面每次显示 | `onShow(() => {})` | 从其他页面返回或从后台切回时触发 |
| 页面隐藏 | `onHide(() => {})` | 跳转其他页面或切后台 |
| DOM 首次就绪 | `onReady(() => {})` | 仅触发一次，适合获取节点信息 |
| 页面卸载 | `onUnload(() => {})` | 清理定时器、取消请求 |
| 组件初始化 | `onMounted(() => {})` | Vue 标准钩子，组件内使用 |
| 监听返回按钮 | `onBackPress(() => {})` | 返回 `true` 阻止默认返回 |

**使用原则**：
- 页面级逻辑（获取参数、每次刷新数据）用 UniApp 钩子（`onLoad`/`onShow`）
- 组件级逻辑（初始化、销毁）用 Vue 钩子（`onMounted`/`onUnmounted`）
- `onShow` 中避免重复初始化，用标志位或判断数据是否已加载

```typescript
import { onLoad, onShow } from '@dcloudio/uni-app'

onLoad((options) => {
  const id = options.id
  loadDetail(id)
})

onShow(() => {
  if (needsRefresh.value) {
    fetchList()
    needsRefresh.value = false
  }
})
```

## 多端适配规范

### 表单校验规范

前端提交前必须校验，不等后端返回错误：

| 校验类型 | 实现方式 | 示例 |
|---------|---------|------|
| 必填 | 表单提交前检查空值 | `if (!form.name) { showToast('请输入名称'); return }` |
| 格式 | 正则校验 | 手机号、邮箱、身份证 |
| 长度 | 字符串长度判断 | `if (form.name.length > 50) { ... }` |
| 范围 | 数值范围判断 | 金额 > 0 |
| 一致性 | 双字段比对 | 确认密码与密码一致 |

```typescript
const validateForm = (): boolean => {
  if (!form.productName.trim()) {
    uni.showToast({ title: '请输入商品名称', icon: 'none' })
    return false
  }
  if (form.price <= 0) {
    uni.showToast({ title: '价格必须大于0', icon: 'none' })
    return false
  }
  return true
}
```

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

| 能力 | 微信小程序 | H5 | App (Android/iOS) |
|------|-----------|-----|-------------------|
| 登录 | `uni.login({ provider: 'weixin' })` → 后端 wxLogin | 账号密码登录 | `plus.oauth.getServices()` |
| 支付 | `uni.requestPayment({ provider: 'wxpay' })` | `window.location.href = payUrl` | `uni.requestPayment({ provider: 'alipay' })` |
| 分享 | `onShareAppMessage()` | `navigator.share()` 或自定义 | `plus.share.sendWithSystem()` |
| 扫码 | `uni.scanCode()` | 不支持（需摄像头） | `uni.scanCode()` |
| 推送 | 微信模板消息 | WebSocket / SSE | `plus.push.addEventListener()` |
| 存储 | `uni.setStorageSync` / `uni.getStorageSync` | localStorage | `plus.io` |
| 网络状态 | `uni.getNetworkType()` | `navigator.onLine` | `uni.getNetworkType()` |
| 文件选择 | `uni.chooseImage()` | `<input type="file">` | `plus.gallery.pick()` |

### 样式单位

| 单位 | 使用场景 | 说明 |
|------|---------|------|
| `rpx` | 宽度、间距、字体 | 响应式单位，750rpx = 屏幕宽度 |
| `px` | 边框、阴影 | 固定尺寸 |
| `%` | 弹性布局 | 相对父容器 |
| `vh/vw` | 全屏布局 | 相对视口（仅 H5/App，小程序不支持） |

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
export const useCartStore = defineStore('cart', () => {
  const items = ref<CartItem[]>([])

  const totalCount = computed(() =>
    items.value.reduce((sum, item) => sum + item.quantity, 0)
  )

  const addItem = (product: ProductInfo, quantity = 1) => {
    const existing = items.value.find(i => i.productId === product.id)
    if (existing) {
      existing.quantity += quantity
    } else {
      items.value.push({ productId: product.id, productName: product.productName, price: product.price, quantity })
    }
    uni.setStorage({ key: 'cart', data: items.value })
  }

  return { items, totalCount, addItem }
})
```

## 网络请求规范

> **Token 安全说明**：Token 存储在本地存储（`uni.getStorageSync`），适用于 SaaS 后台管理系统场景。如需更高安全性，考虑 HttpOnly Cookie 或加密存储方案。

```typescript
const BASE_URL = import.meta.env.VITE_API_BASE_URL

export const request = <T>(options: UniApp.RequestOptions): Promise<T> => {
  return new Promise((resolve, reject) => {
    uni.request({
      url: `${BASE_URL}${options.url}`,
      method: options.method || 'GET',
      data: options.data,
      header: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${uni.getStorageSync('token')}`
      },
      success: (res) => {
        if (res.statusCode === 200) {
          const data = res.data as ResponseData<T>
          if (data.code === 200) {
            resolve(data.data)
          } else {
            uni.showToast({ title: data.message || '请求失败', icon: 'none' })
            reject(new Error(data.message))
          }
        } else if (res.statusCode === 401) {
          uni.navigateTo({ url: '/pages/guest/auth/login' })
          reject(new Error('Unauthorized'))
        } else {
          reject(new Error(`HTTP ${res.statusCode}`))
        }
      },
      fail: (err) => {
        uni.showToast({ title: '网络异常', icon: 'none' })
        reject(err)
      }
    })
  })
}
```

## 路由配置规范（pages.json）

| 规范 | 说明 |
|------|------|
| 路径格式 | `pages/{角色}/{模块}/{页面}` |
| 导航栏 | 每个页面配置 `navigationBarTitleText` |
| TabBar | 仅 guest 角色使用底部导航 |
| 分包 | 主包 ≤ 2MB，超出使用 `subPackages` 分包 |
| 懒加载 | 非首屏页面使用分包懒加载 |

## 字段一致性原则

| 层级 | 命名规范 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `product_name` |
| 后端 DTO 字段 | camelCase | `productName` |
| 前端表单字段 | camelCase（与 DTO 一致） | `productName` |
| 前端列表字段 | camelCase（与 DTO 一致） | `productName` |
| API 参数 | camelCase（与 DTO 一致） | `productName` |

## 禁用规则

| 禁用项 | 说明 |
|--------|------|
| `any` 类型 | 禁止使用 `any`，必须定义具体类型 |
| `v-html` | 小程序不支持，使用 `rich-text` 组件 |
| DOM 操作 | 禁止直接操作 DOM，使用 Vue 响应式 |
| 全局事件总线 | 禁止 `uni.$emit`/`uni.$on`，使用 Pinia 替代 |
| `window/document` | 禁止直接使用，需条件编译包裹 |
| 同步存储 | 大数据使用 `uni.setStorage` 异步版本 |

## 性能规范

| 场景 | 规范 |
|------|------|
| 图片 | 使用 `mode="aspectFill"` / `mode="widthFix"`，启用懒加载 |
| 列表 | 上拉加载分页，单页 ≤ 20 条 |
| 缓存 | 列表页使用 `uni.setStorageSync` 缓存首屏数据 |
| 分包 | 主包 ≤ 2MB，总包 ≤ 20MB（微信小程序） |
| 请求 | 并行请求使用 `Promise.all`，避免瀑布式请求 |
| 页面栈 | 页面栈 ≤ 10 层，超层使用 `uni.reLaunch` |
