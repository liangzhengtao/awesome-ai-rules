# GitHub Copilot AI 编程规则
> Last updated: 2026 | Targets: GitHub Copilot (multi-model)

## 核心原则

- 使用 `.github/copilot-instructions.md` 配置项目规则
- 善用代码补全和内联建议
- 利用 Chat 面板进行复杂任务
- 使用 `/` 命令快速操作
- 保持代码简洁清晰

## 配置文件

### .github/copilot-instructions.md 示例

```markdown
# GitHub Copilot 项目规则

## 项目概述
这是一个 [项目类型] 项目，使用 [技术栈]。

## 技术栈
- 框架：[框架名称]
- 语言：[编程语言]
- 数据库：[数据库]
- 部署：[部署方式]

## 代码规范

### 命名规范
- 组件：PascalCase
- 函数/变量：camelCase
- 常量：UPPER_SNAKE_CASE
- 文件：kebab-case

### 文件结构
```
src/
├── components/     # 通用组件
├── pages/          # 页面组件
├── hooks/          # 自定义 Hooks
├── services/       # API 服务
├── utils/          # 工具函数
└── types/          # TypeScript 类型
```

### 代码风格
- 使用 TypeScript strict mode
- 使用 ESLint + Prettier
- 优先使用 const
- 使用 early return 减少嵌套
- 函数不超过 50 行
- 文件不超过 300 行

## 特殊指令

### 代码生成
- 生成代码时添加 JSDoc 注释
- 为函数添加参数验证
- 使用 TypeScript 类型定义
- 处理所有可能的错误

### 测试
- 为新功能编写单元测试
- 使用 AAA 模式（Arrange-Act-Assert）
- 测试边界情况和错误情况
- 测试覆盖率不低于 80%

### 安全
- 验证所有用户输入
- 使用参数化查询防止 SQL 注入
- 对敏感数据进行加密
- 实施适当的认证和授权

### 性能
- 避免 N+1 查询
- 使用缓存减少重复计算
- 懒加载非关键资源
- 优化图片和静态资源
```

## 最佳实践

### 代码补全

```typescript
// ✅ 写清晰的注释让 Copilot 理解意图

// 计算两个日期之间的天数差
function daysBetween(date1: Date, date2: Date): number {
  // Copilot 会自动补全实现
}

// 验证邮箱格式
function isValidEmail(email: string): boolean {
  // Copilot 会提供正则表达式
}

// 使用 JSDoc 让 Copilot 生成更好的代码

/**
 * 发送邮件通知
 * @param to - 收件人邮箱
 * @param subject - 邮件主题
 * @param body - 邮件内容
 * @returns 是否发送成功
 */
async function sendEmail(to: string, subject: string, body: string): Promise<boolean> {
  // Copilot 会根据 JSDoc 生成实现
}
```

### Chat 面板使用

```markdown
# 使用 / 命令

/explain    # 解释代码
/fix        # 修复问题
/test       # 生成测试
/doc        # 生成文档
/optimize   # 优化代码

# 示例
/explain 这段代码的作用是什么？

/fix 这个函数有 bug，请修复

/test 请为这个函数生成测试

/doc 请为这个模块生成文档
```

### 内联编辑

```typescript
// 选中代码后按 Cmd+I 打开内联编辑

// 原代码
function add(a, b) {
  return a + b;
}

// 使用 Cmd+I 输入：添加类型定义和错误处理

// Copilot 会生成
function add(a: number, b: number): number {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError('Arguments must be numbers');
  }
  return a + b;
}
```

## 常用提示模板

### 代码生成

```typescript
// 生成 React 组件
// 一个带有 loading 状态和错误处理的用户列表组件
interface UserListProps {
  users: User[];
  loading: boolean;
  error: Error | null;
}

// 生成 API 路由
// Express.js 路由，处理用户 CRUD 操作
// GET /api/users - 获取用户列表
// POST /api/users - 创建用户
// GET /api/users/:id - 获取单个用户
// PUT /api/users/:id - 更新用户
// DELETE /api/users/:id - 删除用户

// 生成数据库查询
// 使用 Prisma 查询用户，支持分页和筛选
```

### 代码审查

```markdown
在 Chat 面板中：

请审查这个文件的代码质量：
@file:src/services/user.ts

关注点：
- 错误处理是否完善
- 是否有安全漏洞
- 性能是否有问题
- 是否遵循最佳实践
```

### Bug 修复

```markdown
在 Chat 面板中：

遇到以下错误：
TypeError: Cannot read property 'id' of undefined

相关代码：
@file:src/components/UserCard.tsx

请分析原因并提供修复方案。
```

### 重构

```markdown
在 Chat 面板中：

请重构这个函数：
@file:src/utils/helpers.ts#L10-L50

目标：
- 提高可读性
- 减少重复代码
- 改善性能
- 添加类型安全
```

## 工作流优化

### 1. 新功能开发

```
1. 写清晰的注释描述需求
2. 让 Copilot 生成代码框架
3. 使用 Chat 面板完善细节
4. 运行测试验证
5. 使用 /test 生成测试
```

### 2. Bug 修复

```
1. 复制错误信息到 Chat
2. @ 引用相关文件
3. 让 Copilot 分析原因
4. 应用修复方案
5. 验证修复
```

### 3. 代码重构

```
1. 选中要重构的代码
2. 使用 Cmd+I 或 Chat 描述目标
3. 审查生成的代码
4. 运行测试确保不破坏功能
5. 应用重构
```

## 高级技巧

### 使用上下文

```typescript
// ✅ 在文件开头添加上下文注释
/**
 * 用户管理模块
 * 
 * 这个模块处理用户的 CRUD 操作，
 * 包括创建、读取、更新和删除用户。
 * 
 * 使用 PostgreSQL 数据库和 Prisma ORM。
 * 
 * @module UserService
 */

// Copilot 会根据上下文生成更准确的代码
```

### 使用类型定义

```typescript
// ✅ 定义清晰的接口让 Copilot 理解数据结构

interface User {
  id: string;
  username: string;
  email: string;
  profile: UserProfile;
  createdAt: Date;
  updatedAt: Date;
}

interface UserProfile {
  firstName: string;
  lastName: string;
  avatar?: string;
  bio?: string;
}

// Copilot 会根据接口生成正确的代码
function createUser(data: CreateUserInput): Promise<User> {
  // Copilot 知道返回类型应该是 User
}
```

### 使用测试驱动开发

```typescript
// ✅ 先写测试，让 Copilot 生成实现

describe('UserService', () => {
  describe('createUser', () => {
    it('should create a new user', async () => {
      // Arrange
      const userData = {
        username: 'testuser',
        email: 'test@example.com',
        password: 'Password123',
      };

      // Act
      const user = await userService.createUser(userData);

      // Assert
      expect(user).toBeDefined();
      expect(user.username).toBe(userData.username);
      expect(user.email).toBe(userData.email);
      expect(user.id).toBeDefined();
    });

    it('should throw error for duplicate email', async () => {
      // Copilot 会根据测试生成实现
    });
  });
});
```

## 项目特定规则

### React 项目

```markdown
## React 特定规则

### 组件
- 使用函数组件和 Hooks
- 使用 TypeScript 定义 Props
- 使用 React.memo 优化性能
- 使用 useCallback/useMemo 缓存

### 状态管理
- 本地状态使用 useState
- 复杂状态使用 useReducer
- 全局状态使用 Zustand/Jotai
- 服务端状态使用 React Query

### 样式
- 使用 Tailwind CSS
- 使用 CSS Modules
- 避免内联样式
```

### Python 项目

```markdown
## Python 特定规则

### 代码风格
- 遵循 PEP 8
- 使用 type hints
- 使用 Black 格式化
- 使用 isort 排序导入

### 项目结构
- 使用 src layout
- 分离 tests 目录
- 使用 pyproject.toml

### 依赖管理
- 使用 Poetry
- 锁定依赖版本
- 分离 dev 和 prod 依赖
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 注释太模糊
// 处理数据
function processData(data) {
  // Copilot 不知道要处理什么
}

// ❌ 不使用类型
function getUser(id) {
  // Copilot 不知道返回类型
}

// ❌ 代码太复杂
function complexFunction(a, b, c, d, e, f, g) {
  // 参数太多，Copilot 难以理解
}
```

### ✅ 推荐
```typescript
// ✅ 注释具体
// 将用户数据转换为 CSV 格式
function convertToCSV(users: User[]): string {
  // Copilot 会生成正确的实现
}

// ✅ 使用类型
function getUser(id: string): Promise<User | null> {
  // Copilot 知道返回类型
}

// ✅ 参数简洁
function createUser(options: CreateUserOptions): Promise<User> {
  // 使用对象参数，更清晰
}
```

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 技术栈：
- 特殊指令：
```
