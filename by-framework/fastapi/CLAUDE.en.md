# CLAUDE.md - FastAPI Project Conventions

## Build & Run

- Dependency management: `uv` or `poetry` — lock file must be committed to the repository
- Start dev server: `uvicorn app.main:app --reload --port 8000`
- Run tests: `pytest -xvs`; Lint: `ruff check . && ruff format --check .`
- Python version: 3.11+, fully leverage type annotations

## Project Structure

- `app/main.py` - Entry point, registers routers and middleware
- `app/routers/` - Routes split by business domain (e.g., `users.py`, `orders.py`)
- `app/models/` - SQLAlchemy ORM models; `app/schemas/` - Pydantic v2 schemas
- `app/services/` - Business logic layer; `app/dependencies.py` - Shared dependency injection
- `alembic/` - Database migration scripts

## Async Model

- Route functions default to `async def`; use `def` only when calling synchronous blocking libraries
- Database: `sqlalchemy[asyncio]` + `asyncpg` — calling synchronous blocking I/O from async code is forbidden
- Background tasks: use `BackgroundTasks` or Celery/ARQ — never perform long-running operations inside request handlers

## Pydantic v2 Conventions

- Request bodies and responses must use Pydantic models — returning bare `dict` is forbidden
- Use `ConfigDict(from_attributes=True)` instead of the legacy `orm_mode`
- Field validation via `Field(ge=0, max_length=100)` and `@field_validator`

## Database & Migrations

- SQLAlchemy 2.0 style: use `Mapped[]` type annotations and `mapped_column()`
- Database sessions via dependency injection: `async def get_db() -> AsyncGenerator[AsyncSession]`
- All schema changes must go through Alembic: `alembic revision --autogenerate -m "description"`

## Dependency Injection

- Common dependencies (auth, pagination, DB session) defined in `dependencies.py`
- Use `Annotated[User, Depends(get_current_user)]` to simplify type declarations
- Use the yield pattern for resource lifecycle management; override with `app.dependency_overrides` in tests

## Router Organization

- Each router uses `APIRouter(prefix="/resource-name", tags=["tag"])`, registered centrally in `main.py`
- RESTful naming: `list_users`, `get_user`, `create_user`, `delete_user`

## Testing Conventions

- Use `httpx.AsyncClient` + `pytest-asyncio` for API testing
- Test database: SQLite in-memory mode or Testcontainers; fixtures manage data with per-test rollback
