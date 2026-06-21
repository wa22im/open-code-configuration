---
name: codebase-memory
description: Use when working in a project codebase — answers structural questions (functions, call chains, routes, layers, dependencies) in sub-ms by querying the persistent codebase knowledge graph. Use when exploring an unfamiliar repository, tracing call paths across services, getting an architecture overview, finding dead code, or any task that would otherwise require many grep/read cycles. Triggers on phrases like "find the function that", "trace the call from", "what depends on", "give me the architecture of", or "index this project".
---

# Codebase Memory MCP

A persistent, per-project knowledge graph for any codebase the agent touches.
Each project is indexed once, persisted as SQLite under
`~/.cache/codebase-memory-mcp/<project-key>/`, and shared across every opencode
session that opens that project. The server uses git-based change detection to
keep the graph fresh — re-indexing is incremental, not a full rebuild. Average
repo is queryable in milliseconds; structural questions that would otherwise
require many `grep` / `read` cycles resolve in a single MCP call.

The graph stores language nodes (functions, methods, classes, interfaces,
modules, routes, resources), edge types (calls, imports, inherits, implements,
HTTP_CALLS, …), and Architecture Decision Records. It also embeds an
interactive 3D graph UI (UI binary variant) at `http://localhost:9749` for
visual exploration while the agent works.

## MCP tools

### Indexing

| Tool | Purpose |
| --- | --- |
| `index_repository` | Index a repository into the graph. Auto-sync keeps it fresh after that. |
| `list_projects` | List all indexed projects with node/edge counts. |
| `delete_project` | Remove a project and all its graph data. |
| `index_status` | Check indexing status of a project. |

### Querying

| Tool | Purpose |
| --- | --- |
| `search_graph` | Structured search by label, name pattern, file pattern, degree filters. Pagination via limit/offset. |
| `trace_path` | BFS traversal — who calls a function and what it calls (alias: `trace_call_path`). Depth 1–5. |
| `detect_changes` | Map git diff to affected symbols + blast radius with risk classification. |
| `query_graph` | Execute Cypher-like graph queries (read-only). |
| `get_graph_schema` | Node/edge counts, relationship patterns, property definitions per label. Run this first. |
| `get_code_snippet` | Read source code for a function by qualified name. |
| `get_architecture` | Codebase overview: languages, packages, routes, hotspots, clusters, ADR. |
| `search_code` | Grep-like text search within indexed project files. |
| `manage_adr` | CRUD for Architecture Decision Records. |
| `ingest_traces` | Ingest runtime traces to validate `HTTP_CALLS` edges. |

## When to use which tool

- **First time in a project** → `index_status` → if absent, `index_repository`,
  then `get_graph_schema` to learn the node/edge vocabulary.
- **"How is this project structured?"** → `get_architecture` for a one-shot
  overview (languages, packages, entry points, routes, hotspots, clusters).
- **"What calls X?" / "What does X call?"** → `trace_path` with
  `direction="inbound"` or `direction="outbound"`.
- **"Find the function / class / route named …"** → `search_graph` with a
  regex `name_pattern`; refine with `label` and `file_pattern` filters.
- **"What did my last commit change and what depends on it?"** →
  `detect_changes` to get the diff → symbol mapping with risk classification.
- **"Read the body of function Y"** → first `search_graph` to discover the
  qualified name `<project>.<path_parts>.<name>`, then `get_code_snippet`.
- **"Grep inside the indexed codebase"** → `search_code` (graph-augmented,
  only over files already in the graph).
- **Ad-hoc openCypher questions** → `query_graph`. Always run
  `get_graph_schema` first to learn the labels and edge types in play.
- **Recording or revisiting a design decision** → `manage_adr` to persist ADRs
  into the graph so they survive session restarts.
- **Validating HTTP call edges against real traffic** → `ingest_traces`.

## Operational notes

- The binary is registered as the `codebase-memory` MCP server in
  `~/.config/opencode/opencode.jsonc` and runs as
  `~/.local/bin/codebase-memory-mcp --ui=true --port=9749`. The UI variant
  embeds an HTTP server alongside the MCP stdio transport, so the 3D graph
  visualization at `http://localhost:9749` is available whenever the agent is
  connected to the server.
- Per-project data lives at `~/.cache/codebase-memory-mcp/<project-key>/` and
  is shared across all opencode sessions. Each project is indexed once; git
  diffs drive incremental updates.
- `auto_index` is enabled in the binary's own config, so the first time a
  project is opened the graph is built without an explicit call.
- The `~/.cache/codebase-memory-mcp/**` path is allow-listed under
  `permission.external_directory` so the agent can read and write graph
  artifacts without per-call permission prompts.
