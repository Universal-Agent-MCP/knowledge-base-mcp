# Story 1.2: Configure Environment Variables for Neo4j Connection

Status: ready-for-dev

## Story

As a **System Admin**,
I want to **configure Neo4j connection details via environment variables**,
So that **credentials are never stored in code and I can switch environments easily**.

## Acceptance Criteria

1. **Given** I have a `.env.example` file in the project root
   **When** I copy it to `.env` and fill in my Neo4j credentials (`NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD`)
   **Then** the application reads configuration from environment variables
   **And** credentials are validated on startup with clear error messages if missing

## Tasks / Subtasks

- [ ] Task 1: Create .env.example template (AC: #1)
  - [ ] Subtask 1.1: Define NEO4J_URI variable with example bolt://localhost:7687
  - [ ] Subtask 1.2: Define NEO4J_USER variable with example neo4j
  - [ ] Subtask 1.3: Define NEO4J_PASSWORD placeholder
  - [ ] Subtask 1.4: Define NEO4J_DATABASE with default neo4j
  - [ ] Subtask 1.5: Add optional EMBEDDING_MODEL variable

- [ ] Task 2: Implement settings.py configuration loader (AC: #1)
  - [ ] Subtask 2.1: Use python-dotenv to load .env file
  - [ ] Subtask 2.2: Create Settings dataclass/class with all config fields
  - [ ] Subtask 2.3: Implement get_settings() singleton function
  - [ ] Subtask 2.4: Add type hints for all configuration values

- [ ] Task 3: Implement validation on startup (AC: #1)
  - [ ] Subtask 3.1: Check for required environment variables
  - [ ] Subtask 3.2: Raise ConfigurationError with clear message if missing
  - [ ] Subtask 3.3: Validate NEO4J_URI format (must start with bolt:// or bolt+s://)
  - [ ] Subtask 3.4: Log successful configuration load at INFO level

- [ ] Task 4: Add configuration to .gitignore (AC: #1)
  - [ ] Subtask 4.1: Ensure .env is in .gitignore
  - [ ] Subtask 4.2: Keep .env.example tracked in git

- [ ] Task 5: Write unit tests for settings (AC: #1)
  - [ ] Subtask 5.1: Test successful configuration loading
  - [ ] Subtask 5.2: Test missing required variables raise error
  - [ ] Subtask 5.3: Test invalid URI format raises error

## Dev Notes

### Environment Variables Specification

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `NEO4J_URI` | Yes | - | Neo4j connection URI (bolt:// or bolt+s://) |
| `NEO4J_USER` | Yes | - | Neo4j username |
| `NEO4J_PASSWORD` | Yes | - | Neo4j password |
| `NEO4J_DATABASE` | No | `neo4j` | Target database name |
| `EMBEDDING_MODEL` | No | `all-MiniLM-L6-v2` | SentenceTransformer model |

### .env.example Template

```bash
# Neo4j Connection (Required)
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-secure-password
NEO4J_DATABASE=neo4j

# Embedding Model (Optional)
EMBEDDING_MODEL=all-MiniLM-L6-v2
```

### Settings Implementation Pattern

```python
# src/config/settings.py
from dataclasses import dataclass
from functools import lru_cache
import os
from dotenv import load_dotenv

from src.utils.exceptions import ConfigurationError

@dataclass(frozen=True)
class Settings:
    """Application configuration loaded from environment variables."""
    neo4j_uri: str
    neo4j_user: str
    neo4j_password: str
    neo4j_database: str = "neo4j"
    embedding_model: str = "all-MiniLM-L6-v2"

    def __post_init__(self) -> None:
        """Validate configuration after initialization."""
        if not self.neo4j_uri.startswith(("bolt://", "bolt+s://")):
            raise ConfigurationError(
                f"NEO4J_URI must start with bolt:// or bolt+s://, got: {self.neo4j_uri}"
            )

@lru_cache()
def get_settings() -> Settings:
    """Load and cache application settings."""
    load_dotenv()
    
    required_vars = ["NEO4J_URI", "NEO4J_USER", "NEO4J_PASSWORD"]
    missing = [var for var in required_vars if not os.getenv(var)]
    
    if missing:
        raise ConfigurationError(
            f"Missing required environment variables: {', '.join(missing)}"
        )
    
    return Settings(
        neo4j_uri=os.getenv("NEO4J_URI", ""),
        neo4j_user=os.getenv("NEO4J_USER", ""),
        neo4j_password=os.getenv("NEO4J_PASSWORD", ""),
        neo4j_database=os.getenv("NEO4J_DATABASE", "neo4j"),
        embedding_model=os.getenv("EMBEDDING_MODEL", "all-MiniLM-L6-v2"),
    )
```

### Add ConfigurationError to exceptions.py

```python
class ConfigurationError(KnowledgeBaseError):
    """Raised when configuration is invalid or missing."""
    pass
```

### Security Requirements (NFR9, NFR10)

- **Never** hardcode credentials in source code
- Use `bolt://` for local development
- Use `bolt+s://` for production/cloud (encrypted)
- Ensure .env is always in .gitignore

### Logging Pattern

```python
import logging

logger = logging.getLogger(__name__)

# On successful config load:
logger.info("Configuration loaded successfully from environment")
logger.debug(f"Neo4j URI: {settings.neo4j_uri}")
```

### Previous Story Dependencies

- Story 1-1 creates the project structure including:
  - `src/config/__init__.py`
  - `src/config/settings.py` (empty or minimal)
  - `src/utils/exceptions.py` (base exceptions)

### References

- [Source: docs/architecture.md#Authentication-&-Security] - Credential storage and protocol requirements
- [Source: docs/architecture.md#Infrastructure-&-Deployment] - Environment config pattern
- [Source: docs/epics.md#Story-1.2] - Original acceptance criteria
- [Source: docs/architecture.md#Process-Patterns] - Error handling patterns

## Dev Agent Record

### Context Reference

Builds on Story 1-1 project structure. Uses python-dotenv for configuration.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Debug Log References

N/A

### Completion Notes List

- Implements NFR9 (environment-based credentials)
- Implements NFR10 (bolt protocol support)
- Follows architecture exception patterns

### File List

Files to modify/create:
- `.env.example` - Environment template (create)
- `src/config/settings.py` - Configuration loader (implement)
- `src/utils/exceptions.py` - Add ConfigurationError
- `.gitignore` - Ensure .env excluded
- `tests/config/test_settings.py` - Unit tests
