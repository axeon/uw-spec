# Java SaaS开发评审检查清单

> **评审原则**：每个检查项的标准以 [310-java-uniweb-dev/SKILL.md](../../310-java-uniweb-dev/SKILL.md) 和 [210-java-uniweb-design/SKILL.md](../../210-java-uniweb-design/SKILL.md) 中的原始约定为准。下表"源技能依据"列指向源技能的具体章节，评审时必须回到源技能核实。

## 1. TDD实践

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 测试先行 | 210 阶段有 Red 骨架，310 阶段实现变 Green | 310 → 开发步骤 2.4 |
| 覆盖率 | 行覆盖≥80%，分支覆盖≥70% | 310 → 开发步骤 2.4 |
| 测试结构 | @ExtendWith(MockitoExtension.class) + MockedStatic（DaoManager/FusionCache/GlobalLocker/AuthServiceHelper） | 310 → 开发步骤 2.4 Mock模式 |
| 断言质量 | 断言明确，边界测试完整，无 fail() 残留，无 TODO 残留 | 310 → 开发步骤 2.4 |
| Mock 模式 | 所有依赖统一使用 MockedStatic + try-with-resources | 310 → 开发步骤 2.4 Mock模式 |
| 命名规范 | test{Method}_{Scenario}_{ExpectedResult}，有 @DisplayName | 310 → 开发步骤 2.4 |
| **TODO 清理** | **已实现的 Helper 方法上方 `// TODO: [Tn]` 行已删除** | 310 → 开发步骤 2.3 |

## 2. UniWeb 规范

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 禁用 Lombok | 无 @Data/@Getter/@Setter/@RequiredArgsConstructor 等注解 | 210 → 架构约定 |
| DAO 使用 | 使用 DaoManager（ResponseData 风格），非 DaoFactory（抛异常风格） | 310 → 开发步骤 2.3 |
| 响应格式 | 统一使用 ResponseData\<T\>，非自定义 Result | 310 → 开发步骤 2.3 |
| 多租户 | AuthQueryParam 自动注入 saasId，手动 bindMchId()/bindUserId() | 310 → 开发步骤 2.3 |
| 缓存 | FusionCache/GlobalCache/GlobalLocker | 310 → 开发步骤 2.3 |
| 分布式 ID | daoManager.getSequenceId(Entity.class)，非自增 | 310 → 开发步骤 2.3 |
| 权限注解 | @MscPermDeclare，非 Spring Security 注解 | 210 → 架构约定 |
| 分页查询 | AuthQueryParam + daoManager.list()，非 MyBatis-Plus Page | 310 → 开发步骤 2.3 |
| Helper 命名 | service 包下统一 {Module}Helper，无 Service/ServiceImpl | 210 → 架构约定 |
| Controller 分包 | 按角色分包（saas/mch/admin/rpc），无混合角色包 | 210 → 架构约定 |
| **Helper 方法风格** | **全部 static 方法，DaoManager 通过 getInstance() 静态获取** | 310 → 开发步骤 2.3 |

## 3. 代码质量

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 单一职责 | 每个 Helper 负责一个模块的业务逻辑 | 310 → 开发步骤 2.3 |
| 分层清晰 | Controller → Helper → DaoManager，无跨层调用 | 310 → 开发步骤 2.3 |
| 命名规范 | 类名大驼峰，方法名小驼峰，布尔变量 is/has/should 前缀 | 310 → 开发步骤 2.3 |
| 复杂度 | 圈复杂度≤10，方法行数≤50 | 310 → 开发步骤 2.3 |
| 依赖获取 | `DaoManager.getInstance()` 静态获取，不使用构造器注入 | 310 → 开发步骤 2.3 |
| 差量更新 | DataEntity setter 自动记录变更，支持差量更新 | 310 → 开发步骤 2.3 |

## 4. 安全性

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| 输入校验 | @RequestBody 参数加 @Valid，DTO 有 @NotNull/@Size 等约束 | 310 → 开发步骤 2.3 |
| SQL 注入 | 使用 DaoManager QueryParam 参数化查询，禁止拼接 SQL | 310 → 开发步骤 2.3 |
| 认证授权 | 每个 Controller 方法有 @MscPermDeclare | 210 → 架构约定 |
| 敏感信息 | 密码加密存储，日志中不打印敏感字段 | 310 → 开发步骤 2.3 |
| 租户隔离 | AuthQueryParam 自动注入 saasId，禁止绕过租户隔离查询 | 310 → 开发步骤 2.3 |

## 5. 210→310 衔接验证

| 检查项 | 要求 | 源技能依据 |
|--------|------|-----------|
| TASKS.md 消费 | 310 按 TASKS.md 任务卡片执行，卡片中的方法列表与实际实现一致 | 310 → 210→310 衔接协议 |
| TODO 清理完整性 | `grep "// TODO:" src/main/java/` 结果为空（全部已清理） | 310 → 开发步骤 2.2 TODO扫描确认 |
| TDD Red→Green | `grep "TDD Red" src/test/java/` 结果为空（全部测试已变绿） | 310 → 开发步骤 2.4 |
| mvn test 全绿 | `mvn test` 全部通过，无失败用例 | 310 → 开发步骤 2.5 模块验证 |
