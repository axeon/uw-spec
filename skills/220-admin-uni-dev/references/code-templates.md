# 管理端UniApp页面代码模板

> 以下模板展示通用模式。实际开发中，类型从 `@/api/` 的生成代码导入，API 函数也从 `@/api/` 导入。

---

## 列表页模板 (list.vue)

管理端列表页：顶部搜索 + 筛选 + 数据列表 + 悬浮新增按钮

```vue
<!-- src/packages/{module}/{role}/{Feature}/{Entity}/index.vue -->
<!-- 示例：src/packages/uwAuthCenter/root/account/group/index.vue -->
<template>
  <view class="page-container">
    <!-- 强制使用 GbNavbar 自定义导航栏 -->
    <GbNavbar
      :title="t('/uwAuthCenter/root/account/group/title')"
      @back="goBack"
    />

    <!-- 搜索筛选区 -->
    <view class="search-bar">
      <uv-search
        v-model="keyword"
        :placeholder="t('/uwAuthCenter/root/account/group/searchPlaceholder')"
        :showAction="true"
        :actionText="t('search')"
        @search="handleSearch"
        @custom="handleSearch"
      />
      <view class="filter-row">
        <text
          v-for="filter in filters"
          :key="filter.value"
          class="filter-item"
          :class="{ active: currentFilter === filter.value }"
          @click="currentFilter = filter.value"
        >
          {{ filter.label }}
        </text>
      </view>
    </view>

    <!-- 数据列表 -->
    <view class="list">
      <view
        v-for="item in dataList"
        :key="item.id"
        class="list-item"
        @click="handleDetail(item)"
      >
        <view class="item-header">
          <text class="item-title">{{ item.{entity}Name }}</text>
          <uv-tag
            :text="item.statusText"
            :type="item.status === 1 ? 'success' : 'info'"
          />
        </view>
        <view class="item-meta">
          <text class="meta-text">{{ item.createTime }}</text>
          <text class="meta-text">{{ item.operator }}</text>
        </view>

        <!-- 操作按钮组（带权限控制） -->
        <view v-if="item.state !== -1" class="item-actions">
          <view
            v-if="item.state === 0 && usePermission('enable')"
            class="btn btn-success"
            @click="handleEnable(item)"
          >
            {{ t("status.enabled") }}
          </view>
          <view
            v-if="item.state === 1 && usePermission('disable')"
            class="btn btn-info"
            @click="handleDisable(item)"
          >
            {{ t("status.disabled") }}
          </view>
          <view
            v-if="item.state === 1 && usePermission('update')"
            class="btn btn-primary"
            @click="handleEdit(item)"
          >
            {{ t("edit") }}
          </view>
          <view
            v-if="item.state === 0 && usePermission('delete')"
            class="btn btn-danger"
            @click="handleDelete(item)"
          >
            {{ t("delete") }}
          </view>
        </view>
      </view>

      <!-- 空状态 -->
      <view v-if="!loading && dataList.length === 0" class="empty">
        <uv-icon name="folder" size="60" color="#c8c9cc"></uv-icon>
        <text class="empty__text">{{
          t("/uwAuthCenter/root/account/group/noData")
        }}</text>
      </view>

      <!-- 加载状态 -->
      <view v-if="dataList.length > 0" class="loadmore">
        {{
          loading
            ? t("loading")
            : finished
              ? t("noMore")
              : t("loadMore")
        }}
      </view>
    </view>

    <!-- 悬浮新增按钮 -->
    <GbFabButton v-if="usePermission('save')" @click="handleAdd" />
  </view>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { onLoad, onPullDownRefresh, onReachBottom } from '@dcloudio/uni-app'
import { useI18n } from 'vue-i18n'
import GbNavbar from '@/components/GbNavbar/index.vue'
import GbFabButton from '@/components/GbFabButton/index.vue'
import { root{Feature}{Entity}List } from '@/api/uwAuthCenterRootApi'
import type { {Entity}Info } from '@/api/uwAuthCenterRootApi'
import { usePermission } from '@/hooks/usePermission'

const { t } = useI18n()

// ✅ 注意：pageSize 必须在 queryParams 之前声明！
const pageSize = 10

const dataList = ref<{Entity}Info[]>([])
const loading = ref(false)
const finished = ref(false)
const keyword = ref('')
const currentFilter = ref('all')
const queryParams = reactive({
  $pg: 1,
  $rn: pageSize,
  keyword: '',
})

const filters = [
  { label: t('all'), value: 'all' },
  { label: t('status.pending'), value: 'pending' },
  { label: t('status.approved'), value: 'approved' },
]

const getList = async (append = false) => {
  if (loading.value) return
  loading.value = true
  try {
    const res = await root{Feature}{Entity}List({
      ...queryParams,
      keyword: keyword.value || undefined,
    })
    if (res.state === 'success' && res.data) {
      const list = res.data?.list || []

      // ✅ 刷新时重置列表和完成状态
      if (!append) {
        dataList.value = list
        finished.value = false
      } else {
        dataList.value = [...dataList.value, ...list]
      }

      // ✅ 通过返回数据长度判断是否还有更多
      finished.value = list.length < pageSize
    } else if (res.state !== 'success') {
      uni.showToast({ title: res.msg || t('/uwAuthCenter/root/account/group/loadError'), icon: 'none' })
    }
  } catch (err) {
    console.error(err)
    uni.showToast({ title: t('/uwAuthCenter/root/account/group/loadError'), icon: 'none' })
  } finally {
    loading.value = false
    uni.stopPullDownRefresh()
  }
}

const handleSearch = () => {
  queryParams.$pg = 1
  queryParams.keyword = keyword.value
  getList(false)
}

const handleLoadMore = () => {
  if (finished.value || loading.value) return
  queryParams.$pg++
  getList(true)
}

const handleDetail = (item: {Entity}Info) => {
  uni.navigateTo({ url: `/packages/uwAuthCenter/root/account/group/detail/index?id=${item.id}` })
}

const handleAdd = () => {
  uni.navigateTo({ url: `/packages/uwAuthCenter/root/account/group/form/index` })
}

const handleEdit = (item: {Entity}Info) => {
  uni.navigateTo({ url: `/packages/uwAuthCenter/root/account/group/form/index?id=${item.id}` })
}

const handleEnable = (item: {Entity}Info) => {
  uni.showModal({
    title: t('confirm'),
    content: t('/uwAuthCenter/root/account/group/enableConfirm', { groupName: item.groupName }),
    success: async (res) => {
      if (res.confirm) {
        try {
          await root{Feature}{Entity}Enable({ id: item.id, remark: '' })
          uni.showToast({ title: t('saveSuccess'), icon: 'success' })
          getList(false)
        } catch (error: any) {
          uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
        }
      }
    },
  })
}

const handleDisable = (item: {Entity}Info) => {
  uni.showModal({
    title: t('confirm'),
    content: t('/uwAuthCenter/root/account/group/disableConfirm', { groupName: item.groupName }),
    success: async (res) => {
      if (res.confirm) {
        try {
          await root{Feature}{Entity}Disable({ id: item.id, remark: '' })
          uni.showToast({ title: t('saveSuccess'), icon: 'success' })
          getList(false)
        } catch (error: any) {
          uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
        }
      }
    },
  })
}

const handleDelete = (item: {Entity}Info) => {
  uni.showModal({
    title: t('confirmDelete'),
    content: t('/uwAuthCenter/root/account/group/deleteConfirm', { groupName: item.groupName }),
    confirmColor: '#fa3534',
    success: async (res) => {
      if (res.confirm) {
        try {
          await root{Feature}{Entity}Delete({ id: item.id, remark: '' })
          uni.showToast({ title: t('deleteSuccess'), icon: 'success' })
          getList(false)
        } catch (error: any) {
          uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
        }
      }
    },
  })
}

const goBack = () => {
  uni.navigateBack()
}

// ✅ 下拉刷新
onPullDownRefresh(() => {
  queryParams.$pg = 1
  getList(false)
})

// ✅ 上拉加载
onReachBottom(() => {
  if (finished.value || loading.value) return
  queryParams.$pg++
  getList(true)
})

onMounted(() => {
  getList()
})
</script>

<style lang="scss" scoped>
.page-container {
  min-height: 100vh;
  background-color: #f5f6f8;
  padding-bottom: 180rpx;
}

.search-bar {
  padding: 16rpx 24rpx 0;
}

.filter-row {
  display: flex;
  gap: 16rpx;
  padding: 16rpx 0;
}

.filter-item {
  padding: 8rpx 24rpx;
  font-size: 26rpx;
  color: #666;
  border-radius: 32rpx;
  background-color: #fff;

  &.active {
    color: #fff;
    background-color: var(--color-primary);
  }
}

.list {
  padding: 8rpx 24rpx 0;
}

.list-item {
  margin-top: 20rpx;
  background-color: #fff;
  border-radius: 20rpx;
  padding: 24rpx 28rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.04);
}

.item-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 16rpx;
  border-bottom: 1rpx solid #f0f0f0;
}

.item-title {
  font-size: 32rpx;
  font-weight: 600;
  color: #1a1a1a;
}

.item-meta {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 18rpx 0;
}

.meta-text {
  font-size: 26rpx;
  color: #999;
}

.item-actions {
  display: flex;
  justify-content: flex-end;
  gap: 16rpx;
  padding-top: 20rpx;
  border-top: 1rpx solid #f0f0f0;
  margin-top: 16rpx;
}

.btn {
  padding: 12rpx 32rpx;
  font-size: 26rpx;
  border-radius: 32rpx;
  text-align: center;
}

.btn-success {
  color: #fff;
  background-color: #19be6b;
}

.btn-info {
  color: #fff;
  background-color: #909399;
}

.btn-primary {
  color: #fff;
  background-color: var(--color-primary);
}

.btn-danger {
  color: #fff;
  background-color: #fa3534;
}

.empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 120rpx 0;

  &__text {
    margin-top: 24rpx;
    font-size: 28rpx;
    color: #c8c9cc;
  }
}

.loadmore {
  text-align: center;
  font-size: 24rpx;
  color: #999;
  padding: 32rpx 0;
}
</style>
```

## 详情页模板 (detail.vue)

管理端详情页：信息展示 + 操作按钮（根据权限）

```vue
<!-- src/packages/{module}/{role}/{Feature}/{Entity}/detail/index.vue -->
<!-- 示例：src/packages/uwAuthCenter/root/account/group/detail/index.vue -->
<template>
  <view class="page-container" v-if="detail">
    <!-- 强制使用 GbNavbar 自定义导航栏 -->
    <GbNavbar
      :title="t('/uwAuthCenter/root/account/group/detailTitle')"
      @back="goBack"
    />

    <view class="detail-card">
      <view class="detail-header">
        <text class="detail-title">{{ detail.{entity}Name }}</text>
        <uv-tag
          :text="detail.statusText"
          :type="detail.status === 1 ? 'success' : 'info'"
        />
      </view>
      <view class="detail-section">
        <view class="section-title">{{ t("baseInfo") }}</view>
        <view class="info-row" v-for="field in infoFields" :key="field.key">
          <text class="info-label">{{ field.label }}</text>
          <text class="info-value">{{ detail[field.key] }}</text>
        </view>
      </view>
    </view>

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
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import { useI18n } from 'vue-i18n'
import GbNavbar from '@/components/GbNavbar/index.vue'
import { root{Feature}{Entity}Load, root{Feature}{Entity}Delete } from '@/api/uwAuthCenterRootApi'
import type { {Entity}Info } from '@/api/uwAuthCenterRootApi'
import { usePermission } from '@/hooks/usePermission'

const { t } = useI18n()
const detail = ref<{Entity}Info | null>(null)
const infoFields = [
  { label: t('name'), key: '{entity}Name' },
  { label: t('status'), key: 'statusText' },
  { label: t('createTime'), key: 'createTime' },
]

onLoad(async (options) => {
  const id = options?.id
  if (id) {
    const res = await root{Feature}{Entity}Load({ id: Number(id) })
    detail.value = res.data ?? null
  }
})

const handleEdit = () => {
  uni.navigateTo({ url: `/packages/uwAuthCenter/root/account/group/form/index?id=${detail.value?.id}` })
}

const handleDelete = () => {
  uni.showModal({
    title: t('confirmDelete'),
    content: t('deleteNotRecoverable'),
    success: async (res) => {
      if (res.confirm) {
        try {
          await root{Feature}{Entity}Delete({ id: detail.value!.id })
          uni.showToast({ title: t('deleteSuccess'), icon: 'success' })
          setTimeout(() => uni.navigateBack(), 1500)
        } catch (error: any) {
          uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
        }
      }
    },
  })
}
</script>

<style lang="scss" scoped>
.page-container {
  min-height: 100vh;
  background-color: #f5f6f8;
  padding-bottom: 48rpx;
}

.detail-card {
  margin: 24rpx;
  background-color: #fff;
  border-radius: 20rpx;
  padding: 24rpx 28rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.04);
}

.detail-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-bottom: 24rpx;
  border-bottom: 1rpx solid #f0f0f0;
}

.detail-title {
  font-size: 36rpx;
  font-weight: 600;
  color: #1a1a1a;
}

.detail-section {
  padding-top: 24rpx;
}

.section-title {
  font-size: 28rpx;
  font-weight: 600;
  color: #1a1a1a;
  margin-bottom: 16rpx;
}

.info-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding: 16rpx 0;
  border-bottom: 1rpx solid #f7f7f7;
}

.info-label {
  font-size: 26rpx;
  color: #999;
  flex-shrink: 0;
  margin-right: 32rpx;
}

.info-value {
  font-size: 26rpx;
  color: #333;
  text-align: right;
  word-break: break-all;
}

.action-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 24rpx;
  padding-bottom: calc(24rpx + env(safe-area-inset-bottom));
  background-color: #fff;
  display: flex;
  gap: 16rpx;
  box-shadow: 0 -4rpx 16rpx rgba(0, 0, 0, 0.04);
}
</style>
```

## 表单页模板 (form.vue)

管理端表单页：字段输入 + 底部提交按钮

```vue
<!-- src/packages/{module}/{role}/{Feature}/{Entity}/form/index.vue -->
<!-- 示例：src/packages/uwAuthCenter/root/account/group/form/index.vue -->
<template>
  <view class="page-container">
    <!-- 强制使用 GbNavbar 自定义导航栏 -->
    <GbNavbar
      :title="
        t(
          isEdit
            ? '/uwAuthCenter/root/account/group/editTitle'
            : '/uwAuthCenter/root/account/group/addTitle',
        )
      "
      @back="goBack"
    />

    <view class="form-card">
      <uv-form ref="uvFormRef" :model="form" :rules="rules" labelPosition="top">
        <uv-form-item
          :label="t('/uwAuthCenter/root/account/group/groupNameLabel')"
          prop="{entity}Name"
          required
        >
          <uv-input
            v-model="form.{entity}Name"
            :placeholder="
              t('/uwAuthCenter/root/account/group/groupNamePlaceholder')
            "
          />
        </uv-form-item>
        <uv-form-item
          :label="t('/uwAuthCenter/root/account/group/statusLabel')"
          prop="status"
        >
          <uv-picker
            :columns="[statusOptions]"
            :modelValue="[form.status]"
            @confirm="onStatusChange"
          />
        </uv-form-item>
        <uv-form-item
          :label="t('/uwAuthCenter/root/account/group/remarkLabel')"
          prop="remark"
        >
          <uv-textarea
            v-model="form.remark"
            :placeholder="
              t('/uwAuthCenter/root/account/group/remarkPlaceholder')
            "
          />
        </uv-form-item>
      </uv-form>
    </view>

    <view class="submit-bar">
      <uv-button
        type="primary"
        :text="t('submit')"
        :loading="submitting"
        @click="handleSubmit"
      />
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import { onLoad } from '@dcloudio/uni-app'
import { useI18n } from 'vue-i18n'
import GbNavbar from '@/components/GbNavbar/index.vue'
import { root{Feature}{Entity}Load, root{Feature}{Entity}Create, root{Feature}{Entity}Update } from '@/api/uwAuthCenterRootApi'
import type { {Entity}Form } from '@/api/uwAuthCenterRootApi'

const { t } = useI18n()
const uvFormRef = ref<any>(null)
const submitting = ref(false)
const isEdit = ref(false)

const form = ref<{Entity}Form>({
  {entity}Name: '',
  status: undefined,
  remark: '',
})

const statusOptions = [
  { label: t('status.pending'), value: 0 },
  { label: t('status.enabled'), value: 1 },
  { label: t('status.disabled'), value: 2 },
]

const rules = {
  {entity}Name: [
    { required: true, message: t('/uwAuthCenter/root/account/group/groupNameRequired'), trigger: 'blur' },
    { max: 50, message: t('maxLength', { length: 50 }), trigger: 'blur' },
  ],
}

onLoad(async (options) => {
  const id = options?.id
  if (id) {
    isEdit.value = true
    const res = await root{Feature}{Entity}Load({ id: Number(id) })
    if (res.data) form.value = res.data as unknown as {Entity}Form
  }
})

const onStatusChange = (e: { value: any[] }) => {
  form.value.status = e.value[0]
}

const handleSubmit = async () => {
  const valid = await uvFormRef.value?.validate?.()
  if (!valid) return

  submitting.value = true
  try {
    if (isEdit.value) {
      await root{Feature}{Entity}Update({ data: form.value })
    } else {
      await root{Feature}{Entity}Create({ data: form.value })
    }
    uni.showToast({ title: t('saveSuccess'), icon: 'success' })
    setTimeout(() => uni.navigateBack(), 1500)
  } catch (error: any) {
    uni.showToast({ title: error?.msg || t('submitError'), icon: 'none' })
  } finally {
    submitting.value = false
  }
}
</script>

<style lang="scss" scoped>
.page-container {
  min-height: 100vh;
  background-color: #f5f6f8;
  padding-bottom: 200rpx;
}

.form-card {
  margin: 24rpx;
  background-color: #fff;
  border-radius: 20rpx;
  padding: 24rpx 28rpx;
  box-shadow: 0 8rpx 24rpx rgba(0, 0, 0, 0.04);
}

.submit-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  padding: 24rpx;
  padding-bottom: calc(24rpx + env(safe-area-inset-bottom));
  background-color: #fff;
  box-shadow: 0 -4rpx 16rpx rgba(0, 0, 0, 0.04);
}
</style>
```
