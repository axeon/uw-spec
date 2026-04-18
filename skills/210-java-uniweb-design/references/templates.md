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
| DaoManager | `@Mock` + `@InjectMocks` | 构造器注入，Mockito 自动注入 |
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

## 2. $PackageInfo$ 模板

> 每个 controller 角色包中必须包含此文件。声明角色级基础权限，Controller 方法级 `@MscPermDeclare` 在此基础上做细粒度控制。

```java
@MscPermDeclare(name = "{模块功能描述}", user = UserType.{SAAS/MCH/ADMIN})
package {项目包路径}.controller.{角色};

import uw.auth.service.anno.MscPermDeclare;
import uw.auth.service.vo.UserType;
```

角色路径对照：

| 角色 | @RequestMapping | @MscPermDeclare user | controller 包名 |
|------|----------------|---------------------|----------------|
| SAAS运营商 | `/saas/{module}` | `UserType.SAAS` | `controller.saas` |
| 商户 | `/mch/{module}` | `UserType.MCH` | `controller.mch` |
| 平台管理员 | `/admin/{module}` | `UserType.ADMIN` | `controller.admin` |
| ROOT | `/root/{module}` | `UserType.ROOT` | `controller.root` |
| OPS | `/ops/{module}` | `UserType.OPS` | `controller.ops` |
| RPC内部调用 | `/rpc/{module}` | `UserType.RPC` | `controller.rpc` |

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
 * <p>依赖关系：{Module}Helper</p>
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

    private final {Module}Helper {module}Helper;

    public {ModuleName}Controller({Module}Helper {module}Helper) {
        this.{module}Helper = {module}Helper;
    }
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

> service 包下新建，方法体返回默认值，Javadoc 包含完整设计意图。

### 类级 Javadoc

```java
package {项目包路径}.service;

import {项目包路径}.entity.*;
import uw.auth.service.AuthServiceHelper;
import uw.cache.FusionCache;
import uw.cache.GlobalCache;
import uw.cache.GlobalLocker;
import uw.common.dto.ResponseData;
import uw.dao.DaoManager;
import uw.dao.DataList;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * {ModuleName}Helper - {模块}业务逻辑
 *
 * <p>设计思路：{说明该Helper负责的核心业务逻辑范围，处理哪些业务场景}</p>
 *
 * <p>依赖关系：</p>
 * <ul>
 *   <li>DaoManager - 数据持久化操作（uw-dao）</li>
 *   <li>AuthServiceHelper - 当前用户信息获取（uw-auth-service）</li>
 *   <li>FusionCache - 融合缓存（uw-cache）</li>
 * </ul>
 *
 * <p>需求映射：README.md 中 {模块名} 模块</p>
 *
 * @author {author}
 * @since 1.0.0
 */
@Component
public class {Module}Helper {

    private static final Logger log = LoggerFactory.getLogger({Module}Helper.class);

    private final DaoManager daoManager;

    // ==================== 缓存配置 ====================

    // FusionCache配置：本地缓存最大条目数
    private static final int CACHE_MAX_NUM = 500;
    // FusionCache配置：缓存过期时间（毫秒），30分钟
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    /**
     * 构造器注入依赖
     *
     * @param daoManager 数据访问管理器
     */
    public {Module}Helper(DaoManager daoManager) {
        this.daoManager = daoManager;
    }

    // ==================== CRUD 方法 ====================

    // ... 方法签名见下方
}
```

### 方法级 Javadoc

```java
/**
 * 分页查询{资源}
 *
 * <p>设计思路：构建AuthQueryParam查询条件，调用DaoManager分页查询</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>keyword 通过 QueryParam 的 LIKE_QUERY_ENABLE(true) 启用模糊匹配</li>
 *   <li>AuthQueryParam 自动注入 saasId，按需 bindUserId()/bindMchId()</li>
 *   <li>调用 daoManager.list({Entity}.class, param)</li>
 * </ul>
 *
 * @param param 权限查询参数
 * @return 分页数据列表
 */
public ResponseData<DataList<{Entity}>> list{Entity}(AuthQueryParam param) {
    return daoManager.list({Entity}.class, param);
}

/**
 * 查询{资源}详情
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
public ResponseData<{Entity}> get{Entity}(Long id) {
    return ResponseData.success(null);
}

/**
 * 新增{资源}
 *
 * <p>设计思路：校验 → 填充默认值 → 获取序列ID → 持久化</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>entity.setId(daoManager.getSequenceId({Entity}.class)) 获取分布式ID</li>
 *   <li>entity.setSaasId(AuthServiceHelper.getSaasId()) 注入租户ID</li>
 *   <li>entity.setMchId(AuthServiceHelper.getMchId()) 注入商户ID（如需）</li>
 *   <li>调用 daoManager.save(entity) 持久化</li>
 *   <li>新增数据无需处理缓存（尚未被缓存）</li>
 * </ul>
 *
 * @param entity {资源}实体
 * @return 创建后的实体
 */
public ResponseData<{Entity}> save{Entity}({Entity} entity) {
    return ResponseData.success(null);
}

/**
 * 修改{资源}
 *
 * <p>缓存策略：修改成功后调用 FusionCache.invalidate({Entity}.class, id) 失效缓存</p>
 *
 * <p>实现要求：</p>
 * <ul>
 *   <li>先 daoManager.load({Entity}.class, id) 查原实体，不存在返回 warn</li>
 *   <li>校验状态是否允许修改（如有状态机约束）</li>
 *   <li>设置变更字段（DataEntity setter自动记录到DataUpdateInfo，支持差量更新）</li>
 *   <li>调用 daoManager.update(entity) 差量更新</li>
 *   <li>更新成功后 FusionCache.invalidate({Entity}.class, id)</li>
 * </ul>
 *
 * @param entity {资源}实体（含ID和待更新字段）
 * @return 修改后的实体
 */
public ResponseData<{Entity}> update{Entity}({Entity} entity) {
    return ResponseData.success(null);
}

/**
 * 删除{资源}
 *
 * <p>缓存策略：删除成功后调用 FusionCache.invalidate({Entity}.class, id) 失效缓存</p>
 *
 * @param id 主键ID
 * @return 影响行数
 */
public ResponseData<Integer> delete{Entity}(Long id) {
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
public ResponseData<Integer> process{Entity}(Long id) {
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
public ResponseData<DataList<{Entity}>> list{Entity}Cached(AuthQueryParam param) {
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
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;

/**
 * {Module}HelperTest - {模块}业务逻辑测试
 *
 * <p>测试范围：{Module}Helper 的所有公共方法</p>
 *
 * <p>测试策略：</p>
 * <ul>
 *   <li>Mock DaoManager 进行数据库操作隔离</li>
 *   <li>每个方法覆盖正常 + 边界/异常两个场景</li>
 * </ul>
 *
 * @author {author}
 * @since 1.0.0
 */
@ExtendWith(MockitoExtension.class)
class {Module}HelperTest {

    @Mock
    private DaoManager daoManager;

    @InjectMocks
    private {Module}Helper {module}Helper;

    // ==================== listXxx 测试 ====================

    /**
     * 测试分页查询{资源} - 正常返回分页数据
     *
     * <p>准备数据：构造包含2条{资源}的AuthQueryParam</p>
     * <p>预期结果：返回ResponseData，isSuccess()=true，数据列表size=2</p>
     */
    @Test
    @DisplayName("分页查询{资源} - 正常返回分页数据")
    void testList{Entity}_Normal_ReturnPagedData() {
        // TODO: 开发阶段实现 Mock 和断言
        fail("TDD Red: Helper 方法尚未实现");
    }

    /**
     * 测试分页查询{资源} - 空结果
     *
     * <p>准备数据：构造无匹配条件的AuthQueryParam</p>
     * <p>预期结果：返回ResponseData，isSuccess()=true，数据列表size=0</p>
     */
    @Test
    @DisplayName("分页查询{资源} - 空结果返回空列表")
    void testList{Entity}_NoMatch_ReturnEmptyList() {
        fail("TDD Red: Helper 方法尚未实现");
    }

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

    // ==================== saveXxx 测试 ====================

    @Test
    @DisplayName("新增{资源} - 正常创建并返回实体")
    void testSave{Entity}_Normal_ReturnCreatedEntity() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    @Test
    @DisplayName("新增{资源} - 唯一性冲突返回错误")
    void testSave{Entity}_DuplicateName_ReturnError() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    // ==================== updateXxx 测试 ====================

    @Test
    @DisplayName("修改{资源} - 正常更新并返回实体")
    void testUpdate{Entity}_Normal_ReturnUpdatedEntity() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    @Test
    @DisplayName("修改{资源} - ID不存在返回warn")
    void testUpdate{Entity}_NotFound_ReturnWarn() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    // ==================== deleteXxx 测试 ====================

    @Test
    @DisplayName("删除{资源} - 正常删除")
    void testDelete{Entity}_Normal_ReturnSuccess() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    @Test
    @DisplayName("删除{资源} - ID不存在返回warn")
    void testDelete{Entity}_NotFound_ReturnWarn() {
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
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;

import static org.junit.jupiter.api.Assertions.*;

/**
 * {Module}HelperTest - {模块}复杂业务逻辑测试
 *
 * <p>测试范围：{Module}Helper 的所有公共方法，含缓存一致性、并发控制</p>
 *
 * <p>测试策略：</p>
 * <ul>
 *   <li>Mock DaoManager 隔离数据库操作</li>
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

    @Mock
    private DaoManager daoManager;

    @InjectMocks
    private {Module}Helper {module}Helper;

    // ==================== CRUD 测试（同简单模块） ====================
    // ... list/get/save/update/delete 测试方法

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

## 7. Helper 间依赖注入模板

> 当 HelperA 依赖 HelperB 时的构造器注入示例。

```java
@Component
public class OrderHelper {

    private final DaoManager daoManager;
    private final ProductHelper productHelper;

    public OrderHelper(DaoManager daoManager, ProductHelper productHelper) {
        this.daoManager = daoManager;
        this.productHelper = productHelper;
    }

    /**
     * 创建订单
     *
     * <p>跨Helper调用：调用 productHelper.getXxx() 校验商品状态</p>
     */
    public ResponseData<Order> saveOrder(Order order) {
        return ResponseData.success(null);
    }
}
```

对应的测试类 Mock 声明：

```java
@ExtendWith(MockitoExtension.class)
class OrderHelperTest {

    @Mock
    private DaoManager daoManager;

    @Mock
    private ProductHelper productHelper;

    @InjectMocks
    private OrderHelper orderHelper;
}
```
