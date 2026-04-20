# Web Vue 原型设计模板

> 包含 README.md 页面清单模板和 Vue 页面代码模板。遵循字段一致性原则。

---

## 1. README.md 页面清单模板

```markdown
# {项目名} Web端页面清单

## 页面总览

| 页面名 | 所属角色 | 模块 | 复杂度 | 代码策略 | 涉及数据库表 |
|--------|---------|------|--------|---------|-------------|
| ProductList | admin | product | 简单 | 裁剪生成代码 | product_info |
| ProductDetail | admin | product | 简单 | 裁剪生成代码 | product_info |
| OrderList | mch | order | 复杂 | 基于生成器改造 | order_info, order_item |
| UserProfile | guest | user | 简单 | 裁剪生成代码 | user_info |

## 角色权限映射

| 角色 | ProductList | ProductDetail | OrderList | UserProfile |
|------|-------------|---------------|-----------|-------------|
| SAAS | R | R/W | - | - |
| MCH | R | R | R/W | R |
| ADMIN | R/W | R/W | R | - |
| GUEST | - | - | - | R/W |

R=只读（列表/详情），W=写入（新增/编辑/删除）

## PRD功能点映射

| PRD功能点 | 模块 | 页面 | 路由 | 说明 |
|-----------|------|------|------|------|
| 商品管理 | product | ProductList | /admin/product/list | 管理员查看商品列表 |
| 商品详情 | product | ProductDetail | /admin/product/detail/:id | 管理员编辑商品 |
| 订单处理 | order | OrderList | /mch/order/list | 商户处理订单 |
| 个人中心 | user | UserProfile | /guest/user/profile | 用户查看资料 |

## 路由设计

### 路由层级

```
/{角色}/{模块}/{页面}
```

### 路由配置

| 路由路径 | 组件 | 角色守卫 | 说明 |
|----------|------|---------|------|
| /admin/product/list | ProductList.vue | admin | 商品列表 |
| /admin/product/detail/:id | ProductDetail.vue | admin | 商品详情 |
| /mch/order/list | OrderList.vue | mch | 订单列表 |
| /guest/user/profile | UserProfile.vue | guest | 个人资料 |

### 导航结构

**Admin 侧边栏**:
- 商品管理
  - 商品列表 → /admin/product/list

**Mch 侧边栏**:
- 订单管理
  - 订单列表 → /mch/order/list

## 字段一致性检查

### Product 模块

| 数据库字段 | 后端DTO字段 | 前端表单字段 | 表格列 | 说明 |
|------------|-------------|--------------|--------|------|
| product_name | productName | productName | 商品名称 | 文本输入 |
| price | price | price | 价格 | 数字输入 |
| status | status | status | 状态 | 选择器 |
| create_time | createTime | - | 创建时间 | 只读显示 |

### Order 模块

| 数据库字段 | 后端DTO字段 | 前端表单字段 | 表格列 | 说明 |
|------------|-------------|--------------|--------|------|
| order_no | orderNo | orderNo | 订单号 | 文本显示 |
| total_amount | totalAmount | totalAmount | 总金额 | 金额显示 |
| order_status | orderStatus | orderStatus | 订单状态 | 状态标签 |

## 组件清单

### 通用组件

| 组件名 | 路径 | 用途 |
|--------|------|------|
| DataTable | components/DataTable.vue | 通用表格（分页/排序/筛选） |
| SearchForm | components/SearchForm.vue | 通用搜索表单 |
| StatusTag | components/StatusTag.vue | 状态标签 |

### 业务组件

| 组件名 | 路径 | 用途 |
|--------|------|------|
| ProductCard | components/business/ProductCard.vue | 商品卡片 |
| OrderItem | components/business/OrderItem.vue | 订单项展示 |
```

> **降级策略**：如果 230-init 未生成 SearchForm/DataTable/StatusTag 通用组件，列表页模板中可直接使用 Element Plus 原生组件（`<el-form>` + `<el-table>` + `<el-tag>`）替代。

---

## 2. 页面组件模板

### 列表页模板 (List.vue)

```vue
<!-- src/pages/{role}/{module}/List.vue -->
<template>
  <div class="page-container">
    <!-- 搜索表单 -->
    <SearchForm
      v-model="searchForm"
      :fields="searchFields"
      @search="handleSearch"
      @reset="handleReset"
    />

    <!-- 操作按钮 -->
    <div class="operation-bar">
      <el-button type="primary" @click="handleAdd">
        <el-icon><Plus /></el-icon>新增
      </el-button>
    </div>

    <!-- 数据表格 -->
    <DataTable
      :data="tableData"
      :columns="tableColumns"
      :loading="loading"
      :pagination="pagination"
      @page-change="handlePageChange"
      @sort-change="handleSortChange"
    >
      <!-- 状态列 -->
      <template #status="{ row }">
        <StatusTag :status="row.status" />
      </template>

      <!-- 操作列 -->
      <template #actions="{ row }">
        <el-button link type="primary" @click="handleEdit(row)">编辑</el-button>
        <el-button link type="danger" @click="handleDelete(row)">删除</el-button>
      </template>
    </DataTable>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage, ElMessageBox } from 'element-plus'
import { Plus } from '@element-plus/icons-vue'
import SearchForm from '@/components/SearchForm.vue'
import DataTable from '@/components/DataTable.vue'
import StatusTag from '@/components/StatusTag.vue'
import { list{Module}, delete{Module} } from '@/api/{module}'
import type { {Module}QueryParam, {Module}Info } from '@/types/{module}'

const router = useRouter()
const loading = ref(false)

// 搜索表单 - 字段名与后端QueryParam一致
const searchForm = reactive<{Module}QueryParam>({
  keyword: '',
  status: undefined,
  pageNum: 1,
  pageSize: 20
})

// 搜索字段配置
const searchFields = [
  { prop: 'keyword', label: '关键词', type: 'input', placeholder: '请输入名称' },
  { 
    prop: 'status', 
    label: '状态', 
    type: 'select',
    options: [
      { label: '启用', value: 1 },
      { label: '禁用', value: 0 }
    ]
  }
]

// 表格列配置 - dataIndex与后端返回字段一致
const tableColumns = [
  { prop: 'id', label: 'ID', width: 80 },
  { prop: '{module}Name', label: '名称', sortable: true },
  { prop: 'status', label: '状态', slot: 'status' },
  { prop: 'createTime', label: '创建时间', width: 180 },
  { prop: 'actions', label: '操作', width: 150, slot: 'actions', fixed: 'right' }
]

// 表格数据
const tableData = ref<{Module}Info[]>([])
const pagination = reactive({
  total: 0,
  pageNum: 1,
  pageSize: 20
})

// 查询列表
const fetchList = async () => {
  loading.value = true
  try {
    const res = await list{Module}(searchForm)
    tableData.value = res.data.list
    pagination.total = res.data.total
  } finally {
    loading.value = false
  }
}

// 搜索
const handleSearch = () => {
  searchForm.pageNum = 1
  fetchList()
}

// 重置
const handleReset = () => {
  searchForm.keyword = ''
  searchForm.status = undefined
  handleSearch()
}

// 分页变化
const handlePageChange = (page: number, size: number) => {
  searchForm.pageNum = page
  searchForm.pageSize = size
  fetchList()
}

// 排序变化
const handleSortChange = ({ prop, order }: { prop: string; order: string }) => {
  // 处理排序逻辑
  fetchList()
}

// 新增
const handleAdd = () => {
  router.push(`/{role}/{module}/form`)
}

// 编辑
const handleEdit = (row: {Module}Info) => {
  router.push(`/{role}/{module}/form?id=${row.id}`)
}

// 删除
const handleDelete = async (row: {Module}Info) => {
  try {
    await ElMessageBox.confirm('确认删除该记录？', '提示', { type: 'warning' })
    await delete{Module}(row.id)
    ElMessage.success('删除成功')
    fetchList()
  } catch {
    // 取消删除
  }
}

onMounted(fetchList)
</script>

<style scoped>
.page-container {
  padding: 20px;
}
.operation-bar {
  margin: 16px 0;
}
</style>
```

### 表单页模板 (Form.vue)

```vue
<!-- src/pages/{role}/{module}/Form.vue -->
<template>
  <div class="page-container">
    <el-card class="form-card">
      <template #header>
        <span>{{ isEdit ? '编辑' : '新增' }}</span>
      </template>

      <el-form
        ref="formRef"
        :model="formData"
        :rules="formRules"
        label-width="120px"
      >
        <!-- 表单字段 - 字段名与后端DTO一致 -->
        <el-form-item label="名称" prop="{module}Name">
          <el-input v-model="formData.{module}Name" placeholder="请输入名称" />
        </el-form-item>

        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="formData.status">
            <el-radio :label="1">启用</el-radio>
            <el-radio :label="0">禁用</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="描述" prop="description">
          <el-input 
            v-model="formData.description" 
            type="textarea" 
            :rows="4"
            placeholder="请输入描述"
          />
        </el-form-item>

        <el-form-item>
          <el-button type="primary" @click="handleSubmit">保存</el-button>
          <el-button @click="handleCancel">取消</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import { get{Module}Detail, save{Module}, update{Module} } from '@/api/{module}'
import type { {Module}Form } from '@/types/{module}'
import type { FormInstance, FormRules } from 'element-plus'

const route = useRoute()
const router = useRouter()
const formRef = ref<FormInstance>()

const isEdit = ref(false)
const id = ref<number>()

// 表单数据 - 字段名与后端DTO一致
const formData = reactive<{Module}Form>({
  id: undefined,
  {module}Name: '',
  status: 1,
  description: ''
})

// 表单校验规则
const formRules: FormRules = {
  {module}Name: [
    { required: true, message: '请输入名称', trigger: 'blur' },
    { min: 2, max: 50, message: '长度2-50个字符', trigger: 'blur' }
  ],
  status: [
    { required: true, message: '请选择状态', trigger: 'change' }
  ]
}

// 提交
const handleSubmit = async () => {
  if (!formRef.value) return
  
  await formRef.value.validate()
  
  try {
    if (isEdit.value) {
      await update{Module}(formData)
    } else {
      await save{Module}(formData)
    }
    ElMessage.success('保存成功')
    router.back()
  } catch (error) {
    // 错误处理
  }
}

// 取消
const handleCancel = () => {
  router.back()
}

// 加载详情
const loadDetail = async (detailId: number) => {
  try {
    const res = await get{Module}Detail(detailId)
    Object.assign(formData, res.data)
  } catch (error) {
    // 错误处理
  }
}

onMounted(() => {
  const queryId = route.query.id
  if (queryId) {
    isEdit.value = true
    id.value = Number(queryId)
    loadDetail(id.value)
  }
})
</script>

<style scoped>
.page-container {
  padding: 20px;
}
.form-card {
  max-width: 800px;
}
</style>
```

### 详情页模板 (Detail.vue)

```vue
<!-- src/pages/{role}/{module}/Detail.vue -->
<template>
  <div class="page-container">
    <el-card>
      <template #header>
        <div class="card-header">
          <span>详情</span>
          <el-button @click="handleBack">返回</el-button>
        </div>
      </template>

      <el-descriptions :column="2" border>
        <el-descriptions-item label="ID">{{ detail.id }}</el-descriptions-item>
        <el-descriptions-item label="名称">{{ detail.{module}Name }}</el-descriptions-item>
        <el-descriptions-item label="状态">
          <StatusTag :status="detail.status" />
        </el-descriptions-item>
        <el-descriptions-item label="创建时间">{{ detail.createTime }}</el-descriptions-item>
        <el-descriptions-item label="描述" :span="2">{{ detail.description }}</el-descriptions-item>
      </el-descriptions>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import StatusTag from '@/components/StatusTag.vue'
import { get{Module}Detail } from '@/api/{module}'
import type { {Module}Info } from '@/types/{module}'

const route = useRoute()
const router = useRouter()

const detail = ref<Partial<{Module}Info>>({})

const loadDetail = async () => {
  const id = route.params.id
  if (!id) return
  
  try {
    const res = await get{Module}Detail(Number(id))
    detail.value = res.data
  } catch (error) {
    // 错误处理
  }
}

const handleBack = () => {
  router.back()
}

onMounted(loadDetail)
</script>

<style scoped>
.page-container {
  padding: 20px;
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>
```

---

## 3. 路由配置模板

```typescript
// src/router/index.ts
import { createRouter, createWebHistory } from 'vue-router'
import { useUserStore } from '@/stores/user'

const routes = [
  {
    path: '/',
    redirect: '/admin'
  },
  {
    path: '/admin',
    component: () => import('@/layouts/AdminLayout.vue'),
    meta: { role: 'admin' },
    children: [
      {
        path: 'dashboard',
        component: () => import('@/pages/admin/Dashboard.vue'),
        meta: { title: '仪表盘' }
      },
      {
        path: '{module}/list',
        component: () => import('@/pages/admin/{module}/List.vue'),
        meta: { title: '{模块}列表' }
      },
      {
        path: '{module}/detail/:id',
        component: () => import('@/pages/admin/{module}/Detail.vue'),
        meta: { title: '{模块}详情' }
      },
      {
        path: '{module}/form',
        component: () => import('@/pages/admin/{module}/Form.vue'),
        meta: { title: '{模块}表单' }
      }
    ]
  },
  {
    path: '/mch',
    component: () => import('@/layouts/MchLayout.vue'),
    meta: { role: 'mch' },
    children: [
      // 商户端路由...
    ]
  },
  {
    path: '/login',
    component: () => import('@/pages/auth/Login.vue'),
    meta: { public: true }
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
})

// 路由守卫
router.beforeEach((to, from, next) => {
  const userStore = useUserStore()
  
  // 公开页面直接放行
  if (to.meta.public) {
    return next()
  }
  
  // 检查登录状态
  if (!userStore.token) {
    return next('/login')
  }
  
  // 检查角色权限
  if (to.meta.role && to.meta.role !== userStore.role) {
    return next('/403')
  }
  
  next()
})

export default router
```

---

## 4. 字段一致性原则应用

### 命名映射规则

| 层级 | 命名规范 | 示例 |
|------|---------|------|
| 数据库字段 | snake_case | `user_name`, `create_time` |
| 后端DTO字段 | camelCase | `userName`, `createTime` |
| 前端表单字段 | camelCase | `userName`, `createTime` |
| 前端组件prop | camelCase | `userName` |
| 路由参数 | kebab-case | `/user-profile`, `/order-list` |

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
await page.locator('input[name="userName"]').fill('test') // name 属性匹配
```
