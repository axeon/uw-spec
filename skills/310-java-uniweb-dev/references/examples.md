# Java SaaS 开发示例（UniWeb 技术栈）

> 基于 UniWeb + SaaS 技术栈的 TDD 开发示例。禁用 Lombok，使用 DaoManager/FusionCache/ResponseData。

## TDD 开发流程

```
Red（220设计阶段）→ 编写测试骨架（fail）
  ↓
Green（310开发阶段）→ 实现 Helper 逻辑（测试变绿）
  ↓
Refactor（310开发阶段）→ 重构优化（测试保持全绿）
```

## Helper 实现 + TDD 示例

### 1. 设计阶段产出（220，Red）

```java
@Component
public class ProductHelper {

    private final DaoManager daoManager;

    public ProductHelper(DaoManager daoManager) {
        this.daoManager = daoManager;
    }

    public ResponseData<Product> getProduct(Long id) {
        return ResponseData.success(null);
    }

    public ResponseData<Product> saveProduct(Product entity) {
        return ResponseData.success(null);
    }
}
```

### 2. 设计阶段测试骨架（220，Red）

```java
@ExtendWith(MockitoExtension.class)
class ProductHelperTest {

    @Mock
    private DaoManager daoManager;

    @InjectMocks
    private ProductHelper productHelper;

    @Test
    @DisplayName("查询商品详情 - 正常返回")
    void testGetProduct_Found_ReturnEntity() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    @Test
    @DisplayName("查询商品详情 - ID不存在返回warn")
    void testGetProduct_NotFound_ReturnWarn() {
        fail("TDD Red: Helper 方法尚未实现");
    }

    @Test
    @DisplayName("新增商品 - 正常创建")
    void testSaveProduct_Normal_ReturnCreated() {
        fail("TDD Red: Helper 方法尚未实现");
    }
}
```

### 3. 开发阶段实现 Helper（310，Green）

```java
@Component
public class ProductHelper {

    private static final Logger log = LoggerFactory.getLogger(ProductHelper.class);
    private final DaoManager daoManager;
    private static final int CACHE_MAX_NUM = 500;
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    public ProductHelper(DaoManager daoManager) {
        this.daoManager = daoManager;
    }

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

### 4. 开发阶段实现测试（310，Green）

```java
@ExtendWith(MockitoExtension.class)
class ProductHelperTest {

    @Mock
    private DaoManager daoManager;

    @InjectMocks
    private ProductHelper productHelper;

    @Test
    @DisplayName("查询商品详情 - 正常返回")
    void testGetProduct_Found_ReturnEntity() {
        Product product = new Product();
        product.setId(1L);
        product.setName("测试商品");
        ResponseData<Product> mockResponse = ResponseData.success(product);

        when(daoManager.load(Product.class, 1L)).thenReturn(mockResponse);

        ResponseData<Product> result = productHelper.getProduct(1L);

        assertTrue(result.isSuccess());
        assertNotNull(result.getData());
        assertEquals(1L, result.getData().getId());
    }

    @Test
    @DisplayName("查询商品详情 - ID不存在返回warn")
    void testGetProduct_NotFound_ReturnWarn() {
        ResponseData<Product> mockResponse = ResponseData.warn("商品不存在");
        when(daoManager.load(Product.class, 999L)).thenReturn(mockResponse);

        ResponseData<Product> result = productHelper.getProduct(999L);

        assertFalse(result.isSuccess());
    }

    @Test
    @DisplayName("新增商品 - 正常创建")
    void testSaveProduct_Normal_ReturnCreated() {
        Product entity = new Product();
        entity.setName("新商品");

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

## FusionCache 使用示例

```java
public ResponseData<Product> getProductCached(Long id) {
    Optional<Product> cached = FusionCache.getIfPresent(Product.class, id);
    if (cached.isPresent()) {
        return ResponseData.success(cached.get());
    }
    ResponseData<Product> result = daoManager.load(Product.class, id);
    if (result.isSuccess() && result.getData() != null) {
        FusionCache.put(Product.class, id, result.getData());
    }
    return result;
}

public ResponseData<Product> updateProduct(Product entity) {
    ResponseData<Product> result = daoManager.update(entity);
    if (result.isSuccess()) {
        FusionCache.invalidate(Product.class, entity.getId());
    }
    return result;
}
```

## GlobalLocker 分布式锁示例

```java
public ResponseData<Integer> processOrder(Long orderId) {
    long stamp = GlobalLocker.tryLock(Order.class, orderId, 30000L);
    if (stamp <= 0) {
        return ResponseData.warn("操作正在处理中，请稍后重试");
    }
    try {
        ResponseData<Order> loadResult = daoManager.load(Order.class, orderId);
        if (!loadResult.isSuccess()) {
            return ResponseData.warn("订单不存在");
        }
        Order order = loadResult.getData();
        order.setOrderStatus(OrderStatus.PROCESSED);
        return daoManager.update(order);
    } finally {
        GlobalLocker.unlock(Order.class, orderId, stamp);
    }
}
```

## AuthQueryParam 多租户查询示例

```java
public ResponseData<DataList<Product>> listProduct(AuthQueryParam param) {
    return daoManager.list(Product.class, param);
}

public ResponseData<DataList<Product>> listMyProducts(AuthQueryParam param) {
    param.bindMchId();
    return daoManager.list(Product.class, param);
}
```

## Controller 调用示例

```java
@RestController
@RequestMapping("/saas/product")
@Tag(name = "Product", description = "商品管理")
@MscPermDeclare(name = "商品管理", user = UserType.SAAS, log = ActionLog.NONE)
public class ProductController {

    private final ProductHelper productHelper;

    public ProductController(ProductHelper productHelper) {
        this.productHelper = productHelper;
    }

    @Operation(summary = "查询商品详情")
    @GetMapping("/detail")
    @MscPermDeclare(name = "商品详情", log = ActionLog.REQUEST)
    public ResponseData<Product> detail(@RequestParam Long id) {
        return productHelper.getProduct(id);
    }

    @Operation(summary = "新增商品")
    @PostMapping("/save")
    @MscPermDeclare(name = "新增商品", log = ActionLog.ALL)
    public ResponseData<Product> save(@RequestBody @Valid Product entity) {
        return productHelper.saveProduct(entity);
    }
}
```
