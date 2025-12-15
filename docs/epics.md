---
stepsCompleted: [1, 2, 3, 4]
inputDocuments:
  - docs/prd.md
  - docs/architecture.md
---

# knowledge-base - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for knowledge-base, decomposing the requirements from the PRD and Architecture into implementable stories.

## Requirements Inventory

### Functional Requirements

**Source Code Indexing (FR1-FR5)**
- FR1: System Admin can index Angular/TypeScript source files from an NX monorepo into the knowledge graph
- FR2: System Admin can specify file paths or glob patterns for selective indexing
- FR3: System Admin can filter indexing by NX library scope (e.g., only index `feature-auth`)
- FR4: System can parse Angular decorators (@Component, @Injectable, @Directive, @Pipe) to identify entity types
- FR5: System can generate vector embeddings for indexed code chunks

**Documentation Indexing (FR6-FR8)**
- FR6: System Admin can index markdown specification documents into the knowledge graph
- FR7: System can extract feature descriptions and generate embeddings for semantic search
- FR8: System can create relationships between documentation and related code entities

**Feature Context Queries (FR9-FR12)**
- FR9: Developers can query feature context by name or description
- FR10: System can return related components, services, and libraries for a queried feature
- FR11: System can include linked documentation in feature context responses
- FR12: Developers can specify relationship traversal depth for context queries

**Semantic Search (FR13-FR15)**
- FR13: Developers can search for features by semantic similarity (meaning, not just keywords)
- FR14: System can return ranked results with similarity scores
- FR15: System can limit search results to a specified number

**Impact Analysis (FR16-FR19)**
- FR16: Developers can analyze which features are affected by changes to a specific feature
- FR17: System can traverse dependency graph to identify affected components
- FR18: Developers can optionally target specific NX libraries for impact analysis
- FR19: System can show affected exports and relationship distances

**Dependency Mapping (FR20-FR22)**
- FR20: Developers can retrieve explicit dependencies between features
- FR21: System can return dependency graph structure for visualization
- FR22: System can import NX project graph (apps, libraries, dependencies) into Neo4j

**MCP Protocol Integration (FR23-FR25)**
- FR23: System can expose all tools via standard MCP protocol (JSON-RPC 2.0)
- FR24: System can provide structured JSON responses compatible with AI agents
- FR25: AI agents (Copilot, Claude) can call MCP tools and receive context automatically

**Configuration & Setup (FR26-FR28)**
- FR26: System Admin can configure Neo4j connection via environment variables
- FR27: System Admin can install the MCP server via pip (requirements.txt)
- FR28: System Admin can configure IDE MCP integration via standard configuration files

**Knowledge Graph Schema (FR29-FR32)**
- FR29: System can store NX project structure (Project, App, Library nodes)
- FR30: System can store Angular entities (Component, Service, Directive, Pipe nodes)
- FR31: System can store feature and document nodes with embeddings
- FR32: System can create relationships (DEPENDS_ON, EXPORTS, USES_SERVICE, IMPLEMENTED_BY, etc.)

### NonFunctional Requirements

**Performance (NFR1-NFR4)**
- NFR1: Query response time < 500ms for context retrieval (P0)
- NFR2: Full indexing time < 5 minutes for complete repo (P0)
- NFR3: Incremental indexing time < 10 seconds for changed files (P1)
- NFR4: Vector search latency < 200ms for semantic queries (P1)

**Reliability (NFR5-NFR8)**
- NFR5: Context relevance accuracy > 80% of retrieved code chunks relevant (P0)
- NFR6: Indexing completeness 100% of source code entities indexed (P0)
- NFR7: Feature coverage 100% of PRD features linked to code (P0)
- NFR8: Graceful degradation on Neo4j connection issues (P1)

**Security (NFR9-NFR11)**
- NFR9: All credentials via environment variables, never in code (P0)
- NFR10: Neo4j connections use bolt or bolt+s protocols (P0)
- NFR11: Source code stays local, only embeddings/metadata stored (P0)

**Integration (NFR12-NFR14)**
- NFR12: Full MCP JSON-RPC 2.0 compatibility (P0)
- NFR13: Works with VS Code MCP configuration in user settings (P0)
- NFR14: Supports Neo4j 5.x with vector index support (P0)

### Additional Requirements

**Starter Template (Architecture-specified)**
- FastMCP Python Project initialization with: fastmcp, neo4j, sentence-transformers, cocoindex, tree-sitter-typescript, python-dotenv

**Infrastructure Requirements**
- Python 3.10+ runtime with async/await patterns
- PyProject.toml with full dependency specification
- `.env.example` configuration template
- `scripts/cypher/` for Neo4j schema deployment (constraints + vector indexes)
- `docker/docker-compose.yml` for optional Neo4j containerization

**Technical Implementation Requirements**
- Feature-based project organization with co-located tests (`_test.py` suffix)
- Custom exception hierarchy (KnowledgeBaseError, IndexingError, QueryError, ConnectionError)
- Structured logging with DEBUG/INFO/WARNING/ERROR levels
- Neo4j connection pooling with context manager pattern
- Type hints on all function signatures
- Consistent API response format: `{status, data, metadata, error}`

**Code Quality Requirements**
- Linting with `ruff`
- Type checking with `mypy`
- Pre-commit hooks for automated checks

### FR Coverage Map

| FR | Epic | Description |
|----|------|-------------|
| FR1 | Epic 2 | Index Angular/TypeScript source files |
| FR2 | Epic 2 | Specify file paths or glob patterns |
| FR3 | Epic 2 | Filter by NX library scope |
| FR4 | Epic 2 | Parse Angular decorators |
| FR5 | Epic 2 | Generate vector embeddings for code |
| FR6 | Epic 3 | Index markdown specs |
| FR7 | Epic 3 | Extract features and generate embeddings |
| FR8 | Epic 3 | Create doc-to-code relationships |
| FR9 | Epic 4 | Query feature context by name |
| FR10 | Epic 4 | Return related components/services |
| FR11 | Epic 4 | Include linked documentation |
| FR12 | Epic 4 | Specify traversal depth |
| FR13 | Epic 4 | Search by semantic similarity |
| FR14 | Epic 4 | Return ranked results with scores |
| FR15 | Epic 4 | Limit search results |
| FR16 | Epic 5 | Analyze affected features |
| FR17 | Epic 5 | Traverse dependency graph |
| FR18 | Epic 5 | Target specific NX libraries |
| FR19 | Epic 5 | Show affected exports/distances |
| FR20 | Epic 5 | Retrieve feature dependencies |
| FR21 | Epic 5 | Return graph structure |
| FR22 | Epic 2 | Import NX project graph |
| FR23 | Epic 1 | Expose tools via MCP protocol (basic server setup) |
| FR24 | Epic 6 | Structured JSON responses |
| FR25 | Epic 6 | AI agents call tools automatically |
| FR26 | Epic 1 | Configure Neo4j via env vars |
| FR27 | Epic 1 | Install via pip |
| FR28 | Epic 1 | IDE MCP configuration |
| FR29 | Epic 1 | Store NX project structure |
| FR30 | Epic 1 | Store Angular entities |
| FR31 | Epic 1 | Store feature/document nodes |
| FR32 | Epic 1 | Create relationships |

---

## Epic List

### Epic 1: Project Foundation & Neo4j Integration

**User Outcome:** System Admin can install the MCP server, configure Neo4j connection, and have the knowledge graph schema deployed with a running MCP server ready for tools.

**FRs covered:** FR23, FR26, FR27, FR28, FR29, FR30, FR31, FR32
**NFRs addressed:** NFR9, NFR10, NFR12, NFR14

**Demo Outcome:** *'Hello World' MCP Server + Schema Verification*
- Run `python src/main.py` → MCP server starts
- Execute placeholder tool `ping` → Returns `{status: 'ok'}`
- Show Neo4j Browser with schema constraints and vector indexes created
- Run verify script showing all node types and indexes exist

**Technical Validation:** All Cypher scripts execute without error

---

### Epic 2: Source Code Indexing

**User Outcome:** System Admin can index Angular/TypeScript source code from an NX monorepo, with code parsed, entities identified, embeddings generated, and NX project graph imported.

**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR22
**NFRs addressed:** NFR2, NFR6, NFR11

**Demo Outcome:** *Index a Sample NX Library*
- Run `index_source_code` on a sample Angular library
- Show Neo4j graph visualization of indexed components, services
- Query: `MATCH (c:Component) RETURN c.name LIMIT 5`
- Show embedding vectors exist on nodes

**Technical Validation:** Benchmark: 1000 files indexed in < 60s

---

### Epic 3: Documentation Indexing

**User Outcome:** System Admin can index markdown specification documents with semantic embeddings, creating relationships between documentation and code entities.

**FRs covered:** FR6, FR7, FR8
**NFRs addressed:** NFR7

**Demo Outcome:** *Index Markdown Specs + Code Linking*
- Run `index_markdown_specs` on sample spec documents
- Show Document nodes with embeddings in Neo4j
- Show DOCUMENTED_BY relationships between specs and code
- Visual proof of the code-to-doc bridge

**Technical Validation:** At least 3 DOCUMENTED_BY relationships created

---

### Epic 4: Feature Context & Semantic Search

**User Outcome:** Developers can query feature context and search for similar features using natural language, getting complete context with relationships.

**FRs covered:** FR9, FR10, FR11, FR12, FR13, FR14, FR15
**NFRs addressed:** NFR1, NFR4, NFR5

**Demo Outcome:** *Developer Query Experience*
- Ask: 'What is User Authentication?' → Full context returned
- Ask: 'Find features similar to login' → Ranked semantic results
- Show AI agent (simulated) receiving structured JSON context

**Technical Validation:** Query latency < 500ms (NFR1 validated)

---

### Epic 5: Impact Analysis & Dependencies

**User Outcome:** Tech Leads can analyze feature impact and dependencies, understanding what will break when making changes.

**FRs covered:** FR16, FR17, FR18, FR19, FR20, FR21
**NFRs addressed:** NFR1, NFR8

**Demo Outcome:** *Tech Lead Change Assessment*
- Query: 'What breaks if I change AuthService?' → Impact analysis
- Show dependency graph visualization
- Demonstrate affected exports and distances

**Technical Validation:** Graph traversal depth ≥ 3 hops tested

---

### Epic 6: Production Integration & Documentation

**User Outcome:** AI agents (GitHub Copilot, Claude) can automatically call MCP tools and receive structured context during development, with complete documentation for users.

**FRs covered:** FR24, FR25
**NFRs addressed:** NFR12, NFR13

**Demo Outcome:** *Full AI Agent Workflow*
- Configure VS Code MCP settings
- Ask GitHub Copilot a project question
- Watch it call MCP tools and provide context-aware response
- Show Quick Start guide completion time < 10 minutes

**Technical Validation:** VS Code MCP config works first try

---

## Epic 1: Project Foundation & Neo4j Integration

System Admin can install the MCP server, configure Neo4j connection, and have the knowledge graph schema deployed with a running MCP server ready for tools.

### Story 1.1: Initialize Python Project with MCP Server Skeleton

**As a** System Admin,
**I want** to clone the project and have a working Python environment with all dependencies installed,
**So that** I can start developing and running the MCP server.

**Acceptance Criteria:**

**Given** I have Python 3.10+ installed
**When** I run `pip install -r requirements.txt`
**Then** all dependencies (fastmcp, neo4j, sentence-transformers, cocoindex, python-dotenv) are installed
**And** I can run `python src/main.py` without import errors

---

### Story 1.2: Configure Environment Variables for Neo4j Connection

**As a** System Admin,
**I want** to configure Neo4j connection details via environment variables,
**So that** credentials are never stored in code and I can switch environments easily.

**Acceptance Criteria:**

**Given** I have a `.env.example` file in the project root
**When** I copy it to `.env` and fill in my Neo4j credentials (`NEO4J_URI`, `NEO4J_USER`, `NEO4J_PASSWORD`)
**Then** the application reads configuration from environment variables
**And** credentials are validated on startup with clear error messages if missing

---

### Story 1.3: Deploy Neo4j Schema with Constraints and Vector Indexes

**As a** System Admin,
**I want** to deploy the knowledge graph schema to Neo4j,
**So that** the database is ready to store code entities and feature embeddings.

**Acceptance Criteria:**

**Given** I have a running Neo4j 5.x instance with vector index support
**When** I run `python scripts/deploy_schema.py`
**Then** node constraints are created (Project, Library, Component, Service, Feature, Document)
**And** vector indexes are created for embeddings (384 dimensions, cosine similarity)
**And** the script is idempotent (can run multiple times safely)

---

### Story 1.4: Implement Neo4j Client with Connection Pooling

**As a** Developer,
**I want** a Neo4j client with connection pooling and context manager pattern,
**So that** I can safely execute Cypher queries with proper resource cleanup.

**Acceptance Criteria:**

**Given** valid Neo4j credentials in environment variables
**When** I use `with neo4j_client.session() as session:` pattern
**Then** the session is automatically closed after the block
**And** connection errors raise `ConnectionError` with helpful messages
**And** the client supports both sync and async operations

---

### Story 1.5: Create Basic MCP Server with Ping Tool

**As a** System Admin,
**I want** a running MCP server with a basic `ping` tool,
**So that** I can verify the MCP protocol is working correctly.

**Acceptance Criteria:**

**Given** the MCP server is started with `python src/main.py`
**When** an MCP client calls the `ping` tool
**Then** it returns `{status: "ok", message: "knowledge-base MCP server is running"}`
**And** the response follows MCP JSON-RPC 2.0 format

---

### Story 1.6: Setup Development Quality Tools

**As a** Developer,
**I want** linting (ruff), type checking (mypy), and pre-commit hooks configured,
**So that** code quality is enforced automatically.

**Acceptance Criteria:**

**Given** I have the project cloned
**When** I run `pre-commit install`
**Then** hooks are installed for ruff and mypy
**And** running `ruff check src/` passes with no errors
**And** running `mypy src/` passes with no type errors

---

## Epic 2: Source Code Indexing

System Admin can index Angular/TypeScript source code from an NX monorepo, with code parsed, entities identified, embeddings generated, and NX project graph imported.

### Story 2.1: Import NX Project Graph into Neo4j

**As a** System Admin,
**I want** to import the NX project graph (apps, libraries, dependencies) into Neo4j,
**So that** the knowledge graph understands the monorepo structure.

**Acceptance Criteria:**

**Given** an NX monorepo with `nx.json` and `project.json` files
**When** I run `index_source_code` with the project root path
**Then** Project, App, and Library nodes are created in Neo4j
**And** DEPENDS_ON relationships are created between libraries
**And** existing nodes are updated (not duplicated) on re-run

---

### Story 2.2: Parse Angular Decorators and Create Entity Nodes

**As a** System Admin,
**I want** the indexer to detect Angular decorators and create corresponding entity nodes,
**So that** components, services, directives, and pipes are properly identified in the graph.

**Acceptance Criteria:**

**Given** TypeScript files with Angular decorators (@Component, @Injectable, @Directive, @Pipe)
**When** the source code indexer processes these files
**Then** Component, Service, Directive, and Pipe nodes are created
**And** each node has properties: `name`, `path`, `selector` (for components), `providedIn` (for services)
**And** EXPORTS relationships link entities to their parent Library

---

### Story 2.3: Implement File Path and Glob Pattern Selection

**As a** System Admin,
**I want** to specify file paths or glob patterns for selective indexing,
**So that** I can index specific parts of the codebase as needed.

**Acceptance Criteria:**

**Given** the `index_source_code` tool with `paths` parameter
**When** I provide `["libs/feature-auth/**/*.ts"]` as the paths
**Then** only files matching the glob pattern are indexed
**And** other files in the monorepo are skipped
**And** the response shows count of indexed vs skipped files

---

### Story 2.4: Filter Indexing by NX Library Scope

**As a** System Admin,
**I want** to filter indexing to a specific NX library scope,
**So that** I can focus on a particular feature area.

**Acceptance Criteria:**

**Given** the `index_source_code` tool with `library_scope` parameter
**When** I provide `library_scope: "feature-auth"`
**Then** only files within that library are indexed
**And** the Library node is linked to all indexed entities
**And** other libraries are not affected

---

### Story 2.5: Generate Vector Embeddings for Code Chunks

**As a** System Admin,
**I want** vector embeddings generated for indexed code entities,
**So that** semantic search can find related code by meaning.

**Acceptance Criteria:**

**Given** indexed Component, Service, or other entity nodes
**When** the indexer processes the file
**Then** embeddings are generated using SentenceTransformer (all-MiniLM-L6-v2)
**And** embeddings are stored as 384-dimension float arrays on nodes
**And** indexing completes in < 5 minutes for 8000 files (NFR2)

---

### Story 2.6: Create CocoIndex ETL Pipeline for Source Code

**As a** Developer,
**I want** a CocoIndex pipeline that orchestrates the indexing process,
**So that** code parsing, embedding generation, and Neo4j insertion are coordinated.

**Acceptance Criteria:**

**Given** the CocoIndex library is installed
**When** `index_source_code` is called
**Then** the pipeline reads files, parses with tree-sitter, generates embeddings
**And** entities are batch-inserted into Neo4j for performance
**And** source code text stays local (only embeddings/metadata stored) - NFR11

---

## Epic 3: Documentation Indexing

System Admin can index markdown specification documents with semantic embeddings, creating relationships between documentation and code entities.

### Story 3.1: Index Markdown Specification Documents

**As a** System Admin,
**I want** to index markdown specification documents into the knowledge graph,
**So that** business context is available alongside code.

**Acceptance Criteria:**

**Given** markdown files in a specs directory (e.g., `docs/specs/*.md`)
**When** I run `index_markdown_specs` with the paths
**Then** Document nodes are created in Neo4j for each file
**And** each Document node has properties: `path`, `title`, `content_hash`
**And** existing documents are updated (not duplicated) on re-run

---

### Story 3.2: Extract Feature Descriptions from Markdown

**As a** System Admin,
**I want** the indexer to extract feature descriptions from markdown content,
**So that** individual features are identified and searchable.

**Acceptance Criteria:**

**Given** markdown documents with structured content (headings, sections)
**When** the markdown indexer processes the files
**Then** Feature nodes are created for identified features
**And** Feature nodes have properties: `name`, `description`, `source_document`
**And** CONTAINS relationships link Documents to their Features

---

### Story 3.3: Generate Embeddings for Document Content

**As a** System Admin,
**I want** vector embeddings generated for document and feature content,
**So that** semantic search finds related documentation.

**Acceptance Criteria:**

**Given** Document and Feature nodes in Neo4j
**When** the indexer processes markdown files
**Then** embeddings are generated using SentenceTransformer
**And** embeddings are stored on both Document and Feature nodes
**And** the document_embedding vector index is used for similarity search

---

### Story 3.4: Create Documentation-to-Code Relationships

**As a** System Admin,
**I want** explicit relationships created between documentation and code entities,
**So that** I can navigate from specs to implementation and vice versa.

**Acceptance Criteria:**

**Given** indexed code entities (Components, Services) and Documents
**When** the indexer detects references (e.g., component names in markdown)
**Then** DOCUMENTED_BY relationships are created from code to documents
**And** IMPLEMENTED_BY relationships are created from features to code
**And** 100% of PRD features are linked to code (NFR7)

---

## Epic 4: Feature Context & Semantic Search

Developers can query feature context and search for similar features using natural language, getting complete context with relationships.

### Story 4.1: Implement query_feature_context MCP Tool

**As a** Developer,
**I want** to query feature context by name or description,
**So that** I get complete information about a feature and its relationships.

**Acceptance Criteria:**

**Given** indexed features in the knowledge graph
**When** I call `query_feature_context` with `feature_name: "User Authentication"`
**Then** the feature node is returned with all properties
**And** related components, services, and libraries are included
**And** response includes linked documentation

---

### Story 4.2: Return Related Code Entities in Context

**As a** Developer,
**I want** feature context to include all related code entities,
**So that** I understand the full implementation scope.

**Acceptance Criteria:**

**Given** a feature with IMPLEMENTED_BY relationships to code
**When** I query the feature context
**Then** all Component, Service, Directive, and Pipe nodes are returned
**And** each entity includes: `type`, `name`, `path`, and relevant metadata
**And** USES_SERVICE relationships between entities are included

---

### Story 4.3: Include Linked Documentation in Responses

**As a** Developer,
**I want** documentation linked to features included in context responses,
**So that** I have business context alongside code references.

**Acceptance Criteria:**

**Given** a feature with DOCUMENTED_BY relationships
**When** I query the feature context
**Then** linked Document nodes are included in the response
**And** each document includes: `path`, `title`, and `excerpt` (first 200 chars)
**And** the response clearly distinguishes code from documentation

---

### Story 4.4: Support Relationship Traversal Depth Parameter

**As a** Developer,
**I want** to specify how deep to traverse relationships,
**So that** I can control the scope of returned context.

**Acceptance Criteria:**

**Given** the `query_feature_context` tool with `depth` parameter
**When** I call with `depth: 1`
**Then** only direct relationships are returned
**When** I call with `depth: 3`
**Then** relationships up to 3 hops away are included
**And** default depth is 2 if not specified

---

### Story 4.5: Implement search_similar_features MCP Tool

**As a** Developer,
**I want** to search for features using natural language,
**So that** I find related features by meaning, not just keywords.

**Acceptance Criteria:**

**Given** features with vector embeddings in Neo4j
**When** I call `search_similar_features` with `query: "user login and authentication"`
**Then** semantically similar features are returned
**And** results are ranked by similarity score
**And** vector search latency < 200ms (NFR4)

---

### Story 4.6: Return Ranked Results with Scores and Limits

**As a** Developer,
**I want** search results ranked by relevance with configurable limits,
**So that** I get the most relevant results efficiently.

**Acceptance Criteria:**

**Given** the `search_similar_features` tool with `limit` parameter
**When** I call with `limit: 5`
**Then** at most 5 results are returned
**And** each result includes a `similarity_score` (0.0 to 1.0)
**And** results are ordered by score descending
**And** query latency < 500ms (NFR1)

---

## Epic 5: Impact Analysis & Dependencies

Tech Leads can analyze feature impact and dependencies, understanding what will break when making changes.

### Story 5.1: Implement analyze_feature_impact MCP Tool

**As a** Tech Lead,
**I want** to analyze which features are affected by changes to a specific feature,
**So that** I understand the blast radius of my changes.

**Acceptance Criteria:**

**Given** indexed features with DEPENDS_ON relationships
**When** I call `analyze_feature_impact` with `feature_name: "AuthService"`
**Then** affected components and features are returned
**And** each affected item includes: `name`, `relationship`, `distance`, `impact_level`
**And** response time < 500ms (NFR1)

---

### Story 5.2: Traverse Dependency Graph for Impact

**As a** Tech Lead,
**I want** the impact analysis to traverse the full dependency graph,
**So that** I see indirect dependencies, not just direct ones.

**Acceptance Criteria:**

**Given** a feature with multi-hop dependencies (A → B → C)
**When** I analyze impact on feature A
**Then** both B (distance: 1) and C (distance: 2) are returned
**And** the maximum traversal depth can be configured
**And** cycles in the graph are handled without infinite loops

---

### Story 5.3: Filter Impact Analysis by Library Scope

**As a** Tech Lead,
**I want** to optionally filter impact analysis to specific NX libraries,
**So that** I focus on what matters for my change.

**Acceptance Criteria:**

**Given** the `analyze_feature_impact` tool with `target_libs` parameter
**When** I call with `target_libs: ["feature-user", "data-access-user"]`
**Then** only impacts within those libraries are returned
**And** impacts in other libraries are excluded
**And** the response shows which libraries were analyzed

---

### Story 5.4: Show Affected Exports and Relationship Distances

**As a** Tech Lead,
**I want** to see which specific exports are affected and how far away they are,
**So that** I can prioritize what to check first.

**Acceptance Criteria:**

**Given** an impact analysis result
**When** I review the response
**Then** affected_exports lists specific component/service names
**And** distance shows hops from the changed feature
**And** impact_level (high/medium/low) is calculated based on relationship type and distance

---

### Story 5.5: Implement get_feature_dependencies MCP Tool

**As a** Tech Lead,
**I want** to retrieve explicit dependencies between features,
**So that** I can understand what must work before my feature.

**Acceptance Criteria:**

**Given** features with dependency relationships
**When** I call `get_feature_dependencies` with `feature_name: "Order Processing"`
**Then** upstream dependencies are returned (features that must exist first)
**And** dependency types are included (blocks, related, parent-child)
**And** reasons for dependencies are shown if available

---

### Story 5.6: Return Dependency Graph Structure for Visualization

**As a** Tech Lead,
**I want** dependency information returned as a graph structure,
**So that** I can visualize dependencies in external tools.

**Acceptance Criteria:**

**Given** the `get_feature_dependencies` response
**When** I access the `dependency_graph` field
**Then** `nodes` array contains all features in the graph
**And** `edges` array contains all relationships with type and direction
**And** the format is compatible with graph visualization libraries (e.g., D3.js, vis.js)

---

## Epic 6: Production Integration & Documentation

AI agents (GitHub Copilot, Claude) can automatically call MCP tools and receive structured context during development, with complete documentation for users.

### Story 6.1: Implement Structured JSON Response Format

**As an** AI Agent (Copilot/Claude),
**I want** all MCP tool responses to follow a consistent structured format,
**So that** I can reliably parse and use the context.

**Acceptance Criteria:**

**Given** any MCP tool response
**When** the tool completes successfully
**Then** response follows format: `{status, data, metadata}`
**And** error responses follow format: `{status: "error", error: {code, message, details}}`
**And** all responses are valid JSON (NFR12)

---

### Story 6.2: Enable Automatic Context Injection for AI Agents

**As an** AI Agent,
**I want** to call MCP tools and receive context automatically,
**So that** I can provide better code suggestions without manual setup.

**Acceptance Criteria:**

**Given** the MCP server is running
**When** an AI agent (Copilot, Claude) calls any tool
**Then** the tool executes and returns structured context
**And** the response includes all relevant nodes and relationships
**And** the agent can use this context in subsequent suggestions

---

### Story 6.3: Create VS Code MCP Configuration

**As a** System Admin,
**I want** to configure VS Code to connect to the MCP server,
**So that** Copilot and other tools can access the knowledge base.

**Acceptance Criteria:**

**Given** the MCP server package is installed
**When** I add configuration to `.vscode/mcp.json`
**Then** VS Code connects to the server on startup
**And** MCP tools appear in the Copilot tool list
**And** configuration works first try with documented settings (NFR13)

---

### Story 6.4: Write Quick Start Documentation

**As a** System Admin,
**I want** a Quick Start guide that gets me from install to first query in < 10 minutes,
**So that** I can evaluate the system quickly.

**Acceptance Criteria:**

**Given** a new user with Python and Neo4j available
**When** they follow the Quick Start guide
**Then** they complete setup in < 10 minutes
**And** they successfully run their first query
**And** the guide covers: install, configure, deploy schema, index, query

---

### Story 6.5: Write MCP Tools API Reference

**As a** Developer,
**I want** complete API documentation for all MCP tools,
**So that** I know the exact inputs and outputs for each tool.

**Acceptance Criteria:**

**Given** the docs/api-reference.md document
**When** I look up any MCP tool
**Then** I find: tool name, description, input parameters, output format, examples
**And** all 6 MCP tools are documented
**And** examples show real request/response payloads
