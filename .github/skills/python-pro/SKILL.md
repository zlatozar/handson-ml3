<!-- You can load it directly into the prompt with `@python-pro` or Copilot will load it based on topics in the description -->
---
name: python-pro
description: Use when building Python 3.13+ applications requiring type safety, robust error handling, or structured logging. Generates type-annotated Python code, configures mypy in strict mode, writes pytest test suites with fixtures and mocking, and validates code with ruff. Invoke for type hints, dataclasses, Protocols, dependency injection, pathlib file operations, structured logging, context managers, enums, and production-grade error handling.
license: MIT
metadata:
  author: Zlatozar Zhelyazkov
  version: "2.0.0"
  domain: language
  triggers: Python development, type hints, typing, Protocol, pytest, mypy, dataclasses, Python best practices, Pythonic code, pathlib, logging, ruff, error handling, context manager, enum, functools, itertools, collections
  role: specialist
  scope: implementation
  output-format: code
  related-skills: pandas-pro, numpy-pro
---

# Python Pro

Modern Python 3.13+ specialist focused on type-safe, production-ready code with strict mypy, ruff linting, and comprehensive pytest coverage.

## When to Use This Skill

- Writing type-safe Python with complete type coverage (mypy strict)
- Setting up pytest test suites with fixtures, parametrize, and mocking
- Creating Pythonic code with comprehensions, generators, context managers
- Structuring error handling with specific exception types and try-except for I/O
- Using `pathlib` for all file system operations (never `os.path`)
- Configuring structured `logging` (never `print()` for anything beyond quick debug)
- Implementing async/await patterns for I/O-bound operations
- Building packages with `uv` and proper project structure
- Performance optimization and profiling

## Core Workflow

1. **Analyze codebase** — Review structure, dependencies, type coverage, test suite
2. **Design interfaces** — Define Protocols, dataclasses, type aliases
3. **Implement** — Write Pythonic code with full type hints, `pathlib`, `logging`, and error handling
4. **Test** — Create comprehensive pytest suite with >90% coverage
5. **Validate** — Run `mypy --strict`, `ruff check`, `ruff format`
   - If mypy fails: fix type errors reported and re-run before proceeding
   - If tests fail: debug assertions, update fixtures, and iterate until green
   - If ruff reports issues: apply auto-fixes with `ruff check --fix`, then re-validate

## Reference Guide

Load detailed guidance based on context:

| Topic | Reference | Load When |
|-------|-----------|-----------|
| Type System | `references/type-system.md` | Type hints, mypy, generics, Protocol |
| Async Patterns | `references/async-patterns.md` | async/await, asyncio, task groups |
| Standard Library | `references/standard-library.md` | pathlib, dataclasses, functools, itertools, logging |
| Testing | `references/testing.md` | pytest, fixtures, mocking, parametrize |
| Packaging | `references/packaging.md` | uv, pip, pyproject.toml, distribution |

## Code Patterns

### Type-Annotated Function with Error Handling

```python
# ❌ AVOID: no types, bare except, os.path, print
import os

def read_config(path):
    try:
        with open(os.path.join("config", path)) as f:
            return f.read()
    except:
        print("Error reading config")
        return None

# ✅ USE: full types, specific exceptions, pathlib, logging
import logging
from pathlib import Path

logger = logging.getLogger(__name__)

def read_config(path: Path) -> dict[str, str]:
    """Read configuration from a file.

    Args:
        path: Path to the configuration file.

    Returns:
        Parsed key-value configuration entries.

    Raises:
        FileNotFoundError: If the config file does not exist.
        ValueError: If a line cannot be parsed.
    """
    try:
        config: dict[str, str] = {}
        with path.open(encoding="utf-8") as f:
            for line in f:
                key, _, value = line.partition("=")
                if not key.strip():
                    raise ValueError(f"Invalid config line: {line!r}")
                config[key.strip()] = value.strip()
        return config
    except FileNotFoundError:
        logger.exception("Config file not found: %s", path)
        raise
```

### Dataclass with Validation

```python
# ❌ AVOID: manual __init__, no validation
class AppConfig:
    def __init__(self, host, port, debug=False):
        self.host = host
        self.port = port
        self.debug = debug

# ✅ USE: dataclass with post_init validation and type hints
from dataclasses import dataclass, field

@dataclass
class AppConfig:
    """Application configuration with port validation."""

    host: str
    port: int
    debug: bool = False
    allowed_origins: list[str] = field(default_factory=list)

    def __post_init__(self) -> None:
        if not (1 <= self.port <= 65535):
            raise ValueError(f"Invalid port: {self.port}")
```

### Structured Logging (not print)

```python
# ❌ AVOID: print for operational messages
print(f"Processing user {user_id}")
print(f"Error: {e}")

# ✅ USE: structured logging with appropriate levels
import logging

logger = logging.getLogger(__name__)

def process_user(user_id: int) -> None:
    """Process a user by ID."""
    logger.info("Processing user %d", user_id)
    try:
        result = fetch_user(user_id)
        logger.debug("User data loaded: %s", result)
    except ConnectionError:
        logger.exception("Failed to process user %d", user_id)
        raise
```

### Pathlib over os.path

```python
# ❌ AVOID: os.path and string concatenation
import os
config_path = os.path.join(os.path.dirname(__file__), '..', 'config', 'settings.toml')
if os.path.exists(config_path):
    with open(config_path) as f:
        content = f.read()

# ✅ USE: pathlib for all file operations
from pathlib import Path

config_path = Path(__file__).parent.parent / "config" / "settings.toml"
if config_path.exists():
    content = config_path.read_text(encoding="utf-8")
```

### Context Manager for Resources

```python
# ❌ AVOID: manual resource cleanup
f = open("data.txt")
data = f.read()
f.close()  # may never run if exception occurs

# ✅ USE: context manager guarantees cleanup
from contextlib import contextmanager
from collections.abc import Iterator

@contextmanager
def managed_connection(url: str) -> Iterator[Connection]:
    """Acquire and release a database connection."""
    conn = create_connection(url)
    try:
        yield conn
    finally:
        conn.close()
```

### Async Pattern with TaskGroup

```python
# ❌ AVOID: sequential I/O
results = []
for url in urls:
    results.append(await fetch(url))

# ✅ USE: concurrent I/O with TaskGroup (Python 3.11+)
import asyncio

async def fetch_all(urls: list[str]) -> list[bytes]:
    """Fetch multiple URLs concurrently."""
    async with asyncio.TaskGroup() as tg:
        tasks = [tg.create_task(fetch(url)) for url in urls]
    return [t.result() for t in tasks]
```

### pytest Fixture and Parametrize

```python
import pytest
from pathlib import Path

@pytest.fixture
def config_file(tmp_path: Path) -> Path:
    """Create a temporary config file for testing."""
    cfg = tmp_path / "config.txt"
    cfg.write_text("host=localhost\nport=8080\n")
    return cfg

@pytest.mark.parametrize(
    ("port", "valid"),
    [
        (8080, True),
        (0, False),
        (99999, False),
    ],
    ids=["valid-port", "zero-port", "overflow-port"],
)
def test_app_config_port_validation(port: int, valid: bool) -> None:
    """Verify AppConfig rejects invalid ports."""
    if valid:
        AppConfig(host="localhost", port=port)
    else:
        with pytest.raises(ValueError, match="Invalid port"):
            AppConfig(host="localhost", port=port)
```

### mypy + ruff Configuration (pyproject.toml)

```toml
[tool.mypy]
python_version = "3.13"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[tool.ruff]
target-version = "py313"
line-length = 100

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "B",    # flake8-bugbear
    "C4",   # flake8-comprehensions
    "UP",   # pyupgrade
    "ARG",  # flake8-unused-arguments
    "PTH",  # flake8-use-pathlib
    "SIM",  # flake8-simplify
    "RUF",  # Ruff-specific rules
    "TID",  # flake8-tidy-imports
]
```

## Constraints

### MUST DO
- Type hints for all function signatures and class attributes
- Use `X | None` instead of `Optional[X]` (Python 3.10+)
- PEP 8 compliance enforced by `ruff format` and `ruff check`
- Comprehensive docstrings (Google style) on all public APIs
- Test coverage exceeding 90% with pytest
- Use `pathlib.Path` for all file system operations (never `os.path`)
- Use `logging` module for all operational messages (never `print()`)
- Structured `try-except` with specific exception types for I/O and data loading
- Dataclasses over manual `__init__` methods
- Context managers for resource handling
- Async/await for I/O-bound operations
- Constants or config objects instead of magic numbers

### MUST NOT DO
- Skip type annotations on public APIs
- Use mutable default arguments
- Use bare `except:` clauses — always catch specific exception types
- Ignore mypy errors in strict mode
- Use `os.path` — use `pathlib` instead (enforced by Ruff `PTH` rules)
- Use `print()` for logging — use `logging` module
- Hardcode secrets or configuration values
- Mix sync and async code improperly
- Use deprecated stdlib modules or patterns
- Use single-letter variable names (except `i`, `j`, `k` in trivial loops)

## Output Templates

When implementing Python features, provide:
1. Module file with complete type hints and Google-style docstrings
2. Test file with pytest fixtures, parametrize, and descriptive test IDs
3. Type checking confirmation (mypy --strict passes)
4. Brief explanation of Pythonic patterns used

