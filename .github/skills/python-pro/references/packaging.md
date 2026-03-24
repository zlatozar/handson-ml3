# Python Packaging and Project Setup

## Project Structure

```
myproject/
├── pyproject.toml          # Project metadata and dependencies
├── README.md               # Project description
├── .gitignore             # Git ignore patterns
├── .python-version        # Python version for pyenv
├── src/
│   └── myproject/
│       ├── __init__.py    # Package initialization
│       ├── py.typed       # PEP 561 type marker
│       ├── core.py        # Core functionality
│       └── utils.py       # Utilities
├── tests/
│   ├── __init__.py
│   ├── conftest.py        # Pytest configuration
│   └── test_core.py       # Tests
└── docs/
    └── index.md           # Documentation
```

## Pyproject.toml Configuration

```toml
[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[project]
name = "myproject"
version = "0.1.0"
description = "A Python project"
readme = "README.md"
requires-python = ">=3.13"
license = {text = "MIT"}
authors = [
    {name = "Your Name", email = "you@example.com"}
]
keywords = ["python", "package"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3.12",
    "Programming Language :: Python :: 3.13",
    "Typing :: Typed",
]

dependencies = [
    "requests>=2.31.0",
    "pydantic>=2.5.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "mypy>=1.7.0",
    "ruff>=0.1.6",
]
docs = [
    "mkdocs>=1.5.0",
    "mkdocs-material>=9.4.0",
]

[project.scripts]
myproject = "myproject.cli:main"

[project.urls]
Homepage = "https://github.com/username/myproject"
Documentation = "https://myproject.readthedocs.io"
Repository = "https://github.com/username/myproject"
Changelog = "https://github.com/username/myproject/blob/main/CHANGELOG.md"

# Tool configurations
[tool.ruff]
line-length = 100
target-version = "py313"

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

[tool.ruff.lint.per-file-ignores]
"__init__.py" = ["F401"]  # Ignore unused imports in __init__.py

[tool.mypy]
python_version = "3.13"
strict = true
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = true

[[tool.mypy.overrides]]
module = "third_party.*"
ignore_missing_imports = true

[tool.pytest.ini_options]
minversion = "7.0"
addopts = [
    "-ra",
    "--strict-markers",
    "--strict-config",
    "--cov=myproject",
    "--cov-report=term-missing",
    "--cov-report=html",
]
testpaths = ["tests"]
pythonpath = ["src"]

[tool.coverage.run]
source = ["src"]
branch = true

[tool.coverage.report]
exclude_lines = [
    "pragma: no cover",
    "def __repr__",
    "raise AssertionError",
    "raise NotImplementedError",
    "if __name__ == .__main__.:",
    "if TYPE_CHECKING:",
]
```

## uv Project Management

`uv` is a fast Python package manager. Use it instead of pip/Poetry for dependency management.

```bash
# Initialize a new project
uv init myproject

# Add dependencies
uv add requests
uv add pydantic

# Add dev dependencies
uv add --dev pytest pytest-cov mypy ruff

# Install all dependencies (creates uv.lock)
uv sync

# Run commands in the managed environment
uv run pytest
uv run mypy --strict src/
uv run ruff check src/
uv run ruff format src/

# Update dependencies
uv lock --upgrade

# Pin exact versions (uv.lock is auto-generated)
# uv.lock should be committed to version control for applications
```

## Virtual Environments

```bash
# ✅ Preferred: uv manages venvs automatically
uv sync  # creates .venv and installs deps

# Using venv (built-in fallback)
python -m venv .venv
source .venv/bin/activate  # Linux/Mac

# Install in editable mode
pip install -e .
pip install -e ".[dev]"    # With optional dependencies


# Using pyenv for Python version management
pyenv install 3.11.6
pyenv local 3.11.6         # Set for current directory
echo "3.11.6" > .python-version
```

## Package __init__.py

```python
# src/myproject/__init__.py
"""MyProject - A Python package."""

from myproject.core import main_function, CoreClass
from myproject.utils import helper_function

__version__ = "0.1.0"
__all__ = ["main_function", "CoreClass", "helper_function"]

# Package-level configuration
import logging

logger = logging.getLogger(__name__)
logger.addHandler(logging.NullHandler())
```

## Type Stub Files (py.typed)

```python
# src/myproject/py.typed
# Empty file indicates package includes type hints

# src/myproject/__init__.pyi (optional stub file)
from typing import Any

__version__: str

def main_function(arg: str) -> dict[str, Any]: ...

class CoreClass:
    def __init__(self, name: str) -> None: ...
    def process(self) -> str: ...
```

## CLI Entry Points

```python
# src/myproject/cli.py
import sys
from typing import NoReturn

def main() -> NoReturn:
    """Main CLI entry point."""
    print("MyProject CLI")
    sys.exit(0)

if __name__ == "__main__":
    main()
```

## Requirements Files

```bash
# requirements.txt - Production dependencies
requests>=2.31.0,<3.0.0
pydantic>=2.5.0,<3.0.0

# requirements-dev.txt - Development dependencies
-r requirements.txt
pytest>=7.4.0
pytest-cov>=4.1.0
mypy>=1.7.0
ruff>=0.1.6

# Export from uv lock file
uv export --format requirements-txt > requirements.txt
uv export --format requirements-txt --all-groups > requirements-dev.txt
```

## Building and Distribution

```bash
# Build package
python -m build

# Check package
twine check dist/*

# Upload to PyPI
twine upload dist/*

# Upload to Test PyPI
twine upload --repository testpypi dist/*

# Install from Test PyPI
pip install --index-url https://test.pypi.org/simple/ myproject
```

## Setuptools Configuration (Legacy)

```python
# setup.py (if not using pyproject.toml)
from setuptools import setup, find_packages

setup(
    name="myproject",
    version="0.1.0",
    packages=find_packages(where="src"),
    package_dir={"": "src"},
    python_requires=">=3.11",
    install_requires=[
        "requests>=2.31.0",
        "pydantic>=2.5.0",
    ],
    extras_require={
        "dev": [
            "pytest>=7.4.0",
            "mypy>=1.7.0",
        ],
    },
    entry_points={
        "console_scripts": [
            "myproject=myproject.cli:main",
        ],
    },
)
```

## Manifest for Package Data

```
# MANIFEST.in
include README.md
include LICENSE
include pyproject.toml
recursive-include src/myproject *.py
recursive-include src/myproject py.typed
recursive-include tests *.py
prune docs/_build
```

## Version Management

```python
# src/myproject/__version__.py
__version__ = "0.1.0"

# src/myproject/__init__.py
from myproject.__version__ import __version__

# Read version in pyproject.toml
import tomli
from pathlib import Path

def get_version() -> str:
    pyproject = Path(__file__).parent.parent / "pyproject.toml"
    with open(pyproject, "rb") as f:
        data = tomli.load(f)
    return data["project"]["version"]
```

## Dependency Management Best Practices

```python
# Pin dependencies for applications (uv.lock handles this automatically)
# For libraries, use ranges:
requests>=2.31.0,<3.0.0
pydantic>=2.5.0,<3.0.0
```

```bash
# Lock files
# uv: uv.lock (auto-generated, commit to VCS for applications)
uv lock                       # Resolve and lock dependencies
uv lock --upgrade             # Upgrade all dependencies
uv lock --upgrade-package requests  # Upgrade single package
```

## CI/CD Integration

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.12", "3.13"]

    steps:
    - uses: actions/checkout@v4
    - name: Install uv
      uses: astral-sh/setup-uv@v5

    - name: Set up Python
      run: uv python install ${{ matrix.python-version }}

    - name: Install dependencies
      run: uv sync

    - name: Run tests
      run: uv run pytest --cov --cov-report=xml

    - name: Type check
      run: uv run mypy --strict src

    - name: Lint and format check
      run: |
        uv run ruff check src tests
        uv run ruff format --check src tests

    - name: Upload coverage
      uses: codecov/codecov-action@v3
```

## Pre-commit Hooks

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.6
    hooks:
      - id: ruff
        args: [--fix, --exit-non-zero-on-fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.7.1
    hooks:
      - id: mypy
        additional_dependencies: [types-requests]
```

```bash
# Install pre-commit
pip install pre-commit
pre-commit install

# Run manually
pre-commit run --all-files
```
