# Microservices AI 编程规则
> Last updated: 2026 | Targets: gRPC / Kafka / Kubernetes 1.29+

## 核心原则

- 单一职责原则
- 服务自治
- 去中心化治理
- 容错设计
- 可观测性

## 代码风格

### 服务结构
```
services/
├── user-service/           # 用户服务
│   ├── cmd/               # 程序入口
│   ├── internal/          # 内部实现
│   │   ├── handler/       # HTTP/RPC 处理器
│   │   ├── service/       # 业务逻辑
│   │   ├── repository/    # 数据访问
│   │   └── model/         # 数据模型
│   ├── pkg/               # 可导出的包
│   ├── api/               # API 定义（Proto/OpenAPI）
│   ├── migrations/        # 数据库迁移
│   ├── Dockerfile
│   └── Makefile
├── order-service/          # 订单服务
└── gateway/                # API 网关
```

### 命名规范
- 服务名：`kebab-case`：`user-service`, `order-service`
- 包名：`snake_case`：`user_service`, `order_service`
- API 版本：`/api/v1/`, `/api/v2/`
- 事件名：`PastTense`：`UserCreated`, `OrderPlaced`

## 最佳实践

### API 设计

```protobuf
// ✅ 使用 Protocol Buffers 定义 API
syntax = "proto3";

package user.v1;

service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(ListUsersRequest) returns (ListUsersResponse);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
  rpc DeleteUser(DeleteUserRequest) returns (google.protobuf.Empty);
}

message User {
  string id = 1;
  string username = 2;
  string email = 3;
  google.protobuf.Timestamp created_at = 4;
  google.protobuf.Timestamp updated_at = 5;
}

message GetUserRequest {
  string id = 1;
}

message ListUsersRequest {
  int32 page = 1;
  int32 size = 2;
}
```

### 服务通信

```go
// ✅ 同步通信 - gRPC Client
type UserServiceClient struct {
    conn   *grpc.ClientConn
    client pb.UserServiceClient
}

func NewUserServiceClient(addr string) (*UserServiceClient, error) {
    conn, err := grpc.Dial(addr,
        grpc.WithTransportCredentials(insecure.NewCredentials()),
        grpc.WithUnaryInterceptor(retryInterceptor(3)),
    )
    if err != nil {
        return nil, err
    }

    return &UserServiceClient{
        conn:   conn,
        client: pb.NewUserServiceClient(conn),
    }, nil
}

func (c *UserServiceClient) GetUser(ctx context.Context, id string) (*User, error) {
    resp, err := c.client.GetUser(ctx, &pb.GetUserRequest{Id: id})
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    return toUser(resp), nil
}

// ✅ 异步通信 - 事件发布
type EventPublisher interface {
    Publish(ctx context.Context, topic string, event interface{}) error
}

type UserCreatedEvent struct {
    UserID    string    `json:"user_id"`
    Username  string    `json:"username"`
    Email     string    `json:"email"`
    CreatedAt time.Time `json:"created_at"`
}

func (s *UserService) CreateUser(ctx context.Context, input *CreateUserInput) (*User, error) {
    user, err := s.repo.Create(ctx, input)
    if err != nil {
        return nil, err
    }

    // 发布事件
    event := UserCreatedEvent{
        UserID:    user.ID,
        Username:  user.Username,
        Email:     user.Email,
        CreatedAt: user.CreatedAt,
    }

    if err := s.publisher.Publish(ctx, "user.created", event); err != nil {
        s.logger.Error("failed to publish user.created event", "error", err)
    }

    return user, nil
}
```

### 事件驱动

```go
// ✅ 事件处理
type EventHandler interface {
    Handle(ctx context.Context, event Event) error
}

type UserCreatedHandler struct {
    notificationService *NotificationService
    analyticsService    *AnalyticsService
}

func (h *UserCreatedHandler) Handle(ctx context.Context, event Event) error {
    var payload UserCreatedEvent
    if err := event.Payload(&payload); err != nil {
        return err
    }

    // 发送欢迎邮件
    if err := h.notificationService.SendWelcomeEmail(ctx, payload.Email); err != nil {
        return fmt.Errorf("failed to send welcome email: %w", err)
    }

    // 记录分析事件
    if err := h.analyticsService.TrackUserRegistration(ctx, payload.UserID); err != nil {
        return fmt.Errorf("failed to track user registration: %w", err)
    }

    return nil
}
```

### 服务发现

```go
// ✅ 使用 Consul 进行服务发现
type ServiceDiscovery struct {
    client *consul.Client
}

func (sd *ServiceDiscovery) Register(service *ServiceInfo) error {
    registration := &consul.AgentServiceRegistration{
        ID:      service.ID,
        Name:    service.Name,
        Port:    service.Port,
        Address: service.Address,
        Check: &consul.AgentServiceCheck{
            HTTP:     fmt.Sprintf("http://%s:%d/health", service.Address, service.Port),
            Interval: "10s",
            Timeout:  "5s",
        },
    }

    return sd.client.Agent().ServiceRegister(registration)
}

func (sd *ServiceDiscovery) Discover(serviceName string) ([]*ServiceInfo, error) {
    services, _, err := sd.client.Health().Service(serviceName, "", true, nil)
    if err != nil {
        return nil, err
    }

    var result []*ServiceInfo
    for _, s := range services {
        result = append(result, &ServiceInfo{
            ID:      s.Service.ID,
            Name:    s.Service.Service,
            Address: s.Service.Address,
            Port:    s.Service.Port,
        })
    }

    return result, nil
}
```

### 熔断器

```go
// ✅ 使用熔断器保护服务调用
type CircuitBreaker struct {
    maxFailures  int
    resetTimeout time.Duration
    failures     int
    lastFailure  time.Time
    state        string // "closed", "open", "half-open"
    mutex        sync.Mutex
}

func (cb *CircuitBreaker) Execute(fn func() error) error {
    cb.mutex.Lock()

    if cb.state == "open" {
        if time.Since(cb.lastFailure) > cb.resetTimeout {
            cb.state = "half-open"
        } else {
            cb.mutex.Unlock()
            return ErrCircuitOpen
        }
    }

    cb.mutex.Unlock()

    err := fn()

    cb.mutex.Lock()
    defer cb.mutex.Unlock()

    if err != nil {
        cb.failures++
        cb.lastFailure = time.Now()

        if cb.failures >= cb.maxFailures {
            cb.state = "open"
        }

        return err
    }

    cb.failures = 0
    cb.state = "closed"

    return nil
}
```

### 链路追踪

```go
// ✅ 使用 OpenTelemetry 进行链路追踪
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/trace"
)

type UserService struct {
    tracer trace.Tracer
    repo   UserRepository
}

func (s *UserService) GetUser(ctx context.Context, id string) (*User, error) {
    ctx, span := s.tracer.Start(ctx, "UserService.GetUser")
    defer span.End()

    span.SetAttributes(attribute.String("user.id", id))

    user, err := s.repo.FindByID(ctx, id)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return nil, err
    }

    span.SetStatus(codes.Ok, "")
    return user, nil
}
```

### 配置管理

```go
// ✅ 使用配置中心
type Config struct {
    ServiceName string        `env:"SERVICE_NAME" envDefault:"user-service"`
    Port        int           `env:"PORT" envDefault:"8080"`
    Database    DatabaseConfig
    Redis       RedisConfig
    Kafka       KafkaConfig
}

type DatabaseConfig struct {
    Host     string `env:"DB_HOST" envDefault:"localhost"`
    Port     int    `env:"DB_PORT" envDefault:"5432"`
    User     string `env:"DB_USER"`
    Password string `env:"DB_PASSWORD"`
    Name     string `env:"DB_NAME"`
}

func LoadConfig() (*Config, error) {
    cfg := &Config{}

    if err := env.Parse(cfg); err != nil {
        return nil, err
    }

    return cfg, nil
}
```

### 健康检查

```go
// ✅ 健康检查端点
func (h *Handler) HealthCheck(w http.ResponseWriter, r *http.Request) {
    checks := map[string]string{
        "database": "ok",
        "redis":    "ok",
        "kafka":    "ok",
    }

    // 检查数据库
    if err := h.db.Ping(); err != nil {
        checks["database"] = "error"
    }

    // 检查 Redis
    if err := h.redis.Ping(); err != nil {
        checks["redis"] = "error"
    }

    // 检查 Kafka
    if err := h.kafka.Ping(); err != nil {
        checks["kafka"] = "error"
    }

    status := "healthy"
    httpStatus := http.StatusOK

    for _, check := range checks {
        if check != "ok" {
            status = "unhealthy"
            httpStatus = http.StatusServiceUnavailable
            break
        }
    }

    response := map[string]interface{}{
        "status": status,
        "checks": checks,
    }

    w.Header().Set("Content-Type", "application/json")
    w.WriteHeader(httpStatus)
    json.NewEncoder(w).Encode(response)
}
```

## 数据库策略

### 每服务一数据库
```yaml
# docker-compose.yml
services:
  user-db:
    image: postgres:15
    environment:
      POSTGRES_DB: user_service
      POSTGRES_USER: user_service
      POSTGRES_PASSWORD: secret

  order-db:
    image: postgres:15
    environment:
      POSTGRES_DB: order_service
      POSTGRES_USER: order_service
      POSTGRES_PASSWORD: secret
```

### Saga 模式
```go
// ✅ 编排式 Saga
type OrderSaga struct {
    steps []SagaStep
}

type SagaStep struct {
    Name     string
    Execute  func(ctx context.Context) error
    Compensate func(ctx context.Context) error
}

func (s *OrderSaga) Execute(ctx context.Context) error {
    executedSteps := make([]SagaStep, 0, len(s.steps))

    for _, step := range s.steps {
        if err := step.Execute(ctx); err != nil {
            // 补偿已执行的步骤
            for i := len(executedSteps) - 1; i >= 0; i-- {
                if err := executedSteps[i].Compensate(ctx); err != nil {
                    // 记录补偿失败
                }
            }
            return err
        }
        executedSteps = append(executedSteps, step)
    }

    return nil
}
```

## 部署策略

### Docker 配置
```dockerfile
# ✅ 多阶段构建
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /app/server ./cmd/server

FROM alpine:3.19

RUN apk --no-cache add ca-certificates tzdata

WORKDIR /app
COPY --from=builder /app/server .

EXPOSE 8080
CMD ["./server"]
```

### Kubernetes 部署
```yaml
# ✅ Kubernetes Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
        - name: user-service
          image: user-service:latest
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

## 常见陷阱

### ❌ 避免
```go
// ❌ 同步调用链过长
func PlaceOrder(ctx context.Context, order *Order) error {
    user := userService.GetUser(order.UserID)      // 调用用户服务
    products := productService.GetItems(order.Items) // 调用产品服务
    payment := paymentService.Process(order.Total)   // 调用支付服务
    notification.Send(user.Email, payment)           // 调用通知服务
    return nil
}

// ❌ 共享数据库
// 多个服务直接访问同一个数据库
```

### ✅ 推荐
```go
// ✅ 使用事件驱动
func PlaceOrder(ctx context.Context, order *Order) error {
    // 创建订单
    if err := repo.Create(ctx, order); err != nil {
        return err
    }

    // 发布事件
    return publisher.Publish(ctx, "order.created", OrderCreatedEvent{
        OrderID: order.ID,
        UserID:  order.UserID,
        Items:   order.Items,
    })
}

// ✅ 每服务独立数据库
// 每个服务有自己的数据库，通过 API 或事件通信
```

## 依赖推荐

- **通信**: gRPC / REST / GraphQL
- **消息队列**: Kafka / RabbitMQ / NATS
- **服务发现**: Consul / etcd / Kubernetes
- **配置中心**: Consul / etcd / Apollo
- **链路追踪**: Jaeger / Zipkin / OpenTelemetry
- **监控**: Prometheus + Grafana
- **日志**: ELK Stack / Loki

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- 服务数量：
- 通信方式：gRPC / REST / 混合
- 消息队列：
- 服务发现：
- 部署平台：Kubernetes / Docker Swarm
```
