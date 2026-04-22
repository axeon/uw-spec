# Java SaaS 开发示例（UniWeb 技术栈）

> 基于 UniWeb + SaaS 技术栈的 TDD 开发示例。禁用 Lombok，使用 DaoManager/FusionCache/ResponseData。

## TDD 开发流程

```
Red（210设计阶段）→ 编写测试骨架（fail）
  ↓
Green（310开发阶段）→ 实现 Helper 逻辑（测试变绿）
  ↓
Refactor（310开发阶段）→ 重构优化（测试保持全绿）
```

## Helper 实现 + TDD 示例

### 1. 设计阶段产出（210，Red）

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
}
```

### 2. 设计阶段测试骨架（210，Red）

```java
@ExtendWith(MockitoExtension.class)
class ProductHelperTest {

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
}
```

### 3. 开发阶段实现 Helper（310，Green）

```java
public class ProductHelper {

    private static final Logger log = LoggerFactory.getLogger(ProductHelper.class);
    private static final DaoManager daoManager = DaoManager.getInstance();
    private static final int CACHE_MAX_NUM = 500;
    private static final long CACHE_EXPIRE_MILLIS = 1800_000L;

    public static ResponseData<Product> getProduct(Long id) {
        return daoManager.load(Product.class, id);
    }

    public static ResponseData<Product> saveProduct(Product entity) {
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

    @Test
    @DisplayName("查询商品详情 - 正常返回")
    void testGetProduct_Found_ReturnEntity() {
        Product product = new Product();
        product.setId(1L);
        product.setName("测试商品");
        ResponseData<Product> mockResponse = ResponseData.success(product);

        try (MockedStatic<DaoManager> daoMock = mockStatic(DaoManager.class)) {
            DaoManager mockInstance = mock(DaoManager.class);
            daoMock.when(DaoManager::getInstance).thenReturn(mockInstance);
            when(mockInstance.load(Product.class, 1L)).thenReturn(mockResponse);

            ResponseData<Product> result = ProductHelper.getProduct(1L);

            assertTrue(result.isSuccess());
            assertNotNull(result.getData());
            assertEquals(1L, result.getData().getId());
        }
    }

    @Test
    @DisplayName("查询商品详情 - ID不存在返回warn")
    void testGetProduct_NotFound_ReturnWarn() {
        ResponseData<Product> mockResponse = ResponseData.warn("商品不存在");

        try (MockedStatic<DaoManager> daoMock = mockStatic(DaoManager.class)) {
            DaoManager mockInstance = mock(DaoManager.class);
            daoMock.when(DaoManager::getInstance).thenReturn(mockInstance);
            when(mockInstance.load(Product.class, 999L)).thenReturn(mockResponse);

            ResponseData<Product> result = ProductHelper.getProduct(999L);

            assertFalse(result.isSuccess());
        }
    }

    @Test
    @DisplayName("新增商品 - 正常创建")
    void testSaveProduct_Normal_ReturnCreated() {
        Product entity = new Product();
        entity.setName("新商品");

        try (MockedStatic<DaoManager> daoMock = mockStatic(DaoManager.class);
             MockedStatic<AuthServiceHelper> authMock = mockStatic(AuthServiceHelper.class)) {
            DaoManager mockInstance = mock(DaoManager.class);
            daoMock.when(DaoManager::getInstance).thenReturn(mockInstance);
            authMock.when(AuthServiceHelper::getSaasId).thenReturn(1001L);
            when(mockInstance.getSequenceId(Product.class)).thenReturn(1L);
            when(mockInstance.save(any(Product.class))).thenReturn(ResponseData.success(entity));

            ResponseData<Product> result = ProductHelper.saveProduct(entity);

            assertTrue(result.isSuccess());
        }
    }
}
```

## FusionCache 使用示例

```java
public static ResponseData<Product> getProductCached(Long id) {
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

public static ResponseData<Product> updateProduct(Product entity) {
    ResponseData<Product> result = daoManager.update(entity);
    if (result.isSuccess()) {
        FusionCache.invalidate(Product.class, entity.getId());
    }
    return result;
}
```

## GlobalLocker 分布式锁示例

```java
public static ResponseData<Integer> processOrder(Long orderId) {
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
public static ResponseData<DataList<Product>> listProduct(AuthQueryParam param) {
    return daoManager.list(Product.class, param);
}

public static ResponseData<DataList<Product>> listMyProducts(AuthQueryParam param) {
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

    @Operation(summary = "查询商品详情")
    @GetMapping("/detail")
    @MscPermDeclare(name = "商品详情", log = ActionLog.REQUEST)
    public ResponseData<Product> detail(@RequestParam Long id) {
        return ProductHelper.getProduct(id);
    }

    @Operation(summary = "新增商品")
    @PostMapping("/save")
    @MscPermDeclare(name = "新增商品", log = ActionLog.ALL)
    public ResponseData<Product> save(@RequestBody @Valid Product entity) {
        return ProductHelper.saveProduct(entity);
    }
}
```
