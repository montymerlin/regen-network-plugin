---
name: regen-code
description: >
  Explore the Regen Network codebase. Use when the user asks about "regen code",
  "regen architecture", "regen codebase", "regen tech stack", "trace call chain",
  "find function in regen", "regen repo", "regen ledger code", or wants to
  understand the technical implementation of Regen Network software.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Explore the Regen Network codebase using the KOI code graph and GitHub document index.

## Tools

### Repository overviews
- `mcp__plugin_regen-network_regen-koi__get_repo_overview` with `repository` parameter (e.g. `"regen-ledger"`, `"regen-web"`, `"regen-data-standards"`, `"regenie-corpus"`) — structured overview of a specific repo
- `mcp__plugin_regen-network_regen-koi__get_tech_stack` — technology stack information across all indexed repos

### Code graph queries
Call `mcp__plugin_regen-network_regen-koi__query_code_graph` with `query_type` and `entity_name` (not `target`) for the entity you're asking about. Use `repo_name` (not `repository`) to filter by repo. Example query types:
- `find_callers` + `entity_name: "MsgCreateBatch"` — what calls this entity
- `find_callees` + `entity_name: "SomeKeeper"` — what this entity calls
- `trace_call_chain` + `from_entity` + `to_entity` — path between two entities
- `find_orphaned_code` — code with no callers
- `get_entity_stats` — full graph statistics (no entity_name needed)
- `list_modules` — all modules in the codebase
- `keeper_for_msg` + `entity_name` — find the Keeper handling a Msg

Use this for architectural questions ("how does the ecocredit module work?", "what calls MsgCreateBatch?").

### GitHub documentation search
- `mcp__plugin_regen-network_regen-koi__search_github_docs` — search across indexed repositories for code, documentation, and configuration files

### SPARQL queries
For advanced users, `mcp__plugin_regen-network_regen-koi__sparql_query` executes raw SPARQL against the Apache Jena knowledge graph. Only use this if the user specifically asks for SPARQL or if the other tools don't return sufficient detail.

## Presenting results

- For architecture questions, describe the module structure and key relationships before diving into specifics
- For call chain traces, present as a readable flow (A calls B calls C) rather than raw data
- Always name the repository and file path when referencing specific code
- Link to GitHub source files when URLs are available in results
