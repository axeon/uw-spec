# 后端技术栈参考文档

> ⚠️ **按需加载**：以下文档为后端开发提供技术栈参考，**仅在当前任务涉及对应技术栈时才读取**对应目录的 README.md，不要一次性全部加载。

## 技术栈目录

| 目录 | 技术栈 | 说明 | 索引入口 |
|------|--------|------|---------|
| [uniweb/](uniweb/) | UniWeb 基础框架 | 基础类库（uw-base）、微服务、开发规范 | [uniweb/README.md](uniweb/README.md) |
| [saas/](saas/) | SaaS 业务框架 | AIP/AIS模块、业务服务、开发指南 | [saas/README.md](saas/README.md) |

## 层级关系

```
SaaS 应用层（saas-mall / saas-market / ...）
        │
        ▼
SaaS 基础服务层（saas-base + saas-finance: AIP + AIS + 租户管理 + 商户管理 + 支付通道 + 余额管理 + 对账管理 + 汇率管理 + 消息通知 + 对象存储 + 配置管理 + ...）
）
        │
        ▼
UniWeb 基础框架层（uw-base: uw-common / uw-dao / uw-auth / uw-task / uw-cache / uw-oauth2 / ...）
```

## 使用建议

| 阶段 | 加载策略 |
|------|---------|
| 早期设计/全局了解 | 仅读取 `uniweb/README.md` 或 `saas/README.md` 建立全局认知 |
| 模块设计 | 读取 README 索引 + 对应模块的详细文档 |
| 编码开发 | 读取 `dev-standards.md` + 当前使用的核心类库文档 |
| 创建新项目 | 读取 `service-template.md` 或 `new-app-guide.md` |
