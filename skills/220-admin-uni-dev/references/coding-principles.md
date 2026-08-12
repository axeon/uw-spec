# Admin UniApp 编码原则

> version: "1.1.0"
> 被 220-admin-uni-dev、221-admin-uni-dev-review、620-feature-dev、720-bugfix-dev 共同引用。
> 本文件是编码规范的**唯一权威来源**，其他文件不再重复列举规则。

## 四条核心原则

### 原则一：集中管理（Single Source of Truth）

**判断标准**：如果一个配置在多个地方可能被使用，或者属于字典/枚举/映射类数据，它应该集中管理。

| 集中到哪里    | 管什么                         | 格式                     |
| ------------- | ------------------------------ | ------------------------ |
| `hooks/`      | 组合函数（权限判断、业务逻辑） | `useXxx()` 函数          |
| `store/`      | 全局状态（认证、菜单权限）     | Pinia setup 风格         |
| `components/` | UI 组件（扁平组织）            | 可复用组件               |
| `utils/`      | 工具函数                       | 纯函数                   |
| `api/`        | API 调用封装                   | 代码生成器产出，只读不改 |

**具体做法**：

- 页面内**禁止**定义重复的类型接口，应从 `@/api/` 的类型中导入
- 相同的业务逻辑抽取到 `hooks/useXxx.ts`
- 权限判断统一使用 `usePermission` composable
- API 调用只通过 `api/` 层，禁止页面内直接 `uni.request`

### 原则二：类型安全（No Escape Hatches）

**判断标准**：如果 TypeScript 编译器无法推断类型，说明代码有问题。

| 禁止               | 替代方案                                                             |
| ------------------ | -------------------------------------------------------------------- |
| `any` 类型         | 定义具体类型或使用泛型                                               |
| 无类型 Props       | 使用 `defineProps<T>()` 泛型风格                                     |
| 无类型 Emits       | 使用 `defineEmits<T>()` 泛型风格                                     |
| 隐式 any 参数      | 使用具体的接口类型                                                   |
| `@ts-ignore`       | 修正类型定义                                                         |
| 松散相等 `==`/`!=` | 严格相等 `===`/`!==`（含 `null`/`undefined` 判断，避免隐式类型转换） |

> **UniApp 框架限制**：`switch` 组件的 `@change` 事件回调参数类型在 UniApp 中为 `any`，此为框架限制，可保留。

### 原则三：项目一致性（Use What Exists）

**判断标准**：在编写任何代码前，先检查项目中是否已有相同或类似功能的实现。

| 场景         | 做法                                                           |
| ------------ | -------------------------------------------------------------- |
| 网络请求     | 使用 `api/` 封装，禁止直接 `uni.request`                       |
| 权限判断     | 使用 `usePermission` composable                                |
| 导航跳转     | 使用 `uni.navigateTo` / `uni.navigateBack`，导航栈模式         |
| 列表数据     | 统一使用 `res.data?.list`（`res.data?.results` 仅兼容旧代码）  |
| 表单校验     | 前端提交前校验，不等后端返回                                   |
| Toast 提示   | 使用 `uni.showToast`，统一格式                                 |
| 页面生命周期 | 页面级用 `onLoad`/`onShow`，组件级用 `onMounted`/`onUnmounted` |

### 原则四：代码可读性（Self-Documenting Code）

**判断标准**：一个新团队成员能否在不看注释的情况下理解代码意图。

| 禁止                            | 替代方案                     |
| ------------------------------- | ---------------------------- |
| v-for 单字母变量 `v-for="i in"` | 描述性名称 `v-for="item in"` |
| 数组方法单字母参数 `(a, b) =>`  | `(item, index) =>`           |
| 硬编码魔法数字/字符串           | 提取为命名常量               |
| 嵌套三元 `a ? b : c ? d : e`    | computed 属性或方法          |
| 超长单文件组件                  | 拆分为子组件和 composables   |

### 原则五：主题适配（Theme-Aware Design）

**判断标准**：切换主题后，所有业务颜色是否立即同步变化。

| 禁止                     | 替代方案                             |
| ------------------------ | ------------------------------------ |
| 硬编码颜色值 `#409EFF`   | 使用 CSS 变量 `var(--color-primary)` |
| 在 script 中定义颜色常量 | 在 style 中使用 CSS 变量             |
| 自定义组件不支持主题切换 | 组件内部使用 CSS 变量定义颜色        |

**具体做法**：

- 所有业务颜色必须使用 CSS 变量定义
- uv-ui 组件主题已通过 CSS 变量覆盖实现同步切换
- 新增组件时，直接使用项目现有 CSS 变量

### 原则六：样式优先使用 Tailwind（Tailwind First · 推荐）

**判断标准**：**优先使用 Tailwind**，但不做硬性强制；以"哪种写法更清晰、更易维护"为最终判断标准。项目已集成 `tailwindcss@^4.3` 与 `weapp-tailwindcss`，全平台可用。

| 场景                          | 推荐做法                                                          | 备选                                              |
| ----------------------------- | ----------------------------------------------------------------- | ------------------------------------------------- |
| 布局（flex/grid/gap/间距）    | 直接用 Tailwind：`flex items-center justify-between gap-4 p-6`    | 复杂/可复用样式可用 SCSS + BEM                    |
| 尺寸/边距                     | Tailwind 数值/任意值：`w-full h-24 mt-4`、`w-[112rpx] h-[112rpx]` | 数值密集且叠加多时可抽 SCSS 类                    |
| 圆角/阴影/边框                | `rounded-full shadow-lg border border-gray-200`                   | 复用度高时抽 SCSS 组件类                          |
| 文字样式                      | `text-sm font-semibold text-gray-500 truncate`                    | 大段文本样式可用 SCSS                             |
| **主题色**                    | 通过 CSS 变量：`class="text-[var(--color-primary)]"`              | 用 Tailwind 内置调色板（无法主题切换，❌ 不推荐）  |
| **rpx 单位**                  | Tailwind 任意值：`class="p-[24rpx] rounded-[20rpx] h-[88rpx]"`    | -                                                 |
| 状态类（active/disabled/...） | `:class="[base, isActive && 'bg-primary text-white']"`            | 也可通过 SCSS `&--active` 组织                    |
| 复杂动画/伪元素/深度选择器    | `<style scoped>` SCSS                                             | -                                                 |
| 可复用业务组件内部样式        | SCSS + BEM（避免使用方传大量原子类）                              | -                                                 |

**具体做法**：

- **默认写 Tailwind**：模板里简单、一次性的布局/间距/字体等样式，优先用原子类。
- **可以用 SCSS 的场景**（无需强制回避）：
  1. 可复用业务组件（`GbNavbar`、`GbFabButton` 等）内部样式，外部通过 props 控制主题
  2. 5 个以上原子类叠加同一元素、可读性明显下降
  3. `:deep()` 深度选择器覆盖第三方组件（uv-ui 等）
  4. `@keyframes` 自定义动画
  5. 伪元素 `::before` / `::after`
  6. `env(safe-area-inset-bottom)` 等 CSS 环境变量
  7. 项目已有大量 SCSS，保持一致性优先
- **主题色不用 Tailwind 内置调色板**（`text-blue-500` 等），主题相关一律通过 CSS 变量 + `text-[var(--color-primary)]` / `bg-[var(--color-primary)]` 任意值语法。
- **rpx 单位用任意值语法**：`w-[750rpx]`、`p-[24rpx]`、`text-[26rpx]`；避免 Tailwind 默认的 `rem` 单位污染移动端布局。
- **小程序限制**：`weapp-tailwindcss` 已处理绝大多数原子类；`space-x-*` / `divide-*` 等依赖后代通配选择器的 API 在小程序里表现不稳定，改用 `gap-*` 与显式 `border`。
- **不要写自定义 utility 类**：如需复用，抽成组件而非用 `@apply` 造工具类。

**参考性检查**（不作为红线，仅用于统计 Tailwind 使用比例）：

```bash
# 统计 <style scoped> 中仍以 SCSS 声明简单属性的位置
grep -rn --include="*.vue" -E '(display:\s*flex|padding:\s*[0-9]+rpx|margin:\s*[0-9]+rpx|font-size:\s*[0-9]+rpx|color:\s*#|background-color:\s*#)' src/packages/ src/components/
```

- 命中并不意味着违规，应结合"是否为可复用样式、是否复杂、是否可读"综合判断
- 如果一行 SCSS 的语义更清楚，就用 SCSS；不必为了 Tailwind 而 Tailwind

**示例对比**：

场景 A · 简单一次性布局 → **推荐 Tailwind**

```vue
<template>
  <view class="flex items-center justify-between px-[28rpx] py-[24rpx] bg-white rounded-[20rpx]">
    <text class="text-[32rpx] font-semibold text-[#1a1a1a]">部门名称</text>
    <text class="text-[24rpx] text-[var(--color-primary)]">正常</text>
  </view>
</template>
```

场景 B · 可复用业务组件（如 GbFabButton）→ **推荐 SCSS**

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
  background: linear-gradient(135deg, var(--color-primary), var(--color-primary-secondary));
  box-shadow: 0 8rpx 24rpx rgba(60, 156, 255, 0.4);

  &__icon {
    font-size: 60rpx;
    color: #fff;
  }
}
</style>
```

### 原则七：列表页面规范（List Page Standard）

**判断标准**：列表页面是否支持下拉刷新、上拉加载，且功能正常工作。

#### 6.1 变量声明顺序

**禁止**：在常量声明之前使用该常量

```typescript
// ❌ 错误示例
const queryParams = reactive({
  $rn: pageSize, // pageSize 还没声明！
});
const pageSize = 10;
```

**正确做法**：先声明常量，再使用

```typescript
// ✅ 正确示例
const pageSize = 10; // 先声明常量
const queryParams = reactive({
  $rn: pageSize, // 再使用
});
```

#### 6.2 列表状态管理

| 状态变量   | 类型           | 说明                         |
| ---------- | -------------- | ---------------------------- |
| `dataList` | `ref<T[]>`     | 列表数据                     |
| `loading`  | `ref<boolean>` | 加载状态，防止重复请求       |
| `finished` | `ref<boolean>` | 是否加载完成，没有更多数据了 |
| `pageSize` | `number`       | 每页条数，使用常量定义       |

#### 6.3 上拉加载判断逻辑

**禁止**：依赖 `total` 总数判断（可能不准确）

```typescript
// ❌ 不推荐
const total = ref(0);
const finished = computed(() => dataList.value.length >= total.value);
```

**正确做法**：通过返回数据长度判断

```typescript
// ✅ 推荐
const pageSize = 10;
const finished = ref(false);

const getList = async (append = false) => {
  // ...
  const list = res.data?.list || [];

  // 刷新时重置完成状态
  if (!append) {
    dataList.value = list;
    finished.value = false;
  } else {
    dataList.value = [...dataList.value, ...list];
  }

  // 通过返回数据长度判断是否还有更多
  finished.value = list.length < pageSize;
};
```

#### 6.4 下拉刷新与上拉加载实现

```typescript
// 下拉刷新
onPullDownRefresh(() => {
  queryParams.$pg = 1;
  getList(false); // 不追加，重新加载
});

// 上拉加载
onReachBottom(() => {
  if (finished.value || loading.value) return; // 已完成或加载中，直接返回
  queryParams.$pg++;
  getList(true); // 追加数据
});
```

**pages.json 配置**：

```json
{
  "style": {
    "navigationBarTitleText": "xxx管理",
    "navigationStyle": "custom",
    "enablePullDownRefresh": true // 开启下拉刷新
  }
}
```

## 数据结构规范

> 所有 API 响应遵循统一的包装类型，解析时必须按以下规则。

| API 类型     | 返回类型                    | 取值方式                                                     |
| ------------ | --------------------------- | ------------------------------------------------------------ |
| 列表 API     | `ResponseData<DataList<T>>` | `res.data?.list` → `T[]`（`res.data?.results` 仅兼容旧代码） |
| 实体 API     | `ResponseData<T>`           | `res.data` → `T`                                             |
| 无返回值 API | `ResponseData<void>`        | 检查 `res.state === 'success'`                               |

**禁止使用**：`res.data?.rows`

## 自动化验证

开发完成后，执行以下命令验证编码规范：

```bash
cd E:/work/uw-uniapp-template-main

# 1. any 类型检查（应为 0，框架限制的 switch 事件除外）
grep -rn ': any' src/ --include="*.vue" --include="*.ts" | grep -v 'node_modules' | grep -v '@change' | wc -l

# 2. 直接 uni.request 调用（应为 0，应使用封装的 request）
grep -rn 'uni\.request(' src/pages/ --include="*.vue" | wc -l

# 3. DataList 字段错误（应为 0，禁止用 rows；results 仅旧代码兼容）
grep -rn 'res\.data\?\.rows\b' src/ --include="*.vue" --include="*.ts" | wc -l

# 4. ref 双重调用（应为 0，运行时必崩）
grep -rn 'ref<.*>(.*)(.*)' src/ --include="*.vue" --include="*.ts" | wc -l

# 5. v-for 无 key（应为 0）
grep -rn 'v-for=' src/pages/ --include="*.vue" | grep -v ':key=' | wc -l

# 6. 变量声明顺序检查：查找在 reactive 中使用了后面才声明的变量的情况
# 检查模式：const queryParams = reactive({... xyz ...}); const xyz = ...
echo "检查变量声明顺序问题..."

# 7. TypeScript 编译检查
pnpm vue-tsc --noEmit

# 8. 多端编译验证
pnpm build:h5
pnpm build:mp-weixin
```

## 项目基础架构速查

### 目录结构

> 以当前项目 `uw-uniapp-template-main` 实际目录为标准。

```
src/pages/                         # 通用页面（home/login/mine/tools）
src/packages/                      # 分包页面
  ├── about/                       # 通用页面：关于
  ├── resetPassword/               # 通用页面：重置密码
  ├── userInfo/                    # 通用页面：个人资料
  └── {module}/                    # 业务模块
      └── {role}/                  # admin / mch / saas
          └── {Feature}/           # 功能模块
              └── {Entity}/        # 实体/页面
                  └── index.vue
src/components/                    # 组件（扁平组织）
src/hooks/                         # 组合函数
src/store/                         # Pinia 状态管理
  ├── index.ts                     # Pinia 实例
  ├── main.ts                      # 全局状态
  └── user.ts                      # 用户状态
src/api/                           # API 调用（代码生成器产出，只读不改）
src/utils/                         # 工具函数
pages.json                         # 页面路由配置
```

### 页面路由配置

```json
{
  "pages": [
    {
      "path": "pages/home/index",
      "style": { "navigationBarTitleText": "工作台" }
    },
    {
      "path": "pages/tools/index",
      "style": { "navigationBarTitleText": "常用工具" }
    },
    {
      "path": "pages/mine/index",
      "style": { "navigationBarTitleText": "我的" }
    },
    {
      "path": "pages/login/index",
      "style": { "navigationBarTitleText": "登录" }
    }
  ],
  "subPackages": [
    {
      "root": "packages",
      "pages": [
        {
          "path": "uwAuthCenter/root/account/group/index",
          "style": {
            "navigationBarTitleText": "部门管理",
            "navigationStyle": "custom",
            "enablePullDownRefresh": true
          }
        }
      ]
    }
  ],
  "tabBar": {
    "list": [
      { "pagePath": "pages/home/index", "text": "管理菜单" },
      { "pagePath": "pages/tools/index", "text": "常用工具" },
      { "pagePath": "pages/mine/index", "text": "我的" }
    ]
  }
}
```

### 页面生命周期选择

- **页面级逻辑**（获取参数、每次刷新数据）→ `onLoad` / `onShow`
- **组件级逻辑**（初始化、销毁）→ `onMounted` / `onUnmounted`
- **当前项目固定 3 个 TabBar 入口**（管理菜单、常用工具、我的），其余页面采用导航栈模式
