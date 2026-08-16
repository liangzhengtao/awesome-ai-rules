# TypeScript AI 编程规则
> Last updated: 2026 | Targets: TypeScript 5.x / strict mode

## 核心原则

- 使用 TypeScript strict 模式
- 优先使用类型推断，减少显式类型注解
- 使用 discriminated unions 处理状态
- 利用模板字面量类型
- 使用 satisfies 操作符

## 代码风格

### 命名规范
- 类型/接口：PascalCase：`UserProfile`, `ApiResponse`
- 泛型参数：单大写或描述性 PascalCase：`T`, `Data`, `Error`
- 枚举：PascalCase，成员 PascalCase：`enum Status { Active, Inactive }`
- 常量：camelCase 或 UPPER_SNAKE_CASE
- 工具类型：PascalCase：`DeepPartial`, `RequiredKeys`

### 文件结构
```
src/
├── types/                # 类型定义
│   ├── index.ts
│   ├── api.ts
│   └── models.ts
├── utils/                # 工具函数
│   ├── type-guards.ts
│   └── helpers.ts
├── services/             # 服务层
│   └── api-client.ts
└── index.ts
```

## 最佳实践

### 类型定义

```typescript
// ✅ 使用 interface 定义对象形状
interface User {
  id: string;
  username: string;
  email: string;
  profile: UserProfile;
  createdAt: Date;
  updatedAt: Date;
}

// ✅ 使用 type 定义联合类型、交叉类型、工具类型
type UserStatus = 'active' | 'inactive' | 'suspended';

type UserWithPosts = User & {
  posts: Post[];
};

type PartialUser = Partial<User>;
type RequiredUser = Required<User>;

// ✅ 使用 discriminated unions
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function handleResult(result: Result<User>) {
  if (result.success) {
    console.log(result.data); // TypeScript 知道这里有 data
  } else {
    console.error(result.error); // TypeScript 知道这里有 error
  }
}

// ✅ 使用 const assertions
const ROUTES = {
  HOME: '/',
  USERS: '/users',
  USER_DETAIL: '/users/:id',
} as const;

type Route = (typeof ROUTES)[keyof typeof ROUTES];
// type Route = "/" | "/users" | "/users/:id"
```

### 泛型约束

```typescript
// ✅ 使用 extends 约束泛型
interface HasId {
  id: string;
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// ✅ 使用 keyof 约束
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

// ✅ 使用条件类型
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false

// ✅ 使用 infer 提取类型
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type C = UnwrapPromise<Promise<string>>; // string
type D = UnwrapPromise<number>; // number
```

### 工具类型

```typescript
// ✅ DeepPartial
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

// ✅ DeepReadonly
type DeepReadonly<T> = {
  readonly [P in keyof T]: T[P] extends object ? DeepReadonly<T[P]> : T[P];
};

// ✅ Pick with type safety
type PickByType<T, U> = {
  [P in keyof T as T[P] extends U ? P : never]: T[P];
};

interface Example {
  id: number;
  name: string;
  active: boolean;
}

type StringProps = PickByType<Example, string>; // { name: string }
type NumberProps = PickByType<Example, number>; // { id: number }

// ✅ 使用 Record
type UserRoles = Record<string, string[]>;

const roles: UserRoles = {
  admin: ['read', 'write', 'delete'],
  user: ['read'],
};
```

### 类型守卫

```typescript
// ✅ 自定义类型守卫
function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'username' in obj &&
    'email' in obj
  );
}

// ✅ 使用
function processInput(input: unknown) {
  if (isUser(input)) {
    console.log(input.username); // TypeScript 知道这是 User
  }
}

// ✅ 断言函数
function assertDefined<T>(value: T | null | undefined, name: string): asserts value is T {
  if (value === null || value === undefined) {
    throw new Error(`Expected ${name} to be defined`);
  }
}

// ✅ 使用
function processUser(user: User | null) {
  assertDefined(user, 'user');
  console.log(user.username); // TypeScript 知道 user 不是 null
}
```

### 模板字面量类型

```typescript
// ✅ 事件名称
type EventName = `on${Capitalize<'click' | 'hover' | 'focus'>}`;
// type EventName = "onClick" | "onHover" | "onFocus"

// ✅ CSS 单位
type CSSUnit = 'px' | 'rem' | 'em' | '%' | 'vh' | 'vw';
type CSSValue = `${number}${CSSUnit}`;

const width: CSSValue = '100px';

// ✅ 路径参数
type ExtractPathParams<T extends string> =
  T extends `${string}:${infer Param}/${infer Rest}`
    ? Param | ExtractPathParams<Rest>
    : T extends `${string}:${infer Param}`
    ? Param
    : never;

type Params = ExtractPathParams<'/users/:id/posts/:postId'>;
// type Params = "id" | "postId"
```

### satisfies 操作符

```typescript
// ✅ 使用 satisfies 保持类型推断
interface Config {
  apiUrl: string;
  timeout: number;
  features: {
    darkMode: boolean;
    notifications: boolean;
  };
}

const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  features: {
    darkMode: true,
    notifications: false,
  },
} satisfies Config;

// 仍然可以访问具体类型
config.features.darkMode; // boolean (不是 boolean | undefined)
```

### 装饰器

```typescript
// ✅ 方法装饰器
function Log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} returned:`, result);
    return result;
  };

  return descriptor;
}

// ✅ 类装饰器
function Sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@Sealed
class Example {
  @Log
  method(arg: string) {
    return arg;
  }
}
```

## 测试类型

```typescript
// ✅ 测试类型
import { expectTypeOf } from 'expect-type';

describe('Type tests', () => {
  it('should have correct types', () => {
    expectTypeOf(findById<User>).returns.toEqualTypeOf<User | undefined>();
    expectTypeOf(getProperty<User, 'id'>).returns.toBeString();
  });
});
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 使用 any
function process(data: any) {
  return data.whatever; // 没有类型检查
}

// ❌ 类型断言滥用
const user = {} as User; // 可能不安全

// ❌ 非空断言滥用
const element = document.getElementById('app')!; // 可能为 null

// ❌ 忽略 TypeScript 错误
// @ts-ignore
const value: string = 123;
```

### ✅ 推荐
```typescript
// ✅ 使用 unknown
function process(data: unknown) {
  if (isUser(data)) {
    console.log(data.username);
  }
}

// ✅ 使用类型守卫
function processInput(input: unknown) {
  if (isUser(input)) {
    // 安全使用 input
  }
}

// ✅ 正确处理可能的 null
const element = document.getElementById('app');
if (element) {
  element.innerHTML = 'Hello';
}

// ✅ 修复而不是忽略错误
const value: string = String(123);
```

## 依赖推荐

- **构建工具**: tsup / esbuild / tsc
- **运行时**: Node.js / Deno / Bun
- **测试**: Vitest / Jest + ts-jest
- **代码规范**: ESLint + typescript-eslint
- **格式化**: Prettier

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- TypeScript 版本：
- 目标环境：Node.js / Browser / Deno
- 模块系统：ESM / CommonJS
```
