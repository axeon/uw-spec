# UniApp开发示例

## 项目配置

### manifest.json
```json
{
  "name": "nova-guest",
  "appid": "__UNI__XXXXXXX",
  "description": "Nova Guest App",
  "versionName": "1.0.0",
  "versionCode": "100",
  "mp-weixin": {
    "appid": "wxXXXXXXXXXXXXXXXX",
    "setting": { "urlCheck": false }
  }
}
```

### pages.json
```json
{
  "pages": [
    { "path": "pages/index/index", "style": { "navigationBarTitleText": "首页" } },
    { "path": "pages/product/detail", "style": { "navigationBarTitleText": "商品详情" } }
  ],
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarTitleText": "Nova",
    "navigationBarBackgroundColor": "#F8F8F8"
  },
  "tabBar": {
    "list": [
      { "pagePath": "pages/index/index", "text": "首页" },
      { "pagePath": "pages/category/category", "text": "分类" },
      { "pagePath": "pages/cart/cart", "text": "购物车" },
      { "pagePath": "pages/user/user", "text": "我的" }
    ]
  }
}
```

## 页面示例

### 首页
```vue
<template>
  <view class="container">
    <search-bar @search="onSearch" />
    <swiper class="banner" :indicator-dots="true">
      <swiper-item v-for="item in banners" :key="item.id">
        <image :src="item.image" mode="aspectFill" />
      </swiper-item>
    </swiper>
    <product-list :list="products" @click="goDetail" />
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import { productApi } from '@/api/product'

const banners = ref<Banner[]>([])
const products = ref<Product[]>([])

onLoad(() => fetchData())

const fetchData = async () => {
  const [bannerRes, productRes] = await Promise.all([
    productApi.getBanners(),
    productApi.getList({ page: 1, size: 10 })
  ])
  banners.value = bannerRes.data
  products.value = productRes.data.list
}

const goDetail = (id: string) => {
  uni.navigateTo({ url: `/pages/product/detail?id=${id}` })
}
</script>
```

## API封装

```typescript
// utils/request.ts
type Method = 'GET' | 'POST' | 'PUT' | 'DELETE'

export const request = <T>(options: { url: string; method?: Method; data?: any }): Promise<T> => {
  return new Promise((resolve, reject) => {
    uni.request({
      url: `${import.meta.env.VITE_API_URL}${options.url}`,
      method: options.method || 'GET',
      data: options.data,
      header: { 'Authorization': `Bearer ${uni.getStorageSync('token')}` },
      success: (res) => {
        if (res.statusCode === 200) resolve(res.data as T)
        else if (res.statusCode === 401) {
          uni.navigateTo({ url: '/pages/login/login' })
          reject(new Error('Unauthorized'))
        } else reject(new Error('Request failed'))
      },
      fail: reject
    })
  })
}
```

## 微信登录

```typescript
// api/auth.ts
export const authApi = {
  wxLogin: () => {
    return new Promise((resolve, reject) => {
      uni.login({
        provider: 'weixin',
        success: (res) => {
          if (res.code) {
            request.post('/auth/wx-login', { code: res.code })
              .then(resolve)
              .catch(reject)
          }
        },
        fail: reject
      })
    })
  }
}
```

## 平台差异

| 特性 | 微信小程序 | H5 | App |
|-----|-----------|-----|-----|
| 登录 | wx.login | 普通登录 | plus.oauth |
| 支付 | wx.requestPayment | 跳转支付 | uni.requestPayment |
| 分享 | onShareAppMessage | 浏览器分享 | plus.share |
| 扫码 | wx.scanCode | 不支持 | uni.scanCode |

## TDD示例

```typescript
// 🔴 Red - 编写测试
describe('calculateOrderAmount', () => {
  it('应该正确计算订单金额', () => {
    const items = [{ price: 100, quantity: 2 }, { price: 50, quantity: 1 }]
    const amount = calculateOrderAmount(items, 10)
    expect(amount.totalAmount).toBe(260)
  })
})

// 🟢 Green - 实现功能
export function calculateOrderAmount(items: OrderItem[], shippingFee: number) {
  const goodsAmount = items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  const finalShippingFee = goodsAmount >= 199 ? 0 : shippingFee
  return { goodsAmount, shippingFee: finalShippingFee, totalAmount: goodsAmount + finalShippingFee }
}
```
