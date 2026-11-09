# Roadmap — regen-network

Where this plugin could go. Items here are aspirations, not commitments. When an item is evaluated, the outcome moves to the Decided section with a pointer to DECISIONS.md.

**Statuses:** `active` (in progress) · `idea` (worth evaluating) · `parked` (inspiration, no timeline) · `decided` (evaluated — see DECISIONS.md)

---

## Near-term

- **Hand the skills to upstream** — propose that the regen-ai marketplace become the one place for Regen's AI tools (KOI, ledger, Compute, Claims, Registry Review, skills), serving Claude Code and Codex from one repo with pinned servers and marketplace auto-update, and that Gaia AI build these skills natively into the [regen-ai marketplace](https://github.com/gaiaaiagent/regen-ai-claude) (KOI/ledger skills into their `koi` and `ledger` plugins, footprint into a `compute` plugin for `regen-compute`) and take over maintenance. Once they do, this repo points to theirs and is archived. Proposal and intro message drafted 2026-11-09; nothing sent. Supersedes listing this repo as a separate marketplace entry. `status: active`
- **Drop the `mcp<2` pin** — once `regen-python-mcp` bounds `mcp` below 2 or moves to the 2.x `MCPServer` API (`/regen-update` checks). `status: idea`
- **Skills-only variant** — ship the ten skills without servers and rely on upstream `koi@regen-ai` and `ledger@regen-ai`. Removes double-loading and leaves server ownership with Gaia AI; costs cross-plugin tool prefixes (`mcp__plugin_koi_regen-koi__*`), a hard dependency on upstream merging the `mcp<2` fix, and still needs `regen-compute` from somewhere. Worth deciding with the upstream maintainers if they take the marketplace entry. `status: idea`
- **Registry Review skill** — implement `regen-review` properly if the Registry Review MCP becomes publicly installable (today it installs from the private `regen-network/registry-review-mcp`). `status: parked`
- **Drop the `regen-` prefix from skill dirs** — would turn `regen-network:regen-search` into `regen-network:search` (flagged 2026-04-27). `status: idea`

## Future explorations

- **Cross-skill workflows** — Compose skills into higher-level workflows, e.g., "analyze this credit class" could invoke regen-credits → regen-market → regen-portfolio in sequence. Currently each skill is independent. `status: idea`
- **Offline/degraded mode** — When MCP servers can't connect (network issues, missing dependencies), provide cached or static fallback data for common queries. `status: idea`
- **Notification skill** — Alert on governance proposals approaching vote deadline, or significant market movements. Would need a scheduling mechanism. `status: idea`

## Parking lot

- **Additional MCP integrations** — IBC-connected chains (Osmosis DEX data, Cosmos Hub governance), off-chain data sources (Toucan, Verra registry APIs). `status: parked`
- **Curriculum/learning skill** — Guided introduction to Regen Network concepts for newcomers, using KOI commons content. `status: parked`

## Decided

- **Adopted agentic scaffold** — → Decision 001. `status: decided`
- **Dual-distribution packaging** — → Decision 002. `status: decided`
- **Reframe regen-network as a host-agnostic package** — → Decision 003. `status: decided`
- **Own the package, restore the ledger, pin servers, leave registry-review out** — → Decision 005. `status: decided`
