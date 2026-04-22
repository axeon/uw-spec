# 设计阶段 TDD 指南

> 设计阶段（220）的 TDD 实践指南。设计即代码，测试先行——在设计阶段产出测试骨架，确保每个 Helper 方法都有可执行的测试契约。

## 核心理念

| 传统设计 | TDD驱动设计（本技能） |
|---------|-------------------|
| Helper 方法体返回 `null` | Helper 方法体返回 `null` + 配套测试骨架 |
| 编译通过即设计完成 | 测试骨架编译 + 运行（全红）即设计完成 |
| 测试留到开发阶段 | 测试在设计阶段就位，开发阶段从 Green 开始 |
| 设计文档描述行为 | 测试代码即行为规约，编译器强制一致性 |

## TDD 三阶段映射

| TDD 阶段 | 对应阶段 | 产出 |
|----------|---------|------|
| **Red**（设计阶段 220） | 编写测试骨架 + Helper 签名 | 测试全红，但可编译运行 |
| **Green**（开发阶段 310） | 实现 Helper 逻辑 | 测试逐个变绿 |
| **Refactor**（开发阶段 310） | 重构优化 | 测试保持全绿 |

## 职责划分

| 测试类型 | 负责角色 | 负责阶段 | 说明 |
|---------|---------|---------|------|
| 单元测试（Helper） | 架构师 / Java 开发工程师 | 220（骨架）+ 310（实现） | 本技能产出 |
| 测试脚本 | 测试工程师 | 330+ | 测试用例自动化 |
| API 测试 | 测试工程师 | 410+ | 接口全量验证 |
| E2E 测试 | 测试工程师 | 410+ | 端到端流程验证 |
| 压力测试 | 测试工程师 | 按需 | JMeter 性能测试 |
| 安全测试 | 测试工程师 | 按需 | OWASP/Trivy 扫描 |

> 测试工程师不参与单元测试编写。单元测试是开发工程师的本职工作，从设计阶段的骨架到开发阶段的实现，全程由开发侧负责。

## 测试策略矩阵

### 按模块复杂度分类

| 模块分类 | Helper 方法类型 | 测试策略 | 覆盖重点 |
|---------|---------------|---------|---------|
| 简单模块 | listXxx / getXxx / saveXxx / updateXxx / deleteXxx | 每个方法 1 个基础测试 | 参数校验、返回类型 |
| 复杂模块 | 上述 + 业务流程方法 | 每个方法 2-5 个测试用例 | 状态流转、并发控制、缓存一致性 |
| 跨模块 | 涉及多个 Helper 协作 | 集成测试骨架 | 模块间契约 |

### 按方法类型测试用例数

| 方法类型 | 基础用例 | 边界用例 | 异常用例 | 合计 |
|---------|---------|---------|---------|------|
| listXxx | 正常分页查询 1 | 空结果 1 | - | 2 |
| getXxx | 正常查询 1 | 不存在ID 1 | - | 2 |
| saveXxx | 正常新增 1 | 唯一性冲突 1 | 必填字段缺失 1 | 3 |
| updateXxx | 正常修改 1 | 不存在ID 1 | 状态不允许修改 1 | 3 |
| deleteXxx | 正常删除 1 | 不存在ID 1 | 关联数据阻止删除 1 | 3 |
| enableXxx / disableXxx | 正常操作 1 | 不存在ID 1 | 重复操作 1 | 3 |
| 业务流程方法 | 正常流程 1 | 每个分支 1 | 每个异常 1 | 3-5 |

### 缓存测试策略

| 缓存类型 | 测试要点 | Mock 策略 |
|---------|---------|----------|
| FusionCache | get 命中/未命中、put 写入、invalidate 失效 | Mock FusionCache 静态方法 |
| GlobalCache | get/put/delete 操作 | Mock GlobalCache 实例 |
| GlobalLocker | tryLock 成功/失败、unlock 释放 | Mock GlobalLocker 静态方法 |

## 设计阶段测试产出要求

### 产出物

| 产出物 | 位置 | 说明 |
|--------|------|------|
| 测试类 | `src/test/java/{包路径}/service/{Module}HelperTest.java` | 每个 Helper 一个测试类 |
| 测试方法 | 测试类内 | 每个 Helper 方法 ≥ 2 个测试方法 |
| README.md 测试章节 | README.md | 测试策略概览 + 覆盖率目标 |

### 测试命名规范

| 规则 | 格式 | 示例 |
|------|------|------|
| 测试类 | `{Class}Test` | `ProductHelperTest` |
| 测试方法 | `test{Method}_{Scenario}_{ExpectedResult}` | `testSaveProduct_NameDuplicate_ThrowException` |
| 测试方法（简化） | `test{Method}_{ExpectedResult}` | `testGetById_NotFound_ReturnWarn` |

### 测试目录结构

```
src/test/java/{包路径}/
├── service/
│   ├── {ModuleA}HelperTest.java      # 每个Helper对应一个测试类
│   ├── {ModuleB}HelperTest.java
│   └── ...
└── README.md                          # 测试说明（可选）
```

## Mock 技术选型

Helper 使用纯 static 方法，所有依赖通过静态 API 获取，测试统一使用 `MockedStatic` 隔离。

| 依赖 | Mock 方式 | 说明 |
|------|----------|------|
| DaoManager | `MockedStatic<DaoManager>` + `mock(DaoManager.class)` | mock `getInstance()` 返回 mocked 实例 |
| FusionCache | `MockedStatic<FusionCache>` | try-with-resources 包裹，验证 get/put/invalidate |
| GlobalCache | `MockedStatic<GlobalCache>` | try-with-resources 包裹 |
| GlobalLocker | `MockedStatic<GlobalLocker>` | try-with-resources 包裹，验证 tryLock/unlock |
| AuthServiceHelper | `MockedStatic<AuthServiceHelper>` | try-with-resources 包裹，模拟 getSaasId/getMchId |

**MockedStatic 使用模式**（所有依赖统一）：

```java
@Test
void testGetXxx_CacheHit_ReturnEntity() {
    try (MockedStatic<FusionCache> fusionCacheMock = mockStatic(FusionCache.class);
         MockedStatic<AuthServiceHelper> authMock = mockStatic(AuthServiceHelper.class)) {
        authMock.when(AuthServiceHelper::getSaasId).thenReturn(1001L);
        fusionCacheMock.when(() -> FusionCache.getIfPresent(Entity.class, 1L))
            .thenReturn(Optional.ofNullable(entity));
        // ... 断言
    }
}
```

**框架依赖**：

| 框架 | 用途 | scope |
|------|------|-------|
| JUnit 5 | 测试框架 | test |
| Mockito | Mock 框架（含 MockedStatic） | test |
| Spring Boot Test | 集成测试支持 | test |

## 设计阶段 vs 开发阶段测试职责

| 阶段 | 职责 | 测试状态 |
|------|------|---------|
| 设计阶段（220） | 产出测试骨架（方法签名 + 断言结构 + Javadoc） | 全红（方法体返回null） |
| 开发阶段（310） | 实现 Helper 逻辑，测试逐个变绿 | 红 → 绿 |
| 重构阶段（310） | 优化实现，测试保持全绿 | 全绿 |

**设计阶段测试代码特征**：
- 测试类使用 `@ExtendWith(MockitoExtension.class)`，MockedStatic 在开发阶段补充
- 方法签名完整（`@Test` + `@DisplayName` + 访问修饰符 + 参数）
- Javadoc 包含测试意图、准备数据、预期结果
- 方法体使用 `fail("TDD Red: [Tn] Helper 方法尚未实现")` 标记为 Red 状态，Tn 为任务 ID
- 不需要 `@BeforeEach` 的 Mock 初始化（MockitoExtension 自动处理）
- MockedStatic 静态方法 Mock 在开发阶段实现时补充

## 验收标准

设计阶段测试骨架的完成标准：

- [ ] 每个 Helper 类有对应 `{Module}HelperTest.java`
- [ ] 每个 Helper 方法有 ≥ 2 个测试方法
- [ ] 所有测试类可编译通过
- [ ] `mvn test -pl {模块}` 可运行（全红是预期）
- [ ] 测试方法有 `@DisplayName` 注解
- [ ] 测试方法有 Javadoc 说明测试意图
- [ ] README.md 包含测试策略章节
