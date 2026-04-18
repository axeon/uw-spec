# TDD驱动开发指南

> 跨技能共用的TDD红绿重构说明和通用示例。被 310-java-uniweb-dev、320-web-vue-dev、320-md-uniapp-dev、620-feature-java-uniweb-dev、630-feature-web-vue-dev、631-feature-md-uniapp-dev、720-bugfix-java-uniweb、730-bugfix-web-vue、731-bugfix-md-uniapp 等技能引用。

## TDD开发原则

1. **测试先行**：先写测试，再写实现
2. **红-绿-重构**：测试失败 -> 实现通过 -> 重构优化
3. **验收标准**：每个需求都要有可验证的验收标准
4. **持续测试**：测试贯穿整个开发周期

## 红绿重构循环

```
Red -> 编写测试（失败）
  |
Green -> 最少代码实现（通过）
  |
Refactor -> 重构优化（保持通过）
```

## 后端TDD示例（Java）

### Red - 编写测试

```java
@Test
public void testQueryUserById_WithNullResult() {
    Long nonExistentUserId = 99999L;
    assertThrows(UserNotFoundException.class, () -> {
        userService.queryById(nonExistentUserId);
    });
}
```

### Green - 最少实现

```java
public User queryById(Long id) {
    User user = userMapper.selectById(id);
    if (user == null) {
        throw new UserNotFoundException("用户不存在: " + id);
    }
    return user;
}
```

### Refactor - 重构优化

```java
public User queryById(Long id) {
    return Optional.ofNullable(userMapper.selectById(id))
        .orElseThrow(() -> new UserNotFoundException("用户不存在: " + id));
}
```

## 前端TDD示例（TypeScript）

### Red - 编写测试

```typescript
describe('calculateOrderAmount', () => {
  it('应该正确计算订单金额', () => {
    const items = [{ price: 100, quantity: 2 }]
    const result = calculateOrderAmount(items, 10)
    expect(result.totalAmount).toBe(210)
  })
})
```

### Green - 实现功能

```typescript
export function calculateOrderAmount(items: OrderItem[], shippingFee: number) {
  const goodsAmount = items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  const finalShippingFee = goodsAmount >= 199 ? 0 : shippingFee
  return { goodsAmount, shippingFee: finalShippingFee, totalAmount: goodsAmount + finalShippingFee }
}
```

## Git提交规范

### Bug修复提交

```bash
git commit -m "fix(BUG-{编号}): 修复描述

- 修改内容1
- 修改内容2
- 添加单元测试

Closes BUG-{编号}"
```

### 功能添加提交

```bash
git commit -m "feat({模块}): 功能描述

- 新增内容1
- 新增内容2
- 添加单元测试"
```
