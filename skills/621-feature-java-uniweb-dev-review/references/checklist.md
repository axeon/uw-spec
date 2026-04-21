# 功能Java后端开发评审检查清单

## 方案一致性检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | 接口路径 | 与技术方案完全一致 | Critical |
| 2 | DTO字段 | 与技术方案定义一致, 未新增未授权字段 | Major |
| 3 | Helper方法 | 方法签名、参数、返回值与方案一致 | Major |
| 4 | 数据库操作 | 表名、字段名与DDL一致 | Critical |
| 5 | 权限注解 | 与方案中的角色权限映射一致 | Major |
| 6 | 缓存策略 | 与方案中的缓存设计一致 | Minor |

## TDD实践检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | Red→Green | 从210骨架实现为Green, 非跳过重写 | Major |
| 2 | 覆盖率 | 行覆盖≥80%, 分支覆盖≥70% | Major |
| 3 | 测试质量 | @ExtendWith+@Mock+@InjectMocks, @DisplayName | Major |
| 4 | 无fail()残留 | 无 fail("Not implemented") 等占位代码 | Critical |
| 5 | Mock合理性 | 仅Mock外部依赖, 不Mock被测逻辑 | Minor |

## UniWeb规范检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | 数据访问 | 使用DaoManager, 禁止直接JPA/MyBatis | Critical |
| 2 | 响应格式 | 使用ResponseData/DataList统一响应 | Major |
| 3 | 权限查询 | 使用AuthQueryParam | Major |
| 4 | 缓存 | 使用FusionCache, 非Redis直接调用 | Major |
| 5 | 禁用Lombok | 无@Data/@Getter/@Setter等注解 | Major |
| 6 | 构造器注入 | 使用构造器注入, 非@Autowired字段注入 | Minor |

## 代码质量检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | 圈复杂度 | ≤10 | Major |
| 2 | 方法行数 | ≤50行 | Minor |
| 3 | 命名规范 | 类名PascalCase, 方法名camelCase | Minor |
| 4 | 分层清晰 | Controller不包含业务逻辑 | Major |
| 5 | 异常处理 | 业务异常明确, 不吞异常 | Major |

## 安全性检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | 输入校验 | @Valid注解, DTO校验完整 | Critical |
| 2 | SQL安全 | QueryParam参数化, 无字符串拼接SQL | Critical |
| 3 | 权限控制 | @MscPermDeclare声明完整 | Major |
| 4 | 租户隔离 | 查询包含saas_id条件 | Critical |
| 5 | 敏感数据 | 密码/密钥不明文存储和日志 | Critical |

## 影响范围检查

| # | 检查项 | 标准 | 严重程度 |
|---|--------|------|---------|
| 1 | 变更文件 | 仅修改功能相关文件 | Major |
| 2 | 无副作用 | 未修改公共工具类/基类（除非必要） | Major |
| 3 | 向后兼容 | 未修改现有接口签名 | Critical |
