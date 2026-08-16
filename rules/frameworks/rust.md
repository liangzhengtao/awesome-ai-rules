# Rust AI 编程规则
> Last updated: 2026 | Targets: Rust 1.75+ / Axum 0.7+ / Tokio

## 核心原则

- 遵循 Rust 所有权和借用规则
- 优先使用零成本抽象
- 合理使用 trait 实现多态
- 充分利用模式匹配
- 编写惯用的 Rust 代码（Idiomatic Rust）

## 代码风格

### 命名规范
- 模块/文件：snake_case：`user_service.rs`, `config.rs`
- 类型/结构体/枚举：PascalCase：`UserProfile`, `DatabaseError`
- 函数/方法：snake_case：`get_user`, `create_item`
- 常量/静态：SCREAMING_SNAKE_CASE：`MAX_CONNECTIONS`, `DEFAULT_PORT`
- 生命周期：短小写：`'a`, `'b`, `'de`
- 泛型类型参数：单大写字母或 PascalCase：`T`, `U`, `Item`

### 文件结构
```
src/
├── main.rs                # 程序入口
├── lib.rs                 # 库入口
├── config/
│   ├── mod.rs
│   └── settings.rs
├── api/
│   ├── mod.rs
│   ├── routes/
│   │   ├── mod.rs
│   │   ├── users.rs
│   │   └── items.rs
│   └── middleware/
│       ├── mod.rs
│       └── auth.rs
├── models/
│   ├── mod.rs
│   ├── user.rs
│   └── item.rs
├── services/
│   ├── mod.rs
│   ├── user_service.rs
│   └── item_service.rs
├── db/
│   ├── mod.rs
│   ├── connection.rs
│   └── repository.rs
├── errors/
│   ├── mod.rs
│   └── types.rs
└── utils/
    ├── mod.rs
    └── helpers.rs
```

## 最佳实践

### 错误处理

```rust
// ✅ 使用 thiserror 定义错误类型
use thiserror::Error;

#[derive(Debug, Error)]
pub enum AppError {
    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Not found: {resource} with id {id}")]
    NotFound { resource: String, id: String },

    #[error("Validation error: {0}")]
    Validation(String),

    #[error("Unauthorized: {0}")]
    Unauthorized(String),

    #[error("Internal error: {0}")]
    Internal(#[from] anyhow::Error),
}

// 实现 IntoResponse 用于 Axum
impl IntoResponse for AppError {
    fn into_response(self) -> Response {
        let (status, error_message) = match &self {
            AppError::NotFound { .. } => (StatusCode::NOT_FOUND, self.to_string()),
            AppError::Validation(_) => (StatusCode::BAD_REQUEST, self.to_string()),
            AppError::Unauthorized(_) => (StatusCode::UNAUTHORIZED, self.to_string()),
            _ => (
                StatusCode::INTERNAL_SERVER_ERROR,
                "Internal server error".to_string(),
            ),
        };

        let body = Json(serde_json::json!({
            "error": error_message,
            "status": status.as_u16(),
        }));

        (status, body).into_response()
    }
}

// ✅ 使用 Result 别名
pub type Result<T> = std::result::Result<T, AppError>;
```

### Axum 路由

```rust
// ✅ api/routes/users.rs
use axum::{
    extract::{Path, Query, State},
    http::StatusCode,
    Json,
};
use serde::{Deserialize, Serialize};
use uuid::Uuid;

use crate::errors::Result;
use crate::models::user::{User, CreateUser, UpdateUser};
use crate::services::UserService;

#[derive(Debug, Deserialize)]
pub struct Pagination {
    pub page: Option<i64>,
    pub size: Option<i64>,
}

#[derive(Debug, Serialize)]
pub struct UserResponse {
    pub id: Uuid,
    pub username: String,
    pub email: String,
    pub created_at: chrono::NaiveDateTime,
}

impl From<User> for UserResponse {
    fn from(user: User) -> Self {
        Self {
            id: user.id,
            username: user.username,
            email: user.email,
            created_at: user.created_at,
        }
    }
}

// GET /api/users
pub async fn list_users(
    State(state): State<AppState>,
    Query(pagination): Query<Pagination>,
) -> Result<Json<Vec<UserResponse>>> {
    let page = pagination.page.unwrap_or(1);
    let size = pagination.size.unwrap_or(20);

    let users = state
        .user_service
        .list(page, size)
        .await?;

    let response: Vec<UserResponse> = users.into_iter().map(Into::into).collect();

    Ok(Json(response))
}

// GET /api/users/:id
pub async fn get_user(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> Result<Json<UserResponse>> {
    let user = state
        .user_service
        .get_by_id(id)
        .await?
        .ok_or(AppError::NotFound {
            resource: "User".to_string(),
            id: id.to_string(),
        })?;

    Ok(Json(user.into()))
}

// POST /api/users
pub async fn create_user(
    State(state): State<AppState>,
    Json(input): Json<CreateUser>,
) -> Result<(StatusCode, Json<UserResponse>)> {
    let user = state.user_service.create(input).await?;

    Ok((StatusCode::CREATED, Json(user.into())))
}

// PUT /api/users/:id
pub async fn update_user(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
    Json(input): Json<UpdateUser>,
) -> Result<Json<UserResponse>> {
    let user = state.user_service.update(id, input).await?;

    Ok(Json(user.into()))
}

// DELETE /api/users/:id
pub async fn delete_user(
    State(state): State<AppState>,
    Path(id): Path<Uuid>,
) -> Result<StatusCode> {
    state.user_service.delete(id).await?;

    Ok(StatusCode::NO_CONTENT)
}
```

### Trait 定义

```rust
// ✅ 定义清晰的 trait
use async_trait::async_trait;
use uuid::Uuid;

use crate::errors::Result;
use crate::models::user::{User, CreateUser, UpdateUser};

#[async_trait]
pub trait UserRepository: Send + Sync {
    async fn find_by_id(&self, id: Uuid) -> Result<Option<User>>;
    async fn find_by_username(&self, username: &str) -> Result<Option<User>>;
    async fn find_by_email(&self, email: &str) -> Result<Option<User>>;
    async fn find_all(&self, page: i64, size: i64) -> Result<Vec<User>>;
    async fn create(&self, input: &CreateUser) -> Result<User>;
    async fn update(&self, id: Uuid, input: &UpdateUser) -> Result<User>;
    async fn delete(&self, id: Uuid) -> Result<()>;
}

// ✅ 实现 trait
pub struct PostgresUserRepository {
    pool: sqlx::PgPool,
}

impl PostgresUserRepository {
    pub fn new(pool: sqlx::PgPool) -> Self {
        Self { pool }
    }
}

#[async_trait]
impl UserRepository for PostgresUserRepository {
    async fn find_by_id(&self, id: Uuid) -> Result<Option<User>> {
        let user = sqlx::query_as!(
            User,
            r#"
            SELECT id, username, email, created_at, updated_at
            FROM users
            WHERE id = $1
            "#,
            id
        )
        .fetch_optional(&self.pool)
        .await?;

        Ok(user)
    }

    // ... 其他方法实现
}
```

### 泛型和约束

```rust
// ✅ 使用泛型约束
use std::fmt::Display;

pub fn print_item<T>(item: &T)
where
    T: Display + Debug,
{
    println!("Item: {}", item);
    println!("Debug: {:?}", item);
}

// ✅ 使用 trait 对象
pub trait Validator<T> {
    fn validate(&self, input: &T) -> Result<(), ValidationError>;
}

pub struct UserValidator;

impl Validator<CreateUser> for UserValidator {
    fn validate(&self, input: &CreateUser) -> Result<(), ValidationError> {
        if input.username.len() < 3 {
            return Err(ValidationError::new("Username too short"));
        }

        if !input.email.contains('@') {
            return Err(ValidationError::new("Invalid email"));
        }

        Ok(())
    }
}

// ✅ 服务层使用泛型
pub struct UserService<R: UserRepository> {
    repository: R,
}

impl<R: UserRepository> UserService<R> {
    pub fn new(repository: R) -> Self {
        Self { repository }
    }

    pub async fn get_by_id(&self, id: Uuid) -> Result<User> {
        self.repository
            .find_by_id(id)
            .await?
            .ok_or(AppError::NotFound {
                resource: "User".to_string(),
                id: id.to_string(),
            })
    }
}
```

### 序列化/反序列化

```rust
// ✅ 使用 serde
use serde::{Deserialize, Serialize};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct User {
    pub id: Uuid,
    pub username: String,
    #[serde(skip_serializing)]  // 序列化时跳过
    pub password_hash: String,
    pub email: String,
    #[serde(with = "chrono::serde::ts_seconds")]
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Deserialize)]
pub struct CreateUser {
    #[serde(deserialize_with = "validate_username")]
    pub username: String,
    pub email: String,
    #[serde(deserialize_with = "validate_password")]
    pub password: String,
}

// ✅ 自定义验证
fn validate_username<'de, D>(deserializer: D) -> std::result::Result<String, D::Error>
where
    D: serde::Deserializer<'de>,
{
    let s = String::deserialize(deserializer)?;

    if s.len() < 3 {
        return Err(serde::de::Error::custom("Username must be at least 3 characters"));
    }

    if !s.chars().all(|c| c.is_alphanumeric() || c == '_') {
        return Err(serde::de::Error::custom(
            "Username can only contain alphanumeric characters and underscores",
        ));
    }

    Ok(s)
}
```

### 异步编程

```rust
// ✅ 使用 tokio
use tokio::task;

// 并发执行多个任务
pub async fn fetch_multiple_resources(ids: Vec<Uuid>) -> Result<Vec<User>> {
    let handles: Vec<_> = ids
        .into_iter()
        .map(|id| {
            let repo = self.repository.clone();
            task::spawn(async move { repo.find_by_id(id).await })
        })
        .collect();

    let mut users = Vec::new();

    for handle in handles {
        let result = handle.await??;
        if let Some(user) = result {
            users.push(user);
        }
    }

    Ok(users)
}

// ✅ 使用 select! 处理多个异步操作
use tokio::select;
use tokio::time::{sleep, Duration};

async fn fetch_with_timeout(url: &str) -> Result<String> {
    select! {
        result = reqwest::get(url) => {
            let response = result?;
            Ok(response.text().await?)
        }
        _ = sleep(Duration::from_secs(10)) => {
            Err(AppError::Timeout("Request timed out".to_string()))
        }
    }
}
```

## 性能优化

```rust
// ✅ 使用 Cow 避免不必要的克隆
use std::borrow::Cow;

fn process_name(name: Cow<str>) -> String {
    if name.contains("test") {
        name.to_uppercase()  // 创建新 String
    } else {
        name.into_owned()    // 如果是 Owned，直接返回
    }
}

// ✅ 使用迭代器适配器
fn process_items(items: Vec<Item>) -> Vec<ProcessedItem> {
    items
        .into_iter()
        .filter(|item| item.is_valid())
        .map(|item| item.process())
        .collect()
}

// ✅ 使用 SmallVec 避免小集合的堆分配
use smallvec::SmallVec;

fn get_tags() -> SmallVec<[String; 4]> {
    let mut tags = SmallVec::new();
    tags.push("tag1".to_string());
    tags.push("tag2".to_string());
    tags
}
```

## 常见陷阱

### ❌ 避免
```rust
// ❌ 不必要的克隆
fn process(user: &User) -> String {
    let name = user.name.clone();  // 如果只需要借用，不需要克隆
    format!("Hello, {}", name)
}

// ❌ 滥用 unwrap
fn get_user(id: Uuid) -> User {
    repository.find_by_id(id).unwrap()  // 可能 panic
}

// ❌ 使用 String 而不是 &str
fn greet(name: String) {  // 如果只需要读取，使用 &str
    println!("Hello, {}", name);
}
```

### ✅ 推荐
```rust
// ✅ 使用借用
fn process(user: &User) -> String {
    format!("Hello, {}", user.name)
}

// ✅ 使用 ? 传播错误
fn get_user(id: Uuid) -> Result<User> {
    repository
        .find_by_id(id)?
        .ok_or(AppError::NotFound { /* ... */ })
}

// ✅ 使用 &str 参数
fn greet(name: &str) {
    println!("Hello, {}", name);
}
```

## 依赖推荐

- **Web 框架**: Axum / Actix-web
- **异步运行时**: Tokio
- **数据库**: SQLx / Diesel / SeaORM
- **序列化**: Serde
- **错误处理**: thiserror + anyhow
- **日志**: tracing + tracing-subscriber
- **配置**: config-rs
- **测试**: 内置测试 + tokio::test
- **代码规范**: clippy + rustfmt

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- Rust 版本：
- Web 框架：
- 数据库：
- 异步运行时：
```
