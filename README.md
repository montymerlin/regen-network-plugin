# Regen Network Plugin

Access the Regen Network ecosystem through a host-agnostic MCP + skills package: knowledge commons, blockchain ledger, ecological credit markets, and AI compute footprinting.

This plugin is an integration layer: it bundles three independently maintained upstream MCP servers and ten skills into one installable package. The servers are built and published by the Regen Network and Gaia AI teams (see [Credits](#credits)); this repo wires them together with skills, documentation and host packaging, maintained by [Monty Bryant](https://github.com/montymerlin).

## Install

In Claude Code:

```
/plugin install regen-network --marketplace montymerlin/regen-network-plugin
```

Or from a shell: `claude plugin marketplace add montymerlin/regen-network-plugin`, then `claude plugin install regen-network@regen-network`. Cowork, Cursor, VS Code and other MCP hosts are covered in [SETUP.md](SETUP.md).

## What is Regen Network?

[Regen Network](https://regen.network) is a public blockchain purpose-built for ecological assets. Built on the Cosmos SDK, it provides an on-chain registry for verified ecological outcomes — primarily **ecocredits**, which represent quantified, verified improvements to land, biodiversity, or climate.

Key parts of the ecosystem this plugin connects to:

- **Regen Ledger** — the blockchain itself. Stores ecocredit classes, project registrations, credit batch issuances, retirements, marketplace sell orders, staking, and on-chain governance.
- **KOI (Knowledge of Impact)** — Regen's knowledge commons. A semantic index of tens of thousands of documents (about 38,000 in November 2026) spanning forum posts, Notion workspace, GitHub repos, podcast transcripts, YouTube videos, and ecosystem publications. Supports hybrid vector + keyword + knowledge graph search.
- **Regen Compute** — connects AI compute usage to verified ecological credit retirement. Estimates the energy and CO2 cost of an AI session and provides a direct path to retire ecocredits as compensation.
- **Registry Review** — an LLM-assisted workflow for reviewing carbon credit project documents against methodology requirements. Its server is not publicly installable, so this package does not bundle it (see below).

## MCP Servers

Three MCP servers ship with this package, each pinned to a tested version and connecting to a different layer of the Regen ecosystem:

### `regen-koi` — Knowledge Commons

**Package:** [`regen-koi-mcp`](https://github.com/gaiaaiagent/regen-koi-mcp) (npm), pinned `1.6.6` — maintained by Gaia AI

Connects to the KOI semantic index, Regen's living knowledge base. Supports three search modes (lexical BM25, semantic vector, and hypothetical document embedding), entity resolution (people, projects, organisations), knowledge graph traversal, SPARQL queries over the linked data graph, and code graph queries for the Regen Ledger codebase itself.

Used by: `/regen-search`, `/regen-digest`, `/regen-code`

### `regen-ledger` — Regen Ledger Blockchain

**Package:** [`regen-python-mcp`](https://github.com/gaiaaiagent/regen-python-mcp) (PyPI), pinned `0.1.0` with `mcp<2` — maintained by Gaia AI

`regen-python-mcp` imports `mcp.server.fastmcp`, which the `mcp` 2.x SDK removed, so the server runs as `uvx --with 'mcp<2' regen-python-mcp==0.1.0` (the same fix as [gaiaaiagent/regen-ai-claude#4](https://github.com/gaiaaiagent/regen-ai-claude/pull/4)). The pin can go once the package bounds `mcp` itself or moves to the 2.x API.

Connects to Regen Ledger via gRPC. Exposes read-only queries across: ecocredit classes, projects, and batches; marketplace sell orders; on-chain governance proposals, votes, tallies, and parameters; staking, delegation, and validator data; bank balances and supply; and community pool.

Used by: `/regen-credits`, `/regen-market`, `/regen-governance`, `/regen-portfolio`

### `regen-compute` — AI Footprint & Credit Retirement

**Package:** [`regen-compute`](https://github.com/regen-network/regen-compute) (npm), pinned `0.3.4` — maintained by Regen Network

Connects to the Regen Compute service. Estimates the energy and CO2 footprint of an AI session, browses available credits on the marketplace, and starts credit retirement through a Regen Marketplace credit card link, which the user completes in their browser. The server can also sign on-chain retirements and cross-chain ecoBridge payments itself if a wallet mnemonic is in its environment; this package never configures one (see [Wallet keys](#wallet-keys)).

Used by: `/regen-footprint`, `/regen-credits`, `/regen-market`

### Not bundled: `registry-review`

Gaia AI's [Registry Review MCP](https://github.com/gaiaaiagent/regen-ai-claude/tree/main/plugins/registry-review) runs an 8-stage review of carbon credit project documents against registry methodology requirements. Its install source is the private `regen-network/registry-review-mcp` GitHub repo (fetched over SSH), and the PyPI name is unpublished, so a public package cannot start it. If you have access to that repo, install the `registry-review` plugin from the upstream [regen-ai marketplace](https://github.com/gaiaaiagent/regen-ai-claude). `/regen-review` explains this rather than attempting a review.

## Skills

Ten skills are included, each routing through one or more of the MCP servers above.

### `/regen-search` — Search the knowledge commons

Searches KOI using hybrid retrieval (vector + keyword + knowledge graph). Supports source filtering (forum, GitHub, Notion, podcasts, YouTube), date-bounded queries, and entity-centric lookups. For entity queries (people, projects, organisations), resolves structured entity data and retrieves all related documents.

**Trigger phrases:** "search regen", "search KOI", "what does regen know about", "find in regen commons", "regen docs about"
**MCPs used:** `regen-koi`

---

### `/regen-digest` — Weekly activity digest

Generates a structured summary of recent Regen Network activity using the KOI knowledge base. Covers governance, development, community, and media activity. Can be scoped to a custom date range or filtered by domain.

**Trigger phrases:** "regen digest", "what's new at regen", "regen weekly summary", "regen activity update", "regen network news"
**MCPs used:** `regen-koi`

---

### `/regen-credits` — Explore ecocredits

Provides a unified view of the Regen credit ecosystem: credit types and classes, active projects, issuance batches, live marketplace listings with pricing, and network-wide aggregate stats (total retirements, credits issued vs retired, hectares under stewardship).

**Trigger phrases:** "ecocredits", "credit classes", "what credits are available", "carbon credits on regen", "biodiversity credits"
**MCPs used:** `regen-ledger`, `regen-compute`

---

### `/regen-market` — Market analysis

Analyses the Regen credit marketplace: current sell orders (by batch or seller), allowed payment denominations, market trends over time, and methodology comparisons across credit classes. Also surfaces a user-friendly browsable view of currently purchasable credits with pricing.

**Trigger phrases:** "regen market", "credit prices", "sell orders", "market trends", "compare methodologies", "marketplace activity"
**MCPs used:** `regen-ledger`, `regen-compute`

---

### `/regen-governance` — On-chain governance

Queries Regen Ledger governance: proposals (list, filter by status, full detail), vote tallies, individual voter positions, deposit status, governance parameters (voting period, quorum, threshold, veto threshold), and community pool balance.

**Trigger phrases:** "regen governance", "proposals", "regen votes", "voting results", "community pool", "active proposals"
**MCPs used:** `regen-ledger`

---

### `/regen-portfolio` — Wallet impact analysis

Analyses the ecological impact and holdings of a `regen1...` wallet address: credit holdings and retirements, token balances (REGEN, ecocredits, basket tokens), staking positions and validator delegations, active marketplace listings, and basket token composition.

**Trigger phrases:** "regen portfolio", "wallet impact", "analyze address", "ecological impact of address", "my regen credits"
**MCPs used:** `regen-ledger`

---

### `/regen-footprint` — AI session footprint & offset

Estimates the energy and CO2 cost of the current AI session (by duration and tool call count), shows matching credits available for retirement, and, if you ask, generates a Regen Marketplace credit card link to retire them. Confirms credit, quantity and cost before any retirement call, and can fetch the on-chain retirement certificate afterwards.

**Trigger phrases:** "offset my session", "environmental cost of AI", "retire credits", "session footprint", "carbon offset"
**MCPs used:** `regen-compute`

---

### `/regen-code` — Codebase exploration

Explores the Regen Network codebase using KOI's code graph and GitHub document index. Supports call chain tracing, caller lookups, orphaned code detection, and architectural queries across Regen Ledger modules. Also provides structured repo overviews and tech stack summaries for individual repos (`regen-ledger`, `regen-web`, `regen-data-standards`, etc.).

**Trigger phrases:** "regen architecture", "regen codebase", "regen tech stack", "trace call chain", "find function in regen"
**MCPs used:** `regen-koi`

---

### `/regen-review` — Registry document review *(unavailable)*

Explains that the Registry Review MCP is not publicly installable, points users with access to the upstream `registry-review` plugin, and offers the KOI and ledger lookups this package can do instead.

**Trigger phrases:** "registry review", "review project documents", "document compliance check"
**MCPs used:** none (its server is not bundled)

---

### `/regen-update` — Check for upstream updates

Compares the three pinned MCP packages with their latest published versions, checks whether the ledger server still needs the `mcp<2` workaround, fetches release notes for newer versions, and cross-references the tools the servers expose against what the skills use, surfacing tools with no skill coverage yet.

**Trigger phrases:** "check for regen updates", "what's new in regen MCPs", "are there new regen tools", "update regen plugin"
**MCPs used:** none (uses npm registry, PyPI API, and GitHub releases)

## Host Positioning

This package is host-agnostic: any environment that supports MCP and markdown skills can use it.

- **Cowork** remains the smoothest end-user experience — one-click plugin install, skill slash commands in the `/` menu, and automatic MCP loading.
- **Claude Code, Cursor, VS Code, and other MCP-capable hosts** use the same underlying MCPs and skills, typically through natural-language invocation and host-specific MCP configuration.

## Setup

See [SETUP.md](SETUP.md) for per-host installation and MCP configuration (Cowork, Claude Code CLI, Cursor, VS Code, and generic MCP hosts).

## Wallet keys

Everything in this package works without configuration, and none of it needs a wallet. `regen-compute` reads two optional variables, `REGEN_WALLET_MNEMONIC` (direct on-chain retirement) and `ECOBRIDGE_EVM_MNEMONIC` (cross-chain payment via ecoBridge). With either set, the server signs and broadcasts irreversible transactions that spend real funds, with no browser step between the agent and the chain.

This package does not set them, and its skills never suggest setting them or ask for a seed phrase. If you choose to enable direct signing, do it yourself from the [regen-compute docs](https://github.com/regen-network/regen-compute), use a dedicated low-balance wallet, and keep the phrase out of chat and out of any file an agent edits.

## Using this beside the upstream regen-ai plugins

The [regen-ai marketplace](https://github.com/gaiaaiagent/regen-ai-claude) ships `koi` and `ledger` as separate MCP-only plugins. Installing `koi@regen-ai` or `ledger@regen-ai` alongside this package starts the same servers twice, under different names (`mcp__plugin_koi_regen-koi__*` and `mcp__plugin_regen-network_regen-koi__*`). Nothing breaks, but you pay for two processes and twice the tool definitions, and the agent sees duplicate tools. Install one or the other: this package if you want the skills, the upstream plugins if you only want the raw tools.

## Dependencies & Updates

This plugin is a thin routing layer over three upstream packages, and it pins each one to a version that has been started and exercised (see `.mcp.json`). Pins trade automatic updates for predictability: an upstream release cannot break the plugin overnight, as the unpinned `mcp` dependency did to the ledger server in 2026.

**Skills and pins move together.** Skill files reference specific tools, so a server update that adds or renames tools needs a matching skill change. Both ship as a new plugin version.

**To check for updates:** run `/regen-update`. It compares pinned and published versions, reads release notes, and lists tools no skill covers yet.

**To bump a pin:** edit the version in `.mcp.json` (and the snippets in `SETUP.md`), start the server once to confirm it initialises and lists its tools, re-run `/regen-update` for coverage, then bump the version in `.claude-plugin/plugin.json` and `marketplace.json` and add a CHANGELOG entry.

## Credits

The MCP servers this package depends on are other people's work:

- [`regen-koi-mcp`](https://github.com/gaiaaiagent/regen-koi-mcp) and [`regen-python-mcp`](https://github.com/gaiaaiagent/regen-python-mcp), by [Gaia AI](https://github.com/gaiaaiagent), who also maintain the KOI index and the [regen-ai Claude marketplace](https://github.com/gaiaaiagent/regen-ai-claude)
- [`regen-compute`](https://github.com/regen-network/regen-compute), by [Regen Network](https://github.com/regen-network)
- The `mcp<2` fix for the ledger server follows Gregory Landua's [gaiaaiagent/regen-ai-claude#4](https://github.com/gaiaaiagent/regen-ai-claude/pull/4)

This repo adds the skills, docs and packaging, and is not an official Regen Network or Gaia AI release.

## Links

- [Regen Network](https://regen.network)
- [Regen Marketplace](https://app.regen.network)
- [Regen Compute](https://compute.regen.network)
- [KOI MCP](https://github.com/gaiaaiagent/regen-koi-mcp)
- [Ledger MCP](https://github.com/gaiaaiagent/regen-python-mcp)
- [Compute MCP](https://github.com/regen-network/regen-compute)
- [Upstream regen-ai marketplace](https://github.com/gaiaaiagent/regen-ai-claude)

## License

Apache-2.0, see [LICENSE](LICENSE). The upstream MCP servers carry their own licences.
