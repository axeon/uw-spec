# Admin Web端原型评审检查清单

> **评审原则**：每个检查项的标准以 [220-admin-web-design/SKILL.md](../../220-admin-web-design/SKILL.md) 中的原始约定为准。下表"源技能依据"列指向源技能的具体章节，评审时必须回到源技能核实。

## 需求符合度检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 功能覆盖 | PRD中所有功能点都有对应页面 | Critical | 220 → Phase 0 需求确认 |
| 2 | 业务流程 | 核心业务流程可在原型中走通 | Critical | 220 → Phase 0 需求确认 |
| 3 | 验收标准 | 每个AC在原型中有体现 | Major | 220 → Phase 0 需求确认 |

## 用户体验检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 布局合理 | 页面布局清晰, 信息层级分明 | Major | 220 → references/web-design-spec.md |
| 2 | 操作简洁 | 核心操作路径最短 | Major | 220 → Phase 0 页面分类决策 |
| 3 | 反馈明确 | 操作成功/失败有明确反馈 | Major | 220 → references/web-design-spec.md |
| 4 | 导航清晰 | 导航结构清晰, 面包屑完整 | Minor | 220 → references/web-dev-standards.md 布局规范 |

## 视觉设计检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 风格统一 | 全站风格统一, 符合ElementPlus规范 | Major | 220 → references/web-design-spec.md |
| 2 | 配色合理 | 配色方案一致, 对比度足够 | Minor | 220 → references/web-design-spec.md 色彩规范 |
| 3 | 间距一致 | 元素间距统一 | Minor | 220 → references/web-design-spec.md 间距规范 |

## 技术可行性检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 交互可实现 | 所有交互效果在Vue3+ElementPlus可实现 | Major | 220 → Phase 2 Step 5 |
| 2 | 性能可接受 | 页面加载和交互响应可接受 | Minor | 220 → Phase 2 Step 5 |
| 3 | 路由配置 | 路由配置完整, 权限守卫到位 | Major | 220 → Phase 2 Step 3 |

## 一致性检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 组件复用 | 相同功能使用相同组件 | Major | 220 → references/web-dev-standards.md |
| 2 | 命名规范 | 页面/组件命名符合PascalCase规范 | Minor | 220 → references/web-dev-standards.md 命名规范 |
| 3 | **字段一致** | **表单字段名与数据库字段名一致（camelCase）** | Critical | 220 → Phase 2 Step 4 字段一致性 |
| 4 | 状态完整 | 所有状态流转有对应页面 | Major | 220 → Phase 0 需求确认 |

## 源技能约定检查（必查）

> 这些检查项来自源技能 220 的架构约定，评审时必须对照源技能原文。

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | **页面路径角色化** | **页面路径第一级必须为用户角色（/saas/、/mch/、/admin/、/guest/）** | Critical | 220 → 架构约定 |
| 2 | **角色移动后修正** | **页面已从 pages/ 移动到目标角色目录，导入路径已修正** | Major | 220 → Phase 2 Step 2 |
| 3 | **SFC 结构顺序** | **template → script setup lang="ts" → style scoped lang="scss"** | Major | 220 → references/web-dev-standards.md SFC结构顺序 |
| 4 | **导入顺序** | **类型→组件→API→composables→响应式状态→方法→生命周期** | Minor | 220 → references/web-dev-standards.md `<script setup>` 规范 |
| 5 | **路由配置** | **角色级路由已配置，路由 meta 含 roles 字段** | Major | 220 → Phase 2 Step 3 |
| 6 | **编译通过** | **`npm run build` 编译通过，`npx vue-tsc --noEmit` 无类型错误** | Critical | 220 → Phase 2 Step 5 |
| 7 | **README 完整** | **含页面总览、角色权限映射、PRD功能点映射、路由设计** | Major | 220 → Phase 1 必须章节 |

## 管理端专项检查

| # | 检查项 | 标准 | 严重程度 | 源技能依据 |
|---|--------|------|---------|-----------|
| 1 | 数据表格 | 筛选、排序、分页、批量操作 | Major | 220 → references/web-dev-standards.md ElementPlus 表格规范 |
| 2 | 表单设计 | 字段分组、快捷编辑、验证反馈 | Major | 220 → references/web-dev-standards.md |
| 3 | 权限控制 | 菜单权限、按钮权限、数据权限 | Critical | 220 → Phase 2 Step 3 路由配置 |
| 4 | 数据可视化 | 图表直观、实时更新 | Minor | 220 → Phase 0 页面分类决策 |
