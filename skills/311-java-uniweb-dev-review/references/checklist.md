# Java SaaS开发评审检查清单

> 基于 UniWeb + SaaS 技术栈。禁用 Lombok，使用 DaoManager/FusionCache/ResponseData。

## 1. TDD实践

| 检查项 | 要求 |
|--------|------|
| 测试先行 | 220 阶段有 Red 骨架，310 阶段实现变 Green |
| 覆盖率 | 行覆盖≥80%，分支覆盖≥70% |
| 测试结构 | @ExtendWith(MockitoExtension.class) + @Mock + @InjectMocks |
| 断言质量 | 断言明确，边界测试完整，无 fail() 残留 |
| Mock 模式 | DaoManager 用 @Mock，FusionCache/GlobalLocker/AuthServiceHelper 用 MockedStatic |
| 命名规范 | test{Method}_{Scenario}_{ExpectedResult}，有 @DisplayName |

**示例（UniWeb 风格）**：
```java
@ExtendWith(MockitoExtension.class)
class ProductHelperTest {

    @Mock
    private DaoManager daoManager;

    @InjectMocks
    private ProductHelper productHelper;

    @Test
    @DisplayName("查询商品详情 - 正常返回实体")
    void testGetProduct_Found_ReturnEntity() {
        Product product = new Product();
        product.setId(1L);
        when(daoManager.load(Product.class, 1L)).thenReturn(ResponseData.success(product));

        ResponseData<Product> result = productHelper.getProduct(1L);

        assertTrue(result.isSuccess());
        assertEquals(1L, result.getData().getId());
    }

    @Test
    @DisplayName("新增商品 - 注入saasId并持久化")
    void testSaveProduct_Normal_ReturnCreated() {
        Product entity = new Product();
        try (MockedStatic<AuthServiceHelper> authMock = mockStatic(AuthServiceHelper.class)) {
            authMock.when(AuthServiceHelper::getSaasId).thenReturn(1001L);
            when(daoManager.getSequenceId(Product.class)).thenReturn(1L);
            when(daoManager.save(any(Product.class))).thenReturn(ResponseData.success(entity));

            ResponseData<Product> result = productHelper.saveProduct(entity);

            assertTrue(result.isSuccess());
        }
    }
}
```

## 2. UniWeb 规范

| 检查项 | 要求 |
|--------|------|
| 禁用 Lombok | 无 @Data/@Getter/@Setter/@RequiredArgsConstructor 等注解 |
| DAO 使用 | 使用 DaoManager（ResponseData 风格），非 DaoFactory（抛异常风格） |
| 响应格式 | 统一使用 ResponseData\<T\>，非自定义 Result |
| 多租户 | AuthQueryParam 自动注入 saasId，手动 bindMchId()/bindUserId() |
| 缓存 | FusionCache/GlobalCache/GlobalLocker，非 Spring @Cacheable |
| 分布式 ID | daoManager.getSequenceId(Entity.class)，非自增 |
| 权限注解 | @MscPermDeclare，非 Spring Security 注解 |
| 分页查询 | AuthQueryParam + daoManager.list()，非 MyBatis-Plus Page |
| Helper 命名 | service 包下统一 {Module}Helper，无 Service/ServiceImpl |
| Controller 分包 | 按角色分包（saas/mch/admin/rpc），无混合角色包 |

**示例（UniWeb 风格 Helper）**：
```java
@Component
public class ProductHelper {

    private static final DaoManager daoManager = DaoManager.getInstance();

    public ResponseData<Product> getProduct(Long id) {
        return daoManager.load(Product.class, id);
    }

    public ResponseData<Product> saveProduct(Product entity) {
        entity.setId(daoManager.getSequenceId(Product.class));
        entity.setSaasId(AuthServiceHelper.getSaasId());
        return daoManager.save(entity);
    }
}
```

**示例（UniWeb 风格 Controller）**：
```java
@RestController
@RequestMapping("/saas/product")
@MscPermDeclare(name = "商品管理", user = UserType.SAAS, log = ActionLog.NONE)
public class ProductController {

    private final ProductHelper productHelper;

    public ProductController(ProductHelper productHelper) {
        this.productHelper = productHelper;
    }

    @PostMapping("/save")
    @MscPermDeclare(name = "新增商品", log = ActionLog.ALL)
    public ResponseData<Product> save(@RequestBody @Valid Product entity) {
        return productHelper.saveProduct(entity);
    }
}
```

## 3. 代码质量

| 检查项 | 要求 |
|--------|------|
| 单一职责 | 每个 Helper 负责一个模块的业务逻辑 |
| 分层清晰 | Controller → Helper → DaoManager，无跨层调用 |
| 命名规范 | 类名大驼峰，方法名小驼峰，布尔变量 is/has/should 前缀 |
| 复杂度 | 圈复杂度≤10，方法行数≤50 |
| 构造器注入 | 通过构造器注入依赖，不使用 @Autowired |
| 差量更新 | DataEntity setter 自动记录变更，支持差量更新 |

## 4. 安全性

| 检查项 | 要求 |
|--------|------|
| 输入校验 | @RequestBody 参数加 @Valid，DTO 有 @NotNull/@Size 等约束 |
| SQL 注入 | 使用 DaoManager QueryParam 参数化查询，禁止拼接 SQL |
| 认证授权 | 每个 Controller 方法有 @MscPermDeclare |
| 敏感信息 | 密码加密存储，日志中不打印敏感字段 |
| 租户隔离 | AuthQueryParam 自动注入 saasId，禁止绕过租户隔离查询 |
