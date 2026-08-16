# Testing AI 编程规则
> Last updated: 2026 | Targets: Vitest / Playwright / React Testing Library

## 核心原则

- 测试金字塔：单元测试 > 集成测试 > E2E 测试
- 测试行为而非实现
- 使用 AAA 模式（Arrange-Act-Assert）
- 保持测试独立和可重复
- 测试覆盖率不是唯一指标

## 代码风格

### 测试文件结构
```
src/
├── components/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx      # 单元测试
│   │   └── Button.stories.tsx   # Storybook
├── services/
│   ├── api.ts
│   └── api.test.ts
└── utils/
    ├── helpers.ts
    └── helpers.test.ts

tests/
├── unit/                # 单元测试
├── integration/         # 集成测试
├── e2e/                 # 端到端测试
│   ├── fixtures/        # 测试数据
│   └── pages/           # Page Objects
└── support/             # 测试工具
```

### 命名规范
- 测试文件：`*.test.ts` 或 `*.spec.ts`
- 测试描述：使用 `should` 或动词开头
- 测试函数：`it('should do something')` 或 `test('does something')`

## 最佳实践

### 单元测试

```typescript
// ✅ Jest 单元测试
describe('UserService', () => {
  let userService: UserService;
  let mockUserRepository: jest.Mocked<UserRepository>;

  beforeEach(() => {
    mockUserRepository = {
      findById: jest.fn(),
      findByEmail: jest.fn(),
      create: jest.fn(),
      update: jest.fn(),
      delete: jest.fn(),
    };
    userService = new UserService(mockUserRepository);
  });

  describe('getUserById', () => {
    it('should return user when found', async () => {
      // Arrange
      const userId = '123';
      const expectedUser = { id: userId, name: 'John' };
      mockUserRepository.findById.mockResolvedValue(expectedUser);

      // Act
      const result = await userService.getUserById(userId);

      // Assert
      expect(result).toEqual(expectedUser);
      expect(mockUserRepository.findById).toHaveBeenCalledWith(userId);
    });

    it('should throw NotFoundError when user not found', async () => {
      // Arrange
      const userId = '999';
      mockUserRepository.findById.mockResolvedValue(null);

      // Act & Assert
      await expect(userService.getUserById(userId)).rejects.toThrow(
        NotFoundError
      );
    });
  });
});
```

### React 组件测试

```typescript
// ✅ React Testing Library
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('LoginForm', () => {
  it('should submit form with valid credentials', async () => {
    // Arrange
    const onSubmit = jest.fn();
    const user = userEvent.setup();

    render(<LoginForm onSubmit={onSubmit} />);

    // Act
    await user.type(screen.getByLabelText('Email'), 'user@example.com');
    await user.type(screen.getByLabelText('Password'), 'password123');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    // Assert
    await waitFor(() => {
      expect(onSubmit).toHaveBeenCalledWith({
        email: 'user@example.com',
        password: 'password123',
      });
    });
  });

  it('should show error message for invalid email', async () => {
    // Arrange
    const user = userEvent.setup();
    render(<LoginForm onSubmit={jest.fn()} />);

    // Act
    await user.type(screen.getByLabelText('Email'), 'invalid');
    await user.click(screen.getByRole('button', { name: 'Login' }));

    // Assert
    expect(screen.getByText('Invalid email format')).toBeInTheDocument();
  });
});
```

### 集成测试

```typescript
// ✅ API 集成测试
import request from 'supertest';
import { createApp } from '../src/app';
import { setupTestDatabase, cleanupTestDatabase } from './helpers';

describe('Users API Integration', () => {
  let app: Express;

  beforeAll(async () => {
    await setupTestDatabase();
    app = createApp();
  });

  afterAll(async () => {
    await cleanupTestDatabase();
  });

  beforeEach(async () => {
    await clearUsers();
  });

  describe('POST /api/users', () => {
    it('should create user and return 201', async () => {
      const response = await request(app)
        .post('/api/users')
        .send({
          username: 'testuser',
          email: 'test@example.com',
          password: 'Password123',
        })
        .expect(201);

      expect(response.body.data).toMatchObject({
        username: 'testuser',
        email: 'test@example.com',
      });
      expect(response.body.data.id).toBeDefined();
    });

    it('should return 409 for duplicate email', async () => {
      // Create first user
      await request(app)
        .post('/api/users')
        .send({
          username: 'user1',
          email: 'test@example.com',
          password: 'Password123',
        });

      // Try to create with same email
      const response = await request(app)
        .post('/api/users')
        .send({
          username: 'user2',
          email: 'test@example.com',
          password: 'Password456',
        })
        .expect(409);

      expect(response.body.error.code).toBe('DUPLICATE_EMAIL');
    });
  });
});
```

### E2E 测试

```typescript
// ✅ Playwright E2E 测试
import { test, expect } from '@playwright/test';

test.describe('User Registration Flow', () => {
  test('should register new user successfully', async ({ page }) => {
    // Navigate to registration page
    await page.goto('/register');

    // Fill in registration form
    await page.getByLabel('Username').fill('newuser');
    await page.getByLabel('Email').fill('new@example.com');
    await page.getByLabel('Password').fill('SecurePassword123');
    await page.getByLabel('Confirm Password').fill('SecurePassword123');

    // Submit form
    await page.getByRole('button', { name: 'Register' }).click();

    // Verify success
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome, newuser!')).toBeVisible();
  });

  test('should show error for existing email', async ({ page }) => {
    await page.goto('/register');

    await page.getByLabel('Username').fill('existinguser');
    await page.getByLabel('Email').fill('existing@example.com');
    await page.getByLabel('Password').fill('SecurePassword123');
    await page.getByLabel('Confirm Password').fill('SecurePassword123');

    await page.getByRole('button', { name: 'Register' }).click();

    await expect(page.getByText('Email already exists')).toBeVisible();
  });
});

// ✅ Page Object Model
class RegisterPage {
  constructor(private page: Page) {}

  async goto() {
    await this.page.goto('/register');
  }

  async fillForm(data: {
    username: string;
    email: string;
    password: string;
    confirmPassword: string;
  }) {
    await this.page.getByLabel('Username').fill(data.username);
    await this.page.getByLabel('Email').fill(data.email);
    await this.page.getByLabel('Password').fill(data.password);
    await this.page.getByLabel('Confirm Password').fill(data.confirmPassword);
  }

  async submit() {
    await this.page.getByRole('button', { name: 'Register' }).click();
  }

  async register(data: {
    username: string;
    email: string;
    password: string;
  }) {
    await this.fillForm({
      ...data,
      confirmPassword: data.password,
    });
    await this.submit();
  }
}
```

### 测试数据工厂

```typescript
// ✅ 使用工厂函数创建测试数据
import { faker } from '@faker-js/faker';

interface UserFactoryOptions {
  id?: string;
  username?: string;
  email?: string;
  role?: 'user' | 'admin';
}

function createUser(options: UserFactoryOptions = {}): User {
  return {
    id: options.id || faker.string.uuid(),
    username: options.username || faker.internet.username(),
    email: options.email || faker.internet.email(),
    role: options.role || 'user',
    createdAt: new Date(),
    updatedAt: new Date(),
  };
}

function createUsers(count: number, options: UserFactoryOptions = {}): User[] {
  return Array.from({ length: count }, () => createUser(options));
}

// 使用
const user = createUser({ role: 'admin' });
const users = createUsers(10);
```

### Mock 和 Stub

```typescript
// ✅ Mock 外部服务
jest.mock('../src/services/email', () => ({
  sendEmail: jest.fn().mockResolvedValue({ success: true }),
}));

// ✅ Mock API 响应
import { rest } from 'msw';
import { setupServer } from 'msw/node';

const server = setupServer(
  rest.get('/api/users', (req, res, ctx) => {
    return res(
      ctx.json({
        data: [{ id: '1', username: 'testuser' }],
        pagination: { page: 1, size: 20, total: 1 },
      })
    );
  }),

  rest.post('/api/users', (req, res, ctx) => {
    return res(
      ctx.status(201),
      ctx.json({
        data: { id: '2', username: 'newuser' },
      })
    );
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
```

### 快照测试

```typescript
// ✅ 组件快照测试
describe('UserCard', () => {
  it('should match snapshot', () => {
    const user = createUser();
    const { container } = render(<UserCard user={user} />);
    expect(container).toMatchSnapshot();
  });

  it('should match snapshot with admin badge', () => {
    const user = createUser({ role: 'admin' });
    const { container } = render(<UserCard user={user} />);
    expect(container).toMatchSnapshot();
  });
});
```

### 测试覆盖率

```json
// jest.config.js
module.exports = {
  collectCoverage: true,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  collectCoverageFrom: [
    'src/**/*.{ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/index.ts',
    '!src/**/*.stories.tsx',
  ],
};
```

## 测试策略

### 测试金字塔
```
         /\
        /  \        E2E Tests (10%)
       /    \       - 关键用户流程
      /------\
     /        \     Integration Tests (20%)
    /          \    - API 端点
   /            \   - 数据库操作
  /--------------\
 /                \ Unit Tests (70%)
/                  \ - 业务逻辑
--------------------  - 工具函数
                      - 组件
```

### 测试分类

```typescript
// ✅ 使用 describe 分组
describe('UserService', () => {
  describe('getUserById', () => {
    it('should return user when found', async () => {});
    it('should throw when user not found', async () => {});
  });

  describe('createUser', () => {
    it('should create user successfully', async () => {});
    it('should throw for duplicate email', async () => {});
  });
});

// ✅ 使用标签分类
test.skip('legacy feature', () => {});
test.todo('implement this later');
test.only('debug this test', () => {});
```

## 性能测试

```typescript
// ✅ 性能基准测试
import { performance } from 'perf_hooks';

describe('Performance', () => {
  it('should process 1000 items in under 100ms', () => {
    const items = createItems(1000);

    const start = performance.now();
    const result = processItems(items);
    const end = performance.now();

    expect(end - start).toBeLessThan(100);
    expect(result).toHaveLength(1000);
  });
});
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 测试实现细节
it('should call internal method', () => {
  const spy = jest.spyOn(service, 'internalMethod');
  service.doSomething();
  expect(spy).toHaveBeenCalled(); // 测试实现而非行为
});

// ❌ 测试依赖外部服务
it('should send email', async () => {
  await service.sendWelcomeEmail(user); // 实际发送邮件
});

// ❌ 不稳定的测试
it('should work', async () => {
  const result = await fetchData(); // 可能因网络问题失败
  expect(result).toBeDefined();
});

// ❌ 测试之间有依赖
let userId: string;
it('should create user', async () => {
  userId = await createUser(); // 影响下一个测试
});
it('should get user', async () => {
  await getUser(userId); // 依赖上一个测试
});
```

### ✅ 推荐
```typescript
// ✅ 测试行为
it('should return user data', async () => {
  const user = await service.getUserById('123');
  expect(user).toMatchObject({
    id: '123',
    name: 'John',
  });
});

// ✅ Mock 外部依赖
it('should send welcome email', async () => {
  const sendEmail = jest.spyOn(emailService, 'sendEmail');
  await service.register(userData);
  expect(sendEmail).toHaveBeenCalledWith(
    userData.email,
    'Welcome!'
  );
});

// ✅ 使用测试数据库
beforeAll(async () => {
  await setupTestDatabase();
});

afterEach(async () => {
  await clearTestData();
});

// ✅ 每个测试独立
it('should create and get user', async () => {
  const userId = await createUser(userData);
  const user = await getUser(userId);
  expect(user).toBeDefined();
});
```

## 依赖推荐

- **测试框架**: Jest / Vitest / Mocha
- **组件测试**: React Testing Library / Vue Test Utils
- **E2E 测试**: Playwright / Cypress
- **API 测试**: Supertest / HTTPX
- **Mock**: Jest Mock / MSW / sinon
- **测试数据**: Faker.js / fishery
- **覆盖率**: c8 / Istanbul

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 测试框架：
- E2E 工具：
- 覆盖率目标：
- CI 集成：
```
