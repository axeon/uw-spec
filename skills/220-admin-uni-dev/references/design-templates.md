# 管理端UniApp移动端设计模板

> 包含 README.md 页面清单模板和 TASKS.md 任务卡片模板。遵循字段一致性原则。

---

## 1. README.md 页面清单模板

````markdown
# {project-name} 管理端移动端页面清单

## 页面总览

| 页面名         | 所属角色 | 模块         | 功能    | 实体    | 复杂度 | 涉及API                                        | 平台   |
| -------------- | -------- | ------------ | ------- | ------- | ------ | ---------------------------------------------- | ------ |
| group-list     | root     | uwAuthCenter | account | group   | 简单   | rootAccountGroupList                           | 全平台 |
| group-detail   | root     | uwAuthCenter | account | group   | 简单   | rootAccountGroupLoad                           | 全平台 |
| group-form     | root     | uwAuthCenter | account | group   | 中等   | rootAccountGroupCreate, rootAccountGroupUpdate | 全平台 |
| critLog-list   | root     | uwAuthCenter | log     | critLog | 复杂   | rootLogCritLogList                             | 全平台 |
| critLog-detail | root     | uwAuthCenter | log     | critLog | 中等   | rootLogCritLogLoad                             | 全平台 |

## 角色权限映射

| 角色 | group-list | group-form | critLog-list | critLog-detail |
| ---- | ---------- | ---------- | ------------ | -------------- |
| root | R/W        | R/W        | R            | R              |
| saas | -          | -          | -            | -              |
| mch  | -          | -          | -            | -              |

R=只读（列表/详情），W=写入（新增/编辑/删除/审核）

## PRD功能点映射

| PRD功能点 | 模块         | 功能    | 实体    | 页面           | 路径                                                | 说明                |
| --------- | ------------ | ------- | ------- | -------------- | --------------------------------------------------- | ------------------- |
| 部门管理  | uwAuthCenter | account | group   | group-list     | packages/uwAuthCenter/root/account/group/index      | 管理员查看/管理部门 |
| 部门编辑  | uwAuthCenter | account | group   | group-form     | packages/uwAuthCenter/root/account/group/form/index | 新增/编辑部门       |
| 关键日志  | uwAuthCenter | log     | critLog | critLog-list   | packages/uwAuthCenter/root/log/critLog/index        | 查看关键日志        |
| 日志详情  | uwAuthCenter | log     | critLog | critLog-detail | packages/uwAuthCenter/root/log/critLog/detail/index | 查看日志详情        |

## 路由设计

### pages.json 配置

```json
{
  "pages": [
    {
      "path": "pages/home/index",
      "style": {
        "navigationBarTitleText": "工作台",
        "navigationStyle": "custom"
      }
    },
    {
      "path": "pages/tools/index",
      "style": {
        "navigationBarTitleText": "常用工具",
        "navigationStyle": "custom"
      }
    },
    {
      "path": "pages/mine/index",
      "style": { "navigationBarTitleText": "我的", "navigationStyle": "custom" }
    },
    {
      "path": "pages/login/index",
      "style": { "navigationBarTitleText": "登录", "navigationStyle": "custom" }
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
        },
        {
          "path": "uwAuthCenter/root/account/group/detail/index",
          "style": {
            "navigationBarTitleText": "部门详情",
            "navigationStyle": "custom"
          }
        },
        {
          "path": "uwAuthCenter/root/account/group/form/index",
          "style": {
            "navigationBarTitleText": "编辑部门",
            "navigationStyle": "custom"
          }
        },
        {
          "path": "uwAuthCenter/root/log/critLog/index",
          "style": {
            "navigationBarTitleText": "关键日志",
            "navigationStyle": "custom",
            "enablePullDownRefresh": true
          }
        },
        {
          "path": "uwAuthCenter/root/log/critLog/detail/index",
          "style": {
            "navigationBarTitleText": "日志详情",
            "navigationStyle": "custom"
          }
        }
      ]
    }
  ],
  "tabBar": {
    "color": "#999999",
    "selectedColor": "#2979ff",
    "backgroundColor": "#ffffff",
    "borderStyle": "black",
    "list": [
      { "pagePath": "pages/home/index", "text": "管理菜单" },
      { "pagePath": "pages/tools/index", "text": "常用工具" },
      { "pagePath": "pages/mine/index", "text": "我的" }
    ]
  }
}
```
````

> **注意**：当前项目固定 3 个 TabBar 入口（管理菜单、常用工具、我的），其余业务页面放在 `subPackages` 中，采用导航栈模式。

## 字段一致性检查

> 表单字段名必须与后端 DTO 字段名一致（camelCase）。类型从 `@/api/` 导入，不另外定义。

| 后端DTO字段 | 前端字段   | 列表项   | 说明     |
| ----------- | ---------- | -------- | -------- |
| groupName   | groupName  | 部门名称 | 文本输入 |
| groupDesc   | groupDesc  | 部门描述 | 文本输入 |
| state       | state      | 状态     | 状态标签 |
| createDate  | createDate | 创建时间 | 只读显示 |

## 平台适配策略

| 平台       | 适配要点                                                                                 |
| ---------- | ---------------------------------------------------------------------------------------- |
| 微信小程序 | 使用 rpx 单位；登录用 `uni.login({ provider: 'weixin' })`；条件编译用 `#ifdef MP-WEIXIN` |
| H5         | 响应式设计；账号密码登录；条件编译用 `#ifdef H5`                                         |
| App        | iOS/Android 原生体验；处理安全区和刘海屏；条件编译用 `#ifdef APP-PLUS`                   |

> **强制要求**：所有平台相关能力（登录、扫码、文件选择、`window`/`document` 访问等）必须使用 `#ifdef` / `#ifndef` 条件编译，禁止在运行时使用 `typeof window !== 'undefined'`、`navigator.userAgent` 等判断。

````

---

## 2. TASKS.md 模板

```markdown
# 前端页面开发任务

## 页面分类

| 页面 | 分类 | 说明 |
|------|------|------|
| group-list | 简单 | 列表/搜索/启用/禁用/删除 |
| group-detail | 简单 | 部门信息展示 |
| group-form | 中等 | 新增/编辑部门 |
| critLog-list | 复杂 | 关键日志列表/详情弹窗 |
| critLog-detail | 中等 | 日志详情展示 |

## 并行分组

| 组别 | 任务 | 说明 |
|------|------|------|
| 组1 | P1, P2, P3 | root角色 - account功能 |
| 组2 | P4, P5 | root角色 - log功能 |

## 进度

- [ ] P1: group-list（简单）
- [ ] P2: group-detail（简单）
- [ ] P3: group-form（中等）
- [ ] P4: critLog-list（复杂）
- [ ] P5: critLog-detail（中等）
````
