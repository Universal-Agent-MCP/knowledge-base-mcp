---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
inputDocuments:
  - docs/prd.md
  - docs/analysis/research/technical-neo4j-mcp-knowledge-base-research-2024-12-14.md
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2024-12-14'
project_name: 'knowledge-base'
user_name: 'manhhaycode'
date: '2024-12-14'
---

# Architecture Decision Document - knowledge-base

**Author:** manhhaycode  
**Date:** 2024-12-14

---

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**

The PRD defines 32 functional requirements organized into 8 categories:

| Category | Requirements | Key Architectural Implications |
|----------|-------------|--------------------------------|
| **Source Code Indexing** | FR1-FR5 | AST parsing, embedding generation, NX library awareness |
| **Documentation Indexing** | FR6-FR8 | Markdown parsing, semantic embeddings, code-doc linking |
| **Feature Context Queries** | FR9-FR12 | Graph traversal, relationship depth control, context aggregation |
| **Semantic Search** | FR13-FR15 | Vector indexes, similarity scoring, result ranking |
| **Impact Analysis** | FR16-FR19 | Dependency graph traversal, multi-hop analysis |
| **Dependency Mapping** | FR20-FR22 | NX project graph import, graph structure output |
| **MCP Protocol Integration** | FR23-FR25 | JSON-RPC 2.0, structured responses, tool definitions |
| **Configuration & Setup** | FR26-FR28 | Environment variables, pip installation, IDE integration |
| **Knowledge Graph Schema** | FR29-FR32 | Node types, relationship types, embedding properties |

**Non-Functional Requirements:**

| Category | Requirements | Architectural Impact |
|----------|-------------|---------------------|
| **Performance** | NFR1-NFR4 | Query < 500ms, full index < 5min, incremental < 10s, vector search < 200ms |
| **Reliability** | NFR5-NFR8 | 80% context relevance, 100% indexing completeness, graceful degradation |
| **Security** | NFR9-NFR11 | Environment-based credentials, bolt protocol, local-only source code |
| **Integration** | NFR12-NFR14 | MCP JSON-RPC 2.0, VS Code MCP config, Neo4j 5.x compatibility |

**Scale & Complexity:**

- **Primary Domain:** Developer Tool / API Backend
- **Complexity Level:** Medium
- **Estimated Architectural Components:** 6 major components
- **Target Users:** 4 developers (small team)
- **Target Codebase:** Angular 17 + NX monorepo (~8,000 files based on research estimates)

### Technical Constraints & Dependencies

| Constraint | Description |
|------------|-------------|
| **External Database** | Neo4j is user-provided (local, Docker, or cloud) - not bundled |
| **MCP Protocol** | Must follow JSON-RPC 2.0 specification for Copilot/Claude compatibility |
| **Embedding Dimensions** | 384 dimensions (all-MiniLM-L6-v2) - affects vector index configuration |
| **Target Framework** | Angular 17 + NX monorepo - requires Angular decorator awareness |
| **Python Runtime** | Python-based MCP server using FastMCP SDK |

### Cross-Cutting Concerns Identified

1. **Error Handling:** Consistent error responses across all MCP tools
2. **Logging:** Structured logging for debugging indexing and query issues
3. **Configuration:** Environment-based configuration for all external services
4. **Vector Embedding:** Consistent embedding model usage across all indexing operations
5. **Graph Connection:** Neo4j connection pooling and retry logic

---

## Starter Template Evaluation

### Primary Technology Domain

**API Backend / Developer Tool** - This is a Python MCP server that exposes tools via the Model Context Protocol, connecting to Neo4j for graph storage and queries.

### Starter Options Considered

| Option | Description | Fit Assessment |
|--------|-------------|----------------|
| **FastMCP Official Template** | Official MCP SDK for Python | ✅ Best fit - designed for MCP servers |
| **Plain Python + FastAPI** | Generic API framework | ❌ Does not include MCP protocol handling |
| **NestJS MCP Template** | TypeScript MCP server | ❌ PRD specifies Python implementation |

### Selected Starter: FastMCP Python Project

**Rationale for Selection:**
- Official Python SDK for MCP protocol
- Automatic JSON schema generation from type hints
- Built-in tool definition decorators
- Native async support for database operations
- Recommended in PRD and technical research

**Initialization Command:**

```bash
# Create project directory
mkdir knowledge-base-mcp
cd knowledge-base-mcp

# Initialize Python project
python -m venv .venv
source .venv/bin/activate

# Install core dependencies
pip install fastmcp neo4j sentence-transformers cocoindex tree-sitter-typescript
```

**Architectural Decisions Provided by Starter:**

| Category | Decision |
|----------|----------|
| **Language & Runtime** | Python 3.10+, async/await patterns |
| **Protocol Handling** | FastMCP handles JSON-RPC 2.0 automatically |
| **Tool Definitions** | Decorator-based with automatic schema generation |
| **Type Safety** | Python type hints for all tool inputs/outputs |

---

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Database: Neo4j 5.x with vector index support
- MCP Framework: FastMCP (Python)
- Embedding Model: all-MiniLM-L6-v2 (384 dimensions)
- ETL Pipeline: CocoIndex for source code parsing

**Important Decisions (Shape Architecture):**
- Code Parsing: Tree-sitter for TypeScript/Angular
- Connection Management: Neo4j Python driver with connection pooling
- Configuration: Environment variables via python-dotenv

**Deferred Decisions (Post-MVP):**
- Git hook automation for incremental sync
- Web UI dashboard
- Multi-project support
- RBAC for multi-tenant scenarios

### Data Architecture

| Decision | Choice | Version | Rationale |
|----------|--------|---------|-----------|
| **Graph Database** | Neo4j | 5.x+ (Enterprise or Community) | Native vector indexes, mature graph engine, Cypher query language |
| **Vector Index** | Neo4j native | N/A | Integrated with graph queries, no external vector DB needed |
| **Embedding Dimensions** | 384 | N/A | Matches all-MiniLM-L6-v2 output |
| **Similarity Function** | Cosine | N/A | Standard for text embeddings |

**Neo4j Schema (from research):**

```cypher
// Node Constraints
CREATE CONSTRAINT Project_name IF NOT EXISTS FOR (p:Project) REQUIRE p.name IS UNIQUE;
CREATE CONSTRAINT Library_name IF NOT EXISTS FOR (l:Library) REQUIRE l.name IS UNIQUE;
CREATE CONSTRAINT Component_path IF NOT EXISTS FOR (c:Component) REQUIRE c.path IS UNIQUE;
CREATE CONSTRAINT Service_path IF NOT EXISTS FOR (s:Service) REQUIRE s.path IS UNIQUE;
CREATE CONSTRAINT Feature_name IF NOT EXISTS FOR (f:Feature) REQUIRE f.name IS UNIQUE;
CREATE CONSTRAINT Document_path IF NOT EXISTS FOR (d:Document) REQUIRE d.path IS UNIQUE;

// Vector Indexes
CREATE VECTOR INDEX feature_embedding IF NOT EXISTS
FOR (f:Feature) ON (f.embedding)
OPTIONS {indexConfig: {`vector.dimensions`: 384, `vector.similarity_function`: 'cosine'}};

CREATE VECTOR INDEX document_embedding IF NOT EXISTS
FOR (d:Document) ON (d.embedding)
OPTIONS {indexConfig: {`vector.dimensions`: 384, `vector.similarity_function`: 'cosine'}};

CREATE VECTOR INDEX code_chunk_embedding IF NOT EXISTS
FOR (cc:CodeChunk) ON (cc.embedding)
OPTIONS {indexConfig: {`vector.dimensions`: 384, `vector.similarity_function`: 'cosine'}};
```

### Authentication & Security

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Credential Storage** | Environment variables | NFR9 - never in code |
| **Neo4j Protocol** | bolt:// or bolt+s:// | NFR10 - secure transport |
| **Source Code Storage** | Embeddings only in DB | NFR11 - source stays local |
| **MCP Transport** | stdio (local) | Standard for VS Code MCP integration |

### API & Communication Patterns

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Protocol** | MCP (JSON-RPC 2.0) | Required for Copilot/Claude integration |
| **Tool Response Format** | Structured JSON objects | Compatible with AI agent parsing |
| **Error Format** | `{error: {code, message}}` | MCP protocol standard |

**MCP Tools API (from PRD):**

| Tool | Input | Output |
|------|-------|--------|
| `index_source_code` | `{paths: string[], library_scope?: string}` | `{indexed: number, status: string}` |
| `index_markdown_specs` | `{paths: string[]}` | `{indexed: number, status: string}` |
| `query_feature_context` | `{feature_name: string, depth?: number}` | `{feature: object, related: array}` |
| `search_similar_features` | `{query: string, limit?: number}` | `{results: array, scores: array}` |
| `analyze_feature_impact` | `{feature_name: string, target_libs?: string[]}` | `{affected: array, depth: number}` |
| `get_feature_dependencies` | `{feature_name: string}` | `{dependencies: array, graph: object}` |

### Infrastructure & Deployment

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Neo4j Deployment** | User-managed (local, cloud, or Docker) | User controls infrastructure and deployment method |
| **MCP Server Execution** | Python process via VS Code | Standard MCP server pattern |
| **Environment Config** | `.env` file + python-dotenv | Simple, standard for Python projects |

**Neo4j Setup Options:**

Users are responsible for setting up their own Neo4j instance. The MCP server connects via environment variables:

```bash
# .env file
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-secure-password
NEO4J_DATABASE=neo4j
```

**Supported Neo4j Deployment Methods:**

1. **Neo4j Desktop** (Recommended for development)
   - Download from neo4j.com/download
   - Create a new database with Neo4j 5.x+
   - Enable APOC plugin if needed
   - Copy connection details to `.env`

2. **Neo4j AuraDB** (Managed cloud service)
   - Create account at neo4j.com/cloud/aura
   - Create a new instance (Free tier available)
   - Use provided connection URI (bolt+s://)
   - Copy credentials to `.env`

3. **Docker** (For users who prefer containerization)
   ```bash
   docker run -d --name neo4j-kb \
     -p 7474:7474 -p 7687:7687 \
     -e NEO4J_AUTH=neo4j/password \
     neo4j:5-enterprise
   ```

4. **Self-hosted** (Production deployments)
   - Install Neo4j 5.x+ on server
   - Configure bolt connector
   - Set up authentication
   - Provide connection details via environment variables

**Required Neo4j Configuration:**
- Neo4j version: 5.x or higher (for vector index support)
- Plugins: APOC (optional but recommended)
- Memory: Minimum 2GB heap for development, scale based on data size

### Decision Impact Analysis

**Implementation Sequence:**
1. Neo4j schema deployment (Cypher scripts)
2. NX project graph import script
3. CocoIndex ETL pipeline setup
4. FastMCP server with core tools
5. VS Code MCP configuration

**Cross-Component Dependencies:**
- CocoIndex → Neo4j (writes indexed data)
- MCP Server → Neo4j (reads for queries)
- MCP Server → SentenceTransformer (generates query embeddings)

---

## Implementation Patterns & Consistency Rules

### Pattern Categories Defined

**Critical Conflict Points Identified:** 12 areas where AI agents could make different choices

### Naming Patterns

**Database Naming Conventions:**

| Element | Convention | Example |
|---------|------------|---------|
| Node Labels | PascalCase | `Component`, `Service`, `Feature` |
| Relationship Types | SCREAMING_SNAKE_CASE | `DEPENDS_ON`, `IMPLEMENTED_BY`, `USES_SERVICE` |
| Properties | snake_case | `file_path`, `embedding`, `created_at` |
| Indexes | snake_case with prefix | `feature_embedding`, `idx_component_path` |

**API Naming Conventions:**

| Element | Convention | Example |
|---------|------------|---------|
| MCP Tool Names | snake_case | `query_feature_context`, `index_source_code` |
| Tool Parameters | snake_case | `feature_name`, `library_scope` |
| Response Fields | snake_case | `indexed_count`, `affected_components` |

**Code Naming Conventions (Python):**

| Element | Convention | Example |
|---------|------------|---------|
| Functions | snake_case | `get_feature_context()`, `index_files()` |
| Classes | PascalCase | `Neo4jClient`, `FeatureIndexer` |
| Constants | SCREAMING_SNAKE_CASE | `DEFAULT_EMBEDDING_MODEL`, `MAX_QUERY_DEPTH` |
| Files | snake_case | `neo4j_client.py`, `mcp_tools.py` |
| Directories | snake_case | `indexers/`, `queries/` |

### Structure Patterns

**Project Organization:**
- **Feature-based organization** for indexers and queries
- **Co-located tests** with `_test.py` suffix
- **Shared utilities** in dedicated `utils/` directory

**File Structure Patterns:**

| Pattern | Rule |
|---------|------|
| Configuration | `config/` directory with environment-specific files |
| Cypher Scripts | `scripts/cypher/` for schema and migrations |
| Test Fixtures | `tests/fixtures/` for sample data |

### Format Patterns

**API Response Formats:**

All MCP tool responses follow a consistent structure with detailed metadata for debugging and monitoring.

#### Use Case 1: index_source_code

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "indexed_files": [
            {
                "path": "libs/feature-auth/src/lib/login/login.component.ts",
                "type": "Component",
                "entities_extracted": 3,
                "embedding_generated": true
            },
            {
                "path": "libs/feature-auth/src/lib/services/auth.service.ts",
                "type": "Service",
                "entities_extracted": 5,
                "embedding_generated": true
            }
        ],
        "summary": {
            "total_files": 2,
            "components": 1,
            "services": 1,
            "total_entities": 8
        }
    },
    "metadata": {
        "indexed_count": 2,
        "duration_ms": 1234,
        "library_scope": "feature-auth",
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

**Error Response:**
```python
{
    "status": "error",
    "error": {
        "code": "INDEXING_FAILED",
        "message": "Failed to parse TypeScript file: libs/feature-auth/src/lib/login/login.component.ts",
        "details": {
            "file_path": "libs/feature-auth/src/lib/login/login.component.ts",
            "parse_error": "Unexpected token at line 45"
        }
    },
    "metadata": {
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

#### Use Case 2: query_feature_context

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "feature": {
            "name": "User Authentication",
            "description": "Login and session management for users",
            "implemented_by": [
                {
                    "type": "Component",
                    "name": "LoginComponent",
                    "path": "libs/feature-auth/src/lib/login/login.component.ts",
                    "selector": "app-login"
                },
                {
                    "type": "Service",
                    "name": "AuthService",
                    "path": "libs/feature-auth/src/lib/services/auth.service.ts",
                    "provided_in": "root"
                }
            ],
            "related_features": [
                {
                    "name": "User Profile Management",
                    "relationship": "DEPENDS_ON",
                    "similarity_score": 0.85
                }
            ],
            "documentation": [
                {
                    "path": "docs/specs/authentication-feature.md",
                    "type": "Feature Specification",
                    "excerpt": "Users must be able to login with email and password..."
                }
            ]
        },
        "graph_depth": 2,
        "total_relationships": 12
    },
    "metadata": {
        "query_time_ms": 234,
        "depth_traversed": 2,
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

**Error Response:**
```python
{
    "status": "error",
    "error": {
        "code": "FEATURE_NOT_FOUND",
        "message": "Feature 'Payment Processing' not found in knowledge graph",
        "details": {
            "feature_name": "Payment Processing",
            "suggestions": ["User Authentication", "Order Processing", "Invoice Generation"]
        }
    },
    "metadata": {
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

#### Use Case 3: search_similar_features

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "results": [
            {
                "feature_name": "User Authentication",
                "description": "Login and session management",
                "similarity_score": 0.92,
                "implemented_by_count": 3,
                "primary_library": "feature-auth"
            },
            {
                "feature_name": "OAuth Integration",
                "description": "Third-party authentication via OAuth 2.0",
                "similarity_score": 0.87,
                "implemented_by_count": 2,
                "primary_library": "feature-auth"
            },
            {
                "feature_name": "Session Management",
                "description": "User session tracking and timeout",
                "similarity_score": 0.81,
                "implemented_by_count": 2,
                "primary_library": "data-access-session"
            }
        ],
        "total_results": 3,
        "query": "user login and authentication"
    },
    "metadata": {
        "query_time_ms": 156,
        "limit": 5,
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

#### Use Case 4: analyze_feature_impact

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "feature_name": "User Authentication",
        "affected_components": [
            {
                "name": "UserProfileComponent",
                "path": "libs/feature-user/src/lib/profile/profile.component.ts",
                "relationship": "USES_SERVICE",
                "distance": 1,
                "impact_level": "high"
            },
            {
                "name": "DashboardComponent",
                "path": "apps/main-app/src/app/dashboard/dashboard.component.ts",
                "relationship": "DEPENDS_ON",
                "distance": 2,
                "impact_level": "medium"
            }
        ],
        "affected_libraries": [
            {
                "name": "feature-user",
                "type": "feature",
                "affected_exports": ["UserProfileComponent", "UserSettingsComponent"]
            },
            {
                "name": "data-access-user",
                "type": "data-access",
                "affected_exports": ["UserService"]
            }
        ],
        "impact_summary": {
            "total_affected": 5,
            "max_depth": 3,
            "high_impact": 2,
            "medium_impact": 2,
            "low_impact": 1
        }
    },
    "metadata": {
        "query_time_ms": 312,
        "target_libraries": ["feature-user", "data-access-user"],
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

#### Use Case 5: get_feature_dependencies

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "feature_name": "Order Processing",
        "dependencies": [
            {
                "feature_name": "User Authentication",
                "dependency_type": "blocks",
                "reason": "Must verify user identity before processing orders"
            },
            {
                "feature_name": "Inventory Management",
                "dependency_type": "blocks",
                "reason": "Must check product availability"
            },
            {
                "feature_name": "Payment Processing",
                "dependency_type": "related",
                "reason": "Often used together but not strictly required"
            }
        ],
        "dependency_graph": {
            "nodes": [
                {"id": "order-processing", "label": "Order Processing"},
                {"id": "user-auth", "label": "User Authentication"},
                {"id": "inventory", "label": "Inventory Management"},
                {"id": "payment", "label": "Payment Processing"}
            ],
            "edges": [
                {"from": "order-processing", "to": "user-auth", "type": "blocks"},
                {"from": "order-processing", "to": "inventory", "type": "blocks"},
                {"from": "order-processing", "to": "payment", "type": "related"}
            ]
        }
    },
    "metadata": {
        "query_time_ms": 189,
        "total_dependencies": 3,
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

#### Use Case 6: index_markdown_specs

**Success Response:**
```python
{
    "status": "success",
    "data": {
        "indexed_documents": [
            {
                "path": "docs/specs/authentication-feature.md",
                "features_extracted": 2,
                "embedding_generated": true,
                "linked_to_code": true
            },
            {
                "path": "docs/specs/user-profile-feature.md",
                "features_extracted": 3,
                "embedding_generated": true,
                "linked_to_code": true
            }
        ],
        "summary": {
            "total_documents": 2,
            "total_features": 5,
            "total_code_links": 8
        }
    },
    "metadata": {
        "indexed_count": 2,
        "duration_ms": 892,
        "timestamp": "2024-12-14T10:30:00Z"
    }
}
```

**Common Error Codes:**

| Error Code | Description | HTTP Equivalent |
|------------|-------------|----------------|
| `INDEXING_FAILED` | File parsing or indexing error | 500 |
| `QUERY_FAILED` | Database query execution error | 500 |
| `FEATURE_NOT_FOUND` | Requested feature doesn't exist | 404 |
| `INVALID_PARAMETERS` | Invalid input parameters | 400 |
| `CONNECTION_ERROR` | Neo4j connection failure | 503 |
| `EMBEDDING_ERROR` | Embedding generation failure | 500 |

**Data Exchange Formats:**

| Format | Rule |
|--------|------|
| JSON field naming | snake_case |
| Date/time format | ISO 8601 strings (`2024-12-14T10:30:00Z`) |
| Boolean values | `true` / `false` |
| Null handling | Explicit `null`, never omit fields |
| Arrays | Always arrays, even for single items |

### Communication Patterns

**Logging Patterns:**

| Level | Usage | Example |
|-------|-------|---------|
| DEBUG | Detailed tracing | `"Indexing file: {path}"` |
| INFO | Operation completion | `"Indexed 45 files in 2.3s"` |
| WARNING | Recoverable issues | `"Skipped unsupported file type: {ext}"` |
| ERROR | Failures requiring attention | `"Neo4j connection failed: {error}"` |

**Log Format:**

```python
import logging

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
```

### Process Patterns

**Error Handling Patterns:**

```python
# Custom exception hierarchy
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

**Neo4j Connection Pattern:**

```python
from contextlib import contextmanager
from neo4j import GraphDatabase

class Neo4jClient:
    def __init__(self, uri: str, user: str, password: str):
        self._driver = GraphDatabase.driver(uri, auth=(user, password))
    
    @contextmanager
    def session(self):
        session = self._driver.session()
        try:
            yield session
        finally:
            session.close()
    
    def close(self):
        self._driver.close()
```

### Enforcement Guidelines

**All AI Agents MUST:**

1. Follow snake_case for all Python code and API parameters
2. Use the defined error response format for all failures
3. Log at appropriate levels using the structured format
4. Use the Neo4jClient context manager for all database operations
5. Include type hints on all function signatures

**Pattern Enforcement:**

- Linting: `ruff` for Python code style
- Type checking: `mypy` for type hint validation
- Pre-commit hooks: Automated checks before commits

---

## Project Structure & Boundaries

### Complete Project Directory Structure

```
knowledge-base/
├── README.md
├── pyproject.toml
├── requirements.txt
├── .env.example
├── .gitignore
├── .pre-commit-config.yaml
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
│   │   ├── server.py               # FastMCP server setup
│   │   └── tools/
│   │       ├── __init__.py
│   │       ├── index_source_code.py
│   │       ├── index_markdown_specs.py
│   │       ├── query_feature_context.py
│   │       ├── search_similar_features.py
│   │       ├── analyze_feature_impact.py
│   │       └── get_feature_dependencies.py
│   │
│   ├── indexers/
│   │   ├── __init__.py
│   │   ├── source_code_indexer.py   # CocoIndex pipeline for code
│   │   ├── markdown_indexer.py      # CocoIndex pipeline for docs
│   │   ├── nx_graph_importer.py     # NX project graph import
│   │   └── angular_parser.py        # Angular decorator detection
│   │
│   ├── queries/
│   │   ├── __init__.py
│   │   ├── feature_context.py       # Feature context queries
│   │   ├── semantic_search.py       # Vector similarity search
│   │   ├── impact_analysis.py       # Dependency traversal
│   │   └── dependency_graph.py      # Dependency mapping
│   │
│   ├── db/
│   │   ├── __init__.py
│   │   ├── neo4j_client.py          # Neo4j connection management
│   │   └── embeddings.py            # SentenceTransformer wrapper
│   │
│   └── utils/
│       ├── __init__.py
│       └── exceptions.py            # Custom exception classes
│
├── scripts/
│   ├── cypher/
│   │   ├── 001_create_constraints.cypher
│   │   ├── 002_create_vector_indexes.cypher
│   │   └── 003_seed_project_structure.cypher
│   ├── import_nx_graph.py           # Standalone NX import script
│   └── deploy_schema.py             # Schema deployment script
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py                  # Pytest fixtures
│   ├── fixtures/
│   │   ├── sample_angular_project/  # Sample NX project for testing
│   │   └── sample_markdown_docs/    # Sample spec documents
│   ├── unit/
│   │   ├── test_angular_parser.py
│   │   ├── test_neo4j_client.py
│   │   └── test_embeddings.py
│   ├── integration/
│   │   ├── test_indexers.py
│   │   └── test_queries.py
│   └── e2e/
│       └── test_mcp_tools.py
│
├── docs/
│   ├── prd.md                       # Product Requirements Document
│   ├── architecture.md              # This document
│   └── api-reference.md             # MCP Tools API Reference
│
└── docker/
    └── docker-compose.yml           # Neo4j + optional services
```

### Architectural Boundaries

**API Boundaries:**

| Boundary | Protocol | Components |
|----------|----------|------------|
| MCP Interface | JSON-RPC 2.0 (stdio) | `mcp/server.py` ↔ VS Code/Copilot |
| Neo4j Connection | Bolt protocol | `db/neo4j_client.py` ↔ Neo4j |
| External Embeddings | Python API | `db/embeddings.py` ↔ SentenceTransformer |

**Component Boundaries:**

```
┌─────────────────────────────────────────────────────────────────┐
│                         MCP Layer                                │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │ FastMCP Server (main.py)                                 │   │
│   │   └── Tools: index_*, query_*, search_*, analyze_*       │   │
│   └─────────────────────────────────────────────────────────┘   │
│                              │                                   │
├──────────────────────────────┼───────────────────────────────────┤
│                         Core Layer                               │
│   ┌───────────────┐    ┌─────────────┐    ┌─────────────────┐   │
│   │   Indexers    │    │   Queries   │    │      Utils      │   │
│   │ source_code   │    │ feature_ctx │    │  exceptions.py  │   │
│   │ markdown      │    │ semantic    │    │                 │   │
│   │ nx_graph      │    │ impact      │    │                 │   │
│   └───────────────┘    └─────────────┘    └─────────────────┘   │
│           │                   │                                  │
├───────────┼───────────────────┼──────────────────────────────────┤
│                         Data Layer                               │
│   ┌───────────────────────────────────────────────────────────┐ │
│   │  Neo4jClient (db/neo4j_client.py)                          │ │
│   │  EmbeddingService (db/embeddings.py)                       │ │
│   └───────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
                    ┌─────────────────────┐
                    │       Neo4j         │
                    │  (External Service) │
                    └─────────────────────┘
```

**Data Boundaries:**

| Boundary | Rule |
|----------|------|
| Source Code | Never stored in Neo4j - only embeddings and metadata |
| Embeddings | Stored as 384-dimension float arrays on nodes |
| Configuration | Never in code - always environment variables |

### Requirements to Structure Mapping

**Feature/FR Category Mapping:**

| FR Category | Primary Directory | Key Files |
|-------------|-------------------|-----------|
| Source Code Indexing (FR1-5) | `src/indexers/` | `source_code_indexer.py`, `angular_parser.py` |
| Documentation Indexing (FR6-8) | `src/indexers/` | `markdown_indexer.py` |
| Feature Context Queries (FR9-12) | `src/queries/` | `feature_context.py` |
| Semantic Search (FR13-15) | `src/queries/` | `semantic_search.py` |
| Impact Analysis (FR16-19) | `src/queries/` | `impact_analysis.py` |
| Dependency Mapping (FR20-22) | `src/queries/`, `src/indexers/` | `dependency_graph.py`, `nx_graph_importer.py` |
| MCP Protocol (FR23-25) | `src/mcp/` | `server.py`, `tools/*.py` |
| Configuration (FR26-28) | `src/config/` | `settings.py` |
| Knowledge Graph Schema (FR29-32) | `scripts/cypher/` | `*.cypher` |

**Cross-Cutting Concerns:**

| Concern | Location | Key Files |
|---------|----------|-----------|
| Error Handling | `src/utils/` | `exceptions.py` |
| Neo4j Connection | `src/db/` | `neo4j_client.py` |
| Embedding Generation | `src/db/` | `embeddings.py` |
| Configuration | `src/config/` | `settings.py` |

### Integration Points

**Internal Communication:**
- All MCP tools call into `indexers/` or `queries/` modules
- `indexers/` and `queries/` use `db/neo4j_client.py` for database access
- `db/embeddings.py` provides embedding generation for both indexing and querying

**External Integrations:**

| Integration | Connection | Configuration |
|-------------|------------|---------------|
| Neo4j Database | Bolt protocol | `NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD` |
| VS Code MCP | stdio (process communication) | `.vscode/mcp.json` |
| SentenceTransformer | Python library (local) | `EMBEDDING_MODEL` env var |

**Data Flow:**

```
┌─────────────────┐    ┌──────────────┐    ┌─────────────┐
│  Source Files   │───▶│  CocoIndex   │───▶│   Neo4j     │
│  (Angular/NX)   │    │  Pipeline    │    │  Knowledge  │
│                 │    │              │    │   Graph     │
└─────────────────┘    └──────────────┘    └─────────────┘
                              │                   ▲
                              ▼                   │
                       ┌──────────────┐    ┌──────┴──────┐
                       │ Sentence     │    │ MCP Server  │
                       │ Transformer  │    │ (Queries)   │
                       └──────────────┘    └─────────────┘
                                                  ▲
                                                  │
                                           ┌──────┴──────┐
                                           │ GitHub      │
                                           │ Copilot     │
                                           └─────────────┘
```

---

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
- Python + FastMCP + Neo4j 5.x are all compatible
- CocoIndex has native Neo4j target support
- SentenceTransformer embedding dimensions (384) match vector index configuration
- All technology versions are current and stable (December 2024)

**Pattern Consistency:**
- snake_case naming consistent across Python code and API
- MCP tool definitions follow PRD specifications exactly
- Error handling patterns support MCP protocol requirements
- Logging patterns enable debugging across all components

**Structure Alignment:**
- Project structure separates MCP layer, core logic, and data access
- Clear boundaries between indexing and querying components
- Test structure mirrors source structure for easy navigation
- Configuration centralized in `src/config/`

### Requirements Coverage Validation ✅

**Functional Requirements Coverage:**

| Category | FRs | Status | Notes |
|----------|-----|--------|-------|
| Source Code Indexing | FR1-FR5 | ✅ Covered | `indexers/source_code_indexer.py`, `angular_parser.py` |
| Documentation Indexing | FR6-FR8 | ✅ Covered | `indexers/markdown_indexer.py` |
| Feature Context Queries | FR9-FR12 | ✅ Covered | `queries/feature_context.py` |
| Semantic Search | FR13-FR15 | ✅ Covered | `queries/semantic_search.py` |
| Impact Analysis | FR16-FR19 | ✅ Covered | `queries/impact_analysis.py` |
| Dependency Mapping | FR20-FR22 | ✅ Covered | `queries/dependency_graph.py`, `indexers/nx_graph_importer.py` |
| MCP Protocol | FR23-FR25 | ✅ Covered | `mcp/server.py`, `mcp/tools/*.py` |
| Configuration | FR26-FR28 | ✅ Covered | `config/settings.py`, environment variables |
| Knowledge Graph Schema | FR29-FR32 | ✅ Covered | `scripts/cypher/*.cypher` |

**Non-Functional Requirements Coverage:**

| NFR | Target | Architectural Support |
|-----|--------|----------------------|
| NFR1 (Query < 500ms) | ✅ | Vector indexes, optimized Cypher queries |
| NFR2 (Full index < 5min) | ✅ | CocoIndex Rust performance, batch operations |
| NFR3 (Incremental < 10s) | ✅ | CocoIndex change detection |
| NFR4 (Vector search < 200ms) | ✅ | Native Neo4j vector indexes |
| NFR5 (80% relevance) | ✅ | Hybrid search (vector + graph) |
| NFR6 (100% completeness) | ✅ | CocoIndex with comprehensive file patterns |
| NFR7 (100% feature coverage) | ✅ | Feature-to-code relationship mapping |
| NFR8 (Graceful degradation) | ✅ | Exception hierarchy with fallback patterns |
| NFR9 (Env credentials) | ✅ | `python-dotenv`, never in code |
| NFR10 (Bolt protocol) | ✅ | Neo4j Python driver |
| NFR11 (Local source code) | ✅ | Only embeddings stored, source stays local |
| NFR12 (MCP compliance) | ✅ | FastMCP handles protocol |
| NFR13 (VS Code integration) | ✅ | Standard MCP configuration |
| NFR14 (Neo4j 5.x) | ✅ | Neo4j 5.x+ with vector index support |

### Implementation Readiness Validation ✅

**Decision Completeness:**
- All critical technology decisions documented with specific versions
- Implementation patterns comprehensive with examples
- Consistency rules enforceable via linting and pre-commit hooks
- Examples provided for all major patterns

**Structure Completeness:**
- Complete project tree with all files and directories
- All required configuration files specified
- Test structure defined with fixtures
- Deployment scripts for schema and NX import

**Pattern Completeness:**
- Naming conventions for DB, API, and code fully specified
- Error handling patterns with custom exception hierarchy
- Logging patterns with structured format
- Neo4j connection pattern with context manager

### Gap Analysis Results

**No Critical Gaps Identified** ✅

**Minor Areas for Future Enhancement:**
1. Monitoring/metrics collection (post-MVP)
2. Rate limiting for MCP tools (post-MVP)
3. Caching layer for frequent queries (post-MVP)
4. Git hook automation scripts (Phase 4)

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed (32 FRs, 14 NFRs)
- [x] Scale and complexity assessed (Medium, 4 developers)
- [x] Technical constraints identified (Neo4j external, MCP protocol, embedding dimensions)
- [x] Cross-cutting concerns mapped (error handling, logging, config, embeddings)

**✅ Architectural Decisions**
- [x] Critical decisions documented with versions (Neo4j 5.x, FastMCP, all-MiniLM-L6-v2)
- [x] Technology stack fully specified
- [x] Integration patterns defined (MCP, Bolt, Python library calls)
- [x] Performance considerations addressed (vector indexes, batch operations)

**✅ Implementation Patterns**
- [x] Naming conventions established (snake_case, PascalCase rules)
- [x] Structure patterns defined (feature-based, co-located tests)
- [x] Communication patterns specified (logging format, error responses)
- [x] Process patterns documented (error handling, connection management)

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established (MCP, Core, Data layers)
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION ✅

**Confidence Level:** High - All validation checks passed

**Key Strengths:**
1. Clear technology decisions with specific versions
2. Comprehensive patterns preventing AI agent conflicts
3. Well-defined project structure with clear boundaries
4. Complete requirements coverage verified

**Areas for Future Enhancement:**
1. Monitoring and metrics (post-MVP)
2. Caching layer for performance optimization (post-MVP)
3. Git hook automation (Phase 4)

---

## Architecture Completion Summary

### Workflow Completion

**Architecture Decision Workflow:** COMPLETED ✅  
**Total Steps Completed:** 8  
**Date Completed:** 2024-12-14  
**Document Location:** `docs/architecture.md`

### Final Architecture Deliverables

**📋 Complete Architecture Document**
- All architectural decisions documented with specific versions
- Implementation patterns ensuring AI agent consistency
- Complete project structure with all files and directories
- Requirements to architecture mapping
- Validation confirming coherence and completeness

**🏗️ Implementation Ready Foundation**
- 15+ architectural decisions made
- 12 implementation patterns defined
- 6 major architectural components specified
- 46 requirements fully supported (32 FR + 14 NFR)

**📚 AI Agent Implementation Guide**
- Technology stack with verified versions
- Consistency rules that prevent implementation conflicts
- Project structure with clear boundaries
- Integration patterns and communication standards

### Implementation Handoff

**For AI Agents:**
This architecture document is your complete guide for implementing knowledge-base. Follow all decisions, patterns, and structures exactly as documented.

**First Implementation Priority:**

```bash
# 1. Create project structure
mkdir -p knowledge-base/{src/{mcp/tools,indexers,queries,db,config,utils},scripts/cypher,tests/{unit,integration,e2e,fixtures},docs}

# 2. Set up Neo4j (user's responsibility - choose one method)
# Option A: Neo4j Desktop - Download and create database from neo4j.com/download
# Option B: Neo4j AuraDB - Create cloud instance at neo4j.com/cloud/aura
# Option C: Docker - Run: docker run -d -p 7474:7474 -p 7687:7687 -e NEO4J_AUTH=neo4j/password neo4j:5
# Option D: Self-hosted - Install Neo4j 5.x+ on your server

# 3. Configure environment variables
cat > .env << EOF
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=your-password
NEO4J_DATABASE=neo4j
EMBEDDING_MODEL=sentence-transformers/all-MiniLM-L6-v2
EOF

# 4. Create Python environment and install dependencies
python -m venv .venv && source .venv/bin/activate
pip install fastmcp neo4j sentence-transformers cocoindex tree-sitter-typescript python-dotenv ruff mypy pytest
```

**Development Sequence:**
1. Initialize project using documented structure
2. Deploy Neo4j schema (`scripts/cypher/*.cypher`)
3. Implement data layer (`db/neo4j_client.py`, `db/embeddings.py`)
4. Build indexers (`indexers/*.py`)
5. Build queries (`queries/*.py`)
6. Implement MCP tools (`mcp/tools/*.py`)
7. Configure VS Code MCP integration

### Quality Assurance Checklist

**✅ Architecture Coherence**
- [x] All decisions work together without conflicts
- [x] Technology choices are compatible
- [x] Patterns support the architectural decisions
- [x] Structure aligns with all choices

**✅ Requirements Coverage**
- [x] All functional requirements are supported
- [x] All non-functional requirements are addressed
- [x] Cross-cutting concerns are handled
- [x] Integration points are defined

**✅ Implementation Readiness**
- [x] Decisions are specific and actionable
- [x] Patterns prevent agent conflicts
- [x] Structure is complete and unambiguous
- [x] Examples are provided for clarity

---

**Architecture Status:** READY FOR IMPLEMENTATION ✅

**Next Phase:** Begin implementation using the architectural decisions and patterns documented herein.

**Document Maintenance:** Update this architecture when major technical decisions are made during implementation.
