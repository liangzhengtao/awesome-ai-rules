# API Design AI 编程规则
> Last updated: 2026 | Targets: REST / OpenAPI 3.1 / JSON:API

## 核心原则

- 遵循 RESTful 设计原则
- 使用清晰一致的命名规范
- 提供完善的错误处理
- 支持版本控制
- 编写详细的 API 文档

## 代码风格

### 命名规范
- URL 使用 kebab-case：`/api/v1/user-profiles`
- 查询参数使用 camelCase：`?pageSize=10&pageNumber=1`
- 请求/响应体使用 camelCase：`{ "userName": "John" }`
- HTTP 方法语义正确：GET（查询）、POST（创建）、PUT（全量更新）、PATCH（部分更新）、DELETE（删除）

### URL 结构
```
/api/v1/{resource}              # 资源集合
/api/v1/{resource}/{id}         # 单个资源
/api/v1/{resource}/{id}/{sub}   # 子资源
```

## 最佳实践

### RESTful API 设计

```typescript
// ✅ 用户 API 设计示例

// 获取用户列表
GET /api/v1/users?page=1&size=20&sort=-createdAt

// 响应
{
  "data": [
    {
      "id": "123",
      "username": "john_doe",
      "email": "john@example.com",
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 100,
    "totalPages": 5
  }
}

// 获取单个用户
GET /api/v1/users/123

// 响应
{
  "data": {
    "id": "123",
    "username": "john_doe",
    "email": "john@example.com",
    "profile": {
      "firstName": "John",
      "lastName": "Doe",
      "avatar": "https://example.com/avatar.jpg"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "updatedAt": "2024-01-01T00:00:00Z"
  }
}

// 创建用户
POST /api/v1/users
Content-Type: application/json

{
  "username": "jane_doe",
  "email": "jane@example.com",
  "password": "securePassword123"
}

// 响应 (201 Created)
{
  "data": {
    "id": "456",
    "username": "jane_doe",
    "email": "jane@example.com",
    "createdAt": "2024-01-02T00:00:00Z"
  }
}

// 更新用户
PATCH /api/v1/users/456
Content-Type: application/json

{
  "email": "newemail@example.com"
}

// 删除用户
DELETE /api/v1/users/456

// 响应 (204 No Content)
```

### 错误处理

```typescript
// ✅ 统一错误响应格式
interface ErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Array<{
      field: string;
      message: string;
    }>;
    requestId: string;
    timestamp: string;
  };
}

// 400 Bad Request
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      },
      {
        "field": "password",
        "message": "Password must be at least 8 characters"
      }
    ],
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// 401 Unauthorized
{
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Authentication required",
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "code": "FORBIDDEN",
    "message": "Insufficient permissions",
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// 404 Not Found
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Resource not found",
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// 429 Too Many Requests
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded. Please try again later.",
    "retryAfter": 60,
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}

// 500 Internal Server Error
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred",
    "requestId": "req_abc123",
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

### API 版本控制

```typescript
// ✅ URL 版本控制
/api/v1/users
/api/v2/users

// ✅ Header 版本控制
GET /api/users
Accept: application/vnd.myapi.v2+json

// ✅ 查询参数版本控制
GET /api/users?version=2
```

### 分页

```typescript
// ✅ 分页请求
GET /api/v1/users?page=1&size=20&sort=-createdAt&filter[status]=active

// ✅ 分页响应
{
  "data": [...],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 100,
    "totalPages": 5,
    "hasNext": true,
    "hasPrev": false
  },
  "links": {
    "self": "/api/v1/users?page=1&size=20",
    "next": "/api/v1/users?page=2&size=20",
    "last": "/api/v1/users?page=5&size=20"
  }
}

// ✅ 游标分页（适用于大数据集）
GET /api/v1/users?cursor=eyJpZCI6MTAwfQ==&limit=20

{
  "data": [...],
  "pagination": {
    "nextCursor": "eyJpZCI6MTIwfQ==",
    "hasMore": true
  }
}
```

### 过滤和搜索

```typescript
// ✅ 过滤
GET /api/v1/users?filter[status]=active&filter[role]=admin

// ✅ 搜索
GET /api/v1/users?search=john

// ✅ 字段选择
GET /api/v1/users?fields=id,username,email

// ✅ 嵌套资源过滤
GET /api/v1/users/123/posts?filter[status]=published&sort=-createdAt
```

### 认证和授权

```typescript
// ✅ JWT 认证
// 请求
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

// 响应
{
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  }
}

// ✅ 使用 Token
GET /api/v1/users/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// ✅ OAuth2 认证
POST /api/v1/auth/oauth/callback
Content-Type: application/json

{
  "provider": "google",
  "code": "4/0AX4XfWh...",
  "redirectUri": "https://example.com/callback"
}
```

### 速率限制

```typescript
// ✅ 速率限制响应头
HTTP/1.1 200 OK
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640000000

// 超出限制
HTTP/1.1 429 Too Many Requests
Retry-After: 60
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1640000000
```

### 缓存

```typescript
// ✅ ETag 缓存
GET /api/v1/users/123

// 响应
HTTP/1.1 200 OK
ETag: "33a64df5"
Cache-Control: max-age=3600

// 后续请求
GET /api/v1/users/123
If-None-Match: "33a64df5"

// 如果未修改
HTTP/1.1 304 Not Modified
```

### HATEOAS

```typescript
// ✅ 超媒体链接
{
  "data": {
    "id": "123",
    "username": "john_doe",
    "email": "john@example.com"
  },
  "links": {
    "self": "/api/v1/users/123",
    "posts": "/api/v1/users/123/posts",
    "followers": "/api/v1/users/123/followers",
    "update": {
      "href": "/api/v1/users/123",
      "method": "PATCH"
    },
    "delete": {
      "href": "/api/v1/users/123",
      "method": "DELETE"
    }
  }
}
```

### Webhook

```typescript
// ✅ Webhook 配置
POST /api/v1/webhooks
Content-Type: application/json

{
  "url": "https://example.com/webhook",
  "events": ["user.created", "user.updated", "order.completed"],
  "secret": "whsec_..."
}

// ✅ Webhook 事件格式
{
  "id": "evt_abc123",
  "type": "user.created",
  "createdAt": "2024-01-01T00:00:00Z",
  "data": {
    "userId": "123",
    "username": "john_doe"
  },
  "signature": "sha256=..."
}
```

## OpenAPI 规范

```yaml
# ✅ OpenAPI 3.0 规范示例
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0
  description: User management API

servers:
  - url: https://api.example.com/v1
    description: Production

paths:
  /users:
    get:
      summary: Get user list
      operationId: listUsers
      tags:
        - Users
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: size
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserListResponse'

    post:
      summary: Create user
      operationId: createUser
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserRequest'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserResponse'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: string
        username:
          type: string
        email:
          type: string
          format: email
        createdAt:
          type: string
          format: date-time

    CreateUserRequest:
      type: object
      required:
        - username
        - email
        - password
      properties:
        username:
          type: string
          minLength: 3
          maxLength: 50
        email:
          type: string
          format: email
        password:
          type: string
          minLength: 8

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

## 测试

```typescript
// ✅ API 测试
import request from 'supertest';
import { app } from '../src/app';

describe('Users API', () => {
  describe('GET /api/v1/users', () => {
    it('should return paginated users', async () => {
      const response = await request(app)
        .get('/api/v1/users')
        .query({ page: 1, size: 10 })
        .set('Authorization', `Bearer ${token}`)
        .expect(200);

      expect(response.body.data).toBeInstanceOf(Array);
      expect(response.body.pagination).toBeDefined();
      expect(response.body.pagination.page).toBe(1);
    });
  });

  describe('POST /api/v1/users', () => {
    it('should create a new user', async () => {
      const response = await request(app)
        .post('/api/v1/users')
        .send({
          username: 'newuser',
          email: 'new@example.com',
          password: 'Password123',
        })
        .expect(201);

      expect(response.body.data.id).toBeDefined();
      expect(response.body.data.username).toBe('newuser');
    });

    it('should return 400 for invalid input', async () => {
      const response = await request(app)
        .post('/api/v1/users')
        .send({
          username: 'ab', // Too short
          email: 'invalid',
          password: '123', // Too short
        })
        .expect(400);

      expect(response.body.error.code).toBe('VALIDATION_ERROR');
      expect(response.body.error.details).toBeInstanceOf(Array);
    });
  });
});
```

## 常见陷阱

### ❌ 避免
```typescript
// ❌ 不一致的命名
GET /api/v1/getUsers      // 使用动词
GET /api/v1/user_list     // 使用下划线
GET /api/v1/UserList      // 使用 PascalCase

// ❌ 错误的 HTTP 方法
POST /api/v1/users/delete  // 应该用 DELETE
GET /api/v1/users/create   // 应该用 POST

// ❌ 暴露内部实现
{
  "id": 123,
  "password_hash": "$2b$10$...",
  "internal_role_id": 5,
  "database_table": "users"
}
```

### ✅ 推荐
```typescript
// ✅ 一致的命名
GET /api/v1/users          // 使用名词复数
GET /api/v1/user-profiles  // 使用 kebab-case

// ✅ 正确的 HTTP 方法
DELETE /api/v1/users/123
POST /api/v1/users

// ✅ 隐藏内部实现
{
  "id": "123",
  "username": "john_doe",
  "email": "john@example.com",
  "role": "admin"
}
```

## 依赖推荐

- **API 框架**: Express / Fastify / NestJS (Node.js), FastAPI (Python), Gin (Go)
- **验证**: Zod / Joi / Yup (Node.js), Pydantic (Python)
- **文档**: Swagger / Redoc / Stoplight
- **测试**: Jest + Supertest (Node.js), pytest + httpx (Python)
- **网关**: Kong / APISIX / Traefik

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- API 风格：REST / GraphQL / gRPC
- 认证方式：JWT / OAuth2 / API Key
- 版本控制方式：URL / Header / Query
- 文档工具：Swagger / Redoc
```
