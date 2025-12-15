# Story 1.1: Initialize Python Project with MCP Server Skeleton

Status: ready-for-dev

## Story

As a **System Admin**,
I want to **clone the project and have a working Python environment with all dependencies installed**,
So that **I can start developing and running the MCP server**.

## Acceptance Criteria

1. **Given** I have Python 3.10+ installed
   **When** I run `pip install -r requirements.txt`
   **Then** all dependencies (fastmcp, neo4j, sentence-transformers, cocoindex, python-dotenv) are installed
   **And** I can run `python src/main.py` without import errors

## Tasks / Subtasks

- [ ] Task 1: Initialize pyproject.toml with project metadata (AC: #1)
  - [ ] Subtask 1.1: Create pyproject.toml with name, version, description
  - [ ] Subtask 1.2: Configure Python 3.10+ requirement
  - [ ] Subtask 1.3: Define project entry point

- [ ] Task 2: Configure all dependencies (AC: #1)
  - [ ] Subtask 2.1: Add core dependencies to pyproject.toml
    - fastmcp (MCP protocol handling)
    - neo4j (Neo4j driver with connection pooling)
    - sentence-transformers (embedding generation)
    - cocoindex (ETL pipeline orchestration)
    - tree-sitter-typescript (TypeScript/Angular parsing)
    - python-dotenv (environment configuration)
  - [ ] Subtask 2.2: Add development dependencies
    - ruff (linting)
    - mypy (type checking)
    - pytest (testing)
    - pre-commit (git hooks)
  - [ ] Subtask 2.3: Generate requirements.txt from pyproject.toml

- [ ] Task 3: Create project directory structure (AC: #1)
  - [ ] Subtask 3.1: Create src/ directory with __init__.py
  - [ ] Subtask 3.2: Create src/main.py as MCP server entry point
  - [ ] Subtask 3.3: Create src/config/__init__.py and settings.py
  - [ ] Subtask 3.4: Create src/mcp/__init__.py and server.py
  - [ ] Subtask 3.5: Create src/db/__init__.py
  - [ ] Subtask 3.6: Create src/utils/__init__.py and exceptions.py
  - [ ] Subtask 3.7: Create scripts/cypher/ directory

- [ ] Task 4: Implement minimal MCP server skeleton (AC: #1)
  - [ ] Subtask 4.1: Implement src/main.py with FastMCP server initialization
  - [ ] Subtask 4.2: Configure server to use stdio transport
  - [ ] Subtask 4.3: Verify server starts without errors

- [ ] Task 5: Create configuration boilerplate (AC: #1)
  - [ ] Subtask 5.1: Create .env.example with template variables
  - [ ] Subtask 5.2: Implement settings.py to load environment variables
  - [ ] Subtask 5.3: Create .gitignore for Python projects

- [ ] Task 6: Verify setup works end-to-end (AC: #1)
  - [ ] Subtask 6.1: Run `pip install -e .` to install in editable mode
  - [ ] Subtask 6.2: Run `python src/main.py` and verify no import errors
  - [ ] Subtask 6.3: Document any platform-specific notes

## Dev Notes

### Critical Technical Requirements

**Python Version:**
- Python 3.10+ required (specified in pyproject.toml)
- Use async/await patterns throughout for database operations

**Dependency Versions (from Architecture):**
| Package | Purpose | Notes |
|---------|---------|-------|
| `fastmcp` | MCP protocol handling | Official Python SDK, handles JSON-RPC 2.0 automatically |
| `neo4j` | Database driver | Python driver with connection pooling support |
| `sentence-transformers` | Embeddings | Will use all-MiniLM-L6-v2 (384 dimensions) |
| `cocoindex` | ETL pipeline | For orchestrating indexing workflows |
| `tree-sitter-typescript` | Parsing | For Angular/TypeScript AST analysis |
| `python-dotenv` | Configuration | Load .env files |

### Project Structure (CRITICAL - Follow Exactly)

```
knowledge-base/
├── README.md
├── pyproject.toml
├── requirements.txt
├── .env.example
├── .gitignore
│
├── src/
│   ├── __init__.py
│   ├── main.py                     # MCP server entry point
│   ├── config/
│   │   ├── __init__.py
│   │   └── settings.py             # Environment configuration
│   │
│   ├── mcp/
│   │   ├── __init__.py
│   │   └── server.py               # FastMCP server setup
│   │
│   ├── db/
│   │   └── __init__.py
│   │
│   └── utils/
│       ├── __init__.py
│       └── exceptions.py           # Custom exception classes
│
└── scripts/
    └── cypher/
```

### Naming Conventions (MUST Follow)

| Element | Convention | Example |
|---------|------------|---------|
| Python Files | snake_case | `neo4j_client.py`, `mcp_tools.py` |
| Python Functions | snake_case | `get_feature_context()` |
| Python Classes | PascalCase | `Neo4jClient`, `FeatureIndexer` |
| Constants | SCREAMING_SNAKE_CASE | `DEFAULT_EMBEDDING_MODEL` |
| Directories | snake_case | `indexers/`, `queries/` |

### Environment Variables Template (.env.example)

```bash
# Neo4j Connection
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-secure-password
NEO4J_DATABASE=neo4j

# Embedding Model (optional - has defaults)
EMBEDDING_MODEL=all-MiniLM-L6-v2
```

### Custom Exception Hierarchy (Create in exceptions.py)

```python
class KnowledgeBaseError(Exception):
    """Base exception for all knowledge-base errors"""
    pass

class IndexingError(KnowledgeBaseError):
    """Raised when indexing fails"""
    pass

class QueryError(KnowledgeBaseError):
    """Raised when a query fails"""
    pass

class ConnectionError(KnowledgeBaseError):
    """Raised when Neo4j connection fails"""
    pass
```

### Minimal main.py Implementation

```python
from fastmcp import FastMCP

mcp = FastMCP("knowledge-base")

@mcp.tool()
def ping() -> dict:
    """Health check tool to verify MCP server is running."""
    return {"status": "ok", "message": "knowledge-base MCP server is running"}

if __name__ == "__main__":
    mcp.run()
```

### Code Quality Requirements

- Type hints on ALL function signatures
- Linting with `ruff`
- Type checking with `mypy`
- Pre-commit hooks for automated checks

### References

- [Source: docs/architecture.md#Starter-Template-Evaluation] - FastMCP selection rationale
- [Source: docs/architecture.md#Project-Structure-&-Boundaries] - Complete directory structure
- [Source: docs/architecture.md#Naming-Patterns] - Naming conventions
- [Source: docs/architecture.md#Process-Patterns] - Exception hierarchy
- [Source: docs/epics.md#Story-1.1] - Original acceptance criteria

## Dev Agent Record

### Context Reference

This story file contains all context from architecture.md and epics.md needed for implementation.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Debug Log References

N/A - Initial story creation

### Completion Notes List

- Story ready for development
- All architectural patterns documented
- No previous story learnings (this is Story 1.1)

### File List

Files to be created:
- `pyproject.toml` - Project configuration
- `requirements.txt` - Pip dependencies
- `.env.example` - Environment template
- `.gitignore` - Git ignore rules
- `src/__init__.py`
- `src/main.py` - MCP server entry point
- `src/config/__init__.py`
- `src/config/settings.py` - Environment loading
- `src/mcp/__init__.py`
- `src/mcp/server.py` - FastMCP setup
- `src/db/__init__.py`
- `src/utils/__init__.py`
- `src/utils/exceptions.py` - Custom exceptions
- `scripts/cypher/` - Directory for Cypher scripts
