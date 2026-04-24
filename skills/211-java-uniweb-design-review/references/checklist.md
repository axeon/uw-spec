# UniWeb 后端设计评审检查清单

> **评审原则**：每个检查项的标准以 [210-java-uniweb-design/SKILL.md](../../210-java-uniweb-design/SKILL.md) 中的原始约定为准。下表"源技能依据"列指向源技能的具体章节，评审时必须回到源技能核实。

## 1. README.md 完整性

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 模块总览 | 每个模块有名称、说明、复杂度、代码策略、涉及实体 | 210 → Phase 1 必须章节 |
| 模块依赖关系 | Mermaid 图，无循环依赖 | 210 → Phase 1 必须章节 |
| PRD 功能点映射 | 每个 PRD 功能点对应到模块和接口 | 210 → Phase 1 必须章节 |
| 角色权限映射 | SAAS/MCH/ADMIN/ROOT × 模块的 R/W 矩阵 | 210 → Phase 1 必须章节 |
| 全局缓存策略 | FusionCache/GlobalCache 选型、Key规范、TTL、更新机制 | 210 → Phase 1 必须章节 |
| 性能预算 | 数据量估算（3月/1年/3年）、索引策略、RT/QPS目标 | 210 → Phase 1 必须章节 |
| 外部依赖 | saas-finance、第三方接口、异步任务（如有） | 210 → Phase 1 按需章节 |
| 全局错误码 | 业务错误码范围分配（如有） | 210 → Phase 1 按需章节 |
| 状态机 | 有状态实体的状态流转图和操作映射（如有） | 210 → Phase 1 按需章节 |
| 测试策略 | 测试分层（单元/集成）、覆盖率目标、Mock策略 | 210 → Phase 1 必须章节 |

## 2. TASKS.md 完整性

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 并行分组 | 按依赖图拓扑排序，无依赖同组并行，有依赖串行 | 210 → Phase 1.5 并行分组 |
| 任务卡片字段 | 每张卡片包含：PRD/数据库/待实现/测试骨架/技术栈/待实现方法/依赖任务 | 210 → Phase 1.5 任务卡片编写规则 |
| 任务 ID 一致性 | 卡片中的 Tn ID 与代码中 `// TODO: [Tn]` 标记一一对应 | 210 → Phase 1.5 任务卡片编写规则 |
| 联调验证清单 | 末尾有自动化检查项（grep + mvn test + 启动检查） | 210 → Phase 1.5 联调验证清单 |

## 3. 编译与 Swagger

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| mvn compile | 无编译错误 | 210 → Phase 2 Step 5 |
| mvn test-compile | 测试代码编译无错误 | 210 → Phase 2 Step 5 |
| mvn test | 测试可运行（全红为预期，无编译/运行时错误） | 210 → Phase 2 Step 5 |
| Swagger UI | 启动后所有接口可展示 | 210 → Phase 2 Step 5 |
| Bean 注入 | 无循环依赖、无缺失 Bean | 210 → Phase 2 Step 5 |

## 4. Controller 质量

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 权限注解 | 每个方法有 `@MscPermDeclare`，user 类型与 README 角色映射一致 | 210 → Phase 2 Step 2 权限注解 |
| **Guest 特殊规则** | **Guest 控制器使用 `AuthType.USER`（仅验证登录），不使用 `AuthType.PERM`** | 210 → Phase 2 Step 2 映射表 |
| 类级 Javadoc | 包含设计思路、依赖关系、需求映射 | 210 → Phase 2 Step 2 |
| 方法级 Javadoc | 包含设计思路 + 实现要求（具体步骤） | 210 → Phase 2 Step 2 |
| URL 规范 | 第一级为角色（/saas/, /mch/, /admin/, /rpc/ 等） | 210 → 架构约定 |
| $PackageInfo$ | 仅标准角色（saas/mch/admin/root/ops/rpc）的 controller 包下有 $PackageInfo$.java，**guest 角色不适用** | 210 → 架构约定 |
| 响应格式 | 统一使用 ResponseData\<T\> | 210 → Phase 2 Step 2 |
| HTTP 方法 | 查询用 GET，写入用 POST，符合 RESTful 规范 | 210 → Phase 2 Step 2 |
| 方法裁剪 | 不需要的方法已删除（如不允许删除则无 delete 方法） | 210 → Phase 2 Step 2 |
| **方法体非空** | **方法体必须确定完整调用链路：复杂逻辑调 Helper，简单 CRUD 调 DaoManager，不能留空** | 210 → Phase 2 Step 2 方法体调用 |
| 角色移动 | Controller 已从 admin 移动到目标角色目录，package 声明和 import 已修正 | 210 → Phase 2 Step 2 角色移动 |

**权限注解检查要点**（详见 210 → Phase 2 Step 2 `@MscPermDeclare` 按角色映射表）：
- SAAS/MCH/ADMIN/ROOT/OPS：`AuthType.PERM`
- RPC：`AuthType.NONE`（内部调用无需鉴权）
- **GUEST**：`AuthType.USER`（仅验证用户类型，不验证权限）
- log 级别：查询 REQUEST，写入 ALL，关键操作 CRIT

## 5. Helper 质量

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| **Helper 三条件** | **仅当满足至少一项才创建：逻辑复杂/功能性/多处调用。简单 CRUD 不建 Helper** | 210 → Phase 2 Step 3 创建判断 |
| **横切 Helper** | **跨模块的公共业务规则独立为横切 Helper（如敏感词检测、通知发送），按"一个规则一个 Helper"识别** | 210 → Phase 0 横切关注点识别 |
| 类注解 | 无（纯静态工具类，不加 `@Component`） | 210 → Phase 2 Step 3 |
| 类级 Javadoc | 包含设计思路、依赖关系（DaoManager/FusionCache 等）、需求映射 | 210 → Phase 2 Step 3 |
| 方法级 Javadoc | 包含设计思路 + 实现要求（具体步骤） + 缓存策略（如有） | 210 → Phase 2 Step 3 |
| 方法签名 | `public static`，入参出参明确，方法体返回默认值 | 210 → Phase 2 Step 3 |
| **FusionCache 初始化** | **使用 FusionCache 的 Helper 必须在 `static {}` 块中完成 `FusionCache.config(...)` 初始化，包括 Config 参数和 CacheDataLoader** | 210 → Phase 2 Step 3 缓存初始化 |
| 缓存常量 | `private static final` FusionCache.Config 参数、缓存 Key 常量 | 210 → Phase 2 Step 3 |
| **TODO 标记** | **每个方法体上方有 `// TODO: [Tn] implement {methodName}`，Tn 与 TASKS.md 任务卡片 ID 对应** | 210 → Phase 2 Step 3 TODO 标记 |
| 依赖获取 | `DaoManager.getInstance()` 静态获取，不使用构造器注入 | 210 → Phase 2 Step 3 |

**缓存 API 正确性检查**（详见 [uw-cache.md](../../210-java-uniweb-design/references/backend/uniweb/uw-cache.md)）：
- FusionCache：config() / get() / getIfPresent() / put() / invalidate()
- GlobalCache：put() / get() / delete() / containsKey()
- GlobalLocker：tryLock() / keepLock() / unlock()
- Kryo 序列化必须使用具体实现类，不可用 List/Map/Set 接口

## 6. DTO/VO 质量

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 搜索字段裁剪 | 不需要的 @QueryMeta 搜索字段已删除 | 210 → Phase 2 Step 1 |
| 排序字段 | **不裁剪** ALLOWED_SORT_PROPERTY，保留无害 | 210 → Phase 2 Step 1 |
| 校验注解 | 按设计补充 @NotNull、@Size 等约束 | 210 → Phase 2 Step 1 |
| VO 必要性 | 仅在聚合多表数据时创建，非简单 CRUD 场景 | 210 → Phase 2 Step 4 |
| Entity 不修改 | entity 包下的文件未做任何修改 | 210 → 架构约定 |

## 7. PRD 覆盖度

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 功能点全覆盖 | 每个 PRD 功能点在 README 映射表中有对应接口 | 210 → Phase 1 PRD功能点映射 |
| 接口全覆盖 | 映射表中的接口在 Controller 中有实现 | 210 → Phase 1 PRD功能点映射 |
| 状态机覆盖 | 有状态实体的每个状态转换都有对应接口 | 210 → Phase 1 状态机设计 |
| 角色覆盖 | README 权限映射表中的角色在 Controller 注解中体现 | 210 → Phase 1 角色权限映射 |

## 8. UniWeb 技术栈合规

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 禁用 Lombok | 无 @Data/@Getter/@Setter 等注解 | 210 → 架构约定 |
| DaoManager 使用 | 使用 DaoManager（ResponseData 风格），非 DaoFactory（抛异常风格） | 210 → 架构约定 |
| 分页查询 | 使用 AuthQueryParam 自动注入 saasId | 210 → 架构约定 |
| 分布式 ID | 使用 daoManager.getSequenceId() 而非自增 | 210 → 架构约定 |
| Helper 命名 | service 包下统一 {Module}Helper，无独立 helper 包 | 210 → 架构约定 |
| Controller 分包 | 按角色分包（saas/mch/admin/rpc），无混合角色包 | 210 → 架构约定 |

## 9. 测试骨架质量（TDD Red）

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 测试类存在 | 每个 Helper 有对应 `{Module}HelperTest.java` | 210 → Phase 2 Step 3.5 Helper 单元测试 |
| 测试类位置 | `src/test/java/{包}/service/{Module}HelperTest.java` | 210 → Phase 2 Step 3.5 |
| Mockito 注解 | `@ExtendWith(MockitoExtension.class)` + `MockedStatic DaoManager/FusionCache` | 210 → Phase 2 Step 3.5 |
| 测试方法数 | 每个 Helper 方法 ≥ 2 个测试方法（正常 + 边界/异常） | 210 → Phase 2 Step 3.5 |
| @DisplayName | 每个测试方法有 `@DisplayName("...")` | 210 → Phase 2 Step 3.5 |
| 测试方法 Javadoc | 包含测试意图、准备数据、预期结果 | 210 → Phase 2 Step 3.5 |
| **方法体 TDD Red** | **使用 `fail("TDD Red: [Tn] Helper 方法尚未实现")` 标记 Red，Tn 与 TASKS.md 任务卡片 ID 一致** | 210 → Phase 2 Step 3.5 |
| 命名规范 | `test{Method}_{Scenario}_{ExpectedResult}` | 210 → Phase 2 Step 3.5 |
| Controller 测试 | 每个 Controller 有对应测试类，每个方法 ≥ 1 个测试 | 210 → Phase 2 Step 3.5 Controller 单元测试 |

## 10. 设计完成标准验证

> 逐项检查 210 → 设计完成标准 中的 17 项 checklist。

| # | 检查项 | 源技能依据 |
|---|--------|-----------|
| 1 | README.md 覆盖所有模块和全局策略（含测试策略章节） | 210 → 设计完成标准 |
| 2 | TASKS.md 包含并行分组、任务卡片和联调验证清单 | 210 → 设计完成标准 |
| 3 | 所有 Controller 方法有 Javadoc + @MscPermDeclare | 210 → 设计完成标准 |
| 4 | 所有 Helper 方法有 Javadoc（设计思路 + 实现步骤 + 缓存策略） | 210 → 设计完成标准 |
| 5 | 所有 Helper 方法体上方有 `// TODO: [Tn]` 标记，与 TASKS.md 任务卡片 ID 一一对应 | 210 → 设计完成标准 |
| 6 | 所有使用 FusionCache 的 Helper 有 `static { FusionCache.config(...) }` 初始化块 | 210 → 设计完成标准 |
| 7 | 所有 Helper 有对应测试类 | 210 → 设计完成标准 |
| 8 | 每个 Helper 方法有 ≥ 2 个测试方法（正常 + 边界/异常） | 210 → 设计完成标准 |
| 9 | 所有测试方法体有 `fail("TDD Red: [Tn]")` 标记，与任务 ID 一致 | 210 → 设计完成标准 |
| 10 | 所有 Controller 有对应测试类 | 210 → 设计完成标准 |
| 11 | 每个 Controller 方法有 ≥ 1 个测试方法 | 210 → 设计完成标准 |
| 12 | 测试方法有 @DisplayName + Javadoc | 210 → 设计完成标准 |
| 13 | DTO 搜索字段已按业务需求裁剪 | 210 → 设计完成标准 |
| 14 | `mvn compile` 编译通过 | 210 → 设计完成标准 |
| 15 | `mvn test-compile` 测试编译通过 | 210 → 设计完成标准 |
| 16 | `mvn test` 可运行（全红为预期） | 210 → 设计完成标准 |
| 17 | Swagger UI 可展示所有接口 | 210 → 设计完成标准 |
