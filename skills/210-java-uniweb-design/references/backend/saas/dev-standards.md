## 开发规范

### 项目命名规范

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 父项目 | `saas-{name}` | `saas-mall`, `saas-finance` |
| 应用模块 | `{parent}-app` | `saas-mall-app` |
| 客户端模块 | `{parent}-client` | `saas-finance-client` |
| 公共模块 | `{parent}-common` | `saas-base-common` |

### 包命名规范

```
saas.{module}.{layer}

示例：
saas.mall.controller.saas    # 商城-控制器-运营商端
saas.mall.entity             # 商城-实体类
saas.finance.client.rpc      # 财务-客户端-RPC
saas.base.common.helper      # 基础-公共-帮助类
```

### 类命名规范

| 类型 | 后缀 | 示例 |
|------|------|------|
| 实体类 | 无 | `ProductInfo`, `OrderInfo` |
| 查询参数 | `QueryParam` | `ProductInfoQueryParam` |
| 值对象 | `Vo` | `ProductInfoVo` |
| 帮助类 | `Helper` | `ProductHelper` |
| RPC 接口 | `Rpc` | `ProductRpc` |
| Vendor | `Vendor` | `MallAppVendor` |
| Linker | `Linker` | `WechatPaymentLinker` |
| 常量枚举 | 无 | `StateProduct`, `TypeOrder` |

### API 路径规范

| 角色 | 路径前缀 | 说明 |
|------|----------|------|
| ADMIN | `/admin/*` | 平台管理员 |
| SAAS | `/saas/*` | 运营商 |
| MCH | `/mch/*` | 商户 |
| USER | `/user/*` | 终端用户 |
| GUEST | `/guest/*` | 访客 |
| OPEN | `/open/*` | 公开接口 |
| RPC | `/rpc/*` | 微服务间调用 |

---

## 工具类速查

### saas-base-common 工具类

| 类 | 核心方法 | 用途 |
|------|---------|------|
| `MoneyUtils` | `yuanToFen()`, `fenToYuan()` | 元分转换 |
| `CurrencyUtils` | `getCurrency()`, `getAvailableCurrencies()` | 货币处理 |
| `PriceExpression` | `eval()`, `with()` | 价格规则表达式计算 |
| `SaasIdUtils` | `getVipLevel()`, `SaasIdLen()` | 运营商 ID 工具 |
| `SaasMchIdUtils` | `genMchId()`, `parseMchId()` | 商户 ID 生成解析 |
| `SaasOrderIdUtils` | `genOrderId()`, `parseOrderId()` | 订单 ID 生成解析 |
| `GeoHashUtils` | `encode()`, `decode()` | GeoHash 编码解码 |
| `ChinaIdCardUtils` | `validate()`, `getBirthday()` | 身份证验证 |
| `HumanNameUtils` | `parseName()` | 姓名解析 |
| `ValidateUtils` | `isMobile()`, `isEmail()` | 格式验证 |
| `ChineseUtils` | `toPinyin()`, `isChinese()` | 中文处理 |

### 常量定义

| 常量类 | 说明 |
|--------|------|
| `StateAudit` | 审核状态 |
| `TypeMerchant` | 商户类型 |
| `TypeSaasOrg` | 运营商机构类型 |
| `TypeSpsEvent` | 工单事件类型 |

---

