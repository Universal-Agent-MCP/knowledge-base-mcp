# Story 1.5: Create Basic MCP Server with Ping Tool

Status: ready-for-dev

## Story

As a **System Admin**,
I want a **running MCP server with a basic `ping` tool**,
So that **I can verify the MCP protocol is working correctly**.

## Acceptance Criteria

1. **Given** the MCP server is started with `python src/main.py`
   **When** an MCP client calls the `ping` tool
   **Then** it returns `{status: "ok", message: "knowledge-base MCP server is running"}`
   **And** the response follows MCP JSON-RPC 2.0 format

## Tasks / Subtasks

- [ ] Task 1: Implement MCP server setup (AC: #1)
  - [ ] Subtask 1.1: Create src/mcp/server.py with FastMCP instance
  - [ ] Subtask 1.2: Configure server name as "knowledge-base"
  - [ ] Subtask 1.3: Configure stdio transport for VS Code compatibility

- [ ] Task 2: Implement ping tool (AC: #1)
  - [ ] Subtask 2.1: Create src/mcp/tools/__init__.py
  - [ ] Subtask 2.2: Create src/mcp/tools/ping.py with ping tool
  - [ ] Subtask 2.3: Add proper type hints and docstring
  - [ ] Subtask 2.4: Return structured response format

- [ ] Task 3: Implement main.py entry point (AC: #1)
  - [ ] Subtask 3.1: Import and configure MCP server
  - [ ] Subtask 3.2: Register all tools
  - [ ] Subtask 3.3: Call mcp.run() to start server

- [ ] Task 4: Test MCP server manually (AC: #1)
  - [ ] Subtask 4.1: Start server with python src/main.py
  - [ ] Subtask 4.2: Verify no errors on startup
  - [ ] Subtask 4.3: Test with MCP client or JSON-RPC call

## Dev Notes

### FastMCP Server Setup

```python
# src/mcp/server.py
from fastmcp import FastMCP

# Create MCP server instance
mcp = FastMCP("knowledge-base")

def get_mcp_server() -> FastMCP:
    """Get the MCP server instance."""
    return mcp
```

### Ping Tool Implementation

```python
# src/mcp/tools/ping.py
from src.mcp.server import mcp

@mcp.tool()
def ping() -> dict:
    """
    Health check tool to verify MCP server is running.
    
    Returns a status object indicating the server is operational.
    This tool can be used by MCP clients to verify connectivity.
    
    Returns:
        dict: Status object with 'status' and 'message' fields
    """
    return {
        "status": "ok",
        "message": "knowledge-base MCP server is running"
    }
```

### Main Entry Point

```python
# src/main.py
"""MCP Server entry point for knowledge-base."""
import logging

# Import tools to register them with the server
from src.mcp.tools import ping  # noqa: F401
from src.mcp.server import get_mcp_server

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)

logger = logging.getLogger(__name__)

def main() -> None:
    """Start the MCP server."""
    logger.info("Starting knowledge-base MCP server...")
    mcp = get_mcp_server()
    mcp.run()

if __name__ == "__main__":
    main()
```

### Tools __init__.py

```python
# src/mcp/tools/__init__.py
"""MCP Tools for knowledge-base."""
from src.mcp.tools.ping import ping

__all__ = ["ping"]
```

### Response Format (from Architecture)

All MCP tool responses follow this structure:
```python
{
    "status": "success" | "error",
    "data": {...},  # Tool-specific data
    "metadata": {
        "timestamp": "2024-12-14T10:30:00Z",
        ...
    }
}
```

For the ping tool, simplified response:
```python
{
    "status": "ok",
    "message": "knowledge-base MCP server is running"
}
```

### MCP Protocol Notes

- **Transport:** stdio (standard input/output) for VS Code compatibility
- **Protocol:** JSON-RPC 2.0 (handled by FastMCP)
- **Tool Discovery:** FastMCP auto-generates tool schemas from type hints

### Testing the Server

```bash
# Start the server
python src/main.py

# In another terminal, test with MCP inspector or:
echo '{"jsonrpc":"2.0","method":"tools/call","params":{"name":"ping","arguments":{}},"id":1}' | python src/main.py
```

### Previous Story Dependencies

- Story 1-1: Creates project structure and main.py skeleton
- Story 1-2: Provides settings (not used by ping, but available)
- Story 1-4: Neo4j client available for future tools

### References

- [Source: docs/architecture.md#API-&-Communication-Patterns] - MCP protocol requirements
- [Source: docs/architecture.md#Starter-Template-Evaluation] - FastMCP selection
- [Source: docs/epics.md#Story-1.5] - Original acceptance criteria

## Dev Agent Record

### Context Reference

First functional MCP tool. Validates MCP protocol is working before adding complex tools.

### Agent Model Used

Claude (Anthropic) - Scrum Master Agent

### Completion Notes List

- Implements FR23 (expose tools via MCP protocol)
- Validates NFR12 (MCP JSON-RPC 2.0 compatibility)
- Foundation for all future MCP tools

### File List

Files to create/modify:
- `src/mcp/server.py` - FastMCP server instance
- `src/mcp/tools/__init__.py` - Tools package
- `src/mcp/tools/ping.py` - Ping tool implementation
- `src/main.py` - Entry point (modify to use server)
