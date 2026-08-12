# UniApp 管理端开发示例

> 展示 API 对接、页面开发、平台适配等通用模式。类型从 `@/api/` 导入。示例以当前项目 `uw-uniapp-template-main` 的 `uwAuthCenter/root/account/group` 为参照。

## API 调用模式

> API 调用已由 `src/api/` 封装，页面中禁止直接使用 `uni.request`。

### 列表页 API 对接

```typescript
import { rootAccountGroupList } from '@/api/uwAuthCenterRootApi'
import type { AccountGroupInfo } from '@/api/uwAuthCenterRootApi'

const fetchList = async (isRefresh = false) => {
  if (loading.value) return
  loading.value = true
  try {
    if (isRefresh) pageNum.value = 1
    const res = await rootAccountGroupList({
      $pg: pageNum.value,
      $rn: 20,
      keyword: keyword.value || undefined,
      state: currentFilter.value === 'all' ? undefined : currentFilter.value,
    })
    const list = res.data?.list || []
    if (isRefresh) {
      listData.value = list
    } else {
      listData.value.push(...list)
    }
    hasMore.value = list.length >= 20
    pageNum.value++
  } catch (error) {
    console.error('fetchList error', error)
    uni.showToast({ title: '加载失败，请稍后重试', icon: 'none' })
  } finally {
    loading.value = false
  }
}
```

### 详情页 API 对接

```typescript
import { rootAccountGroupLoad } from '@/api/uwAuthCenterRootApi'
import type { AccountGroupInfo } from '@/api/uwAuthCenterRootApi'

const loadDetail = async (id: number) => {
  const res = await rootAccountGroupLoad({ id })
  detail.value = res.data ?? null
}
```

### 表单提交

```typescript
import { useI18n } from 'vue-i18n'
import { rootAccountGroupSave, rootAccountGroupUpdate } from '@/api/uwAuthCenterRootApi'
import type { AccountGroupForm } from '@/api/uwAuthCenterRootApi'

const { t } = useI18n()

const handleSubmit = async () => {
  const valid = await uvFormRef.value?.validate?.()
  if (!valid) return
  try {
    if (isEdit.value) {
      await rootAccountGroupUpdate({ data: form.value }, { remark: '' })
    } else {
      await rootAccountGroupSave(form.value)
    }
    uni.showToast({ title: t('saveSuccess'), icon: 'success' })
    setTimeout(() => uni.navigateBack(), 1500)
  } catch (error: any) {
    uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
  }
}
```

## 主题变量使用示例

### CSS 变量使用

```scss
/* 正确示例 */
.btn-primary {
  background-color: var(--color-primary);
  color: #fff;
}

.btn-gradient {
  background: linear-gradient(135deg, var(--color-primary), var(--color-primary-secondary));
}

.card-selected {
  background-color: var(--color-primary-bg);
}

/* 错误示例 - 禁止硬编码 */
.btn-primary {
  background-color: #409EFF; /* ❌ 禁止 */
}
```

### 组件中使用主题色

```vue
<template>
  <view class="custom-card">
    <text class="title">标题</text>
    <uv-button type="primary">操作按钮</uv-button>
  </view>
</template>

<style scoped lang="scss">
.custom-card {
  background-color: var(--card-bg);
  border: 1rpx solid var(--border-color);
}

.title {
  color: var(--text-primary);
}
</style>
```

## 平台适配

### 条件编译

```typescript
// #ifdef MP-WEIXIN
// 微信小程序专用代码
// #endif

// #ifdef H5
// H5 专用代码
// #endif

// #ifdef APP-PLUS
// App 专用代码
// #endif
```

### 平台差异对照

| 特性 | 微信小程序 | H5 | App |
|-----|-----------|-----|-----|
| 登录 | `uni.login({ provider: 'weixin' })` | 账号密码登录 | `plus.oauth.getServices()` |
| 扫码 | `uni.scanCode()` | 不支持 | `uni.scanCode()` |
| 推送 | 微信模板消息 | WebSocket | `plus.push.addEventListener()` |

## TDD 示例

```typescript
import { describe, it, expect } from 'vitest'

describe('calculateOrderAmount', () => {
  it('应该正确计算订单金额', () => {
    const items = [{ price: 100, quantity: 2 }, { price: 50, quantity: 1 }]
    const amount = calculateOrderAmount(items, 10)
    expect(amount.totalAmount).toBe(260)
  })
})

export function calculateOrderAmount(items: OrderItem[], shippingFee: number) {
  const goodsAmount = items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  const finalShippingFee = goodsAmount >= 199 ? 0 : shippingFee
  return { goodsAmount, shippingFee: finalShippingFee, totalAmount: goodsAmount + finalShippingFee }
}
```
