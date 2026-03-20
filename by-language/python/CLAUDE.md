# Python 项目规范

## 环境与依赖

- Python >= 3.11，使用 `uv` 管理虚拟环境和依赖（优先），备选 poetry
- 激活环境：`uv venv && source .venv/bin/activate`
- 安装依赖：`uv pip install -e ".[dev]"`，不要用裸 `pip install`
- 锁文件 `uv.lock` 必须提交，手动修改 pyproject.toml 后执行 `uv lock`

## 类型标注（严格模式）

- 所有公开函数必须有完整的参数和返回值类型标注
- 禁止使用 `Any`，除非有注释说明原因
- 容器类型用 `list[str]` 而非 `List[str]`（Python 3.9+ 内置泛型）
- 可选参数用 `str | None = None`，不用 `Optional[str]`
- 运行 `mypy --strict src/` 零错误

## 测试

- 框架：pytest，配置写在 pyproject.toml 的 `[tool.pytest.ini_options]`
- 运行全部测试：`pytest -x -q`，单文件：`pytest tests/test_xxx.py -v`
- 测试文件与源码目录镜像：`src/app/services/user.py` -> `tests/services/test_user.py`
- fixture 放 `tests/conftest.py`，按模块拆分避免单文件过大
- 覆盖率：`pytest --cov=src --cov-report=term-missing`，目标 >= 80%

## Linting 与格式化

- 唯一工具：ruff（替代 flake8 + isort + black）
- 格式化：`ruff format src/ tests/`
- 检查：`ruff check src/ tests/ --fix`
- 提交前必须通过：`ruff check && ruff format --check && mypy --strict src/`

## 项目结构

```
src/
  app/
    __init__.py
    models/       # 数据模型（dataclass / Pydantic）
    services/     # 业务逻辑，不依赖框架
    utils/        # 纯工具函数，无副作用
tests/
pyproject.toml
```

## 常见陷阱

- 默认可变参数：写 `def f(items: list[int] | None = None)` 而非 `def f(items: list[int] = [])`
- 循环导入：services 层不要导入 routes/views，用依赖注入解耦
- 路径处理：用 `pathlib.Path`，不要拼接字符串
- 异常处理：捕获具体异常类型，禁止裸 `except:` 或 `except Exception`
- 字符串拼接：f-string 优先，多行用 `textwrap.dedent` 或括号续行
