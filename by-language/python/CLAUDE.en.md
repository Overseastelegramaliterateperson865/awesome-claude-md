# Python Project Conventions

## Environment & Dependencies

- Python >= 3.11, use `uv` to manage virtual environments and dependencies (preferred), fallback to poetry
- Activate environment: `uv venv && source .venv/bin/activate`
- Install dependencies: `uv pip install -e ".[dev]"` — never use bare `pip install`
- Lock file `uv.lock` must be committed; run `uv lock` after manually editing pyproject.toml

## Type Annotations (Strict Mode)

- All public functions must have complete parameter and return type annotations
- Do not use `Any` unless accompanied by a comment explaining why
- Use built-in generic types: `list[str]` instead of `List[str]` (Python 3.9+ built-in generics)
- Optional parameters: use `str | None = None`, not `Optional[str]`
- Run `mypy --strict src/` with zero errors

## Testing

- Framework: pytest, configured in `[tool.pytest.ini_options]` within pyproject.toml
- Run all tests: `pytest -x -q`; single file: `pytest tests/test_xxx.py -v`
- Test files mirror source directory structure: `src/app/services/user.py` -> `tests/services/test_user.py`
- Fixtures go in `tests/conftest.py`, split by module to avoid oversized files
- Coverage: `pytest --cov=src --cov-report=term-missing`, target >= 80%

## Linting & Formatting

- Single tool: ruff (replaces flake8 + isort + black)
- Format: `ruff format src/ tests/`
- Check: `ruff check src/ tests/ --fix`
- Must pass before commit: `ruff check && ruff format --check && mypy --strict src/`

## Project Structure

```
src/
  app/
    __init__.py
    models/       # Data models (dataclass / Pydantic)
    services/     # Business logic, framework-independent
    utils/        # Pure utility functions, no side effects
tests/
pyproject.toml
```

## Common Pitfalls

- Mutable default arguments: use `def f(items: list[int] | None = None)` not `def f(items: list[int] = [])`
- Circular imports: services layer must not import from routes/views — use dependency injection to decouple
- Path handling: use `pathlib.Path`, never concatenate strings
- Exception handling: catch specific exception types — bare `except:` or `except Exception` is forbidden
- String concatenation: prefer f-strings; for multi-line strings use `textwrap.dedent` or parenthesized continuation
