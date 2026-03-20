# FastAPI 项目规范

## 异步模式

- 所有 I/O 操作（数据库、HTTP、文件）必须用 `async def` + `await`
- CPU 密集型任务用 `run_in_executor` 推到线程池，不要阻塞事件循环
- 数据库用 `sqlalchemy[asyncio]` + `asyncpg`，禁止同步 driver
- HTTP 客户端用 `httpx.AsyncClient`，在 lifespan 中创建共享实例：
  ```python
  async def lifespan(app: FastAPI):
      app.state.http = httpx.AsyncClient()
      yield
      await app.state.http.aclose()
  ```

## Pydantic Models

- 请求/响应模型分开定义：`UserCreate`（入参）、`UserResponse`（出参）、`UserInDB`（内部）
- 所有 model 继承自 `BaseModel`，启用 `model_config = ConfigDict(strict=True)`
- 敏感字段（password）只出现在入参 model，响应 model 用 `model_fields` 显式列出
- 复用字段用 mixin 或基类，不要复制粘贴
- 枚举值用 `StrEnum`，自动序列化为字符串

## Router 组织

- 按领域拆分：`routers/users.py`、`routers/orders.py`，每个文件一个 `APIRouter`
- 前缀和 tags 在 router 定义处声明：`router = APIRouter(prefix="/users", tags=["users"])`
- `main.py` 只做 `app.include_router()`，不写业务逻辑
- 路由函数只做参数校验和调用 service，业务逻辑在 `services/` 层

## 依赖注入

- 数据库 session：`Depends(get_db)` 返回 `AsyncSession`，用 `async with` 管理生命周期
- 当前用户：`Depends(get_current_user)` 返回 `User` model，鉴权失败抛 `HTTPException(401)`
- 共享依赖写在 `deps.py`，不要在 router 文件内重复定义
- 嵌套依赖保持 <= 3 层，过深则重构为 service class

## Alembic 数据库迁移

- 初始化：`alembic init alembic`，配置 `env.py` 使用 async engine
- 生成迁移：`alembic revision --autogenerate -m "add_users_table"`
- 执行迁移：`alembic upgrade head`，回滚：`alembic downgrade -1`
- 每个迁移文件必须有 `upgrade()` 和 `downgrade()`，downgrade 不能为空
- 迁移文件提交到 git，CI 中执行 `alembic check` 确保无遗漏

## 常见陷阱

- 不要在 `async def` 路由中调用同步阻塞函数（如 `requests.get`、`time.sleep`）
- `BackgroundTasks` 适合轻量任务，重任务用 Celery 或 arq
- 避免在 Depends 中做大量计算，它每次请求都会执行
- 响应 model 必须设置 `response_model`，防止意外泄露内部字段
