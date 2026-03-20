# CLAUDE.md - FastAPI 项目规范

## 构建与运行

- 依赖管理：`uv` 或 `poetry`，锁文件必须提交到仓库
- 启动开发服务器：`uvicorn app.main:app --reload --port 8000`
- 运行测试：`pytest -xvs`，Lint 检查：`ruff check . && ruff format --check .`
- Python 版本：3.11+，充分利用类型标注

## 项目结构

- `app/main.py` - 入口，注册 router 和 middleware
- `app/routers/` - 按业务域拆分路由（如 `users.py`、`orders.py`）
- `app/models/` - SQLAlchemy ORM 模型，`app/schemas/` - Pydantic v2 模型
- `app/services/` - 业务逻辑层，`app/dependencies.py` - 公共依赖注入
- `alembic/` - 数据库迁移脚本

## 异步模式

- 路由函数默认 `async def`，仅调用同步阻塞库时用 `def`
- 数据库用 `sqlalchemy[asyncio]` + `asyncpg`，禁止 async 中调同步阻塞 I/O
- 后台任务用 `BackgroundTasks` 或 Celery/ARQ，禁止在请求中做耗时操作

## Pydantic v2 规范

- 请求体和响应体必须定义 Pydantic model，禁止裸 `dict` 返回
- 使用 `ConfigDict(from_attributes=True)` 替代旧版 `orm_mode`
- 字段校验用 `Field(ge=0, max_length=100)` 和 `@field_validator`

## 数据库与迁移

- SQLAlchemy 2.0 风格：使用 `Mapped[]` 类型标注和 `mapped_column()`
- 数据库会话通过依赖注入：`async def get_db() -> AsyncGenerator[AsyncSession]`
- 表结构变更必须通过 Alembic：`alembic revision --autogenerate -m "描述"`

## 依赖注入

- 公共依赖（认证、分页、DB 会话）定义在 `dependencies.py`
- 使用 `Annotated[User, Depends(get_current_user)]` 简化类型声明
- yield 模式管理资源生命周期，测试中用 `app.dependency_overrides` 替换

## Router 组织

- 每个 router 用 `APIRouter(prefix="/资源名", tags=["标签"])`，在 `main.py` 统一注册
- RESTful 命名：`list_users`、`get_user`、`create_user`、`delete_user`

## 测试规范

- 使用 `httpx.AsyncClient` + `pytest-asyncio` 做接口测试
- 测试数据库用 SQLite 内存模式或 Testcontainers，Fixture 管理数据并按用例回滚
