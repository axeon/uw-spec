# AIP 模块（Application Infrastructure Provider - 应用基础设施与产品授权计费）

AIP 模块是整个 SaaS 平台的针对应用基础设施与产品授权的计费和授权核心，负责管理产品、服务、订单、授权许可、余额和发票等业务流程。

## AIP 核心概念

| 概念 | 说明 |
|------|------|
| **Vendor（服务供应商）** | 定义可售卖的服务类型，包含服务代码、计费规则等 |
| **Product（产品）** | 基于 Vendor 封装的可售卖产品，包含定价、库存等 |
| **Order（订单）** | 产品购买记录，支持在线支付 |
| **License（授权许可）** | 服务授权的生命周期管理，支持到期自动禁用 |
| **Balance（余额）** | 预充值余额管理，支持充值、消费、退款 |

## Vendor 类型

Vendor 分为四种类型，分别对应不同的计费模式：

| 类型 | 接口 | 计费方式 | 说明 |
|------|------|----------|------|
| **License** | `AipLicenseVendor` | LicenseValue 计数 | 按使用次数计费，如短信、邮件 |
| **App** | `AipAppVendor` | LicensePeriod 周期 | 按时间周期计费，如功能模块订阅 |
| **AppLicense** | `AipAppLicenseVendor` | 混合计费 | 同时按时间和次数计费 |
| **Task** | `AipTaskVendor` | 单次任务计价 | 一次性任务执行，执行前计算价格 |

## AipVendor 接口定义

```java
/**
 * AipVendor 基础接口
 * 所有的 Vendor 都应该实现这个接口
 */
public interface AipVendor {
    /**
     * 获得展示名称
     */
    String name();

    /**
     * 设置服务代码
     * appName 和 serviceCode 一起组成 licenseCode
     */
    String code();
}
```

## AipAppVendor 接口详解

```java
/**
 * 基于应用功能的 Vendor
 * 应用功能 Vendor 一般只和 license 时间有关
 */
public interface AipAppVendor extends AipVendor {
    /**
     * 初始化数据
     * 在运营商购买服务时调用，用于初始化库表结构和数据
     */
    ResponseData initData(AipVendorExecuteParam runParam);

    /**
     * 销毁数据
     * 在服务到期或退订时调用，用于清理数据
     */
    ResponseData destroyData(AipVendorExecuteParam runParam);
}
```

## AipLicenseVendor 接口详解

```java
/**
 * 基于 License 的 Vendor
 * 一般此类型的 Vendor 仅需要处理 AipLicense 信息即可
 */
public interface AipLicenseVendor extends AipVendor {
    /**
     * 初始化数据
     */
    ResponseData initData(AipVendorExecuteParam runParam);

    /**
     * 销毁数据
     */
    ResponseData destroyData(AipVendorExecuteParam runParam);
}
```

## AIP Helper API 详解

**AipHelper** 是 AIP 模块的核心帮助类，提供授权查询和扣减功能。

```java
// ========== 授权查询 ==========
// 根据 Vendor 类获取 LicenseCode
static String getLicenseCodeByVendor(Class<? extends AipVendor> vendorCls)

// 获取运营商授权 Map
static Map<String, AipLicenseInfoVo> getSaasLicenseMap(long saasId)

// 获取指定授权信息
static AipLicenseInfoVo getLicense(long saasId, String licenseCode)
static AipLicenseInfoVo getLicense(Class<? extends AipVendor> vendorCls, long saasId)

// 获取授权值
static long getLicenseValue(long saasId, String licenseCode)

// ========== 授权扣减 ==========
// 检查并扣减授权
static ResponseData checkAndDeductLicense(long saasId, String licenseCode, long deductValue)

// 批量检查并扣减（带缓存优化）
static ResponseData batchCheckAndDeductLicense(long saasId, String licenseCode, long batchValue)
static ResponseData batchCheckAndDeductLicense(long saasId, String licenseCode, long batchValue, long deductValue)

// ========== 使用示例 ==========
// 示例1：检查用户授权
AipLicenseInfoVo license = AipHelper.getLicense(SaasUserLicenseVendor.class, saasId);
if (license == null || license.getLicenseValue() <= 0) {
    return ResponseData.warn("USER_LICENSE_EXHAUSTED", "用户授权已用完");
}

// 示例2：扣减短信授权
ResponseData result = AipHelper.checkAndDeductLicense(saasId, "saas-base-app:sms", 1);
if (!result.isSuccess()) {
    return ResponseData.warn("SMS_LICENSE_EXHAUSTED", "短信授权不足");
}

// 示例3：批量扣减（高性能场景）
ResponseData result = AipHelper.batchCheckAndDeductLicense(saasId, licenseCode, 100, 1);
```

## 实现 Vendor 的步骤

1. **创建 Vendor 类**，实现对应的 Vendor 接口：

```java
@Service
public class MallAppBaseVendor implements AipAppVendor {
    @Override
    public ResponseData initData(AipVendorExecuteParam runParam) {
        // 初始化商城所需的数据表和默认数据
        return ResponseData.success();
    }

    @Override
    public ResponseData destroyData(AipVendorExecuteParam runParam) {
        // 清理商城数据
        return ResponseData.success();
    }

    @Override
    public String name() {
        return "商城系统";
    }

    @Override
    public String code() {
        return "";
    }
}
```

2. **系统自动注册**：启动时会自动扫描并注册到 saas-base-app

---

