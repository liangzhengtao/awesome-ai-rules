# Python FastAPI AI 编程规则
> Last updated: 2026 | Targets: Python 3.12+ / FastAPI 0.110+ / Pydantic V2

## 核心原则

- 使用 Python 3.10+ 特性
- 充分利用类型提示（Type Hints）
- 遵循异步优先原则
- 使用 Pydantic 进行数据验证
- 编写清晰的 API 文档

## 代码风格

### 命名规范
- 模块/文件名：snake_case：`user_router.py`, `auth_service.py`
- 类名：PascalCase：`UserService`, `CreateUserRequest`
- 函数/变量：snake_case：`get_user`, `create_item`
- 常量：UPPER_SNAKE_CASE：`DATABASE_URL`, `SECRET_KEY`
- 私有成员：下划线前缀：`_internal_method`

### 文件结构
```
app/
├── api/
│   └── v1/
│       ├── endpoints/
│       │   ├── users.py
│       │   ├── items.py
│       │   └── auth.py
│       └── router.py
├── core/
│   ├── config.py
│   ├── security.py
│   └── dependencies.py
├── models/
│   ├── user.py
│   └── item.py
├── schemas/
│   ├── user.py
│   └── item.py
├── services/
│   ├── user_service.py
│   └── item_service.py
├── db/
│   ├── session.py
│   └── base_class.py
├── utils/
│   └── helpers.py
└── main.py
```

## 最佳实践

### 路由定义

```python
# ✅ api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.dependencies import get_db, get_current_user
from app.schemas.user import UserCreate, UserResponse, UserUpdate
from app.services.user_service import UserService

router = APIRouter(prefix="/users", tags=["users"])


@router.post(
    "/",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    summary="创建用户",
    description="创建新用户账户",
)
async def create_user(
    user_in: UserCreate,
    db: AsyncSession = Depends(get_db),
) -> UserResponse:
    """
    创建新用户

    - **username**: 用户名（唯一）
    - **email**: 邮箱地址（唯一）
    - **password**: 密码（至少8位）
    """
    service = UserService(db)

    # 检查用户名是否已存在
    if await service.get_by_username(user_in.username):
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Username already registered",
        )

    user = await service.create(user_in)
    return user


@router.get(
    "/me",
    response_model=UserResponse,
    summary="获取当前用户",
)
async def get_current_user_info(
    current_user: User = Depends(get_current_user),
) -> UserResponse:
    """获取当前登录用户的信息"""
    return current_user


@router.get(
    "/{user_id}",
    response_model=UserResponse,
    summary="获取用户详情",
)
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
) -> UserResponse:
    """根据用户ID获取用户详情"""
    service = UserService(db)
    user = await service.get_by_id(user_id)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    return user
```

### Pydantic 模型

```python
# ✅ schemas/user.py
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, EmailStr, Field, field_validator


class UserBase(BaseModel):
    """用户基础模型"""
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        pattern=r"^[a-zA-Z0-9_]+$",
        description="用户名，只能包含字母、数字和下划线",
    )
    email: EmailStr = Field(..., description="邮箱地址")


class UserCreate(UserBase):
    """创建用户请求模型"""
    password: str = Field(
        ...,
        min_length=8,
        max_length=100,
        description="密码，至少8位",
    )

    @field_validator("password")
    @classmethod
    def validate_password(cls, v: str) -> str:
        if not any(c.isupper() for c in v):
            raise ValueError("Password must contain at least one uppercase letter")
        if not any(c.isdigit() for c in v):
            raise ValueError("Password must contain at least one digit")
        return v


class UserUpdate(BaseModel):
    """更新用户请求模型"""
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    email: Optional[EmailStr] = None
    avatar: Optional[str] = None


class UserResponse(UserBase):
    """用户响应模型"""
    id: int
    is_active: bool
    created_at: datetime
    updated_at: datetime

    class Config:
        from_attributes = True  # 支持 ORM 模型转换


class UserListResponse(BaseModel):
    """用户列表响应"""
    items: list[UserResponse]
    total: int
    page: int
    size: int
```

### 服务层

```python
# ✅ services/user_service.py
from typing import Optional
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate
from app.core.security import get_password_hash


class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_by_id(self, user_id: int) -> Optional[User]:
        """根据ID获取用户"""
        result = await self.db.execute(
            select(User).where(User.id == user_id)
        )
        return result.scalar_one_or_none()

    async def get_by_username(self, username: str) -> Optional[User]:
        """根据用户名获取用户"""
        result = await self.db.execute(
            select(User).where(User.username == username)
        )
        return result.scalar_one_or_none()

    async def get_by_email(self, email: str) -> Optional[User]:
        """根据邮箱获取用户"""
        result = await self.db.execute(
            select(User).where(User.email == email)
        )
        return result.scalar_one_or_none()

    async def create(self, user_in: UserCreate) -> User:
        """创建用户"""
        user = User(
            username=user_in.username,
            email=user_in.email,
            hashed_password=get_password_hash(user_in.password),
        )
        self.db.add(user)
        await self.db.commit()
        await self.db.refresh(user)
        return user

    async def update(self, user: User, user_in: UserUpdate) -> User:
        """更新用户"""
        update_data = user_in.model_dump(exclude_unset=True)

        for field, value in update_data.items():
            setattr(user, field, value)

        await self.db.commit()
        await self.db.refresh(user)
        return user

    async def delete(self, user: User) -> None:
        """删除用户"""
        await self.db.delete(user)
        await self.db.commit()
```

### 依赖注入

```python
# ✅ core/dependencies.py
from typing import AsyncGenerator

from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.config import settings
from app.db.session import async_session_factory
from app.models.user import User
from app.services.user_service import UserService

security = HTTPBearer()


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    """获取数据库会话"""
    async with async_session_factory() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()


async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db),
) -> User:
    """获取当前认证用户"""
    token = credentials.credentials
    payload = decode_token(token)

    if payload is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid authentication credentials",
        )

    user_service = UserService(db)
    user = await user_service.get_by_id(payload.get("sub"))

    if user is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    if not user.is_active:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="Inactive user",
        )

    return user


class PermissionChecker:
    """权限检查器"""

    def __init__(self, required_permissions: list[str]):
        self.required_permissions = required_permissions

    async def __call__(
        self,
        current_user: User = Depends(get_current_user),
    ) -> User:
        user_permissions = get_user_permissions(current_user)

        for permission in self.required_permissions:
            if permission not in user_permissions:
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="Not enough permissions",
                )

        return current_user
```

### 中间件

```python
# ✅ core/middleware.py
import time
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware


class TimingMiddleware(BaseHTTPMiddleware):
    """请求计时中间件"""

    async def dispatch(self, request: Request, call_next):
        start_time = time.time()

        response = await call_next(request)

        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)

        return response


class RequestLoggingMiddleware(BaseHTTPMiddleware):
    """请求日志中间件"""

    async def dispatch(self, request: Request, call_next):
        logger.info(
            f"Request: {request.method} {request.url}",
            extra={
                "method": request.method,
                "url": str(request.url),
                "client": request.client.host if request.client else None,
            },
        )

        response = await call_next(request)

        logger.info(
            f"Response: {response.status_code}",
            extra={"status_code": response.status_code},
        )

        return response
```

### 错误处理

```python
# ✅ core/exceptions.py
from fastapi import HTTPException, status


class AppException(HTTPException):
    """应用自定义异常"""

    def __init__(
        self,
        status_code: int,
        detail: str,
        error_code: str | None = None,
    ):
        super().__init__(status_code=status_code, detail=detail)
        self.error_code = error_code


class NotFoundException(AppException):
    """资源不存在异常"""

    def __init__(self, resource: str, identifier: str | int):
        super().__init__(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"{resource} with id '{identifier}' not found",
            error_code="NOT_FOUND",
        )


class DuplicateException(AppException):
    """重复资源异常"""

    def __init__(self, resource: str, field: str, value: str):
        super().__init__(
            status_code=status.HTTP_409_CONFLICT,
            detail=f"{resource} with {field} '{value}' already exists",
            error_code="DUPLICATE",
        )


class PermissionDeniedException(AppException):
    """权限不足异常"""

    def __init__(self):
        super().__init__(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Permission denied",
            error_code="PERMISSION_DENIED",
        )
```

## 测试

```python
# ✅ tests/test_users.py
import pytest
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import AsyncSession

from app.main import app
from app.models.user import User


@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac


@pytest.fixture
async def test_user(db: AsyncSession) -> User:
    user = User(
        username="testuser",
        email="test@example.com",
        hashed_password="hashed_password",
    )
    db.add(user)
    await db.commit()
    await db.refresh(user)
    return user


@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    response = await client.post(
        "/api/v1/users/",
        json={
            "username": "newuser",
            "email": "new@example.com",
            "password": "StrongPass123",
        },
    )
    assert response.status_code == 201
    data = response.json()
    assert data["username"] == "newuser"
    assert data["email"] == "new@example.com"
    assert "id" in data


@pytest.mark.asyncio
async def test_create_user_duplicate_username(
    client: AsyncClient,
    test_user: User,
):
    response = await client.post(
        "/api/v1/users/",
        json={
            "username": test_user.username,
            "email": "another@example.com",
            "password": "StrongPass123",
        },
    )
    assert response.status_code == 400


@pytest.mark.asyncio
async def test_get_current_user(
    client: AsyncClient,
    test_user: User,
    auth_headers: dict,
):
    response = await client.get(
        "/api/v1/users/me",
        headers=auth_headers,
    )
    assert response.status_code == 200
    data = response.json()
    assert data["id"] == test_user.id
```

## 常见陷阱

### ❌ 避免
```python
# ❌ 在异步函数中使用同步操作
async def get_user():
    time.sleep(1)  # 阻塞事件循环！
    return user

# ❌ 忘记使用 await
async def get_user():
    result = db.execute(select(User))  # 缺少 await
    return result

# ❌ 在路由中直接访问数据库
@router.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    # 业务逻辑应该在 service 层
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()
```

### ✅ 推荐
```python
# ✅ 使用异步库和操作
async def get_user():
    await asyncio.sleep(1)  # 异步等待
    return user

# ✅ 正确使用 await
async def get_user():
    result = await db.execute(select(User))
    return result

# ✅ 使用 service 层处理业务逻辑
@router.get("/users/{user_id}")
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
):
    service = UserService(db)
    user = await service.get_by_id(user_id)

    if not user:
        raise NotFoundException("User", user_id)

    return user
```

## 依赖推荐

- **Web 框架**: FastAPI
- **ASGI 服务器**: Uvicorn / Hypercorn
- **数据库 ORM**: SQLAlchemy 2.0 (async) / Tortoise ORM
- **数据验证**: Pydantic V2
- **数据库迁移**: Alembic
- **认证**: python-jose / PyJWT
- **测试**: Pytest + HTTPX
- **代码规范**: Ruff / Black + isort
- **包管理**: Poetry / PDM

## 项目特定规则

> 💡 在下方添加你的项目特定规则

```markdown
## 项目配置

- 项目名称：
- Python 版本：
- 数据库：
- 缓存：
- 部署方式：
```
