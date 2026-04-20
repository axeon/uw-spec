# Web前端开发评审检查清单

> 基于 Vue3 + TypeScript + Vitest。前端单元测试聚焦 composables/Store/工具函数。

## 1. TDD实践

| 检查项 | 要求 |
|--------|------|
| 测试范围 | composables、Store actions、工具函数必须有测试 |
| 覆盖率 | 核心业务逻辑行覆盖≥70% |
| 框架 | Vitest + Vue Test Utils + @pinia/testing |
| 断言明确 | 使用明确的 expect 断言 |

**composable 测试示例**：
```typescript
import { describe, it, expect } from 'vitest'
import { usePagination } from '@/composables/usePagination'

describe('usePagination', () => {
  it('初始化分页参数', () => {
    const { currentPage, pageSize, total } = usePagination()
    expect(currentPage.value).toBe(1)
    expect(pageSize.value).toBe(10)
    expect(total.value).toBe(0)
  })

  it('计算总页数', () => {
    const { total, pageSize, totalPages } = usePagination()
    total.value = 25
    pageSize.value = 10
    expect(totalPages.value).toBe(3)
  })
})
```

**Store 测试示例**：
```typescript
import { describe, it, expect, beforeEach } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useCartStore } from '@/stores/cart'

describe('cartStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('添加商品到购物车', () => {
    const store = useCartStore()
    store.addItem({ id: 1, productName: '测试商品', price: 100 }, 2)
    expect(store.items.length).toBe(1)
    expect(store.totalCount).toBe(2)
    expect(store.totalPrice).toBe(200)
  })

  it('重复添加增加数量', () => {
    const store = useCartStore()
    store.addItem({ id: 1, productName: '测试商品', price: 100 })
    store.addItem({ id: 1, productName: '测试商品', price: 100 }, 3)
    expect(store.items.length).toBe(1)
    expect(store.totalCount).toBe(4)
  })
})
```

## 2. Vue3规范

| 检查项 | 要求 |
|--------|------|
| 语法 | 使用`<script setup lang="ts">` |
| 响应式 | 正确使用ref/reactive/computed |
| Props | defineProps 泛型风格，类型定义完整 |
| Emits | defineEmits 泛型风格，事件定义清晰 |
| 禁止any | TypeScript类型覆盖率≥90% |
| API对接 | 使用 `src/api/` 统一封装，ResponseData 统一处理 |
| 字段一致性 | 表单字段名与后端 DTO camelCase 一致 |

**示例**
```vue
<script setup lang="ts">
import { ref, computed } from 'vue'

interface User {
  id: number
  name: string
}

const props = defineProps<{
  user: User
  editable?: boolean
}>()

const emit = defineEmits<{
  edit: [userId: number]
}>()

const loading = ref(false)
const displayName = computed(() => props.user.name || '未知')
</script>
```

## 3. Pinia状态管理

| 检查项 | 要求 |
|--------|------|
| Store划分 | 按功能模块划分 |
| State | 结构清晰 |
| Getters | 使用合理 |
| Actions | 职责单一 |

**示例**
```typescript
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import { getUserInfo } from '@/api/user'
import type { UserInfo } from '@/types/user'

export const useUserStore = defineStore('user', () => {
  const userInfo = ref<UserInfo | null>(null)
  const isLogin = computed(() => !!userInfo.value)
  
  async function fetchUserInfo() {
    const res = await getUserInfo()
    userInfo.value = res.data
  }
  
  return { userInfo, isLogin, fetchUserInfo }
})
```

## 4. 代码质量

| 检查项 | 要求 |
|--------|------|
| 目录结构 | 清晰合理 |
| 文件命名 | 规范统一 |
| 类型定义 | 完整，避免any |
| 重复代码 | 抽取复用 |

**目录结构**
```
src/
├── api/           # API接口
├── components/    # 组件
│   ├── common/    # 通用组件
│   └── business/  # 业务组件
├── composables/   # 组合函数
├── stores/        # Pinia Store
├── views/         # 页面
├── utils/         # 工具函数
└── types/         # 类型定义
```

## 5. 性能优化

| 检查项 | 要求 |
|--------|------|
| 懒加载 | 路由组件懒加载 |
| v-for | 使用key |
| computed | 合理使用缓存 |
| watch | 避免不必要的监听 |

**示例**
```vue
<template>
  <UserCard v-for="user in userList" :key="user.id" :user="user" />
</template>

<script setup lang="ts">
const AsyncChart = defineAsyncComponent(() => 
  import('./components/Chart.vue')
)
const totalCount = computed(() => userList.value.length)
</script>
```

## 6. 安全性

| 检查项 | 要求 |
|--------|------|
| v-html | 禁止未转义使用，必须经过 DOMPurify 或自定义过滤器 |
| 路由守卫 | beforeEach 权限控制，未登录跳转登录页 |
| Token管理 | httpOnly cookie 优先，localStorage 需加密 |
| API 错误处理 | 401 自动跳转登录，403 提示无权限 |

**示例**
```typescript
// 路由守卫
router.beforeEach((to, from, next) => {
  const userStore = useUserStore()
  if (to.meta.requiresAuth && !userStore.isLogin) {
    next('/login')
  } else {
    next()
  }
})

// 敏感操作确认
const handleDelete = async () => {
  const confirmed = await ElMessageBox.confirm('确定删除？', '警告')
  if (confirmed) await deleteUser(props.userId)
}
```
