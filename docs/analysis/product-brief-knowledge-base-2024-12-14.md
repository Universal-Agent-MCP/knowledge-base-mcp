---
stepsCompleted: [1, 2, 3, 4, 5, 6]
inputDocuments:
  - docs/analysis/research/technical-neo4j-mcp-knowledge-base-research-2024-12-14.md
workflowType: 'product-brief'
lastStep: 6
status: 'complete'
project_name: 'knowledge-base'
user_name: 'manhhaycode'
date: '2024-12-14'
---

# Product Brief: knowledge-base

**Date:** 2024-12-14
**Author:** manhhaycode

---

## Executive Summary

**Knowledge Base** is an MCP-powered knowledge graph server that transforms disconnected project documentation and source code into an interconnected, AI-queryable knowledge base. By leveraging Neo4j's graph database with semantic embeddings, it enables AI coding agents like GitHub Copilot to understand complete feature context, business relationships, and cross-cutting dependencies in Angular/NX monorepo projects.

The core insight: teams already have detailed markdown specifications defining their business logic, but these specs exist as **isolated documents** without connections between related features. This knowledge gap prevents AI assistants from providing meaningful impact analysis, complete feature context, or accurate implementation guidance.

---

## Core Vision

### Problem Statement

Developers using AI coding assistants in complex Angular/NX monorepos face a critical limitation: **AI agents cannot understand project-specific business logic or feature relationships**. Despite having comprehensive markdown specifications that document individual pages and features, these specs are disconnected islands—they don't capture which features depend on each other, overlap in functionality, or share business rules.

### Problem Impact

- **Developers** must write lengthy prompts explaining project context repeatedly
- **New team members** struggle to understand interconnected business rules during onboarding
- **AI suggestions** miss critical dependencies, leading to incomplete implementations
- **Impact analysis** for PRD breakdowns lacks visibility into related existing features
- **Feature development** takes longer because AI cannot suggest complete, context-aware solutions

### Why Existing Solutions Fall Short

| Current Approach | Limitation |
|------------------|------------|
| Long prompt engineering | Manual, repetitive, doesn't scale |
| Tools like Serena MCP | Discover file paths, but not business/feature relationships |
| Markdown specs | Document individual pages, but no cross-feature connections |
| Copilot out-of-the-box | Limited to current file context, no project knowledge |

### Proposed Solution

A **Neo4j-powered MCP server** that creates a knowledge graph connecting:
- **Source code** (components, services, libraries)
- **Business specifications** (markdown docs with embeddings)
- **Feature relationships** (explicit connections between related features)

Key capabilities:
1. **Feature Context Query** - "Show me everything related to X" with full dependency graph
2. **Impact Analysis** - "What features are affected if I change Y?"
3. **Semantic Search** - Find related code and docs by meaning, not just keywords
4. **Onboarding Context** - "Explain how billing works" with code references

### Key Differentiators

| Differentiator | Value |
|----------------|-------|
| **Graph-based relationships** | Understands that Feature A depends on Feature B, not just flat search |
| **Semantic + Structural** | Vector search (meaning) combined with graph traversal (relationships) |
| **Business-Code Bridge** | Links PRD/specs directly to implementation code |
| **Existing Specs as Foundation** | Builds on your current markdown documentation investment |
| **MCP Protocol Native** | Works directly with Copilot, Claude, and other AI agents |

---

## Target Users

### Primary Users

#### 🧑‍💻 "The Daily Developer" - Junior/Mid-Level Angular Developer

**Profile:**
- Experience: Junior to mid-level developers on an Angular/NX team
- Daily work: Building features in Angular monorepo with frequent AI assistant usage
- Tools: VS Code with GitHub Copilot, referencing functional design documents

**Current Pain Points:**
- AI assistants don't understand project-specific business logic
- Missing context on impact analysis and what changes are needed
- Must repeatedly explain project context through long prompts
- Features have overlapping business rules that AI cannot navigate

**Success Vision:**
- Ask Copilot about a feature and get complete context including related features
- Receive suggestions that respect existing business rules and patterns
- Faster feature development with AI that "knows" the project

---

#### 🆕 "The Onboarding Developer" - New Team Member

**Profile:**
- Experience: New to the team (any skill level)
- Current onboarding time: ~2 months to become productive
- Biggest challenge: Understanding interconnected business logic

**Current Learning Process:**
1. Read Functional Design documents for a feature
2. Interact with the UI to see behavior
3. Compare with codebase to understand implementation
4. Repeat for each feature (slow, disconnected)

**Success Vision:**
- Ask "How does [feature] work?" and get complete answer with code references
- Understand feature dependencies and relationships immediately
- Reduce onboarding time from 2 months to weeks

---

### Secondary Users

#### 👨‍💼 "The Tech Lead" - Engineering Lead/Architect

**Profile:**
- Responsible for code review, task estimation, and feature breakdown
- Needs visibility into change impact across features

**Current Pain Points:**
- Missing or duplicate impact analysis during code review
- Difficult to estimate and breakdown features without full dependency knowledge
- Manual effort to identify which tasks depend on each other

**Success Vision:**
- Review PRs with full visibility into affected features
- Quickly breakdown features into properly sequenced tasks
- Assign dependent tasks to team members with clear relationships

---

### User Journey

| Stage | Daily Developer | Onboarding Developer | Tech Lead |
|-------|-----------------|---------------------|-----------|
| **Discovery** | Hears about knowledge-base from team lead | Introduced during onboarding | Discovers while looking for impact analysis tools |
| **First Use** | Queries feature context during active development | Asks "explain how X works" for first assigned feature | Runs impact analysis on PR |
| **Aha Moment** | AI suggests code that respects existing patterns | Gets complete feature explanation in minutes vs. days | Catches missing dependency before merge |
| **Daily Use** | Feature context queries, impact checks before PRs | Learning new areas through queries | PR review validation, task breakdown |

---

## Success Metrics

### User Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| **Reduced Context-Missing Errors** | Fewer AI suggestions that lack project context | Before/after comparison of prompt retries and context-related issues |
| **Feature Context Query Adoption** | Regular daily usage by team | Frequency of `query_feature_context` tool invocations |
| **Onboarding Acceleration** | New members productive faster | Time-to-first-contribution for onboarding developers |

**Primary Success Indicator:** AI assistants consistently provide context-aware suggestions without requiring manual context prompts.

---

### Business Objectives

| Objective | Target | Timeframe |
|-----------|--------|-----------|
| **Team Adoption Rate** | 80% of target users actively using the system | 1 month post-launch |
| **Productivity Gains** | 80% improvement in feature development efficiency | 3 months post-adoption |
| **Active Users** | 4 team members using regularly | Ongoing |

---

### Key Performance Indicators

| KPI | Description | Target |
|-----|-------------|--------|
| **Feature Relationship Coverage** | % of features with complete relationship mapping in the knowledge graph | 100% of documented features |
| **Query Response Time** | Time to retrieve feature context | < 500ms |
| **Knowledge Graph Completeness** | All source code entities indexed with embeddings | 100% coverage |
| **User Engagement** | Feature context queries per developer per day | > 5 queries/developer/day |

---

## MVP Scope

### Core Features

**MVP includes Phases 1-3** for complete feature context capability:

#### Phase 1: Core Infrastructure
- Neo4j 5.x database with schema deployment
- Vector indexes for semantic search (384 dimensions)
- NX project graph import (apps, libraries, dependencies)
- CocoIndex ETL pipeline for source code parsing

#### Phase 2: MCP Server Foundation
- FastMCP Python server with VS Code integration
- Core MCP tools (see prioritized list below)
- Basic authentication with environment variables

#### Phase 3: Knowledge Enrichment
- Markdown spec document embedding
- Feature-to-code relationship mapping
- Business specification indexing with vector embeddings

---

### MCP Tools (Priority Order)

| Priority | Tool Name | Purpose |
|----------|-----------|---------|
| **P0** | `index_source_code` | Index Angular source files into Neo4j with embeddings |
| **P0** | `index_markdown_specs` | Index existing feature specifications with embeddings |
| **P1** | `query_feature_context` | Get complete feature context with related components, services, and docs |
| **P1** | `search_similar_features` | Semantic search for related features by description |
| **P2** | `analyze_feature_impact` | Analyze which features are affected by changes |
| **P2** | `get_feature_dependencies` | List explicit dependencies between features |
| **P3** | `explain_feature` | Natural language explanation of feature with code refs |
| **P3** | `sync_incremental` | Trigger incremental re-indexing for changed files |

---

### Out of Scope for MVP

| Feature | Reason |
|---------|--------|
| Git hook automation | Manual sync sufficient for initial adoption |
| Web UI dashboard | CLI/MCP tools sufficient for MVP |
| Multi-project support | Single Angular/NX project focus |
| Real-time file watching | On-demand sync is sufficient |
| Advanced security (RBAC) | Environment variables sufficient for team use |

---

### MVP Success Criteria

| Criteria | Target |
|----------|--------|
| **All 3 phases complete** | Full feature context capability |
| **100% feature coverage** | All existing markdown specs indexed |
| **Query response time** | < 500ms for feature context |
| **Team adoption** | 80% of 4 team members using regularly within 1 month |
| **Context-aware AI** | Copilot provides suggestions with project context |
