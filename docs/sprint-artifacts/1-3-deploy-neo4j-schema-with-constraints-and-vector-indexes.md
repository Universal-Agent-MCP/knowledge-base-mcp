# Story 1.3: Deploy Neo4j Schema with Constraints and Vector Indexes

Status: ready-for-dev

## Story

As a **System Admin**,
I want to **deploy the knowledge graph schema to Neo4j**,
So that **the database is ready to store code entities and feature embeddings**.

## Acceptance Criteria

1. **Given** I have a running Neo4j 5.x instance with vector index support
   **When** I run `python scripts/deploy_schema.py`
   **Then** node constraints are created (Project, Library, Component, Service, Feature, Document)
   **And** vector indexes are created for embeddings (384 dimensions, cosine similarity)
   **And** the script is idempotent (can run multiple times safely)

## Tasks / Subtasks

- [ ] Task 1: Create Cypher schema scripts (AC: #1)
  - [ ] Subtask 1.1: Create `scripts/cypher/001_create_constraints.cypher`
  - [ ] Subtask 1.2: Create `scripts/cypher/002_create_vector_indexes.cypher`
  - [ ] Subtask 1.3: Ensure all scripts use IF NOT EXISTS for idempotency

- [ ] Task 2: Implement deploy_schema.py script (AC: #1)
  - [ ] Subtask 2.1: Load Neo4j connection from settings
  - [ ] Subtask 2.2: Read and execute Cypher files in order
  - [ ] Subtask 2.3: Handle connection errors gracefully
  - [ ] Subtask 2.4: Log each constraint/index creation

- [ ] Task 3: Implement schema verification (AC: #1)
  - [ ] Subtask 3.1: Query existing constraints after deployment
  - [ ] Subtask 3.2: Query existing indexes after deployment
  - [ ] Subtask 3.3: Report success/failure with details

- [ ] Task 4: Test idempotency (AC: #1)
  - [ ] Subtask 4.1: Run script twice consecutively
  - [ ] Subtask 4.2: Verify no errors on second run
  - [ ] Subtask 4.3: Verify schema unchanged after second run

## Dev Notes

### Node Constraints (from Architecture)

```cypher
-- scripts/cypher/001_create_constraints.cypher
CREATE CONSTRAINT Project_name IF NOT EXISTS FOR (p:Project) REQUIRE p.name IS UNIQUE;
CREATE CONSTRAINT Library_name IF NOT EXISTS FOR (l:Library) REQUIRE l.name IS UNIQUE;
CREATE CONSTRAINT Component_path IF NOT EXISTS FOR (c:Component) REQUIRE c.path IS UNIQUE;
CREATE CONSTRAINT Service_path IF NOT EXISTS FOR (s:Service) REQUIRE s.path IS UNIQUE;
CREATE CONSTRAINT Feature_name IF NOT EXISTS FOR (f:Feature) REQUIRE f.name IS UNIQUE;
CREATE CONSTRAINT Document_path IF NOT EXISTS FOR (d:Document) REQUIRE d.path IS UNIQUE;
CREATE CONSTRAINT Directive_path IF NOT EXISTS FOR (d:Directive) REQUIRE d.path IS UNIQUE;
CREATE CONSTRAINT Pipe_path IF NOT EXISTS FOR (p:Pipe) REQUIRE p.path IS UNIQUE;
CREATE CONSTRAINT CodeChunk_id IF NOT EXISTS FOR (cc:CodeChunk) REQUIRE cc.id IS UNIQUE;
CREATE CONSTRAINT App_name IF NOT EXISTS FOR (a:App) REQUIRE a.name IS UNIQUE;
```

### Vector Indexes (from Architecture)

```cypher
-- scripts/cypher/002_create_vector_indexes.cypher
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

### deploy_schema.py Implementation

```python
#!/usr/bin/env python3
"""Deploy Neo4j schema constraints and vector indexes."""
import logging
from pathlib import Path

from neo4j import GraphDatabase

from src.config.settings import get_settings
from src.utils.exceptions import ConnectionError

logger = logging.getLogger(__name__)

CYPHER_DIR = Path(__file__).parent / "cypher"

def deploy_schema() -> None:
    """Deploy all Cypher schema scripts in order."""
    settings = get_settings()
    
    try:
        driver = GraphDatabase.driver(
            settings.neo4j_uri,
            auth=(settings.neo4j_user, settings.neo4j_password)
        )
        driver.verify_connectivity()
    except Exception as e:
        raise ConnectionError(f"Failed to connect to Neo4j: {e}")
    
    try:
        with driver.session(database=settings.neo4j_database) as session:
            # Execute Cypher scripts in order
            for cypher_file in sorted(CYPHER_DIR.glob("*.cypher")):
                logger.info(f"Executing: {cypher_file.name}")
                cypher = cypher_file.read_text()
                
                for statement in cypher.split(";"):
                    statement = statement.strip()
                    if statement and not statement.startswith("--"):
                        session.run(statement)
                        logger.debug(f"Executed: {statement[:50]}...")
                
                logger.info(f"Completed: {cypher_file.name}")
            
            # Verify schema
            verify_schema(session)
    finally:
        driver.close()

def verify_schema(session) -> None:
    """Verify constraints and indexes exist."""
    constraints = session.run("SHOW CONSTRAINTS").data()
    indexes = session.run("SHOW INDEXES WHERE type = 'VECTOR'").data()
    
    logger.info(f"Schema deployed: {len(constraints)} constraints, {len(indexes)} vector indexes")
    
    for c in constraints:
        logger.debug(f"  Constraint: {c.get('name')}")
    for i in indexes:
        logger.debug(f"  Vector index: {i.get('name')}")

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    deploy_schema()
    print("✓ Schema deployment complete")
```

### Neo4j Requirements

- **Version:** Neo4j 5.x or higher (required for vector indexes)
- **Plugins:** APOC optional but recommended
- **Memory:** Minimum 2GB heap

### Idempotency Pattern

All Cypher statements use `IF NOT EXISTS` to ensure:
- Script can run multiple times safely
- Existing data is not affected
- No errors on duplicate creation attempts

### Relationship Types (for reference)

| Relationship | From | To | Description |
|--------------|------|-----|-------------|
| DEPENDS_ON | Library | Library | NX library dependencies |
| EXPORTS | Library | Component/Service | Library exports entity |
| USES_SERVICE | Component | Service | Component injects service |
| IMPLEMENTED_BY | Feature | Component/Service | Feature implementation |
| DOCUMENTED_BY | Component/Service | Document | Code documentation link |
| CONTAINS | Document | Feature | Document contains feature |

### Previous Story Dependencies

- Story 1-1: Creates scripts/cypher/ directory
- Story 1-2: Provides get_settings() for Neo4j credentials

### References

- [Source: docs/architecture.md#Data-Architecture] - Complete Neo4j schema
- [Source: docs/architecture.md#Neo4j-Schema] - Constraint and index definitions
- [Source: docs/epics.md#Story-1.3] - Original acceptance criteria

## Dev Agent Record

### Context Reference

Uses settings from Story 1-2. Creates database schema for all subsequent stories.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Completion Notes List

- Critical foundational story - all indexing depends on this schema
- Vector indexes enable semantic search (NFR4)
- Implements FR29-FR32 (knowledge graph schema)

### File List

Files to create:
- `scripts/cypher/001_create_constraints.cypher`
- `scripts/cypher/002_create_vector_indexes.cypher`
- `scripts/deploy_schema.py`
