# Go AI 编程规则
> Last updated: 2026 | Targets: Go 1.22+ / Chi 5.x

## 核心原则

- 遵循 Go 代码规范（Effective Go）
- 优先使用组合而非继承
- 使用接口实现多态
- 合理使用 goroutine 和 channel
- 保持简单和可读

## 代码风格

### 命名规范
- 包名：小写单词：`user`, `http`, `json`
- 结构体/接口：PascalCase：`UserService`, `Reader`
- 函数/方法：PascalCase（导出）/ camelCase（未导出）：`GetUser`, `validateInput`
- 变量/参数：camelCase：`userID`, `maxRetries`
- 常量：PascalCase 或 SCREAMING_SNAKE：`MaxRetries`, `DefaultTimeout`
- 接口：动词结尾：`Reader`, `Writer`, `Closer`

### 文件结构
```
cmd/
└── server/
    └── main.go
internal/
├── config/
│   └── config.go
├── handler/
│   ├── user.go
│   └── item.go
├── middleware/
│   └── auth.go
├── model/
│   ├── user.go
│   └── item.go
├── repository/
│   ├── user.go
│   └── item.go
├── service/
│   ├── user.go
│   └── item.go
└── errors/
    └── errors.go
pkg/
├── logger/
│   └── logger.go
└── validator/
    └── validator.go
```

## 最佳实践

### 错误处理

```go
// ✅ 定义错误类型
package errors

import "fmt"

type AppError struct {
    Code    int
    Message string
    Err     error
}

func (e *AppError) Error() string {
    if e.Err != nil {
        return fmt.Sprintf("%s: %v", e.Message, e.Err)
    }
    return e.Message
}

func (e *AppError) Unwrap() error {
    return e.Err
}

// 预定义错误
var (
    ErrNotFound     = &AppError{Code: 404, Message: "resource not found"}
    ErrUnauthorized = &AppError{Code: 401, Message: "unauthorized"}
    ErrForbidden    = &AppError{Code: 403, Message: "forbidden"}
    ErrValidation   = &AppError{Code: 400, Message: "validation error"}
)

// 创建错误
func NewNotFoundError(resource string, id interface{}) *AppError {
    return &AppError{
        Code:    404,
        Message: fmt.Sprintf("%s with id %v not found", resource, id),
    }
}

// ✅ 使用 %w 包装错误
func GetUser(id string) (*User, error) {
    user, err := repo.FindByID(id)
    if err != nil {
        return nil, fmt.Errorf("failed to get user %s: %w", id, err)
    }
    return user, nil
}

// ✅ 检查错误类型
func HandleError(err error) {
    var appErr *AppError
    if errors.As(err, &appErr) {
        switch appErr.Code {
        case 404:
            // 处理 404
        case 401:
            // 处理 401
        }
    }
}
```

### HTTP 处理器

```go
// ✅ handler/user.go
package handler

import (
    "encoding/json"
    "net/http"

    "github.com/go-chi/chi/v5"
    "github.com/google/uuid"

    "myapp/internal/errors"
    "myapp/internal/model"
    "myapp/internal/service"
)

type UserHandler struct {
    service *service.UserService
}

func NewUserHandler(s *service.UserService) *UserHandler {
    return &UserHandler{service: s}
}

func (h *UserHandler) Routes() chi.Router {
    r := chi.NewRouter()

    r.Get("/", h.ListUsers)
    r.Post("/", h.CreateUser)
    r.Get("/{id}", h.GetUser)
    r.Put("/{id}", h.UpdateUser)
    r.Delete("/{id}", h.DeleteUser)

    return r
}

// ListUsers 获取用户列表
// @Summary 获取用户列表
// @Tags users
// @Accept json
// @Produce json
// @Param page query int false "页码"
// @Param size query int false "每页数量"
// @Success 200 {array} model.User
// @Router /api/users [get]
func (h *UserHandler) ListUsers(w http.ResponseWriter, r *http.Request) {
    page := queryInt(r, "page", 1)
    size := queryInt(r, "size", 20)

    users, err := h.service.List(page, size)
    if err != nil {
        writeError(w, err)
        return
    }

    writeJSON(w, http.StatusOK, users)
}

// GetUser 获取用户详情
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := chi.URLParam(r, "id")

    user, err := h.service.GetByID(id)
    if err != nil {
        writeError(w, err)
        return
    }

    writeJSON(w, http.StatusOK, user)
}

// CreateUser 创建用户
func (h *UserHandler) CreateUser(w http.ResponseWriter, r *http.Request) {
    var input model.CreateUserInput

    if err := json.NewDecoder(r.Body).Decode(&input); err != nil {
        writeError(w, errors.ErrValidation)
        return
    }

    if err := validate.Struct(input); err != nil {
        writeValidationError(w, err)
        return
    }

    user, err := h.service.Create(&input)
    if err != nil {
        writeError(w, err)
        return
    }

    writeJSON(w, http.StatusCreated, user)
}

// 辅助函数
func writeJSON(w http.ResponseWriter, status int, data interface{}) {
    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(status)
    json.NewEncoder(w).Encode(data)
}

func writeError(w http.ResponseWriter, err error) {
    var appErr *errors.AppError
    if errors.As(err, &appErr) {
        writeJSON(w, appErr.Code, map[string]string{
            "error": appErr.Message,
        })
        return
    }

    writeJSON(w, http.StatusInternalServerError, map[string]string{
        "error": "internal server error",
    })
}
```

### Repository 模式

```go
// ✅ repository/user.go
package repository

import (
    "context"
    "database/sql"
    "fmt"

    "github.com/google/uuid"

    "myapp/internal/model"
)

type UserRepository interface {
    FindByID(ctx context.Context, id uuid.UUID) (*model.User, error)
    FindByUsername(ctx context.Context, username string) (*model.User, error)
    FindAll(ctx context.Context, page, size int) ([]*model.User, error)
    Create(ctx context.Context, user *model.User) error
    Update(ctx context.Context, user *model.User) error
    Delete(ctx context.Context, id uuid.UUID) error
}

type postgresUserRepo struct {
    db *sql.DB
}

func NewPostgresUserRepo(db *sql.DB) UserRepository {
    return &postgresUserRepo{db: db}
}

func (r *postgresUserRepo) FindByID(ctx context.Context, id uuid.UUID) (*model.User, error) {
    query := `
        SELECT id, username, email, created_at, updated_at
        FROM users
        WHERE id = $1
    `

    user := &model.User{}
    err := r.db.QueryRowContext(ctx, query, id).Scan(
        &user.ID,
        &user.Username,
        &user.Email,
        &user.CreatedAt,
        &user.UpdatedAt,
    )

    if err == sql.ErrNoRows {
        return nil, nil
    }

    if err != nil {
        return nil, fmt.Errorf("failed to find user: %w", err)
    }

    return user, nil
}

func (r *postgresUserRepo) Create(ctx context.Context, user *model.User) error {
    query := `
        INSERT INTO users (id, username, email, password_hash, created_at, updated_at)
        VALUES ($1, $2, $3, $4, $5, $6)
    `

    _, err := r.db.ExecContext(ctx, query,
        user.ID,
        user.Username,
        user.Email,
        user.PasswordHash,
        user.CreatedAt,
        user.UpdatedAt,
    )

    if err != nil {
        return fmt.Errorf("failed to create user: %w", err)
    }

    return nil
}
```

### 并发处理

```go
// ✅ 使用 errgroup 并发执行
import (
    "golang.org/x/sync/errgroup"
    "golang.org/x/sync/semaphore"
)

func FetchMultipleResources(ids []string) ([]*Resource, error) {
    g, ctx := errgroup.WithContext(context.Background())

    // 限制并发数
    sem := semaphore.NewWeighted(10)

    results := make([]*Resource, len(ids))

    for i, id := range ids {
        i, id := i, id

        g.Go(func() error {
            // 获取信号量
            if err := sem.Acquire(ctx, 1); err != nil {
                return err
            }
            defer sem.Release(1)

            resource, err := fetchResource(ctx, id)
            if err != nil {
                return err
            }

            results[i] = resource
            return nil
        })
    }

    if err := g.Wait(); err != nil {
        return nil, err
    }

    return results, nil
}

// ✅ 使用 context 控制超时
func FetchWithTimeout(ctx context.Context, url string) ([]byte, error) {
    ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
    defer cancel()

    req, err := http.NewRequestWithContext(ctx, "GET", url, nil)
    if err != nil {
        return nil, err
    }

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    return io.ReadAll(resp.Body)
}
```

### 中间件

```go
// ✅ middleware/auth.go
package middleware

import (
    "context"
    "net/http"
    "strings"
)

type contextKey string

const UserIDKey contextKey = "userID"

func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "unauthorized", http.StatusUnauthorized)
            return
        }

        // 移除 Bearer 前缀
        token = strings.TrimPrefix(token, "Bearer ")

        userID, err := validateToken(token)
        if err != nil {
            http.Error(w, "invalid token", http.StatusUnauthorized)
            return
        }

        // 将用户 ID 存入 context
        ctx := context.WithValue(r.Context(), UserIDKey, userID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}

func GetUserID(ctx context.Context) string {
    if userID, ok := ctx.Value(UserIDKey).(string); ok {
        return userID
    }
    return ""
}

// ✅ 中间件链
func SetupRouter(handler *UserHandler) http.Handler {
    r := chi.NewRouter()

    // 全局中间件
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.RequestID)
    r.Use(cors.Handler(cors.Options{
        AllowedOrigins: []string{"*"},
        AllowedMethods: []string{"GET", "POST", "PUT", "DELETE"},
    }))

    // 路由
    r.Route("/api", func(r chi.Router) {
        r.Use(AuthMiddleware)

        r.Mount("/users", handler.Routes())
    })

    return r
}
```

## 测试

```go
// ✅ user_service_test.go
package service_test

import (
    "context"
    "testing"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/mock"

    "myapp/internal/model"
    "myapp/internal/service"
)

// Mock Repository
type mockUserRepo struct {
    mock.Mock
}

func (m *mockUserRepo) FindByID(ctx context.Context, id string) (*model.User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*model.User), args.Error(1)
}

func (m *mockUserRepo) Create(ctx context.Context, user *model.User) error {
    args := m.Called(ctx, user)
    return args.Error(0)
}

func TestUserService_GetByID(t *testing.T) {
    t.Run("success", func(t *testing.T) {
        repo := new(mockUserRepo)
        svc := service.NewUserService(repo)

        expected := &model.User{
            ID:       "1",
            Username: "testuser",
            Email:    "test@example.com",
        }

        repo.On("FindByID", mock.Anything, "1").Return(expected, nil)

        user, err := svc.GetByID("1")

        assert.NoError(t, err)
        assert.Equal(t, expected, user)
        repo.AssertExpectations(t)
    })

    t.Run("not found", func(t *testing.T) {
        repo := new(mockUserRepo)
        svc := service.NewUserService(repo)

        repo.On("FindByID", mock.Anything, "999").Return(nil, nil)

        user, err := svc.GetByID("999")

        assert.Error(t, err)
        assert.Nil(t, user)
        repo.AssertExpectations(t)
    })
}

// ✅ 表驱动测试
func TestValidateInput(t *testing.T) {
    tests := []struct {
        name    string
        input   model.CreateUserInput
        wantErr bool
    }{
        {
            name: "valid input",
            input: model.CreateUserInput{
                Username: "testuser",
                Email:    "test@example.com",
                Password: "Password123",
            },
            wantErr: false,
        },
        {
            name: "invalid email",
            input: model.CreateUserInput{
                Username: "testuser",
                Email:    "invalid",
                Password: "Password123",
            },
            wantErr: true,
        },
        {
            name: "short password",
            input: model.CreateUserInput{
                Username: "testuser",
                Email:    "test@example.com",
                Password: "123",
            },
            wantErr: true,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            err := validate.Struct(tt.input)
            if tt.wantErr {
                assert.Error(t, err)
            } else {
                assert.NoError(t, err)
            }
        })
    }
}
```

## 常见陷阱

### ❌ 避免
```go
// ❌ 忽略错误
result, _ := doSomething()

// ❌ 在循环中启动 goroutine 导致闭包问题
for _, item := range items {
    go func() {
        process(item) // item 可能已经改变
    }()
}

// ❌ 使用 init() 做复杂初始化
func init() {
    // 复杂的初始化逻辑
}
```

### ✅ 推荐
```go
// ✅ 正确处理错误
result, err := doSomething()
if err != nil {
    return fmt.Errorf("doSomething failed: %w", err)
}

// ✅ 正确传递循环变量
for _, item := range items {
    item := item // 重新声明
    go func() {
        process(item)
    }()
}

// ✅ 使用构造函数
func NewService(cfg *Config) (*Service, error) {
    // 初始化逻辑
}
```

## 依赖推荐

- **Web 框架**: Chi / Gin / Echo / Fiber
- **数据库**: sqlx / GORM / Ent
- **配置**: Viper
- **日志**: Zap / Zerolog
- **验证**: go-playground/validator
- **测试**: testing + testify
- **代码规范**: golangci-lint

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- Go 版本：
- Web 框架：
- 数据库：
- 部署方式：
```
