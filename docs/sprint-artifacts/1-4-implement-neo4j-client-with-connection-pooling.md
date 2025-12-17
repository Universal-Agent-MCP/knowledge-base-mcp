---
beads_id: knowledge-base-1ju
beads_parent: knowledge-base-u9s
last_synced: 2025-12-18T00:36:00+07:00
---

# Story 1.4: Implement Neo4j Client with Connection Pooling

Status: ready-for-dev

## Story

As a **Developer**,
I want a **Neo4j client with connection pooling and context manager pattern**,
So that **I can safely execute Cypher queries with proper resource cleanup**.

## Acceptance Criteria

1. **Given** valid Neo4j credentials in environment variables
   **When** I use `with neo4j_client.session() as session:` pattern
   **Then** the session is automatically closed after the block
   **And** connection errors raise `ConnectionError` with helpful messages
   **And** the client supports both sync and async operations

## Tasks / Subtasks

- [ ] Task 1: Implement Neo4jClient class (AC: #1)
  - [ ] Subtask 1.1: Create src/db/neo4j_client.py
  - [ ] Subtask 1.2: Implement __init__ with driver creation
  - [ ] Subtask 1.3: Implement session() context manager
  - [ ] Subtask 1.4: Implement close() method

- [ ] Task 2: Implement async support (AC: #1)
  - [ ] Subtask 2.1: Implement async_session() context manager
  - [ ] Subtask 2.2: Use AsyncGraphDatabase for async driver
  - [ ] Subtask 2.3: Ensure proper async resource cleanup

- [ ] Task 3: Implement error handling (AC: #1)
  - [ ] Subtask 3.1: Catch neo4j.exceptions and wrap in ConnectionError
  - [ ] Subtask 3.2: Add helpful error messages with URI (masked password)
  - [ ] Subtask 3.3: Implement connection verification on init

- [ ] Task 4: Implement singleton pattern (AC: #1)
  - [ ] Subtask 4.1: Create get_neo4j_client() function
  - [ ] Subtask 4.2: Cache client instance for reuse
  - [ ] Subtask 4.3: Handle graceful shutdown

- [ ] Task 5: Write unit tests (AC: #1)
  - [ ] Subtask 5.1: Test session context manager cleanup
  - [ ] Subtask 5.2: Test connection error handling
  - [ ] Subtask 5.3: Test singleton behavior

## Dev Notes

### Neo4jClient Implementation (from Architecture)

```python
# src/db/neo4j_client.py
from contextlib import contextmanager, asynccontextmanager
from functools import lru_cache
import logging
from typing import AsyncGenerator, Generator

from neo4j import GraphDatabase, AsyncGraphDatabase, Session, AsyncSession
from neo4j.exceptions import ServiceUnavailable, AuthError

from src.config.settings import get_settings
from src.utils.exceptions import ConnectionError

logger = logging.getLogger(__name__)


class Neo4jClient:
    """Neo4j database client with connection pooling."""

    def __init__(self, uri: str, user: str, password: str, database: str = "neo4j"):
        """Initialize Neo4j connection with connection pooling."""
        self._uri = uri
        self._database = database

        try:
            self._driver = GraphDatabase.driver(uri, auth=(user, password))
            self._async_driver = AsyncGraphDatabase.driver(uri, auth=(user, password))
            self._verify_connection()
            logger.info(f"Connected to Neo4j at {self._mask_uri(uri)}")
        except AuthError as e:
            raise ConnectionError(f"Neo4j authentication failed: {e}")
        except ServiceUnavailable as e:
            raise ConnectionError(f"Neo4j service unavailable at {self._mask_uri(uri)}: {e}")
        except Exception as e:
            raise ConnectionError(f"Failed to connect to Neo4j: {e}")

    def _verify_connection(self) -> None:
        """Verify database connection is working."""
        self._driver.verify_connectivity()

    def _mask_uri(self, uri: str) -> str:
        """Mask sensitive parts of URI for logging."""
        return uri.replace("://", "://<masked>@") if "@" in uri else uri

    @contextmanager
    def session(self) -> Generator[Session, None, None]:
        """Get a session with automatic cleanup."""
        session = self._driver.session(database=self._database)
        try:
            yield session
        finally:
            session.close()

    @asynccontextmanager
    async def async_session(self) -> AsyncGenerator[AsyncSession, None]:
        """Get an async session with automatic cleanup."""
        session = self._async_driver.session(database=self._database)
        try:
            yield session
        finally:
            await session.close()

    def close(self) -> None:
        """Close all connections."""
        self._driver.close()
        logger.info("Neo4j connection closed")

    async def async_close(self) -> None:
        """Close async driver."""
        await self._async_driver.close()


@lru_cache()
def get_neo4j_client() -> Neo4jClient:
    """Get or create singleton Neo4j client."""
    settings = get_settings()
    return Neo4jClient(
        uri=settings.neo4j_uri,
        user=settings.neo4j_user,
        password=settings.neo4j_password,
        database=settings.neo4j_database,
    )
```

### Usage Pattern

```python
from src.db.neo4j_client import get_neo4j_client

# Sync usage
client = get_neo4j_client()
with client.session() as session:
    result = session.run("MATCH (n) RETURN count(n) as count")
    count = result.single()["count"]

# Async usage
async def query_async():
    client = get_neo4j_client()
    async with client.async_session() as session:
        result = await session.run("MATCH (n) RETURN count(n) as count")
        record = await result.single()
        return record["count"]
```

### Connection Pooling Configuration

The Neo4j Python driver has built-in connection pooling with these defaults:
- Max connection lifetime: 1 hour
- Max connection pool size: 100
- Connection acquisition timeout: 60 seconds

For custom configuration:
```python
self._driver = GraphDatabase.driver(
    uri,
    auth=(user, password),
    max_connection_lifetime=3600,
    max_connection_pool_size=50,
)
```

### Error Messages Pattern

```python
# Good error messages:
"Neo4j authentication failed: Invalid credentials for user 'neo4j'"
"Neo4j service unavailable at bolt://localhost:7687: Connection refused"
"Failed to connect to Neo4j: Database 'knowledge' does not exist"
```

### Graceful Shutdown

```python
# In main.py or shutdown handler
import atexit

def shutdown():
    try:
        client = get_neo4j_client()
        client.close()
    except Exception:
        pass  # Client may not be initialized

atexit.register(shutdown)
```

### Previous Story Dependencies

- Story 1-2: Provides get_settings() with Neo4j credentials
- Story 1-3: Schema should be deployed before using client

### References

- [Source: docs/architecture.md#Process-Patterns] - Neo4j connection pattern
- [Source: docs/architecture.md#Cross-Cutting-Concerns] - Graph connection pooling
- [Source: docs/epics.md#Story-1.4] - Original acceptance criteria

## Dev Agent Record

### Context Reference

Core infrastructure component used by all MCP tools that query Neo4j.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Completion Notes List

- Used by all query and indexing operations
- Implements NFR8 (graceful degradation)
- Connection pooling improves performance

### File List

Files to create:
- `src/db/neo4j_client.py` - Client implementation
- `tests/db/test_neo4j_client.py` - Unit tests
