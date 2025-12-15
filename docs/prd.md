---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]
inputDocuments:
  - docs/analysis/product-brief-knowledge-base-2024-12-14.md
  - docs/analysis/research/technical-neo4j-mcp-knowledge-base-research-2024-12-14.md
documentCounts:
  briefs: 1
  research: 1
  brainstorming: 0
  projectDocs: 0
workflowType: 'prd'
lastStep: 11
project_name: 'knowledge-base'
user_name: 'manhhaycode'
date: '2024-12-14'
---







# Product Requirements Document - knowledge-base

**Author:** manhhaycode
**Date:** 2024-12-14

---

## Executive Summary

**Knowledge Base** is an MCP-powered knowledge graph server that transforms disconnected project documentation and source code into an interconnected, AI-queryable knowledge base. By leveraging Neo4j's graph database with semantic embeddings, it enables AI coding agents like GitHub Copilot to understand complete feature context, business relationships, and cross-cutting dependencies in Angular/NX monorepo projects.

### The Problem

Developers using AI coding assistants in complex Angular/NX monorepos face a critical limitation: AI agents cannot understand project-specific business logic or feature relationships. Despite having comprehensive markdown specifications that document individual pages and features, these specs are disconnected islands—they don't capture which features depend on each other, overlap in functionality, or share business rules.

**Impact:**
- Developers must write lengthy prompts explaining project context repeatedly
- New team members struggle to understand interconnected business rules during onboarding
- AI suggestions miss critical dependencies, leading to incomplete implementations
- Feature development takes longer because AI cannot suggest complete, context-aware solutions

### The Solution

A Neo4j-powered MCP server that creates a knowledge graph connecting:
- **Source code** (components, services, libraries)
- **Business specifications** (markdown docs with embeddings)
- **Feature relationships** (explicit connections between related features)

### What Makes This Special

| Differentiator | Value |
|----------------|-------|
| **Graph-based relationships** | Understands that Feature A depends on Feature B, not just flat search |
| **Semantic + Structural** | Vector search (meaning) combined with graph traversal (relationships) |
| **Business-Code Bridge** | Links PRD/specs directly to implementation code |
| **Existing Specs as Foundation** | Builds on your current markdown documentation investment |
| **MCP Protocol Native** | Works directly with Copilot, Claude, and other AI agents |

## Project Classification

**Technical Type:** `developer_tool`
**Domain:** `scientific` (AI/ML integration for code understanding)
**Complexity:** Medium
**Project Context:** Greenfield - new project

This is a developer tool that integrates with AI coding agents through the Model Context Protocol (MCP). The domain requires understanding of semantic search, knowledge graphs, and AI integration patterns.

---

## Success Criteria

### User Success

| Success Indicator | Description | Target |
|-------------------|-------------|--------|
| **Context-Aware AI Responses** | AI agents understand business context and provide relevant, informed responses | AI suggestions consistently respect existing business rules and patterns |
| **Eliminated Context Prompting** | Developers no longer manually explain project context to AI | < 1 context-related prompt retry per feature implementation |
| **Complete Feature Context** | Queries return business context developers would otherwise need to remember | 100% of indexed features retrievable with full relationship graph |
| **Accelerated Onboarding** | New team members productive faster through AI-assisted learning | Onboarding time reduced from 2 months to 2-3 weeks |

**Primary Success Indicator:** AI assistants consistently provide context-aware suggestions without requiring manual context prompts.

### Business Success

| Metric | Target | Timeframe |
|--------|--------|-----------|
| **Team Adoption Rate** | 80% of target users actively using the system | 1 month post-launch |
| **Productivity Improvement** | 80% improvement in planning, impact analysis, related analysis, and review changes | 3 months post-adoption |
| **Active Users** | 4 team members using regularly | Ongoing |
| **Feature Context Query Usage** | Regular daily usage by development team | > 5 queries/developer/day |

### Technical Success

| Metric | Target | Validation Method |
|--------|--------|-------------------|
| **Indexing Time (Full)** | < 5 minutes for complete repo | Automated benchmark tests |
| **Indexing Time (Incremental)** | < 10 seconds | Automated benchmark tests |
| **Query Latency** | < 500ms for context retrieval | Performance monitoring |
| **Context Relevance** | > 80% of retrieved code chunks relevant | User feedback sampling |
| **Feature Coverage** | 100% of PRD features linked to code | Coverage report |
| **Knowledge Graph Completeness** | 100% of source code entities indexed | Index completeness checks |

### Measurable Outcomes

1. **Before/After Comparison**: Track prompt retries and context-related issues before and after deployment
2. **Query Adoption**: Monitor frequency of `query_feature_context` and `analyze_feature_impact` tool invocations
3. **Time-to-First-Contribution**: Measure onboarding developer productivity improvements
4. **Code Review Quality**: Track reduction in missed dependencies during code reviews

---

## Product Scope

### MVP - Minimum Viable Product

**Includes Phases 1-3 with P0, P1, and P2 tools for complete feature context and impact analysis:**

#### Core Infrastructure (Phase 1)
- Neo4j 5.x database with schema deployment
- Vector indexes for semantic search (384 dimensions)
- NX project graph import (apps, libraries, dependencies)
- CocoIndex ETL pipeline for source code parsing

#### MCP Server Foundation (Phase 2)
- FastMCP Python server with VS Code integration
- All MCP tools P0-P2:
  - `index_source_code` - Index Angular source files with embeddings
  - `index_markdown_specs` - Index feature specifications with embeddings
  - `query_feature_context` - Get complete feature context with relationships
  - `search_similar_features` - Semantic search for related features
  - `analyze_feature_impact` - Analyze change impact across features
  - `get_feature_dependencies` - List explicit feature dependencies

#### Knowledge Enrichment (Phase 3)
- Markdown spec document embedding
- Feature-to-code relationship mapping
- Business specification indexing with vector embeddings

### Growth Features (Post-MVP)

| Feature | Purpose |
|---------|---------|
| `explain_feature` tool | Natural language explanation of feature with code refs |
| `sync_incremental` tool | Trigger incremental re-indexing for changed files |
| Feature relationship visualization | Visual graph of feature connections |

### Vision (Future)

| Feature | Purpose |
|---------|---------|
| Git hook automation | Automatic sync on commits |
| Web UI dashboard | Visual knowledge graph exploration |
| Multi-project support | Multiple Angular/NX projects in one graph |
| Real-time file watching | Automatic indexing on file changes |
| Advanced security (RBAC) | Role-based access control |

---

## User Journeys

### Journey 1: Alex Chen - The Daily Developer Finally Gets Context

Alex is a junior Angular developer on a 4-person team building a complex SaaS platform. Every day, he works on new features using GitHub Copilot, but he's constantly frustrated. When he asks Copilot to help implement a "user permissions" feature, it suggests generic Angular patterns that completely ignore how permissions already work in their existing billing module.

One morning, Alex's tech lead mentions they've set up a new "knowledge-base" MCP server. Skeptical but curious, Alex tries his usual workflow: "Help me add admin permissions to the billing dashboard." This time, something different happens. Copilot's response references their existing `RoleGuard` service, mentions that permissions need to follow the pattern in `feature-user-management`, and even warns that changes might affect the `invoice-generation` module.

Alex's eyes widen. "It actually knows our codebase!" Over the next week, he notices his implementation time dropping significantly. He no longer spends 30 minutes crafting context prompts - the AI just *knows*. When code review comes, his tech lead is impressed: "No missed dependencies this time. Well done."

### Journey 2: Priya Sharma - From Confused to Confident in Weeks

Priya joins as a senior developer but is new to this team's complex Angular monorepo. During her first week, she's assigned to fix a bug in the "order fulfillment" workflow. She opens the codebase and feels overwhelmed - 15 libraries, 200+ components, and a dozen markdown specs that don't obviously connect.

Then her buddy introduces her to the knowledge-base MCP. Priya asks Copilot: "Explain how order fulfillment works in this project." The response is remarkable - it walks through the `OrderService`, links to the functional design doc, shows which libraries are involved, and explains how it connects to inventory management.

In her third week, Priya is already contributing meaningfully. When assigned a new feature, she queries "What features are related to shipping?" and gets a complete map of dependencies. By month two, she's onboarding the *next* new hire using the same tool.

### Journey 3: Marcus Wong - The Tech Lead Who Finally Catches Everything

Marcus leads the team and spends hours every week reviewing PRs. His biggest frustration? Developers often miss dependencies. Last month, a "simple" change to the pricing module broke invoice generation - caught only in production.

With knowledge-base integrated, Marcus's PR review process transforms. When reviewing a permissions PR, he runs `analyze_feature_impact` and sees: "Affects: billing-module, user-management, 3 shared services." He spots that the developer forgot to update the `InvoicePermissions` interface.

During sprint planning, Marcus uses `get_feature_dependencies` to break down a large feature into properly sequenced tasks. "Implement auth changes first - three other features depend on it." The team finishes the sprint without a single blocked task.

### Journey 4: David Park - The System Admin Setting It Up

David is a DevOps engineer tasked with deploying the knowledge-base for his team. He follows the quick-start guide: runs Neo4j in Docker, installs the MCP server package, and runs the initial indexing. The first full index takes 4 minutes for their 8,000-file monorepo.

He configures VS Code's MCP settings and tests with a simple query. Everything works. Two months later, the knowledge graph has become essential. David monitors query latency (consistently under 300ms) and sets up alerts for indexing failures.

### Journey 5: GitHub Copilot - The AI That Finally Understands

As an MCP client, GitHub Copilot connects to the knowledge-base server during VS Code startup. When a developer types a question, Copilot's MCP integration calls `query_feature_context` and receives structured JSON with nodes, relationships, and embeddings.

The AI agent parses the response and incorporates this context into its code suggestions, ensuring they respect existing patterns. When asked "What would break if I change this?", Copilot calls `analyze_feature_impact` and relays the dependency tree - transforming from "smart autocomplete" into a project-aware assistant.

### Journey Requirements Summary

| Journey | Key Capabilities Required |
|---------|--------------------------|
| **Daily Developer** | `query_feature_context`, business-code linking, MCP/VS Code integration |
| **Onboarding Developer** | `search_similar_features`, feature-to-doc linking, relationship graphs |
| **Tech Lead** | `analyze_feature_impact`, `get_feature_dependencies`, dependency visualization |
| **System Admin** | Docker deployment, clear CLI commands, health monitoring |
| **API Consumer (AI)** | MCP protocol compliance, structured JSON responses, rich context output |

---

## Innovation & Novel Patterns

### Core Innovation: Zero-Prompt Context Delivery

The fundamental innovation of **knowledge-base** is eliminating the need for developers to manually explain project context to AI coding assistants. Instead of writing lengthy prompts describing "how billing works" or "which features are related", the knowledge graph automatically provides this context.

**Current State (Manual):**
```
Developer → Writes context prompt → AI understands → Provides suggestion
(Repeated for every question, every feature, every developer)
```

**With Knowledge-Base (Automatic):**
```
Developer → Asks question → MCP provides context automatically → AI understands → Provides suggestion
(Context is always available, never needs re-explanation)
```

### Technology Combination

While individual technologies exist (Neo4j, MCP, embeddings), the combination for **business-code context bridging** is novel:

| Component | Existing Use | Novel Application |
|-----------|--------------|-------------------|
| Neo4j graph | Code dependency analysis | Feature relationship + business logic mapping |
| Semantic embeddings | Document search | Code-to-specification linking |
| MCP protocol | Tool integration | Automatic context injection for AI agents |

### Validation Approach

| Validation | Method |
|------------|--------|
| **Context accuracy** | Compare AI suggestions with/without knowledge-base |
| **Prompt reduction** | Track before/after prompt length and retries |
| **Developer satisfaction** | Survey on "AI understands my project" |

### Risk Mitigation

| Risk | Mitigation |
|------|------------|
| AI still misses context | Continuous improvement of relationship mapping |
| Overhead of maintaining graph | Incremental indexing, low maintenance burden |
| MCP protocol changes | Abstraction layer, follow official SDK |

---

## Developer Tool Specific Requirements

### Technical Architecture

| Component | Technology | Notes |
|-----------|------------|-------|
| **MCP Server** | Python + FastMCP | Primary server implementation |
| **Graph Database** | Neo4j 5.x+ | External service, configured via environment |
| **Embedding Model** | SentenceTransformers | `all-MiniLM-L6-v2` (384 dimensions) |
| **ETL Pipeline** | CocoIndex | Source code and document indexing |

### Installation & Configuration

#### Installation Method
```bash
pip install -r requirements.txt
```

#### Environment Configuration

| Variable | Purpose | Example |
|----------|---------|---------|
| `NEO4J_URL` | Neo4j database connection | `bolt://localhost:7687` |
| `NEO4J_USER` | Database username | `neo4j` |
| `NEO4J_PASSWORD` | Database password | `(secure password)` |

**Note:** Neo4j is an external dependency. Users are responsible for their own Neo4j instance (local, Docker, or cloud-hosted).

### IDE Integration

The MCP server outputs a standard FastMCP interface. Integration is handled by:
- User's IDE MCP configuration (e.g., `.vscode/mcp.json`)
- Any MCP-compatible client (GitHub Copilot, Claude, etc.)

**No IDE-specific code required** - FastMCP handles protocol compliance.

### MCP Tool Interface (API Surface)

| Tool | Input | Output |
|------|-------|--------|
| `index_source_code` | `paths: string[], library_scope?: string` | `{indexed: number, status: string}` |
| `index_markdown_specs` | `paths: string[]` | `{indexed: number, status: string}` |
| `query_feature_context` | `feature_name: string, depth?: number` | `{feature: object, related: array}` |
| `search_similar_features` | `query: string, limit?: number` | `{results: array, scores: array}` |
| `analyze_feature_impact` | `feature_name: string, target_libs?: string[]` | `{affected: array, depth: number}` |
| `get_feature_dependencies` | `feature_name: string` | `{dependencies: array, graph: object}` |

### Documentation Requirements

| Document | Purpose | Priority |
|----------|---------|----------|
| **Quick Start Guide** | Get users from install to first query in < 10 minutes | P0 (MVP) |
| **MCP Tools API Reference** | Complete tool documentation with inputs/outputs | P0 (MVP) |

---

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach:** Problem-Solving MVP
> Solve the core problem (AI doesn't understand project context) with minimal features that prove the value.

**The Minimum That Makes Users Say "This is Useful":**
1. Index source code and markdown specs
2. Query feature context with relationships
3. Analyze feature impact across codebase
4. Search similar features semantically

### MVP Resource Requirements

| Resource | Requirement |
|----------|-------------|
| **Team Size** | 4 developers |
| **Primary Skills** | Python, Neo4j/Cypher, MCP protocol |
| **Timeline** | Phases 1-3 (see Product Scope) |
| **Infrastructure** | Neo4j instance (user-provided) |

### Core User Journeys Supported in MVP

| Journey | MVP Support |
|---------|-------------|
| Daily Developer | ✅ Full support with all P0-P2 tools |
| Onboarding Developer | ✅ Full support with context and search |
| Tech Lead | ✅ Full support with impact analysis |
| System Admin | ✅ Basic setup with environment config |
| API Consumer (AI) | ✅ Full MCP protocol compliance |

### Risk Mitigation Strategy

| Risk Type | Risk | Mitigation |
|-----------|------|------------|
| **Technical** | CocoIndex is early-stage | Pin versions, create abstraction layer |
| **Technical** | Neo4j vector performance at scale | Limit result sets, use composite queries |
| **Market** | MCP protocol could change | Follow official SDK, abstraction layer |
| **Resource** | Small team (4 people) | Focus on P0-P2, defer P3 to post-MVP |

### Contingency Plan

If resources are constrained:
1. **Minimum viable:** P0 tools only (`index_source_code`, `index_markdown_specs`)
2. **Add incrementally:** P1 tools (`query_feature_context`, `search_similar_features`)
3. **Complete MVP:** P2 tools (`analyze_feature_impact`, `get_feature_dependencies`)

---

## Functional Requirements

### Source Code Indexing

- **FR1:** System Admin can index Angular/TypeScript source files from an NX monorepo into the knowledge graph
- **FR2:** System Admin can specify file paths or glob patterns for selective indexing
- **FR3:** System Admin can filter indexing by NX library scope (e.g., only index `feature-auth`)
- **FR4:** System can parse Angular decorators (@Component, @Injectable, @Directive, @Pipe) to identify entity types
- **FR5:** System can generate vector embeddings for indexed code chunks

### Documentation Indexing

- **FR6:** System Admin can index markdown specification documents into the knowledge graph
- **FR7:** System can extract feature descriptions and generate embeddings for semantic search
- **FR8:** System can create relationships between documentation and related code entities

### Feature Context Queries

- **FR9:** Developers can query feature context by name or description
- **FR10:** System can return related components, services, and libraries for a queried feature
- **FR11:** System can include linked documentation in feature context responses
- **FR12:** Developers can specify relationship traversal depth for context queries

### Semantic Search

- **FR13:** Developers can search for features by semantic similarity (meaning, not just keywords)
- **FR14:** System can return ranked results with similarity scores
- **FR15:** System can limit search results to a specified number

### Impact Analysis

- **FR16:** Developers can analyze which features are affected by changes to a specific feature
- **FR17:** System can traverse dependency graph to identify affected components
- **FR18:** Developers can optionally target specific NX libraries for impact analysis
- **FR19:** System can show affected exports and relationship distances

### Dependency Mapping

- **FR20:** Developers can retrieve explicit dependencies between features
- **FR21:** System can return dependency graph structure for visualization
- **FR22:** System can import NX project graph (apps, libraries, dependencies) into Neo4j

### MCP Protocol Integration

- **FR23:** System can expose all tools via standard MCP protocol (JSON-RPC 2.0)
- **FR24:** System can provide structured JSON responses compatible with AI agents
- **FR25:** AI agents (Copilot, Claude) can call MCP tools and receive context automatically

### Configuration & Setup

- **FR26:** System Admin can configure Neo4j connection via environment variables
- **FR27:** System Admin can install the MCP server via pip (requirements.txt)
- **FR28:** System Admin can configure IDE MCP integration via standard configuration files

### Knowledge Graph Schema

- **FR29:** System can store NX project structure (Project, App, Library nodes)
- **FR30:** System can store Angular entities (Component, Service, Directive, Pipe nodes)
- **FR31:** System can store feature and document nodes with embeddings
- **FR32:** System can create relationships (DEPENDS_ON, EXPORTS, USES_SERVICE, IMPLEMENTED_BY, etc.)

---

## Non-Functional Requirements

### Performance

| Requirement | Target | Priority |
|-------------|--------|----------|
| **NFR1:** Query response time | < 500ms for context retrieval | P0 |
| **NFR2:** Full indexing time | < 5 minutes for complete repo | P0 |
| **NFR3:** Incremental indexing time | < 10 seconds for changed files | P1 |
| **NFR4:** Vector search latency | < 200ms for semantic queries | P1 |

### Reliability

| Requirement | Target | Priority |
|-------------|--------|----------|
| **NFR5:** Context relevance accuracy | > 80% of retrieved code chunks relevant | P0 |
| **NFR6:** Indexing completeness | 100% of source code entities indexed | P0 |
| **NFR7:** Feature coverage | 100% of PRD features linked to code | P0 |
| **NFR8:** Error handling | Graceful degradation on Neo4j connection issues | P1 |

### Security

| Requirement | Target | Priority |
|-------------|--------|----------|
| **NFR9:** Credential storage | All credentials via environment variables, never in code | P0 |
| **NFR10:** Network communication | Neo4j connections use bolt or bolt+s protocols | P0 |
| **NFR11:** No code exfiltration | Source code stays local, only embeddings/metadata stored | P0 |

### Integration

| Requirement | Target | Priority |
|-------------|--------|----------|
| **NFR12:** MCP protocol compliance | Full JSON-RPC 2.0 compatibility | P0 |
| **NFR13:** VS Code integration | Works with MCP configuration in user settings | P0 |
| **NFR14:** Neo4j compatibility | Supports Neo4j 5.x with vector index support | P0 |

