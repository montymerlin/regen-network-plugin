---
name: regen-credits
description: >
  Explore ecocredits on Regen Network. Use when the user asks about "regen
  credits", "ecocredits", "credit classes", "credit types", "what credits
  are available", "carbon credits on regen", "biodiversity credits", or
  wants an overview of Regen Network's ecological credit ecosystem.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Provide a unified view of ecological credits on Regen Network by combining Ledger and Compute data.

## Workflow

1. **Credit types and classes** — call `mcp__plugin_regen-network_regen-ledger__list_credit_types` and `mcp__plugin_regen-network_regen-ledger__list_classes` to show what types of credits exist (carbon, biodiversity, marine, etc.) and how they're organized.

2. **Projects** — call `mcp__plugin_regen-network_regen-ledger__list_projects` to show active projects issuing credits. Include project names, jurisdictions, and credit class associations.

3. **Credit batches** — if the user wants detail on a specific class or project, call `mcp__plugin_regen-network_regen-ledger__list_credit_batches` to show issuance history and quantities.

4. **What's for sale** — call `mcp__plugin_regen-network_regen-compute__browse_available_credits` to show live marketplace sell orders with pricing. Filter by `credit_type` (carbon, biodiversity, all) based on user interest.

5. **Network-wide stats** — call `mcp__plugin_regen-network_regen-compute__get_impact_summary` for aggregate context: total retirements, credits issued vs retired, hectares under stewardship.

## Presenting results

- Lead with a high-level summary: how many credit types, active projects, total credits issued
- Present credit types with plain-language descriptions (not just codes)
- When showing marketplace listings, include price per credit and quantity available
- Link to [app.regen.network](https://app.regen.network) for users who want to browse or purchase directly
