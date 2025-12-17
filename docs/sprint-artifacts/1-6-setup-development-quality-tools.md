---
beads_id: knowledge-base-lrf
beads_parent: knowledge-base-u9s
last_synced: 2025-12-18T00:36:00+07:00
---

# Story 1.6: Setup Development Quality Tools

Status: ready-for-dev

## Story

As a **Developer**,
I want **linting (ruff), type checking (mypy), and pre-commit hooks configured**,
So that **code quality is enforced automatically**.

## Acceptance Criteria

1. **Given** I have the project cloned
   **When** I run `pre-commit install`
   **Then** hooks are installed for ruff and mypy
   **And** running `ruff check src/` passes with no errors
   **And** running `mypy src/` passes with no type errors

## Tasks / Subtasks

- [ ] Task 1: Configure ruff linter (AC: #1)
  - [ ] Subtask 1.1: Add ruff to dev dependencies in pyproject.toml
  - [ ] Subtask 1.2: Configure ruff settings in pyproject.toml
  - [ ] Subtask 1.3: Run ruff check src/ and fix any issues
  - [ ] Subtask 1.4: Configure ruff format for code formatting

- [ ] Task 2: Configure mypy type checker (AC: #1)
  - [ ] Subtask 2.1: Add mypy to dev dependencies
  - [ ] Subtask 2.2: Configure mypy settings in pyproject.toml
  - [ ] Subtask 2.3: Run mypy src/ and fix any type errors
  - [ ] Subtask 2.4: Add type stubs for third-party libraries

- [ ] Task 3: Configure pre-commit hooks (AC: #1)
  - [ ] Subtask 3.1: Add pre-commit to dev dependencies
  - [ ] Subtask 3.2: Create .pre-commit-config.yaml
  - [ ] Subtask 3.3: Add ruff hook
  - [ ] Subtask 3.4: Add mypy hook
  - [ ] Subtask 3.5: Add trailing whitespace and EOF hooks

- [ ] Task 4: Configure pytest (AC: #1)
  - [ ] Subtask 4.1: Add pytest and pytest-asyncio to dev dependencies
  - [ ] Subtask 4.2: Configure pytest in pyproject.toml
  - [ ] Subtask 4.3: Create tests/ directory structure
  - [ ] Subtask 4.4: Add sample test to verify setup

- [ ] Task 5: Verify all tools pass (AC: #1)
  - [ ] Subtask 5.1: Run pre-commit install
  - [ ] Subtask 5.2: Run pre-commit run --all-files
  - [ ] Subtask 5.3: Verify all hooks pass

## Dev Notes

### pyproject.toml Configuration

```toml
[project]
name = "knowledge-base"
version = "0.1.0"
description = "MCP server for Angular/NX knowledge graph with Neo4j"
requires-python = ">=3.10"
dependencies = [
    "fastmcp",
    "neo4j",
    "sentence-transformers",
    "cocoindex",
    "tree-sitter-typescript",
    "python-dotenv",
]

[project.optional-dependencies]
dev = [
    "ruff",
    "mypy",
    "pre-commit",
    "pytest",
    "pytest-asyncio",
    "types-python-dotenv",
]

[tool.ruff]
target-version = "py310"
line-length = 100
src = ["src", "tests"]

[tool.ruff.lint]
select = [
    "E",   # pycodestyle errors
    "W",   # pycodestyle warnings
    "F",   # pyflakes
    "I",   # isort
    "B",   # flake8-bugbear
    "C4",  # flake8-comprehensions
    "UP",  # pyupgrade
]
ignore = [
    "E501",  # line too long (handled by formatter)
]

[tool.ruff.format]
quote-style = "double"
indent-style = "space"

[tool.mypy]
python_version = "3.10"
strict = true
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true
disallow_incomplete_defs = true
check_untyped_defs = true

[[tool.mypy.overrides]]
module = [
    "fastmcp.*",
    "cocoindex.*",
    "tree_sitter_typescript.*",
]
ignore_missing_imports = true

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
python_files = ["test_*.py", "*_test.py"]
python_functions = ["test_*"]
```

### .pre-commit-config.yaml

```yaml
repos:
  - repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
      - id: trailing-whitespace
      - id: end-of-file-fixer
      - id: check-yaml
      - id: check-added-large-files

  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.1.9
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.8.0
    hooks:
      - id: mypy
        additional_dependencies:
          - types-python-dotenv
        args: [--config-file=pyproject.toml]
```

### Test Directory Structure

```
tests/
├── __init__.py
├── conftest.py           # Pytest fixtures
├── config/
│   ├── __init__.py
│   └── test_settings.py
├── db/
│   ├── __init__.py
│   └── test_neo4j_client.py
└── mcp/
    ├── __init__.py
    └── tools/
        ├── __init__.py
        └── test_ping.py
```

### conftest.py Fixtures

```python
# tests/conftest.py
import pytest
from unittest.mock import patch
import os

@pytest.fixture
def mock_env_vars():
    """Mock environment variables for testing."""
    env_vars = {
        "NEO4J_URI": "bolt://localhost:7687",
        "NEO4J_USER": "neo4j",
        "NEO4J_PASSWORD": "test-password",
        "NEO4J_DATABASE": "neo4j",
    }
    with patch.dict(os.environ, env_vars):
        yield env_vars

@pytest.fixture
def sample_settings(mock_env_vars):
    """Get settings with mocked environment."""
    from src.config.settings import get_settings
    get_settings.cache_clear()  # Clear cached settings
    return get_settings()
```

### Sample Test

```python
# tests/mcp/tools/test_ping.py
from src.mcp.tools.ping import ping

def test_ping_returns_ok_status():
    """Test ping tool returns expected response."""
    result = ping()

    assert result["status"] == "ok"
    assert "knowledge-base" in result["message"]
    assert "running" in result["message"]
```

### Installation Commands

```bash
# Install dev dependencies
pip install -e ".[dev]"

# Install pre-commit hooks
pre-commit install

# Run all checks
pre-commit run --all-files

# Individual checks
ruff check src/
ruff format src/ --check
mypy src/
pytest
```

### Code Quality Standards

| Tool | Purpose | Pass Criteria |
|------|---------|---------------|
| ruff | Linting + formatting | No errors, consistent style |
| mypy | Type checking | No type errors with strict mode |
| pytest | Testing | All tests pass |
| pre-commit | Automation | All hooks pass before commit |

### Previous Story Dependencies

- Stories 1-1 through 1-5: All code must pass quality checks

### References

- [Source: docs/architecture.md#Enforcement-Guidelines] - Tool requirements
- [Source: docs/architecture.md#Pattern-Enforcement] - Linting and type checking
- [Source: docs/epics.md#Story-1.6] - Original acceptance criteria

## Dev Agent Record

### Context Reference

Final Epic 1 story. Ensures all previous story code meets quality standards.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Completion Notes List

- All code from Stories 1-1 to 1-5 must pass these checks
- Establishes quality baseline for all future development
- Pre-commit hooks prevent low-quality commits

### File List

Files to create/modify:
- `pyproject.toml` - Add dev dependencies and tool configs
- `.pre-commit-config.yaml` - Pre-commit hooks
- `tests/__init__.py`
- `tests/conftest.py` - Shared fixtures
- `tests/config/__init__.py`
- `tests/config/test_settings.py`
- `tests/db/__init__.py`
- `tests/mcp/__init__.py`
- `tests/mcp/tools/__init__.py`
- `tests/mcp/tools/test_ping.py`
