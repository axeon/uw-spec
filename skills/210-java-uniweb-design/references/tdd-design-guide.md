# 设计阶段 TDD 指南

> 设计阶段（210）的 TDD 实践指南。测试先行——在设计阶段产出测试骨架，确保每个 Helper 方法都有可执行的测试契约。

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
| **Red**（设计阶段 210） | 编写测试骨架 + Helper 签名 | 测试全红，但可编译运行 |
| **Green**（开发阶段 310） | 实现 Helper 逻辑 | 测试逐个变绿 |
| **Refactor**（开发阶段 310） | 重构优化 | 测试保持全绿 |

## 职责划分

| 测试类型 | 负责角色 | 负责阶段 | 说明 |
|---------|---------|---------|------|
| 单元测试（Helper） | 架构师 / Java 开发工程师 | 210（骨架）+ 310（实现） | 本技能产出 |
| 测试脚本 | 测试工程师 | 330+ | 测试用例自动化 |
| API 测试 | 测试工程师 | 410+ | 接口全量验证 |
| E2E 测试 | 测试工程师 | 410+ | 端到端流程验证 |
| 压力测试 | 测试工程师 | 按需 | JMeter 性能测试 |
| 安全测试 | 测试工程师 | 按需 | OWASP/Trivy 扫描 |

> 测试工程师不参与单元测试编写。单元测试是开发工程师的本职工作，从设计阶段的骨架到开发阶段的实现，全程由开发侧负责。

## 测试策略

### 测试原则

| 原则 | 说明 |
|------|------|
| **全链路测试** | 使用 `@SpringBootTest` 启动完整 Spring 上下文，测试真实行为 |
| **不 Mock 框架** | 不 Mock DaoManager/FusionCache，测试真实数据库交互 |
| **单 Context** | 所有测试共享同一个 Spring Context，只启动一次 |
| **数据隔离** | 使用测试数据前缀 + 手动清理，不用事务回滚 |

### 测试分层

| 层级 | 测试目标 | 技术方案 |
|------|---------|---------|
| Helper 测试 | 业务逻辑 + 数据访问 | `@SpringBootTest` + 真实数据库 |
| Controller 测试 | API 契约 + 参数校验 | `@SpringBootTest` + `MockMvc` |
| 端到端测试 | 跨模块业务流程 | `@SpringBootTest` + 完整链路 |

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

## 设计阶段测试产出要求

### 产出物

| 产出物 | 位置 | 说明 |
|--------|------|------|
| 测试类 | `src/test/java/{包路径}/service/{Module}HelperTest.java` | 每个 Helper 一个测试类 |
| 测试方法 | 测试类内 | 每个 Helper 方法 ≥ 2 个测试方法 |
| 测试基类 | `src/test/java/{包路径}/BaseIntegrationTest.java` | 共享 Spring Context |
| 测试配置 | `src/test/resources/application-test.yml` | 测试环境配置 |
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
├── BaseIntegrationTest.java           # 测试基类（共享 Context）
├── TestContextConfig.java             # 测试配置类
├── service/
│   ├── {ModuleA}HelperTest.java      # Helper 测试类
│   ├── {ModuleB}HelperTest.java
│   └── ...
├── controller/
│   ├── {ModuleA}ControllerTest.java  # Controller 测试类
│   └── ...
└── resources/
    └── application-test.yml           # 测试配置文件
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

## 测试基类（共享 Spring Context）

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

## 测试配置类

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(basePackages = "${your.base.package}")
public class TestContextConfig {
    // 测试专用的配置覆盖
}
```

## 测试配置文件

```yaml
# application-test.yml
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

## 测试类示例

```java
public class ProductHelperTest extends BaseIntegrationTest {
    
    // 记录本测试类创建的数据ID，用于清理
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
    @DisplayName("保存商品 - 正常保存并返回ID")
    void testSaveProduct_Success_ReturnWithId() {
        Product product = new Product();
        product.setName(testName("iPhone"));
        product.setPrice(new BigDecimal("5999.00"));
        
        ResponseData<Product> result = ProductHelper.saveProduct(product);
        
        assertTrue(result.isSuccess());
        assertNotNull(result.getData().getId());
        createdProductIds.add(result.getData().getId());
    }
    
    @Test
    @DisplayName("查询商品 - 存在则返回，不存在返回warn")
    void testGetProduct_Exists_ReturnEntity_NotExists_ReturnWarn() {
        Product product = new Product();
        product.setName(testName("MacBook"));
        ResponseData<Product> saved = ProductHelper.saveProduct(product);
        Long productId = saved.getData().getId();
        createdProductIds.add(productId);
        
        ResponseData<Product> found = ProductHelper.getProduct(productId);
        assertTrue(found.isSuccess());
        
        ResponseData<Product> notFound = ProductHelper.getProduct(99999999L);
        assertFalse(notFound.isSuccess());
    }
}
```

## 设计阶段 vs 开发阶段测试职责

| 阶段 | 职责 | 测试状态 |
|------|------|---------|
| 设计阶段（210） | 产出测试骨架（方法签名 + 断言结构 + Javadoc） | 全红（方法体返回null） |
| 开发阶段（310） | 实现 Helper 逻辑，测试逐个变绿 | 红 → 绿 |
| 重构阶段（310） | 优化实现，测试保持全绿 | 全绿 |

**设计阶段测试代码特征**：
- 继承 `BaseIntegrationTest`，共享 Spring Context
- 方法签名完整（`@Test` + `@DisplayName` + 访问修饰符 + 参数）
- Javadoc 包含测试意图、准备数据、预期结果
- 方法体使用 `fail("TDD Red: [Tn] Helper 方法尚未实现")` 标记为 Red 状态
- 包含 `cleanTestData()` 方法框架

## Maven 配置

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>3.1.2</version>
    <configuration>
        <reuseForks>true</reuseForks>
        <forkCount>1</forkCount>
        <systemPropertyVariables>
            <spring.profiles.active>test</spring.profiles.active>
        </systemPropertyVariables>
    </configuration>
</plugin>
```

## 验收标准

设计阶段测试骨架的完成标准：

- [ ] 每个 Helper 类有对应 `{Module}HelperTest.java`
- [ ] 每个 Helper 方法有 ≥ 2 个测试方法
- [ ] 所有测试类继承 `BaseIntegrationTest`
- [ ] `TestContextConfig.java` 配置正确
- [ ] `application-test.yml` 配置测试数据库
- [ ] 所有测试类可编译通过
- [ ] `mvn test -pl {模块}` 可运行（全红是预期）
- [ ] 测试方法有 `@DisplayName` 注解
- [ ] 测试方法有 Javadoc 说明测试意图
- [ ] README.md 包含测试策略章节
