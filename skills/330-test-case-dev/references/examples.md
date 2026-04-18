# Playwright 脚本示例

## 1. API 测试脚本

```typescript
import { test, expect } from '@playwright/test';

test.describe('用户管理API', () => {
  test('创建用户 - 正常场景', async ({ request }) => {
    const response = await request.post('/api/v1/users', {
      data: {
        username: 'testuser',
        email: 'test@example.com',
        password: 'Test@123',
      },
    });
    expect(response.status()).toBe(201);
    const body = await response.json();
    expect(body.code).toBe(200);
    expect(body.data.id).toBeDefined();
  });

  test('创建用户 - 缺少必填参数', async ({ request }) => {
    const response = await request.post('/api/v1/users', {
      data: { username: 'testuser' },
    });
    expect(response.status()).toBe(400);
    const body = await response.json();
    expect(body.code).toBe(400);
  });

  test('创建用户 - 无权限访问', async ({ request }) => {
    const response = await request.post('/api/v1/users', {
      headers: { Authorization: 'Bearer guest-token' },
      data: { username: 'testuser', email: 't@e.com', password: 'Test@123' },
    });
    expect(response.status()).toBe(403);
  });
});
```

## 2. E2E 单终端测试脚本

```typescript
import { test, expect } from '@playwright/test';
import { LoginPage } from '../shared/page-objects/login-page';

test.describe('订单管理 - admin-web', () => {
  test.beforeEach(async ({ page }) => {
    const loginPage = new LoginPage(page);
    await loginPage.goto();
    await loginPage.login('admin', 'password');
  });

  test('查看订单列表', async ({ page }) => {
    await page.click('[data-testid="nav-orders"]');
    await expect(page.locator('[data-testid="order-list"]')).toBeVisible();
    await expect(page.locator('[data-testid="order-row"]').first()).toBeVisible();
  });

  test('审核订单', async ({ page }) => {
    await page.click('[data-testid="nav-orders"]');
    await page.click('[data-testid="order-row-001"]');
    await page.click('[data-testid="btn-approve"]');
    await expect(page.locator('[data-testid="order-status"]')).toHaveText('已审核');
  });
});
```

## 3. E2E 跨终端测试脚本

```typescript
import { test, expect } from '@playwright/test';
import { MultiTerminalTest } from '../utils/multi-context';

test.describe('跨终端订单流程', () => {
  let multi: MultiTerminalTest;

  test.beforeAll(async ({ browser }) => {
    multi = new MultiTerminalTest(browser);
  });

  test.afterAll(async () => {
    await multi.closeAll();
  });

  test('用户下单→管理端审核→商家发货→用户确认', async () => {
    const guestPage = await multi.openTerminal('guest', '/login');
    const adminPage = await multi.openTerminal('admin', '/login');

    await guestPage.fill('[data-testid="input-username"]', 'guest');
    await guestPage.fill('[data-testid="input-password"]', 'password');
    await guestPage.click('[data-testid="btn-login"]');

    await guestPage.click('[data-testid="btn-submit-order"]');
    await expect(guestPage.locator('[data-testid="order-status"]')).toHaveText('待审核');

    await adminPage.fill('[data-testid="input-username"]', 'admin');
    await adminPage.fill('[data-testid="input-password"]', 'password');
    await adminPage.click('[data-testid="btn-login"]');

    await adminPage.click('[data-testid="nav-orders"]');
    await adminPage.click('[data-testid="btn-approve"]');
    await expect(adminPage.locator('[data-testid="order-status"]')).toHaveText('待发货');

    await expect(guestPage.locator('[data-testid="order-status"]')).toHaveText('待发货');
  });
});
```

## 4. 多终端 BrowserContext 工具

```typescript
import { Browser, BrowserContext, Page } from '@playwright/test';

type TerminalKey = 'guest' | 'admin' | 'saas' | 'mch';

const TERMINAL_URLS: Record<TerminalKey, string> = {
  guest: process.env.GUEST_BASE_URL || 'http://localhost:3000',
  admin: process.env.ADMIN_BASE_URL || 'http://localhost:3001',
  saas: process.env.SAAS_BASE_URL || 'http://localhost:3002',
  mch: process.env.MCH_BASE_URL || 'http://localhost:3003',
};

export class MultiTerminalTest {
  private contexts: Map<TerminalKey, { context: BrowserContext; page: Page }> = new Map();

  constructor(private browser: Browser) {}

  async openTerminal(terminal: TerminalKey, path: string = '/'): Promise<Page> {
    const context = await this.browser.newContext();
    const page = await context.newPage();
    await page.goto(`${TERMINAL_URLS[terminal]}${path}`);
    this.contexts.set(terminal, { context, page });
    return page;
  }

  getPage(terminal: TerminalKey): Page {
    return this.contexts.get(terminal)!.page;
  }

  async closeAll() {
    for (const { context } of this.contexts.values()) {
      await context.close();
    }
    this.contexts.clear();
  }
}
```

## 5. 共享工具

### 登录/Token管理

```typescript
import { APIRequestContext } from '@playwright/test';

export async function getAuthToken(
  request: APIRequestContext,
  role: string
): Promise<string> {
  const passwords: Record<string, string> = {
    admin: 'admin-password',
    guest: 'guest-password',
  };
  const response = await request.post('/api/auth/login', {
    data: { username: role, password: passwords[role] },
  });
  const body = await response.json();
  return body.data.token;
}
```

### 测试数据生成

```typescript
import { faker } from '@faker-js/faker';

export function generateUser() {
  return {
    username: faker.internet.userName(),
    email: faker.internet.email(),
    phone: faker.phone.number(),
    password: 'Test@' + faker.string.alphanumeric(8),
  };
}
```

## 6. Playwright 配置

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: '.',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [['html', { open: 'never' }], ['json', { outputFile: 'results.json' }]],
  use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    {
      name: 'api',
      testDir: './api',
      use: { baseURL: process.env.API_BASE_URL || 'http://localhost:8080' },
    },
    {
      name: 'e2e',
      testDir: './e2e',
      use: {
        baseURL: process.env.WEB_BASE_URL || 'http://localhost:3000',
        ...devices['Desktop Chrome'],
      },
    },
  ],
});
```
