# FastAPI Project Conventions

## Async Model

- All I/O operations (database, HTTP, file) must use `async def` + `await`
- CPU-intensive tasks should be offloaded to a thread pool via `run_in_executor` — never block the event loop
- Database driver: `sqlalchemy[asyncio]` + `asyncpg` — synchronous drivers are forbidden
- HTTP client: use `httpx.AsyncClient`, create a shared instance in the lifespan handler:
  ```python
  async def lifespan(app: FastAPI):
      app.state.http = httpx.AsyncClient()
      yield
      await app.state.http.aclose()
  ```

## Pydantic Models

- Define separate request and response models: `UserCreate` (input), `UserResponse` (output), `UserInDB` (internal)
- All models inherit from `BaseModel` with `model_config = ConfigDict(strict=True)` enabled
- Sensitive fields (e.g., password) only appear in input models — response models must explicitly list fields via `model_fields`
- Reuse common fields through mixins or base classes — no copy-paste duplication
- Enum values should use `StrEnum` for automatic string serialization

## Router Organization

- Split by domain: `routers/users.py`, `routers/orders.py` — one `APIRouter` per file
- Declare prefix and tags at the router definition: `router = APIRouter(prefix="/users", tags=["users"])`
- `main.py` should only call `app.include_router()` — no business logic here
- Route functions handle parameter validation and call services only — business logic belongs in the `services/` layer

## Dependency Injection

- Database session: `Depends(get_db)` returns an `AsyncSession`, managed with `async with` for lifecycle control
- Current user: `Depends(get_current_user)` returns a `User` model; raises `HTTPException(401)` on auth failure
- Shared dependencies go in `deps.py` — do not redefine them in router files
- Keep dependency nesting <= 3 levels deep; refactor into service classes if deeper

## Alembic Database Migrations

- Initialize: `alembic init alembic`, configure `env.py` to use an async engine
- Generate migration: `alembic revision --autogenerate -m "add_users_table"`
- Apply migration: `alembic upgrade head`; rollback: `alembic downgrade -1`
- Every migration file must have both `upgrade()` and `downgrade()` — downgrade must not be empty
- Migration files must be committed to git; run `alembic check` in CI to catch missing migrations

## Common Pitfalls

- Never call synchronous blocking functions (e.g., `requests.get`, `time.sleep`) inside `async def` routes
- `BackgroundTasks` is suitable for lightweight tasks — use Celery or arq for heavy workloads
- Avoid expensive computation inside `Depends` — it runs on every request
- Always set `response_model` on routes to prevent accidental exposure of internal fields
