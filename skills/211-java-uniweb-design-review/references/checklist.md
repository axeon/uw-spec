# SaaS 后端设计评审检查清单

## 1. README.md 完整性

| 检查项 | 要求 |
|--------|------|
| 模块总览 | 每个模块有名称、说明、复杂度、代码策略、涉及实体 |
| 模块依赖关系 | Mermaid 图，无循环依赖 |
| PRD 功能点映射 | 每个 PRD 功能点对应到模块和接口 |
| 角色权限映射 | SAAS/MCH/ADMIN/ROOT × 模块的 R/W 矩阵 |
| 全局缓存策略 | FusionCache/GlobalCache 选型、Key规范、TTL、更新机制 |
| 性能预算 | 数据量估算（3月/1年/3年）、索引策略、RT/QPS目标 |
| 外部依赖 | saas-finance、第三方接口、异步任务（如有） |
| 全局错误码 | 业务错误码范围分配（如有） |
| 状态机 | 有状态实体的状态流转图和操作映射（如有） |

## 2. 编译与 Swagger

| 检查项 | 要求 |
|--------|------|
| mvn compile | 无编译错误 |
| mvn test-compile | 测试代码编译无错误 |
| mvn test | 测试可运行（全红为预期，无编译/运行时错误） |
| Swagger UI | 启动后所有接口可展示 |
| Bean 注入 | 无循环依赖、无缺失 Bean |

## 3. Controller 质量

| 检查项 | 要求 |
|--------|------|
| 权限注解 | 每个方法有 `@MscPermDeclare`，user 类型与 README 角色映射一致 |
| 类级 Javadoc | 包含设计思路、依赖关系、需求映射 |
| 方法级 Javadoc | 包含设计思路 + 实现要求（具体步骤） |
| URL 规范 | 第一级为角色（/saas/, /mch/, /admin/, /rpc/ 等） |
| $PackageInfo$ | controller下root/ops/admin/saas/mch的二级目录下有 $PackageInfo$.java，用于定义一级菜单信息 |
| 响应格式 | 统一使用 ResponseData\<T\> |
| HTTP 方法 | 查询用 GET，写入用 POST，符合 RESTful 规范 |
| 方法裁剪 | 不需要的方法已删除（如不允许删除则无 delete 方法） |

**权限注解检查要点**：
- 类级别 `@MscPermDeclare` 设置 user 和 name
- 方法级别 `@MscPermDeclare` 设置 name 和 log
- log 级别：查询 REQUEST，写入 ALL，关键操作 CRIT

## 4. Helper 质量

| 检查项 | 要求 |
|--------|------|
| 类注解 | `@Component`，位于 service 包 |
| 类级 Javadoc | 包含设计思路、依赖关系（DaoManager/FusionCache 等）、需求映射 |
| 方法级 Javadoc | 包含设计思路 + 实现要求（具体步骤） + 缓存策略（如有） |
| 方法签名 | 入参出参明确，方法体返回默认值 |
| 缓存常量 | FusionCache 的 localCacheMaxNum、cacheExpireMillis 已定义 |
| 缓存 API | 使用 FusionCache/GlobalCache/GlobalLocker 的正确方法名 |
| 依赖注入 | 构造器注入，不使用 @Autowired |
| 分布式序列 | 新增使用 daoManager.getSequenceId(Entity.class) |
| 租户注入 | 使用 AuthServiceHelper.getSaasId()/getMchId() |
| 差量更新 | DataEntity setter 自动记录变更，支持差量更新 |

**缓存 API 正确性检查**：
- FusionCache：config() / get() / getIfPresent() / put() / invalidate()
- GlobalCache：put() / get() / delete() / containsKey()
- GlobalLocker：tryLock() / keepLock() / unlock()
- Kryo 序列化必须使用具体实现类，不可用 List/Map/Set 接口

## 5. DTO/VO 质量

| 检查项 | 要求 |
|--------|------|
| 搜索字段裁剪 | 不需要的 @QueryMeta 搜索字段已删除 |
| 排序字段裁剪 | 不需要的排序字段已从 ALLOWED_SORT_PROPERTY 移除 |
| 校验注解 | 按设计补充 @NotNull、@Size 等约束 |
| Swagger 注解 | 补充 @Schema(description=...) |
| VO 必要性 | 仅在聚合多表数据时创建，非简单 CRUD 场景 |
| Entity 不修改 | entity 包下的文件未做任何修改 |

## 6. PRD 覆盖度

| 检查项 | 要求 |
|--------|------|
| 功能点全覆盖 | 每个 PRD 功能点在 README 映射表中有对应接口 |
| 接口全覆盖 | 映射表中的接口在 Controller 中有实现 |
| 状态机覆盖 | 有状态实体的每个状态转换都有对应接口 |
| 角色覆盖 | README 权限映射表中的角色在 Controller 注解中体现 |

## 7. UniWeb 技术栈合规

| 检查项 | 要求 |
|--------|------|
| 禁用 Lombok | 无 @Data/@Getter/@Setter 等注解 |
| DaoManager 使用 | 使用 DaoManager（ResponseData 风格），非 DaoFactory（抛异常风格） |
| 分页查询 | 使用 AuthQueryParam 自动注入 saasId |
| 分布式 ID | 使用 daoManager.getSequenceId() 而非自增 |
| Helper 命名 | service 包下统一 {Module}Helper，无独立 helper 包 |
| Controller 分包 | 按角色分包（saas/mch/admin/rpc），无混合角色包 |

## 8. 测试骨架质量（TDD Red）

| 检查项 | 要求 |
|--------|------|
| 测试类存在 | 每个 Helper 有对应 `{Module}HelperTest.java` |
| 测试类位置 | `src/test/java/{包}/service/{Module}HelperTest.java` |
| Mockito 注解 | `@ExtendWith(MockitoExtension.class)` + `@Mock DaoManager` + `@InjectMocks Helper` |
| 测试方法数 | 每个 Helper 方法 ≥ 2 个测试方法（正常 + 边界/异常） |
| @DisplayName | 每个测试方法有 `@DisplayName("...")` |
| 测试方法 Javadoc | 包含测试意图、准备数据、预期结果 |
| 方法体 | 使用 `fail("TDD Red: Helper 方法尚未实现")` 标记 Red |
| 命名规范 | `test{Method}_{Scenario}_{ExpectedResult}` |
| Helper 依赖注入 | Helper 使用构造器注入 DaoManager（非 static final），确保 @Mock 可注入 |
| README 测试策略 | README.md 包含测试策略章节（分层/覆盖率/Mock策略） |
