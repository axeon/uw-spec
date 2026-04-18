## SaaS 财务客户端（saas-finance-client）

saas-finance-client 是独立的财务微服务，提供支付通道、余额管理、对账管理等财务相关功能。

### 引入财务客户端

```xml
<dependency>
    <groupId>saas</groupId>
    <artifactId>saas-finance-client</artifactId>
    <version>2.0.118</version>
</dependency>
```

### 余额管理

余额管理支持三种账户类型：

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| **预存款** | 可充值、消费、退款、取现 | 供应商、分销商 |
| **授信** | 永久授信或临时授信，需还款 | 熟悉的分销商 |
| **佣金** | 仅收入、退款、取现，不可预存 | C 端分销商 |

### FinBalanceHelper API 详解

```java
// ========== 余额查询 ==========
// 获取指定币种余额
static ResponseData<FinBalanceInfoData> getBalance(long saasId, long mchId, String currency)

// 获取所有币种余额列表
static ResponseData<List<FinBalanceInfoData>> listBalance(long saasId, long mchId)

// ========== 授信消费 ==========
static ResponseData<FinBalanceLogData> creditConsume(FinBalanceDealParam balanceDealParam)
static ResponseData<FinBalanceLogData> creditConsumeRefund(FinBalanceRefundParam balanceRefundParam)

// ========== 预存消费 ==========
static ResponseData<FinBalanceLogData> depositConsume(FinBalanceDealParam balanceDealParam)
static ResponseData<FinBalanceLogData> depositConsumeRefund(FinBalanceRefundParam balanceRefundParam)

// ========== 存款收入 ==========
static ResponseData<FinBalanceLogData> depositIncomeStart(FinBalanceDealParam balanceDealParam)
static ResponseData depositIncomeNotifySuccess(FinBalanceNotifyParam balanceNotifyParam)
static ResponseData depositIncomeNotifyFail(FinBalanceNotifyParam balanceNotifyParam)
static ResponseData<FinBalanceLogData> depositIncomeNotifyRefund(FinBalanceRefundParam balanceRefundParam)

// ========== 佣金收入 ==========
static ResponseData<FinBalanceLogData> rebateIncomeStart(FinBalanceDealParam balanceDealParam)
static ResponseData rebateIncomeNotifySuccess(FinBalanceNotifyParam balanceNotifyParam)
static ResponseData rebateIncomeNotifyFail(FinBalanceNotifyParam balanceNotifyParam)
static ResponseData<FinBalanceLogData> rebateIncomeNotifyRefund(FinBalanceRefundParam balanceRefundParam)

// ========== 账户管理 ==========
static ResponseData enable(long saasId, long mchId, String currency)
static ResponseData disable(long saasId, long mchId, String currency)
```

### 支付通道

支付通道通过 AIS Linker 机制实现，支持多种支付方式：

| 支付方式 | Linker 类 | 说明 |
|----------|-----------|------|
| 微信支付 | `WechatPaymentLinker` | 微信 Native/JSAPI/小程序支付 |
| 支付宝 | `AliPayMentPaymentLinker` | 支付宝国内支付 |
| 支付宝国际 | `AliPayGlobalPaymentLinker` | 支付宝跨境支付 |
| 合利宝 | `HelipayPaymentLinker` | 合利宝支付通道 |

### FinPaymentHelper API 详解

```java
// ========== 支付操作 ==========
static ResponseData<FinPayOrderResponse> payOrder(FinPayOrderRequest payOrderRequest)

// ========== 退款操作 ==========
static ResponseData<FinPayRefundResponse> payRefund(FinPayRefundRequest payRefundRequest)

// ========== 使用示例 ==========
// 创建支付订单
FinPayOrderRequest request = new FinPayOrderRequest();
request.setSaasId(saasId);
request.setMchId(mchId);
request.setBizOrderId(orderId);
request.setBizOrderType("MALL_ORDER");
request.setPayChannel(TypeFinPayChannel.WECHAT.getValue());
request.setPayTradeType(TypeFinPayTrade.NATIVE.getValue());
request.setOrderAmount(10000L); // 单位：分
request.setOrderSubject("商品购买");

ResponseData<FinPayOrderResponse> response = FinPaymentHelper.payOrder(request);
response.onSuccess(payResponse -> {
    String payUrl = payResponse.getPayUrl(); // 支付链接或二维码
});
```

### 汇率管理

```java
// ========== FinCurrencyHelper API ==========
// 获取汇率
static String getCurrencyRate(long saasId, String fromCurrency, String toCurrency)

// 汇率计算
static long calcCurrencyRate(String currencyRate, long money)
static long calcCurrencyRate(String currencyRate, BigDecimal money)

// 查询历史汇率
static ResponseData<String> queryCurrencyRateByDate(long saasId, String fromCurrency, 
    String toCurrency, long queryDatePoint)

// 更新缓存
static void updateCache(long saasId)

// ========== 使用示例 ==========
String rate = FinCurrencyHelper.getCurrencyRate(saasId, "USD", "CNY");
long cnyAmount = FinCurrencyHelper.calcCurrencyRate(rate, usdAmount);
```

### 对账接口

业务系统需要实现 `FinOrderBillInterface` 接口以支持对账功能：

```java
public interface FinOrderBillInterface {
    // 获取供应商应付汇总统计
    ResponseData<DataList<SupplierPaymentSummaryResult>> listSupplierPaymentSummary(
        ListMchVerifySummaryParam param);

    // 获取分销商应收汇总统计
    ResponseData<DataList<DistributorReceiptSummaryResult>> listDistributorReceiptSummary(
        ListMchVerifySummaryParam param);

    // 获取业务订单列表
    ResponseData<List<BillOrderVo>> listOrderForVerify(ListOrderParam param);

    // 批量对账
    ResponseData batchVerify(BillVerifyParam param);

    // 批量取消对账
    ResponseData batchCancelVerify(BillVerifyParam param);

    // 批量收款
    ResponseData batchReceipt(BillReceiptParam param);

    // 批量付款
    ResponseData batchPay(BillPaymentParam param);
}
```

### 支付回调接口

业务系统需要实现 `FinPaymentNotifyInterface` 接口以接收支付回调：

```java
public interface FinPaymentNotifyInterface {
    // 支付成功回调
    ResponseData payNotify(FinPayOrderNotifyData payOrderNotifyData);

    // 退款成功回调
    ResponseData refundNotify(FinPayRefundNotifyData payRefundNotifyData);
}
```

---

