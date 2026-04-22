# 设计模板

> 包含 README.md 总体设计模板和代码 Javadoc 模板。全局禁用 Lombok。

---

## 1. README.md 总体设计模板

```markdown
# {项目名称}

## 模块总览

| 模块名 | 说明 | 复杂度 | 代码策略 | 涉及实体 |
|--------|------|--------|---------|---------|
| {module} | {描述} | 简单/复杂 | 裁剪生成代码 / 新建Helper | {Entity列表} |

## 模块依赖关系

​```mermaid
graph TD
    A[模块A] --> B[模块B]
    A --> C[模块C]
    B --> D[模块D]
​```

依赖规则：禁止循环依赖，依赖方向只能从上层指向下层。

## PRD 功能点映射

| PRD功能点 | 模块 | 接口 | 说明 |
|-----------|------|------|------|
| {功能1} | {模块A} | POST /{角色}/{模块}/save | 新增{资源} |
| {功能2} | {模块B} | GET /{角色}/{模块}/list | 查询{资源}列表 |

## 角色权限映射

| 角色 | 模块A | 模块B | 模块C |
|------|-------|-------|-------|
| SAAS | R/W | R | - |
| MCH | R | R/W | R/W |
| ADMIN | R/W | R/W | R/W |
| ROOT | R/W | R/W | R/W |

R=只读（list/detail），W=写入（save/update/delete/enable/disable）

## 状态机设计

### {有状态实体} 状态流转

​```mermaid
stateDiagram-v2
    [*] --> 草稿
    草稿 --> 已提交: submit
    已提交 --> 已审核: approve
    已提交 --> 已驳回: reject
    已驳回 --> 草稿: edit
    已审核 --> [*]
​```

| 状态 | 说明 | 允许操作 |
|------|------|---------|
| 草稿 | 初始状态 | 编辑、提交、删除 |
| 已提交 | 待审核 | 撤回 |
| 已审核 | 审核通过 | - |
| 已驳回 | 审核驳回 | 编辑、重新提交 |

## 全局缓存策略

### 缓存组件选型

| 组件 | 适用场景 | 本项目使用 |
|------|---------|-----------|
| FusionCache | 高频读 + 需要本地加速 | 详情查询、配置数据 |
| GlobalCache | 中频读 + 不占JVM内存 | 列表缓存、计数器 |
| GlobalLocker | 分布式锁 | 订单处理、库存扣减 |

### Key 命名规范

```
{服务名}:{业务标识}:{实体ID}
```

以 Class 作为 cacheName，FusionCache/GlobalCache 自动管理 Key 前缀。

### 缓存场景

| 缓存对象 | 组件 | 本地缓存 | 过期时间 | 更新机制 |
|---------|------|---------|---------|---------|
| {对象A}详情 | FusionCache | 500条 | 30min | 写后失效 invalidate |
| {对象B}计数 | GlobalCache | - | 10min | 写后失效 delete |

### FusionCache 配置要求

- localCacheMaxNum：按数据量设定，单实体缓存 500-1000
- cacheExpireMillis：按变更频率设定（30min/1h/24h）
- Caffeine 不设过期时间（性能劣化200倍），仅设 Redis 过期时间
- Kryo 序列化必须使用具体实现类（ArrayList/LinkedHashMap/HashSet），不可用接口类型

## 性能预算

### 数据量估算

| 核心表 | 3个月 | 1年 | 3年 | 增长率 |
|--------|-------|-----|-----|--------|
| {table_a} | {n}万 | {n}万 | {n}万 | {x}%/月 |

### 索引策略

| 表 | 索引 | 覆盖查询 |
|----|------|---------|
| {table_a} | idx_{col1}_{col2} | {查询场景} |

### 性能指标

| 接口 | 目标 RT (P99) | 目标 QPS | 告警阈值 |
|------|-------------|---------|---------|
| {核心接口} | ≤ 200ms | 100 | RT > 500ms |

## 外部依赖

| 依赖 | 类型 | 用途 | 调用方式 |
|------|------|------|---------|
| saas-finance | 内部服务 | 支付/余额 | FinPaymentHelper / FinBalanceHelper |
| {第三方} | 外部API | {用途} | uw-httpclient |

## 全局错误码

| 错误码范围 | 模块 | 说明 |
|-----------|------|------|
| 10000-10099 | {模块A} | {模块A}业务错误 |
| 10100-10199 | {模块B} | {模块B}业务错误 |

## 测试策略

### 测试分层

| 测试层级 | 工具 | 覆盖范围 | 负责角色 |
|---------|------|---------|---------|
| 单元测试 | JUnit 5 + Mockito | Helper 所有公共方法 | 开发工程师 |
| 集成测试 | Spring Boot Test | Controller → Helper → Dao 链路 | 开发工程师 |
| API/E2E测试 | Playwright | 接口全量验证 + 端到端流程 | 测试工程师 |

### 覆盖率目标

| 指标 | 目标值 |
|------|--------|
| Helper 方法覆盖率 | ≥ 90% |
| 分支覆盖率 | ≥ 80% |
| 核心 CRUD 方法 | 100%（每个 ≥ 2 个测试用例） |

### Mock 策略

| 被Mock组件 | Mock方式 | 说明 |
|-----------|---------|------|
| DaoManager | `MockedStatic` | 静态方法 `DaoManager.getInstance()` 隔离 |
| FusionCache | `MockedStatic` | 静态方法隔离，验证 get/put/invalidate |
| GlobalLocker | `MockedStatic` | 静态方法隔离，验证 tryLock/unlock |
| AuthServiceHelper | `MockedStatic` | 静态方法隔离，模拟 getSaasId/getMchId |

### 关键测试场景

| 模块 | 方法 | 测试场景 | 优先级 |
|------|------|---------|--------|
| {模块A} | saveXxx | 唯一性冲突 | P0 |
| {模块A} | updateXxx | 状态不允许修改 | P0 |
| {模块B} | processXxx | 并发冲突（分布式锁） | P0 |
| {模块B} | getXxx | 缓存命中/未命中 | P1 |
```

---

## 2. $PackageInfo$.java 模板

> 每个 controller 角色包中必须包含此文件。声明角色级基础权限，Controller 方法级 `@MscPermDeclare` 在此基础上做细粒度控制。

```java
package {项目包路径}.controller.{角色}.{模块};

import io.swagger.v3.oas.annotations.Operation;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import uw.auth.service.annotation.MscPermDeclare;
import uw.auth.service.constant.UserType;

/**
 * {模块功能描述} - 角色级权限声明
 */
@RestController
public class $PackageInfo$ {

    @MscPermDeclare(user = UserType.{SAAS/MCH/ADMIN})
    @Operation(summary = "{模块功能描述}", description = "{模块功能描述}")
    @GetMapping("/{角色}/{模块}")
    public void info() {
    }
}
```

角色路径对照：

| 角色 | @RequestMapping | @MscPermDeclare user | @MscPermDeclare auth | controller 包名 | $PackageInfo$ |
|------|----------------|---------------------|---------------------|----------------|-------------|
| SAAS运营商 | `/saas/{module}` | `UserType.SAAS` | `AuthType.PERM` | `controller.saas` | 需要 |
| 商户 | `/mch/{module}` | `UserType.MCH` | `AuthType.PERM` | `controller.mch` | 需要 |
| 平台管理员 | `/admin/{module}` | `UserType.ADMIN` | `AuthType.PERM` | `controller.admin` | 需要 |
| ROOT | `/root/{module}` | `UserType.ROOT` | `AuthType.PERM` | `controller.root` | 需要 |
| OPS | `/ops/{module}` | `UserType.OPS` | `AuthType.PERM` | `controller.ops` | 需要 |
| RPC内部调用 | `/rpc/{module}` | `UserType.RPC` | `AuthType.NONE` | `controller.rpc` | 需要 |
| **GUEST(C端)** | `/guest/{module}` | `UserType.GUEST` | `AuthType.USER` | `controller.guest` | **不需要** |

> **Guest 角色说明**：`AuthType.USER` 仅验证用户类型（已登录），不验证权限菜单。Guest 无后台菜单系统，不适用 `AuthType.PERM`。`$PackageInfo$` 仅用于标准角色（saas/mch/admin/root/ops/rpc），guest 不适用。

---

## 3. Controller Javadoc 模板

> 裁剪代码生成器产出的 Controller 后，为每个保留方法补充 Javadoc。

### 类级 Javadoc

```java
/**
 * {ModuleName}Controller - {模块功能描述}
 *
 * <p>设计思路：{说明该Controller在整体架构中的角色，负责哪些业务场景的接口暴露}</p>
 *
 * <p>简单CRUD：直接调 DaoManager.getInstance()</p>
 * <p>复杂逻辑：调用 {Module}Helper.xxx() 静态方法</p>
 *
 * <p>需求映射：README.md 中 {模块名} 模块，PRD功能点 {xxx}</p>
 *
 * @author {author}
 * @since 1.0.0
 */
@RestController
@RequestMapping("/{角色}/{模块}")
@Tag(name = "{ModuleName}", description = "{模块功能描述}")
@MscPermDeclare(name = "{模块功能}", user = UserType.{SAAS/MCH/ADMIN}, log = ActionLog.NONE)
public class {ModuleName}Controller {

    // 简单CRUD直接调 DaoManager.getInstance()
    // 复杂逻辑调 {Module}Helper.xxx() 静态方法，无需注入
}
```

### 方法级 Javadoc

```java
/**
 * 分页查询{资源}列表
 *
 * <p>设计思路：基于AuthQueryParam分页参数查询当前租户下的{资源}列表，支持关键词模糊搜索</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>调用 {module}Helper.list{Entity}(param) 获取分页数据</li>
 *   <li>返回 ResponseData&lt;DataList&lt;{Entity}&gt;&gt; 统一响应</li>
 * </ul>
 *
 * @param param 权限查询参数，自动注入saasId，含分页/排序/模糊搜索
 * @return 分页{资源}列表
 */
@Operation(summary = "分页查询{资源}")
@GetMapping("/list")
@MscPermDeclare(name = "{资源}列表", log = ActionLog.REQUEST)
public ResponseData<DataList<{Entity}>> list(AuthQueryParam param) {
    return ResponseData.success(null);
}

/**
 * 新增{资源}
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>校验字段唯一性约束（如名称不可重复）</li>
 *   <li>调用 {module}Helper.save{Entity}(entity)</li>
 * </ul>
 *
 * @param entity {资源}实体数据
 * @return 创建成功的{资源}数据
 */
@Operation(summary = "新增{资源}")
@PostMapping("/save")
@MscPermDeclare(name = "新增{资源}", log = ActionLog.ALL)
public ResponseData<{Entity}> save(@RequestBody @Valid {Entity} entity) {
    return ResponseData.success(null);
}

/**
 * 修改{资源}
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>查询原实体，不存在返回 warn</li>
 *   <li>合并变更字段（DataEntity setter自动记录到DataUpdateInfo，支持差量更新）</li>
 *   <li>调用 {module}Helper.update{Entity}(entity)</li>
 * </ul>
 *
 * @param entity {资源}实体数据（含ID和待更新字段）
 * @return 修改后的{资源}数据
 */
@Operation(summary = "修改{资源}")
@PostMapping("/update")
@MscPermDeclare(name = "修改{资源}", log = ActionLog.ALL)
public ResponseData<{Entity}> update(@RequestBody @Valid {Entity} entity) {
    return ResponseData.success(null);
}

/**
 * 删除{资源}
 *
 * <p>设计思路：{逻辑删除/物理删除}，删除前检查是否有关联数据</p>
 *
 * @param id {资源}主键ID
 * @return 操作结果
 */
@Operation(summary = "删除{资源}")
@PostMapping("/delete")
@MscPermDeclare(name = "删除{资源}", log = ActionLog.ALL)
public ResponseData<Integer> delete(@RequestParam Long id) {
    return ResponseData.success(null);
}

/**
 * 启用{资源}
 *
 * <p>设计思路：将资源状态设为启用，需检查当前状态是否允许启用</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>查询原实体，不存在返回 warn</li>
 *   <li>校验当前状态是否为"已停用"，非停用状态不允许启用</li>
 *   <li>调用 {module}Helper.updateState(id, STATE_ENABLED)</li>
 * </ul>
 *
 * @param id {资源}主键ID
 * @return 操作结果
 */
@Operation(summary = "启用{资源}")
@PostMapping("/enable")
@MscPermDeclare(name = "启用{资源}", log = ActionLog.REQUEST)
public ResponseData<Integer> enable(@RequestParam Long id) {
    return ResponseData.success(null);
}

/**
 * 停用{资源}
 *
 * <p>设计思路：将资源状态设为停用，需检查当前状态是否允许停用及是否有关联数据</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>查询原实体，不存在返回 warn</li>
 *   <li>校验当前状态是否为"已启用"，非启用状态不允许停用</li>
 *   <li>检查是否有关联业务数据依赖此资源</li>
 *   <li>调用 {module}Helper.updateState(id, STATE_DISABLED)</li>
 * </ul>
 *
 * @param id {资源}主键ID
 * @return 操作结果
 */
@Operation(summary = "停用{资源}")
@PostMapping("/disable")
@MscPermDeclare(name = "停用{资源}", log = ActionLog.REQUEST)
public ResponseData<Integer> disable(@RequestParam Long id) {
    return ResponseData.success(null);
}
```

---

## 4. Helper 模板

> service 包下新建，纯静态工具类，不加 `@Component`，不使用构造器注入。
> **创建前提**：三条件满足至少一项（逻辑复杂/功能性/多处调用），简单 CRUD 不建 Helper。
>
> **Helper 两种类型**：
> - **模块级 Helper**：按数据库表/模块识别，封装该模块的复杂业务（如 PostQuestionHelper）
> - **横切 Helper**：按 PRD 中跨模块的公共业务规则识别，不绑定单表（如 SensitiveWordHelper）
>
> 横切 Helper 识别方法：通读 PRD，找出"在多个模块中出现相同描述"的业务规则。

```java
package {项目包路径}.service;

import {项目包路径}.entity.*;
import uw.auth.service.AuthServiceHelper;
import uw.cache.FusionCache;
import uw.cache.CacheDataLoader;
import uw.cache.GlobalCache;
import uw.cache.GlobalLocker;
import uw.common.dto.ResponseData;
import uw.dao.DaoManager;
import uw.dao.DataList;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * {ModuleName}Helper - {模块}复杂业务逻辑
 *
 * <p>设计思路：{说明该Helper负责的核心业务逻辑范围，处理哪些业务场景}</p>
 *
 * <p>创建理由：{说明为什么需要Helper，满足三条件中哪些项}</p>
 * <ul>
 *   <li>逻辑复杂：{如状态机、多步流程等}</li>
 *   <li>功能性：{如缓存、分布式锁等}</li>
 *   <li>多处调用：{如N个Controller调用}</li>
 * </ul>
 *
 * <p>需求映射：README.md 中 {模块名} 模块</p>
 *
 * @author {author}
 * @since 1.0.0
 */
public class {Module}Helper {

    private static final Logger log = LoggerFactory.getLogger({Module}Helper.class);
    private static final DaoManager daoManager = DaoManager.getInstance();

    // ==================== 缓存配置 ====================

    // FusionCache配置：本地缓存最大条目数
    private static final int CACHE_MAX_NUM = 500;
    // FusionCache配置：缓存过期时间（毫秒），30分钟
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    // 设计阶段必须完成 FusionCache 初始化
    static {
        FusionCache.config(new FusionCache.Config(
            {Entity}.class,
            CACHE_MAX_NUM,
            CACHE_EXPIRE_MILLIS
        ), new CacheDataLoader<Long, {Entity}>() {
            @Override
            public {Entity} load(Long key) {
                return daoManager.load({Entity}.class, key).getData();
            }
        });
    }

    // ==================== 业务方法 ====================

    // ... 方法签名见下方
}
```

### 方法级 Javadoc

```java
/**
 * 查询{资源}详情（带缓存）
 *
 * <p>缓存策略：FusionCache，本地{CACHE_MAX_NUM}条，过期{CACHE_EXPIRE_MILLIS}ms</p>
 * <p>更新机制：修改/删除时调用 FusionCache.invalidate({Entity}.class, id) 失效</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>先调用 FusionCache.getIfPresent({Entity}.class, id) 查缓存</li>
 *   <li>未命中调用 daoManager.load({Entity}.class, id) 查数据库</li>
 *   <li>查到后调用 FusionCache.put({Entity}.class, id, entity) 写入缓存</li>
 *   <li>判断 isSuccess()，未命中返回 warn</li>
 * </ul>
 *
 * @param id 主键ID
 * @return {资源}实体
 */
public static ResponseData<{Entity}> get{Entity}(Long id) {
    return ResponseData.success(null);
}

/**
 * {复杂业务操作}
 *
 * <p>设计思路：{状态机流转/多步流程/复杂计算等}</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>步骤1：{校验/查询前置数据}</li>
 *   <li>步骤2：{核心业务逻辑}</li>
 *   <li>步骤3：{缓存失效/通知等后置操作}</li>
 * </ul>
 *
 * @param {param1} {参数说明}
 * @return 操作结果
 */
public static ResponseData<{Type}> complexMethod({Params}) {
    return ResponseData.success(null);
}
```

### 分布式锁使用模板（基于 GlobalLocker）

```java
/**
 * 处理{业务操作}
 *
 * <p>并发控制：基于 GlobalLocker 分布式锁，防止并发操作</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>long stamp = GlobalLocker.tryLock({Entity}.class, id, 30000L)</li>
 *   <li>stamp > 0 表示加锁成功，在 finally 中 GlobalLocker.unlock({Entity}.class, id, stamp)</li>
 *   <li>stamp ≤ 0 表示获取锁失败，返回错误提示</li>
 *   <li>业务执行超时（超过锁TTL 30s）：调用 GlobalLocker.keepLock() 续期，或记录告警日志</li>
 * </ul>
 *
 * @param id {资源}ID
 * @return 操作结果
 */
public static ResponseData<Integer> process{Entity}(Long id) {
    return ResponseData.success(null);
}
```

### GlobalCache 使用模板（不占 JVM 内存）

```java
/**
 * 查询{资源}列表（带缓存）
 *
 * <p>缓存策略：GlobalCache（纯Redis，不占JVM内存），过期时间{CACHE_EXPIRE_MILLIS}ms</p>
 * <p>适用场景：列表数据量大，不适合本地缓存</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>构建缓存Key: String cacheKey = saasId + ":" + param.hashCode()</li>
 *   <li>GlobalCache.get(cacheName, cacheKey, clazz) 查缓存</li>
 *   <li>未命中查数据库，结果 GlobalCache.put(cacheName, cacheKey, value, expireMillis)</li>
 *   <li>修改/删除时 GlobalCache.delete(cacheName, cacheKey) 失效</li>
 * </ul>
 */
public static ResponseData<DataList<{Entity}>> list{Entity}Cached(AuthQueryParam param) {
    return ResponseData.success(null);
}
```

---

## 5. VO 模板

> 仅在需要聚合多表数据时创建。VO 用于 Controller 响应，不用于接收请求参数。

```java
package {项目包路径}.vo;

import io.swagger.v3.oas.annotations.media.Schema;

/**
 * {Business}VO - {业务聚合数据}
 *
 * <p>设计思路：{说明为什么需要VO，聚合了哪些数据来源（Entity/其他Helper）}</p>
 *
 * <p>数据来源：</p>
 * <ul>
 *   <li>{EntityA} - {字段1}、{字段2}</li>
 *   <li>{EntityB} - {字段3}</li>
 * </ul>
 *
 * @author {author}
 * @since 1.0.0
 */
@Schema(description = "{业务聚合数据}")
public class {Business}VO {

    @Schema(description = "{主键ID}")
    private Long id;

    @Schema(description = "{字段说明}")
    private {Type} {field};

    public {Business}VO() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public {Type} get{Field}() {
        return {field};
    }

    public void set{Field}({Type} {field}) {
        this.{field} = {field};
    }
}
```

---

## 6. Helper 单元测试模板

> 设计阶段（TDD Red）产出测试骨架，开发阶段实现 Mock 和断言逻辑。

### 简单模块测试模板

```java
package {项目包路径}.service;

import {项目包路径}.entity.{Entity};
import uw.common.dto.ResponseData;
import uw.dao.DaoManager;
import uw.dao.DataList;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.MockedStatic;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

/**
 * {Module}HelperTest - {模块}复杂业务逻辑测试
 *
 * <p>测试范围：{Module}Helper 的所有公共静态方法</p>
 *
 * <p>测试策略：</p>
 * <ul>
 *   <li>MockedStatic DaoManager.getInstance() 隔离数据库</li>
 *   <li>每个方法覆盖正常 + 边界/异常两个场景</li>
 * </ul>
 *
 * @author {author}
 * @since 1.0.0
 */
@ExtendWith(MockitoExtension.class)
class {Module}HelperTest {

    // ==================== getXxx 测试 ====================

    /**
     * 测试查询{资源}详情 - 正常返回实体
     *
     * <p>准备数据：构造存在的{资源}ID</p>
     * <p>预期结果：返回ResponseData，isSuccess()=true，实体非null</p>
     */
    @Test
    @DisplayName("查询{资源}详情 - 正常返回实体")
    void testGet{Entity}_Found_ReturnEntity() {
        // TODO: 开发阶段实现 MockedStatic DaoManager 和断言
        fail("TDD Red: Helper 方法尚未实现");
    }

    /**
     * 测试查询{资源}详情 - ID不存在
     *
     * <p>准备数据：构造不存在的{资源}ID</p>
     * <p>预期结果：返回ResponseData，isSuccess()=false，warn信息</p>
     */
    @Test
    @DisplayName("查询{资源}详情 - ID不存在返回warn")
    void testGet{Entity}_NotFound_ReturnWarn() {
        fail("TDD Red: Helper 方法尚未实现");
    }
}
```

### 复杂模块测试模板（含缓存和锁）

```java
package {项目包路径}.service;

import {项目包路径}.entity.{Entity};
import uw.cache.FusionCache;
import uw.cache.GlobalLocker;
import uw.common.dto.ResponseData;
import uw.dao.DaoManager;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.MockedStatic;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;

/**
 * {Module}HelperTest - {模块}复杂业务逻辑测试
 *
 * <p>测试范围：{Module}Helper 的所有公共静态方法，含缓存一致性、并发控制</p>
 *
 * <p>测试策略：</p>
 * <ul>
 *   <li>MockedStatic DaoManager.getInstance() 隔离数据库操作</li>
 *   <li>MockedStatic FusionCache / GlobalLocker 隔离缓存和锁</li>
 *   <li>MockedStatic AuthServiceHelper 隔离用户上下文</li>
 *   <li>业务流程方法覆盖正常 + 每个分支 + 每个异常</li>
 *   <li>缓存方法验证 get/put/invalidate 调用链</li>
 *   <li>分布式锁方法验证 tryLock/unlock 配对</li>
 * </ul>
 *
 * @author {author}
 * @since 1.0.0
 */
@ExtendWith(MockitoExtension.class)
class {Module}HelperTest {

    // ==================== 业务流程测试 ====================

    /**
     * 测试{业务操作} - 正常流程
     *
     * <p>准备数据：构造合法状态的{资源}ID</p>
     * <p>预期结果：成功执行，状态变更为目标状态</p>
     */
    @Test
    @DisplayName("{业务操作} - 正常流程成功")
    void testProcess{Entity}_Normal_ReturnSuccess() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    /**
     * 测试{业务操作} - 状态不允许
     *
     * <p>准备数据：构造非法状态的{资源}ID</p>
     * <p>预期结果：返回错误，提示状态不允许操作</p>
     */
    @Test
    @DisplayName("{业务操作} - 状态不允许返回错误")
    void testProcess{Entity}_InvalidState_ReturnError() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    /**
     * 测试{业务操作} - 并发冲突
     *
     * <p>准备数据：模拟 GlobalLocker.tryLock 返回 0（获取锁失败）</p>
     * <p>预期结果：返回错误，提示操作正在处理中</p>
     */
    @Test
    @DisplayName("{业务操作} - 并发冲突返回锁失败")
    void testProcess{Entity}_Concurrent_LockFail() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    // ==================== 缓存一致性测试 ====================

    /**
     * 测试缓存失效 - 修改后缓存被清除
     *
     * <p>准备数据：Mock FusionCache 已有缓存数据</p>
     * <p>预期结果：调用 update 后 FusionCache.invalidate 被调用</p>
     */
    @Test
    @DisplayName("缓存失效 - 修改后invalidate被调用")
    void testUpdate{Entity}_CacheInvalidated() {
        fail("TDD Red: Helper 方法尚未实现");
    }
}
```

---

## 7. Controller 单元测试模板

> 设计阶段（TDD Red）产出测试骨架，验证 API 契约：HTTP 方法、路径、参数绑定、权限注解、响应格式、调用链路。
> 测试类位置与源码目录结构对齐：`src/test/java/{包路径}/controller/{角色}/{模块}/{Module}ControllerTest.java`

```java
package {项目包路径}.controller.{角色}.{模块};

import {项目包路径}.entity.{Entity};
import {项目包路径}.dto.{Entity}QueryParam;
import uw.common.dto.ResponseData;
import uw.dao.DataList;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;

/**
 * {Module}ControllerTest - {模块} API 契约测试
 *
 * <p>测试范围：{Module}Controller 的所有接口方法</p>
 *
 * <p>测试策略：</p>
 * <ul>
 *   <li>验证 HTTP 方法和路径映射（GET/POST）</li>
 *   <li>验证 @MscPermDeclare 权限注解正确性</li>
 *   <li>验证请求参数绑定（@RequestBody / @RequestParam）</li>
 *   <li>验证响应格式符合 ResponseData 规范</li>
 *   <li>验证调用链路：Controller → Helper 或 Controller → DaoManager</li>
 * </ul>
 *
 * @author {author}
 * @since 1.0.0
 */
@ExtendWith(MockitoExtension.class)
class {Module}ControllerTest {

    // ==================== 列表查询测试 ====================

    /**
     * 测试列表查询 - 正常返回
     *
     * <p>API: GET /{角色}/{模块}/list</p>
     * <p>预期结果：返回 ResponseData<DataList<{Entity}>></p>
     */
    @Test
    @DisplayName("列表查询 - 正常返回")
    void testList_Normal_ReturnSuccess() {
        fail("TDD Red: Controller 测试尚未实现");
    }

    // ==================== 详情查询测试 ====================

    /**
     * 测试详情查询 - 正常返回
     *
     * <p>API: GET /{角色}/{模块}/get?id={id}</p>
     * <p>预期结果：返回 ResponseData<{Entity}></p>
     */
    @Test
    @DisplayName("详情查询 - 正常返回")
    void testGet_Normal_ReturnSuccess() {
        fail("TDD Red: Controller 测试尚未实现");
    }

    // ==================== 新增测试 ====================

    /**
     * 测试新增 - 正常返回
     *
     * <p>API: POST /{角色}/{模块}/save</p>
     * <p>预期结果：返回 ResponseData<{Entity}>，调用链路：Controller → {Module}Helper.save{Entity}()</p>
     */
    @Test
    @DisplayName("新增 - 正常返回")
    void testSave_Normal_ReturnSuccess() {
        fail("TDD Red: Controller 测试尚未实现");
    }

    // ==================== 修改测试 ====================

    /**
     * 测试修改 - 正常返回
     *
     * <p>API: POST /{角色}/{模块}/update</p>
     * <p>预期结果：返回 ResponseData<{Entity}>，调用链路：Controller → {Module}Helper.update{Entity}()</p>
     */
    @Test
    @DisplayName("修改 - 正常返回")
    void testUpdate_Normal_ReturnSuccess() {
        fail("TDD Red: Controller 测试尚未实现");
    }
}
```

---

## 8. Helper 间调用模板

> Helper 是静态工具类，Helper 间直接通过类名调用静态方法，无需注入。

```java
public class OrderHelper {
    private static final DaoManager daoManager = DaoManager.getInstance();

    /**
     * 创建订单
     *
     * <p>跨Helper调用：直接调用 ProductHelper.getXxx() 校验商品状态</p>
     */
    public static ResponseData<Order> saveOrder(Order order) {
        // 直接调用其他Helper的静态方法
        ResponseData<Product> productResult = ProductHelper.getProduct(order.getProductId());
        // ...
        return ResponseData.success(null);
    }
}
```

对应的测试类 Mock 声明：

```java
@ExtendWith(MockitoExtension.class)
class OrderHelperTest {

    // Helper 是静态类，测试时需要 MockedStatic
    // 开发阶段补充 MockedStatic<OrderHelper> 等

    @Test
    @DisplayName("创建订单 - 正常流程")
    void testSaveOrder_Normal_ReturnSuccess() {
        fail("TDD Red: Helper 方法尚未实现");
    }
}
```
