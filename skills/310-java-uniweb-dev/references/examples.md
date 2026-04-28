# Java SaaS 开发示例（UniWeb 技术栈）

> 基于 UniWeb + SaaS 技术栈的 TDD 开发示例。使用 `@SpringBootTest` 全链路测试，真实数据库交互。

## TDD 开发流程

```
Red（210设计阶段）→ 编写测试骨架（fail）
  ↓
Green（310开发阶段）→ 实现 Helper 逻辑（测试变绿）
  ↓
Refactor（310开发阶段）→ 重构优化（测试保持全绿）
```

## 测试认证工具类

```java
public class TestAuthUtils {
    
    // 测试专用标识，用于数据清理
    public static final Long TEST_SAAS_ID = 666L;
    public static final Long TEST_MCH_ID = 666L;
    public static final Long TEST_USER_ID = 666L;
    
    /**
     * 设置测试用户上下文（使用默认测试用户666）
     */
    public static void setTestUser() {
        setTestUser(TEST_SAAS_ID, TEST_MCH_ID, TEST_USER_ID, UserType.SAAS);
    }
    
    /**
     * 设置测试用户上下文（自定义用户）
     * 反射调用 AuthServiceHelper.setContextToken()
     */
    public static void setTestUser(Long saasId, Long mchId, Long userId, UserType userType) {
        AuthTokenData tokenData = createAuthTokenData(saasId, mchId, userId, userType);
        
        try {
            Method method = AuthServiceHelper.class.getDeclaredMethod("setContextToken", AuthTokenData.class);
            method.setAccessible(true);
            method.invoke(null, tokenData);
        } catch (Exception e) {
            throw new RuntimeException("设置测试用户失败", e);
        }
    }
    
    /**
     * 清理测试用户上下文
     */
    public static void clear() {
        try {
            Method method = AuthServiceHelper.class.getDeclaredMethod("setContextToken", AuthTokenData.class);
            method.setAccessible(true);
            method.invoke(null, (AuthTokenData) null);
        } catch (Exception e) {
            // 清理失败不影响测试
        }
    }
    
    private static AuthTokenData createAuthTokenData(Long saasId, Long mchId, 
                                                      Long userId, UserType userType) {
        AuthTokenData data = new AuthTokenData();
        data.setSaasId(saasId);
        data.setMchId(mchId);
        data.setUserId(userId);
        data.setUserType(userType.getCode());
        data.setLoginName("test_user_" + userId);
        data.setLoginDate(new Date());
        data.setToken(UUID.randomUUID().toString());
        return data;
    }
}
```

## 测试基类

### BaseIntegrationTest.java

```java
@SpringBootTest(classes = TestContextConfig.class)
@TestPropertySource(properties = {
    "spring.profiles.active=test",
    "spring.cloud.nacos.config.enabled=false",
    "spring.cloud.nacos.discovery.enabled=false"
})
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
public abstract class BaseIntegrationTest {
    
    @Autowired
    protected JdbcTemplate jdbcTemplate;
    
    @Autowired
    protected ObjectMapper objectMapper;
    
    // 测试数据标识前缀，用于隔离
    protected static final String TEST_PREFIX = "TEST_" + System.currentTimeMillis() + "_";
    
    @BeforeAll
    void globalSetUp() {
        // 全局初始化，每个测试类执行一次
    }
    
    @BeforeEach
    void setUp() {
        // 设置测试用户（666）
        TestAuthUtils.setTestUser();
    }
    
    @AfterEach
    void tearDown() {
        // 清理本测试方法产生的数据
        cleanTestData();
        // 清理测试用户上下文
        TestAuthUtils.clear();
    }
    
    @AfterAll
    void globalTearDown() {
        // 全局清理，每个测试类执行一次
        cleanAllTestData();
    }
    
    // 清理当前测试产生的数据（由子类实现）
    protected abstract void cleanTestData();
    
    // 清理所有测试数据（按 saas_id = 666 清理）
    protected void cleanAllTestData() {
        jdbcTemplate.execute("SET FOREIGN_KEY_CHECKS = 0");
        jdbcTemplate.execute("DELETE FROM order_item WHERE saas_id = 666");
        jdbcTemplate.execute("DELETE FROM order WHERE saas_id = 666");
        jdbcTemplate.execute("DELETE FROM product WHERE saas_id = 666");
        jdbcTemplate.execute("SET FOREIGN_KEY_CHECKS = 1");
    }
    
    // 辅助方法：生成测试数据名称
    protected String testName(String name) {
        return TEST_PREFIX + name + "_" + UUID.randomUUID().toString().substring(0, 8);
    }
}
```

### TestContextConfig.java

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(basePackages = "com.example.app")
public class TestContextConfig {
    // 测试专用的配置覆盖
}
```

### application-test.yml

```yaml
spring:
  main:
    banner-mode: off
  cloud:
    nacos:
      config:
        enabled: false
      discovery:
        enabled: false

# uw-dao 测试配置（覆盖 Nacos）
uw:
  dao:
    conn-pool:
      root:
        driver: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/test_db?useSSL=false&serverTimezone=Asia/Shanghai
        username: root
        password: root
        min-conn: 1
        max-conn: 5

logging:
  level:
    root: WARN
```

## 设计阶段产出（210，Red）

### Helper 方法签名

```java
public class ProductHelper {

    private static final DaoManager daoManager = DaoManager.getInstance();

    // TODO: [T1] implement getProduct with FusionCache
    public static ResponseData<Product> getProduct(Long id) {
        return ResponseData.success(null);
    }

    // TODO: [T1] implement saveProduct
    public static ResponseData<Product> saveProduct(Product entity) {
        return ResponseData.success(null);
    }
    
    // TODO: [T1] implement updateProduct
    public static ResponseData<Product> updateProduct(Product entity) {
        return ResponseData.success(null);
    }
    
    // TODO: [T1] implement deleteProduct
    public static ResponseData<Integer> deleteProduct(Long id) {
        return ResponseData.success(0);
    }
    
    // TODO: [T1] implement listProduct
    public static ResponseData<DataList<Product>> listProduct(AuthQueryParam param) {
        return ResponseData.success(null);
    }
}
```

### 测试骨架（210，Red）

```java
class ProductHelperTest extends BaseIntegrationTest {

    private final List<Long> createdProductIds = new ArrayList<>();

    @Override
    protected void cleanTestData() {
        if (!createdProductIds.isEmpty()) {
            String ids = createdProductIds.stream()
                .map(String::valueOf)
                .collect(Collectors.joining(","));
            jdbcTemplate.execute(
                "DELETE FROM product WHERE id IN (" + ids + ")"
            );
            createdProductIds.clear();
        }
    }

    @Test
    @DisplayName("查询商品详情 - 正常返回")
    void testGetProduct_Found_ReturnEntity() {
        fail("TDD Red: [T1] getProduct 正常返回");
    }

    @Test
    @DisplayName("查询商品详情 - ID不存在返回warn")
    void testGetProduct_NotFound_ReturnWarn() {
        fail("TDD Red: [T1] getProduct 不存在");
    }

    @Test
    @DisplayName("新增商品 - 正常创建")
    void testSaveProduct_Normal_ReturnCreated() {
        fail("TDD Red: [T1] saveProduct 正常创建");
    }
    
    @Test
    @DisplayName("更新商品 - 差量更新只修改变更字段")
    void testUpdateProduct_DeltaUpdate_OnlyModifiedFields() {
        fail("TDD Red: [T1] updateProduct 差量更新");
    }
    
    @Test
    @DisplayName("删除商品 - 删除后无法查询")
    void testDeleteProduct_Success_CannotQueryAgain() {
        fail("TDD Red: [T1] deleteProduct 删除成功");
    }
    
    @Test
    @DisplayName("列表查询 - 支持分页和排序")
    void testListProduct_WithPagination_ReturnPagedResult() {
        fail("TDD Red: [T1] listProduct 分页查询");
    }
}
```

## 开发阶段实现（310，Green）

### Helper 实现

```java
public class ProductHelper {

    private static final Logger log = LoggerFactory.getLogger(ProductHelper.class);
    private static final DaoManager daoManager = DaoManager.getInstance();
    private static final int CACHE_MAX_NUM = 500;
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    public static ResponseData<Product> getProduct(Long id) {
        // 先查缓存
        Optional<Product> cached = FusionCache.getIfPresent(Product.class, id);
        if (cached.isPresent()) {
            return ResponseData.success(cached.get());
        }
        
        // 查数据库
        ResponseData<Product> result = daoManager.load(Product.class, id);
        if (result.isSuccess() && result.getData() != null) {
            FusionCache.put(Product.class, id, result.getData(), 
                CACHE_MAX_NUM, CACHE_EXPIRE_MILLIS);
        }
        return result;
    }

    public static ResponseData<Product> saveProduct(Product entity) {
        entity.setId(daoManager.getSequenceId(Product.class));
        entity.setSaasId(AuthServiceHelper.getSaasId());
        entity.setCreateDate(new Date());
        entity.setUpdateDate(new Date());
        return daoManager.save(entity);
    }
    
    public static ResponseData<Product> updateProduct(Product entity) {
        entity.setUpdateDate(new Date());
        ResponseData<Product> result = daoManager.update(entity);
        if (result.isSuccess()) {
            FusionCache.invalidate(Product.class, entity.getId());
        }
        return result;
    }
    
    public static ResponseData<Integer> deleteProduct(Long id) {
        ResponseData<Integer> result = daoManager.delete(Product.class, id);
        if (result.isSuccess()) {
            FusionCache.invalidate(Product.class, id);
        }
        return result;
    }
    
    public static ResponseData<DataList<Product>> listProduct(AuthQueryParam param) {
        return daoManager.list(Product.class, param);
    }
}
```

### 测试实现（310，Green）

```java
class ProductHelperTest extends BaseIntegrationTest {

    private final List<Long> createdProductIds = new ArrayList<>();

    @Override
    protected void cleanTestData() {
        if (!createdProductIds.isEmpty()) {
            String ids = createdProductIds.stream()
                .map(String::valueOf)
                .collect(Collectors.joining(","));
            jdbcTemplate.execute(
                "DELETE FROM product WHERE id IN (" + ids + ")"
            );
            createdProductIds.clear();
        }
    }

    @Test
    @DisplayName("查询商品详情 - 正常返回")
    void testGetProduct_Found_ReturnEntity() {
        // 准备：创建测试数据
        Product product = new Product();
        product.setName(testName("iPhone"));
        product.setPrice(new BigDecimal("5999.00"));
        product.setStatus(1);
        ResponseData<Product> saved = ProductHelper.saveProduct(product);
        assertTrue(saved.isSuccess());
        Long productId = saved.getData().getId();
        createdProductIds.add(productId);

        // 执行
        ResponseData<Product> result = ProductHelper.getProduct(productId);

        // 验证
        assertTrue(result.isSuccess());
        assertNotNull(result.getData());
        assertEquals(productId, result.getData().getId());
        assertEquals(product.getName(), result.getData().getName());
    }

    @Test
    @DisplayName("查询商品详情 - ID不存在返回warn")
    void testGetProduct_NotFound_ReturnWarn() {
        // 执行：查询不存在的ID
        ResponseData<Product> result = ProductHelper.getProduct(99999999L);

        // 验证
        assertFalse(result.isSuccess());
        assertEquals("WARN", result.getCode());
    }

    @Test
    @DisplayName("新增商品 - 正常创建")
    void testSaveProduct_Normal_ReturnCreated() {
        // 准备
        Product product = new Product();
        product.setName(testName("MacBook"));
        product.setPrice(new BigDecimal("12999.00"));
        product.setStatus(1);

        // 执行
        ResponseData<Product> result = ProductHelper.saveProduct(product);

        // 验证
        assertTrue(result.isSuccess());
        assertNotNull(result.getData().getId());
        assertTrue(result.getData().getId() > 0);
        assertNotNull(result.getData().getCreateDate());
        
        // 记录ID用于清理
        createdProductIds.add(result.getData().getId());
        
        // 验证真实入库
        ResponseData<Product> loaded = ProductHelper.getProduct(result.getData().getId());
        assertTrue(loaded.isSuccess());
        assertEquals(product.getName(), loaded.getData().getName());
    }
    
    @Test
    @DisplayName("更新商品 - 差量更新只修改变更字段")
    void testUpdateProduct_DeltaUpdate_OnlyModifiedFields() {
        // 准备：创建商品
        Product product = new Product();
        product.setName(testName("iPad"));
        product.setPrice(new BigDecimal("3999.00"));
        product.setDescription("原始描述");
        product.setStatus(1);
        ResponseData<Product> saved = ProductHelper.saveProduct(product);
        Long productId = saved.getData().getId();
        createdProductIds.add(productId);
        Date originalCreateDate = saved.getData().getCreateDate();

        // 修改部分字段
        Product toUpdate = saved.getData();
        toUpdate.setName(testName("iPadPro"));
        toUpdate.setPrice(new BigDecimal("4999.00"));
        // description 和 status 不修改

        // 执行更新
        ResponseData<Product> updated = ProductHelper.updateProduct(toUpdate);

        // 验证
        assertTrue(updated.isSuccess());
        
        // 重新查询验证差量更新
        ResponseData<Product> loaded = ProductHelper.getProduct(productId);
        Product loadedProduct = loaded.getData();
        assertEquals(toUpdate.getName(), loadedProduct.getName());
        assertEquals(new BigDecimal("4999.00"), loadedProduct.getPrice());
        assertEquals("原始描述", loadedProduct.getDescription());  // 未修改
        assertEquals(1, loadedProduct.getStatus());  // 未修改
        assertEquals(originalCreateDate, loadedProduct.getCreateDate());  // 未修改
        assertNotNull(loadedProduct.getUpdateDate());  // 已更新
    }
    
    @Test
    @DisplayName("删除商品 - 删除后无法查询")
    void testDeleteProduct_Success_CannotQueryAgain() {
        // 准备：创建商品
        Product product = new Product();
        product.setName(testName("AirPods"));
        ResponseData<Product> saved = ProductHelper.saveProduct(product);
        Long productId = saved.getData().getId();
        createdProductIds.add(productId);

        // 执行删除
        ResponseData<Integer> deleted = ProductHelper.deleteProduct(productId);

        // 验证删除结果
        assertTrue(deleted.isSuccess());
        assertEquals(1, deleted.getData());
        
        // 验证已删除
        ResponseData<Product> notFound = ProductHelper.getProduct(productId);
        assertFalse(notFound.isSuccess());
        
        // 从清理列表移除（已删除）
        createdProductIds.remove(productId);
    }
    
    @Test
    @DisplayName("列表查询 - 支持分页和排序")
    void testListProduct_WithPagination_ReturnPagedResult() {
        // 准备：批量创建测试数据
        List<Long> ids = new ArrayList<>();
        for (int i = 0; i < 5; i++) {
            Product p = new Product();
            p.setName(testName("Product" + i));
            p.setPrice(new BigDecimal("100"));
            p.setStatus(1);
            ResponseData<Product> saved = ProductHelper.saveProduct(p);
            ids.add(saved.getData().getId());
        }
        createdProductIds.addAll(ids);

        // 查询第一页
        AuthQueryParam param = new AuthQueryParam();
        param.setPage(1);
        param.setSize(3);
        param.SORT_NAME("id").SORT_TYPE("DESC");
        
        ResponseData<DataList<Product>> result = ProductHelper.listProduct(param);

        // 验证分页
        assertTrue(result.isSuccess());
        assertEquals(5, result.getData().getTotal());  // 总共5条
        assertEquals(3, result.getData().getRecords().size());  // 当前页3条
        
        // 验证排序（DESC，第一条应该是最后创建的）
        assertEquals(ids.get(4), result.getData().getRecords().get(0).getId());
    }
}
```

## Controller 全链路测试示例

```java
@SpringBootTest
@AutoConfigureMockMvc
@TestPropertySource(properties = {
    "spring.profiles.active=test",
    "spring.cloud.nacos.config.enabled=false",
    "spring.cloud.nacos.discovery.enabled=false"
})
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;
    
    @Autowired
    private JdbcTemplate jdbcTemplate;

    private final List<Long> createdProductIds = new ArrayList<>();
    
    private static final String TEST_PREFIX = "TEST_" + System.currentTimeMillis() + "_";

    @BeforeEach
    void setUp() {
        // 设置测试用户（666）
        TestAuthUtils.setTestUser();
    }

    @AfterEach
    void cleanTestData() {
        if (!createdProductIds.isEmpty()) {
            String ids = createdProductIds.stream()
                .map(String::valueOf)
                .collect(Collectors.joining(","));
            jdbcTemplate.execute(
                "DELETE FROM product WHERE id IN (" + ids + ")"
            );
            createdProductIds.clear();
        }
        // 清理测试用户上下文
        TestAuthUtils.clear();
    }

    @Test
    @DisplayName("POST /saas/product/save - 正常创建商品")
    void testSaveProduct_Success_Return201() throws Exception {
        ProductDTO dto = new ProductDTO();
        dto.setName(testName("ControllerTestProduct"));
        dto.setPrice(new BigDecimal("999.00"));
        dto.setStatus(1);

        String response = mockMvc.perform(post("/saas/product/save")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(dto)))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.id").exists())
            .andExpect(jsonPath("$.data.name").value(dto.getName()))
            .andReturn()
            .getResponse()
            .getContentAsString();

        // 提取ID用于清理
        Long id = objectMapper.readTree(response)
            .path("data").path("id").asLong();
        createdProductIds.add(id);
    }

    @Test
    @DisplayName("GET /saas/product/detail - 查询存在的商品")
    void testGetProduct_Exists_ReturnEntity() throws Exception {
        // 先通过Helper创建数据
        Product product = new Product();
        product.setName(testName("DetailTest"));
        product.setPrice(new BigDecimal("1999.00"));
        ResponseData<Product> saved = ProductHelper.saveProduct(product);
        Long productId = saved.getData().getId();
        createdProductIds.add(productId);

        // 查询
        mockMvc.perform(get("/saas/product/detail")
                .param("id", productId.toString()))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.id").value(productId))
            .andExpect(jsonPath("$.data.name").value(product.getName()));
    }

    @Test
    @DisplayName("GET /saas/product/list - 分页查询")
    void testListProduct_WithPagination_ReturnPagedResult() throws Exception {
        // 创建测试数据
        for (int i = 0; i < 3; i++) {
            Product p = new Product();
            p.setName(testName("ListTest" + i));
            p.setPrice(new BigDecimal("100"));
            ResponseData<Product> saved = ProductHelper.saveProduct(p);
            createdProductIds.add(saved.getData().getId());
        }

        mockMvc.perform(get("/saas/product/list")
                .param("page", "1")
                .param("size", "10"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.success").value(true))
            .andExpect(jsonPath("$.data.total").value(3))
            .andExpect(jsonPath("$.data.records").isArray())
            .andExpect(jsonPath("$.data.records.length()").value(3));
    }
    
    private String testName(String name) {
        return TEST_PREFIX + name + "_" + UUID.randomUUID().toString().substring(0, 8);
    }
}
```

## Maven 配置

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.1.2</version>
            <configuration>
                <!-- 复用 Spring Context -->
                <reuseForks>true</reuseForks>
                <forkCount>1</forkCount>
                
                <!-- 系统属性 -->
                <systemPropertyVariables>
                    <spring.profiles.active>test</spring.profiles.active>
                </systemPropertyVariables>
            </configuration>
        </plugin>
    </plugins>
</build>
```

## 关键要点

| 要点 | 说明 |
|------|------|
| **全链路测试** | 使用 `@SpringBootTest` 启动完整 Spring 上下文，测试真实数据库交互 |
| **真实依赖** | DaoManager/FusionCache/GlobalLocker 均使用真实实例，测试真实数据库读写 |
| **单 Context** | 所有测试继承 `BaseIntegrationTest`，共享 Spring Context，只启动一次 |
| **测试用户** | 使用 `TestAuthUtils.setTestUser()` 设置测试用户（saasId=mchId=userId=666） |
| **数据隔离** | 使用 `TEST_时间戳_UUID` 前缀 + `@AfterEach` 手动清理，不用事务回滚 |
| **数据清理** | 按 `saas_id = 666` 清理测试数据，方便识别和清理 |
