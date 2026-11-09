---
name: regen-search
description: >
  Search the Regen Network knowledge commons (KOI). Use when the user asks to
  "search regen", "search KOI", "what does regen know about", "find in regen
  commons", "regen docs about", or wants to find information across Regen's
  tens of thousands of indexed documents spanning forum posts, Notion docs, GitHub repos,
  podcasts, and more.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Search the Regen KOI knowledge base using hybrid search (vector + keyword + knowledge graph).

## Search workflow

1. Run `mcp__plugin_regen-network_regen-koi__search` with the user's query. Choose the best intent:
   - `general` — default for topic searches
   - `person_activity` — for "what is [person] working on" queries
   - `person_bio` — for "who is [person]" queries
   - `technical_howto` — for "how do I" implementation questions
   - `concept_explain` — for "what is [concept]" explainers

2. Filter by source when the user specifies one: `notion`, `github`, `discourse`, `youtube`, `podcast`, `web`, `gitlab`. Use `get_stats` with `detailed: true` to see all available sources if unsure.

3. For date-bounded queries, use `published_from` and `published_to` (YYYY-MM-DD format). Sort by `date_desc` for "latest" or "recent" queries.

4. If a result looks highly relevant but the snippet is insufficient, use `mcp__plugin_regen-network_regen-koi__get_full_document` with the result's RID to retrieve the complete content.

5. For entity-related queries (people, projects, organizations), use `mcp__plugin_regen-network_regen-koi__resolve_entity` to get structured entity data, then `mcp__plugin_regen-network_regen-koi__get_entity_documents` to find all documents mentioning that entity.

## Presenting results

- Always include the source URL from result metadata so the user can visit the original
- Group results by source type when returning mixed results (forum posts, Notion pages, GitHub, etc.)
- Indicate confidence scores when relevant — scores above 50% are strong matches
- If no results found, suggest broadening the query or trying a different intent
