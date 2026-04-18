# UniApp 移动端原型设计模板

> 包含 README.md 页面清单模板和 UniApp 页面代码模板。遵循字段一致性原则。

---

## 1. README.md 页面清单模板

```markdown
# {项目名} 移动端页面清单

## 页面总览

| 页面名 | 所属角色 | 模块 | 复杂度 | 代码策略 | 涉及数据库表 | 平台 |
|--------|---------|------|--------|---------|-------------|------|
| product-list | admin | product | 简单 | 裁剪生成代码 | product_info | 全平台 |
| product-detail | admin | product | 简单 | 裁剪生成代码 | product_info | 全平台 |
| order-list | mch | order | 复杂 | 基于生成器改造 | order_info, order_item | 微信小程序 |
| user-profile | guest | user | 简单 | 裁剪生成代码 | user_info | 全平台 |

## 角色权限映射

| 角色 | product-list | product-detail | order-list | user-profile |
|------|-------------|---------------|-----------|-------------|
| SAAS | R | R/W | - | - |
| MCH | R | R | R/W | R |
| ADMIN | R/W | R/W | R | - |
| GUEST | - | - | - | R/W |

R=只读（列表/详情），W=写入（新增/编辑/删除）

## PRD功能点映射

| PRD功能点 | 模块 | 页面 | 路径 | 说明 |
|-----------|------|------|------|------|
| 商品管理 | product | product-list | pages/admin/product/list | 管理员查看商品列表 |
| 商品详情 | product | product-detail | pages/admin/product/detail | 管理员编辑商品 |
| 订单处理 | order | order-list | pages/mch/order/list | 商户处理订单 |
| 个人中心 | user | user-profile | pages/guest/user/profile | 用户查看资料 |

## 路由设计

### 页面路径规范

```
pages/{角色}/{模块}/{页面}
```

### pages.json 配置

```json
{
  "pages": [
    {
      "path": "pages/admin/product/list",
      "style": {
        "navigationBarTitleText": "商品列表"
      }
    },
    {
      "path": "pages/admin/product/detail",
      "style": {
        "navigationBarTitleText": "商品详情"
      }
    },
    {
      "path": "pages/mch/order/list",
      "style": {
        "navigationBarTitleText": "订单列表"
      }
    },
    {
      "path": "pages/guest/user/profile",
      "style": {
        "navigationBarTitleText": "个人资料"
      }
    }
  ],
  "tabBar": {
    "list": [
      { "pagePath": "pages/guest/index/index", "text": "首页" },
      { "pagePath": "pages/guest/category/category", "text": "分类" },
      { "pagePath": "pages/guest/cart/cart", "text": "购物车" },
      { "pagePath": "pages/guest/user/profile", "text": "我的" }
    ]
  }
}
```

## 字段一致性检查

### Product 模块

| 数据库字段 | 后端DTO字段 | 前端表单字段 | 列表项 | 说明 |
|------------|-------------|--------------|--------|------|
| product_name | productName | productName | 商品名称 | 文本输入 |
| price | price | price | 价格 | 数字输入 |
| status | status | status | 状态 | 选择器 |
| create_time | createTime | - | 创建时间 | 只读显示 |

### Order 模块

| 数据库字段 | 后端DTO字段 | 前端表单字段 | 列表项 | 说明 |
|------------|-------------|--------------|--------|------|
| order_no | orderNo | orderNo | 订单号 | 文本显示 |
| total_amount | totalAmount | totalAmount | 总金额 | 金额显示 |
| order_status | orderStatus | orderStatus | 订单状态 | 状态标签 |

## 组件清单

### 通用组件

| 组件名 | 路径 | 用途 |
|--------|------|------|
| DataList | components/DataList.vue | 通用列表（下拉刷新/上拉加载） |
| SearchBar | components/SearchBar.vue | 通用搜索栏 |
| StatusTag | components/StatusTag.vue | 状态标签 |

### 业务组件

| 组件名 | 路径 | 用途 |
|--------|------|------|
| ProductCard | components/business/ProductCard.vue | 商品卡片 |
| OrderItem | components/business/OrderItem.vue | 订单项展示 |

## 平台适配策略

| 平台 | 适配要点 |
|------|---------|
| 微信小程序 | 使用 rpx 单位，遵循微信小程序设计规范 |
| H5 | 响应式设计，适配各种屏幕 |
| App | iOS/Android 原生体验，处理安全区和刘海屏 |
```

---

## 2. 页面组件模板

### 列表页模板 (list.vue)

```vue
<!-- src/pages/{role}/{module}/list.vue -->
<template>
  <view class="page-container">
    <!-- 搜索栏 -->
    <SearchBar
      v-model="keyword"
      placeholder="请输入关键词"
      @search="handleSearch"
    />

    <!-- 数据列表 -->
    <DataList
      :data="listData"
      :loading="loading"
      :has-more="hasMore"
      @refresh="handleRefresh"
      @load-more="handleLoadMore"
    >
      <template #item="{ item }">
        <view class="list-item" @click="handleDetail(item)">
          <view class="item-content">
            <text class="item-title">{{ item.{module}Name }}</text>
            <StatusTag :status="item.status" />
          </view>
          <text class="item-time">{{ item.createTime }}</text>
        </view>
      </template>
    </DataList>

    <!-- 新增按钮 -->
    <view class="fab-btn" @click="handleAdd">
      <text class="fab-icon">+</text>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import SearchBar from '@/components/SearchBar.vue'
import DataList from '@/components/DataList.vue'
import StatusTag from '@/components/StatusTag.vue'
import { list{Module} } from '@/api/{module}'
import type { {Module}Info, {Module}QueryParam } from '@/types/{module}'

// 列表数据
const listData = ref<{Module}Info[]>([])
const loading = ref(false)
const hasMore = ref(true)
const keyword = ref('')

// 查询参数 - 字段名与后端QueryParam一致
const queryParam = ref<{Module}QueryParam>({
  keyword: '',
  pageNum: 1,
  pageSize: 20
})

// 查询列表
const fetchList = async (isRefresh = false) => {
  if (loading.value) return
  loading.value = true

  try {
    const res = await list{Module}(queryParam.value)
    if (isRefresh) {
      listData.value = res.data.list
    } else {
      listData.value.push(...res.data.list)
    }
    hasMore.value = res.data.list.length === queryParam.value.pageSize
  } finally {
    loading.value = false
  }
}

// 搜索
const handleSearch = () => {
  queryParam.value.keyword = keyword.value
  queryParam.value.pageNum = 1
  fetchList(true)
}

// 刷新
const handleRefresh = () => {
  queryParam.value.pageNum = 1
  fetchList(true)
}

// 加载更多
const handleLoadMore = () => {
  if (!hasMore.value) return
  queryParam.value.pageNum++
  fetchList()
}

// 查看详情
const handleDetail = (item: {Module}Info) => {
  uni.navigateTo({
    url: `/pages/{role}/{module}/detail?id=${item.id}`
  })
}

// 新增
const handleAdd = () => {
  uni.navigateTo({
    url: `/pages/{role}/{module}/form`
  })
}

onMounted(() => {
  fetchList()
})
</script>

<style scoped lang="scss">
.page-container {
  min-height: 100vh;
  background: #f5f5f5;
}

.list-item {
  background: #fff;
  padding: 24rpx;
  margin-bottom: 16rpx;

  .item-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16rpx;

    .item-title {
      font-size: 32rpx;
      font-weight: 500;
      color: #333;
    }
  }

  .item-time {
    font-size: 24rpx;
    color: #999;
  }
}

.fab-btn {
  position: fixed;
  right: 32rpx;
  bottom: 120rpx;
  width: 96rpx;
  height: 96rpx;
  background: #007aff;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4rpx 16rpx rgba(0, 122, 255, 0.3);

  .fab-icon {
    font-size: 48rpx;
    color: #fff;
  }
}
</style>
```

### 表单页模板 (form.vue)

```vue
<!-- src/pages/{role}/{module}/form.vue -->
<template>
  <view class="page-container">
    <view class="form-section">
      <!-- 表单字段 - 字段名与后端DTO一致 -->
      <view class="form-item">
        <text class="form-label">名称</text>
        <input
          v-model="formData.{module}Name"
          class="form-input"
          placeholder="请输入名称"
        />
      </view>

      <view class="form-item">
        <text class="form-label">状态</text>
        <view class="radio-group">
          <view
            class="radio-item"
            :class="{ active: formData.status === 1 }"
            @click="formData.status = 1"
          >
            <text>启用</text>
          </view>
          <view
            class="radio-item"
            :class="{ active: formData.status === 0 }"
            @click="formData.status = 0"
          >
            <text>禁用</text>
          </view>
        </view>
      </view>

      <view class="form-item">
        <text class="form-label">描述</text>
        <textarea
          v-model="formData.description"
          class="form-textarea"
          placeholder="请输入描述"
          :maxlength="500"
        />
        <text class="word-count">{{ formData.description?.length || 0 }}/500</text>
      </view>
    </view>

    <!-- 底部按钮 -->
    <view class="footer-bar">
      <button class="btn-cancel" @click="handleCancel">取消</button>
      <button class="btn-submit" :loading="submitting" @click="handleSubmit">保存</button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { get{Module}Detail, save{Module}, update{Module} } from '@/api/{module}'
import type { {Module}Form } from '@/types/{module}'

const formData = reactive<Partial<{Module}Form>>({
  id: undefined,
  {module}Name: '',
  status: 1,
  description: ''
})

const isEdit = ref(false)
const submitting = ref(false)

// 提交
const handleSubmit = async () => {
  // 表单校验
  if (!formData.{module}Name) {
    uni.showToast({ title: '请输入名称', icon: 'none' })
    return
  }

  submitting.value = true
  try {
    if (isEdit.value) {
      await update{Module}(formData as {Module}Form)
    } else {
      await save{Module}(formData as {Module}Form)
    }
    uni.showToast({ title: '保存成功', icon: 'success' })
    setTimeout(() => {
      uni.navigateBack()
    }, 1500)
  } catch (error) {
    uni.showToast({ title: '保存失败', icon: 'none' })
  } finally {
    submitting.value = false
  }
}

// 取消
const handleCancel = () => {
  uni.navigateBack()
}

// 加载详情
const loadDetail = async (id: number) => {
  try {
    const res = await get{Module}Detail(id)
    Object.assign(formData, res.data)
  } catch (error) {
    uni.showToast({ title: '加载失败', icon: 'none' })
  }
}

onMounted(() => {
  const pages = getCurrentPages()
  const currentPage = pages[pages.length - 1]
  const { id } = currentPage.$page?.options || {}

  if (id) {
    isEdit.value = true
    loadDetail(Number(id))
  }
})
</script>

<style scoped lang="scss">
.page-container {
  min-height: 100vh;
  background: #f5f5f5;
  padding-bottom: 120rpx;
}

.form-section {
  background: #fff;
  padding: 0 24rpx;
}

.form-item {
  padding: 24rpx 0;
  border-bottom: 1rpx solid #eee;

  &:last-child {
    border-bottom: none;
  }

  .form-label {
    display: block;
    font-size: 28rpx;
    color: #333;
    margin-bottom: 16rpx;
  }

  .form-input {
    height: 80rpx;
    background: #f5f5f5;
    border-radius: 8rpx;
    padding: 0 24rpx;
    font-size: 28rpx;
  }

  .form-textarea {
    width: 100%;
    height: 200rpx;
    background: #f5f5f5;
    border-radius: 8rpx;
    padding: 24rpx;
    font-size: 28rpx;
  }

  .word-count {
    display: block;
    text-align: right;
    font-size: 24rpx;
    color: #999;
    margin-top: 8rpx;
  }
}

.radio-group {
  display: flex;
  gap: 24rpx;

  .radio-item {
    flex: 1;
    height: 72rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    background: #f5f5f5;
    border-radius: 8rpx;
    font-size: 28rpx;
    color: #666;

    &.active {
      background: #007aff;
      color: #fff;
    }
  }
}

.footer-bar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: #fff;
  padding: 24rpx;
  display: flex;
  gap: 24rpx;
  box-shadow: 0 -2rpx 8rpx rgba(0, 0, 0, 0.05);

  .btn-cancel,
  .btn-submit {
    flex: 1;
    height: 88rpx;
    border-radius: 8rpx;
    font-size: 32rpx;
  }

  .btn-cancel {
    background: #f5f5f5;
    color: #666;
  }

  .btn-submit {
    background: #007aff;
    color: #fff;
  }
}
</style>
```

### 详情页模板 (detail.vue)

```vue
<!-- src/pages/{role}/{module}/detail.vue -->
<template>
  <view class="page-container">
    <view class="detail-section">
      <view class="detail-item">
        <text class="item-label">ID</text>
        <text class="item-value">{{ detail.id }}</text>
      </view>

      <view class="detail-item">
        <text class="item-label">名称</text>
        <text class="item-value">{{ detail.{module}Name }}</text>
      </view>

      <view class="detail-item">
        <text class="item-label">状态</text>
        <StatusTag :status="detail.status" />
      </view>

      <view class="detail-item">
        <text class="item-label">创建时间</text>
        <text class="item-value">{{ detail.createTime }}</text>
      </view>

      <view class="detail-item block">
        <text class="item-label">描述</text>
        <text class="item-value">{{ detail.description || '暂无描述' }}</text>
      </view>
    </view>

    <!-- 底部操作栏 -->
    <view class="footer-bar">
      <button class="btn-edit" @click="handleEdit">编辑</button>
      <button class="btn-delete" @click="handleDelete">删除</button>
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import StatusTag from '@/components/StatusTag.vue'
import { get{Module}Detail, delete{Module} } from '@/api/{module}'
import type { {Module}Info } from '@/types/{module}'

const detail = ref<Partial<{Module}Info>>({})

const loadDetail = async () => {
  const pages = getCurrentPages()
  const currentPage = pages[pages.length - 1]
  const { id } = currentPage.$page?.options || {}

  if (!id) return

  try {
    const res = await get{Module}Detail(Number(id))
    detail.value = res.data
  } catch (error) {
    uni.showToast({ title: '加载失败', icon: 'none' })
  }
}

const handleEdit = () => {
  uni.navigateTo({
    url: `/pages/{role}/{module}/form?id=${detail.value.id}`
  })
}

const handleDelete = () => {
  uni.showModal({
    title: '提示',
    content: '确认删除该记录？',
    success: async (res) => {
      if (res.confirm) {
        try {
          await delete{Module}(detail.value.id!)
          uni.showToast({ title: '删除成功', icon: 'success' })
          setTimeout(() => {
            uni.navigateBack()
          }, 1500)
        } catch (error) {
          uni.showToast({ title: '删除失败', icon: 'none' })
        }
      }
    }
  })
}

onMounted(loadDetail)
</script>

<style scoped lang="scss">
.page-container {
  min-height: 100vh;
  background: #f5f5f5;
  padding-bottom: 120rpx;
}

.detail-section {
  background: #fff;
  padding: 0 24rpx;
}

.detail-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 24rpx 0;
  border-bottom: 1rpx solid #eee;

  &:last-child {
    border-bottom: none;
  }

  &.block {
    flex-direction: column;
    align-items: flex-start;

    .item-label {
      margin-bottom: 16rpx;
    }

    .item-value {
      color: #666;
      line-height: 1.6;
    }
  }

  .item-label {
    font-size: 28rpx;
    color: #999;
  }

  .item-value {
    font-size: 28rpx;
    color: #333;
  }
}

.footer-bar {
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  background: #fff;
  padding: 24rpx;
  display: flex;
  gap: 24rpx;
  box-shadow: 0 -2rpx 8rpx rgba(0, 0, 0, 0.05);

  .btn-edit,
  .btn-delete {
    flex: 1;
    height: 88rpx;
    border-radius: 8rpx;
    font-size: 32rpx;
  }

  .btn-edit {
    background: #007aff;
    color: #fff;
  }

  .btn-delete {
    background: #ff4d4f;
    color: #fff;
  }
}
</style>
```

---

## 3. pages.json 配置模板

```json
{
  "pages": [
    {
      "path": "pages/guest/index/index",
      "style": {
        "navigationBarTitleText": "首页"
      }
    },
    {
      "path": "pages/admin/product/list",
      "style": {
        "navigationBarTitleText": "商品列表"
      }
    },
    {
      "path": "pages/admin/product/detail",
      "style": {
        "navigationBarTitleText": "商品详情"
      }
    },
    {
      "path": "pages/admin/product/form",
      "style": {
        "navigationBarTitleText": "商品编辑"
      }
    },
    {
      "path": "pages/mch/order/list",
      "style": {
        "navigationBarTitleText": "订单列表"
      }
    }
  ],
  "globalStyle": {
    "navigationBarTextStyle": "black",
    "navigationBarBackgroundColor": "#fff",
    "backgroundColor": "#f5f5f5"
  },
  "tabBar": {
    "color": "#999",
    "selectedColor": "#007aff",
    "backgroundColor": "#fff",
    "borderStyle": "black",
    "list": [
      {
        "pagePath": "pages/guest/index/index",
        "text": "首页"
      },
      {
        "pagePath": "pages/guest/category/category",
        "text": "分类"
      },
      {
        "pagePath": "pages/guest/cart/cart",
        "text": "购物车"
      },
      {
        "pagePath": "pages/guest/user/profile",
        "text": "我的"
      }
    ]
  }
}
```

---

## 4. 字段一致性原则应用

### 命名映射规则

| 层级 | 命名规范 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `user_name`, `create_time` |
| 后端DTO字段 | camelCase | `userName`, `createTime` |
| 前端表单字段 | camelCase | `userName`, `createTime` |
| 页面路径 | kebab-case | `user-profile`, `order-list` |

### 表单字段示例

```typescript
// 数据库: user_info 表
// user_name (VARCHAR) - 用户名
// phone (VARCHAR) - 手机号
// status (TINYINT) - 状态
// create_time (DATETIME) - 创建时间

// 后端 UserInfoDto
type UserInfoDto = {
  userName: string    // 对应 user_name
  phone: string       // 对应 phone
  status: number      // 对应 status
  createTime: string  // 对应 create_time
}

// 前端表单
const formData = reactive({
  userName: '',   // 与后端DTO一致
  phone: '',      // 与后端DTO一致
  status: 1       // 与后端DTO一致
})
```

### 测试选择器策略

基于字段一致性，E2E测试可直接使用字段名定位：

```typescript
// 不需要 data-testid，直接使用字段名
await page.getByLabel('用户名').fill('test')      // label 匹配
await page.getByPlaceholder('请输入用户名').fill('test')  // placeholder 匹配
```
