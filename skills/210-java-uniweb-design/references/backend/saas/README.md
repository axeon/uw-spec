# SaaS 开发技术栈参考文档

> ⚠️ **按需加载**：以下文档包含大量 API 细节，**仅在当前任务涉及对应模块时才读取**，不要一次性全部加载。每个文件都是独立的，可单独引用。

## 架构概览

SaaS 开发技术栈是基于 **UniWeb** 框架构建的多租户 SaaS 架构体系。采用微服务设计，由基础服务（saas-base）、财务服务（saas-finance）等多个微服务组成，提供租户管理、支付结算解决方案。

### 微服务架构

SaaS 应用 → 引入 saas-base-common / saas-finance-client → SaaS 基础服务层（saas-base + saas-finance） → UniWeb 基础框架层（uw-base，详见 [uniweb/README.md](../uniweb/README.md)）。

## SAAS微服务清单

| 微服务 | 功能说明 |
|---|---|
| saas-base | 整个 SaaS 平台的核心基础设施服务，负责租户管理、商户管理、产品授权计费（AIP）、应用接口服务（AIS）等基础能力 |
| saas-finance | 是 SaaS 平台的财务核心服务，负责支付通道管理、余额账户管理、对账管理、汇率管理等财务相关业务。 |

### saas-base 内部模块

| 模块 | 功能 |
|------|------|
| AIP 模块 | 产品授权计费 |
| AIS 模块 | 应用接口服务（支付/短信/邮件） |
| 租户管理 | 多租户隔离与管理 |
| 商户管理 | 商户注册与配置 |
| 消息通知 | 站内信与推送 |
| 对象存储 | 文件上传与管理 |
| 配置管理 | 动态配置中心 |

### saas-finance 内部模块

| 模块 | 功能 |
|------|------|
| 支付通道 | 多渠道支付集成 |
| 余额管理 | 账户余额与充值 |
| 对账管理 | 交易对账与核销 |
| 汇率管理 | 多币种汇率转换 |


## 核心类库

| 文档 | 模块 | 核心概念 | 适用场景 |
|------|------|---------|---------|
| [saas-base-common.md](saas-base-common.md) | 基础公共模块 | Maven依赖、uw-base依赖列表 | 了解SaaS公共依赖配置 |
| [aip-module.md](aip-module.md) | AIP授权计费 | Vendor, License, AipHelper | 产品授权、计费扣减 |
| [ais-module.md](ais-module.md) | AIS接口服务 | LinkerType, Linker, AisHelper | 第三方接口集成（支付/短信/邮件） |
| [saas-finance-client.md](saas-finance-client.md) | Saas财务客户端 | FinBalanceHelper, FinPaymentHelper | 提供支付通道，在线支付，退款，余额管理、对账等财务相关功能 |
