# Decisions — regen-network

Architectural decisions for the regen-network plugin, in lightweight ADR format.

---

## Decision 001 — Adopted agentic scaffold (2026-04-21)

**Context:** The plugin had a README and SETUP.md but no agent instruction set (CLAUDE.md), no decision log, no roadmap, and no changelog. As the plugin grows (registry-review skill planned for v0.2.0, potential new MCP integrations), conventions need to be explicit so future sessions don't re-derive them.

**Decision:** Add the standard agentic scaffold (CLAUDE.md, CHANGELOG.md, DECISIONS.md, ROADMAP.md) alongside the existing README.md and SETUP.md.

**Consequences:**
- Agent sessions have explicit conventions for skill authoring, MCP dependencies, and versioning
- Structural decisions have a home rather than being buried in commit messages
- Future directions (registry-review, additional skills) are tracked in ROADMAP.md
- Adds four small files; each serves a distinct purpose

**Alternatives Considered:**
- *Leave as-is* — sufficient at v0.1.0 scale but increasingly fragile as the plugin grows
- *Embed everything in README* — conflates human-facing and agent-facing documentation

---

## Decision 002 — Dual-distribution packaging with marketplace.json (2026-04-21)

**Context:** The plugin README mentioned Cowork as "recommended" and other hosts as alternatives, but had no marketplace.json for Claude Code CLI installation. As the broader plugin portfolio standardised on dual-distribution, regen-plugin needed the same treatment.

**Decision:** Add `.claude-plugin/marketplace.json`, add a Distribution section to CLAUDE.md, and update the directory structure listing.

**Consequences:**
- Users can install via `claude plugins install github.com/regen-network/regen-ai-claude`
- Cowork remains the recommended host (slash command UX), but Claude Code CLI is now a first-class install path
- marketplace.json version must stay in sync with plugin.json on each release

**Alternatives Considered:**
- *Cowork only* — rejected. Claude Code CLI users working with Regen data should be able to install the plugin normally
- *Add COMPATIBILITY.md* — deferred. SETUP.md already covers per-environment configuration; the Distribution section in CLAUDE.md is sufficient for now

---

## Decision 004 — Drop the two unrunnable MCP servers from `.mcp.json` (2026-08-21)

**Context:** Two of the four bundled MCP servers could not start, and every Claude Code session opened with a "MCP regen-network: Server disconnected" error toast. Both failures were verified upstream, not local misconfiguration:

- `regen-network` (`uvx regen-python-mcp`) installs, then crashes on import with `ModuleNotFoundError: No module named 'mcp.server.fastmcp'`. The published package declares an unpinned `mcp` dependency and `FastMCP` has since moved out of `mcp.server.fastmcp`, so a fresh resolve always pulls an incompatible SDK.
- `registry-review` (`uvx registry-review-mcp`) is not on PyPI at all: "registry-review-mcp was not found in the package registry." The README and SETUP.md both link a PyPI page that does not resolve to an installable distribution.

Neither server has ever been reachable from this machine, so removing them costs no working capability.

**Decision:** Remove the `regen-network` and `registry-review` entries from `.mcp.json`, keeping `regen-koi` and `regen-compute`, which both connect. Skills stay installed. Documentation still describes four MCP servers; that drift is deliberately left for a v0.3.0 docs pass rather than rewritten piecemeal here.

**Consequences:**
- Sessions start clean — all remaining servers report Connected
- `regen-review` has no backend and should be treated as non-functional until `registry-review-mcp` is published
- `regen-code`'s ledger-side lookups lose the Python MCP; the KOI code-graph tools still work
- `regen-update` checks four packages and will now report on two that are no longer wired in
- README.md, SETUP.md, and AGENTS.md still say "four MCP servers" and are now inaccurate

**Alternatives Considered:**
- *Fix `regen-python-mcp` upstream* — the right long-term fix (pin `mcp`, or migrate to the current `FastMCP` import path), but it is a separate change in a repo we do not own
- *Disable the whole plugin* — rejected. It would also drop two healthy servers and all ten skills
- *Leave them in and dismiss the toast each session* — rejected. A server that cannot start should not be declared

---

## Decision 003 — Reframe regen-network as a host-agnostic package (2026-04-22)

**Context:** The plugin had already been distributed to multiple hosts, but the repo still read as Claude-first and Cowork-led in a way that overstated the host identity of the package itself. The underlying product is really a bundle of MCP server definitions, markdown skills, and setup guidance that can be used anywhere MCP works. Cowork may still be the smoothest UX, but it should be described as a recommended host experience, not as the product’s defining context.

**Decision:** Reframe the repo as a host-agnostic MCP + skills package. Add canonical `AGENTS.md`, turn `CLAUDE.md` into a thin compatibility wrapper, and rewrite README/SETUP so host differences are presented as setup and UX differences rather than product identity. Keep Cowork recommendations where they reflect real UX advantages.

**Consequences:**
- The repo now matches the compatibility-layer pattern used elsewhere in the workspace
- Documentation is clearer for non-Claude MCP hosts without losing the Cowork recommendation
- The underlying MCP and skill behavior remains unchanged
- Future host-specific notes can live in setup docs without redefining the package itself

**Alternatives Considered:**
- *Leave the Claude-first framing in place* — rejected. It misdescribes what the package actually is
- *Remove all host recommendations entirely* — rejected. Cowork still offers a materially better slash-command UX for some users

---

## Decision 005 — v0.3.0: own the package honestly, restore the ledger, pin everything (2026-11-09)

**Context:** Monty wants to propose that the Regen team adopt this plugin's skills into their Claude marketplace and maintain them, [gaiaaiagent/regen-ai-claude](https://github.com/gaiaaiagent/regen-ai-claude) (maintained by Gaia AI; the `regen-network/regen-ai-claude` fork has never merged a PR). That marketplace lists three single-server MCP-only plugins: `koi`, `ledger`, `registry-review`. Before offering it, a review found the package misdescribed itself: the manifests named Regen Network as author and `regen-network/regen-ai-claude` as the repository; the docs promised four servers while `.mcp.json` declared two; skills called tool names (`mcp__regen-network__*` etc.) that do not exist inside a plugin, where Claude Code exposes `mcp__plugin_regen-network_<server>__*`; servers floated on `@latest`, which is how an unbounded `mcp` dependency broke the ledger server in the first place (Decision 004); and the README said nine skills where there are ten.

On 2026-11-09 the ledger server was re-tested: `uvx regen-python-mcp==0.1.0` still crashes (`mcp` 2.x removed `mcp.server.fastmcp`), while `uvx --with 'mcp<2' regen-python-mcp==0.1.0` initialises (`regen-network-mcp-modular` 1.30.0), lists 45 tools including every tool the skills reference, and answers a live `list_credit_types` call. This is the fix proposed upstream in gaiaaiagent/regen-ai-claude#4 (Gregory Landua). `registry-review`'s upstream install source is `git+ssh://git@github.com/regen-network/registry-review-mcp.git`, which is private (404 from outside), and the PyPI name remains unpublished.

**Decision:**
1. **Ownership:** author, marketplace owner, homepage and repository point at Monty Bryant and `montymerlin/regen-network-plugin`. Upstream servers are credited in a README Credits section, with a line saying this is not an official Regen Network or Gaia AI release. Add an Apache-2.0 `LICENSE` (Monty chose Apache over MIT for continuity with the declared licence and with `regen-compute`).
2. **Restore the ledger server** as `regen-ledger` with the `mcp<2` pin. The key is renamed from `regen-network` to avoid the stuttering tool prefix `mcp__plugin_regen-network_regen-network__*` and to name the layer it serves.
3. **Pin all three servers** to the versions tested on 2026-11-09: `regen-koi-mcp@1.6.6`, `regen-python-mcp==0.1.0` with `mcp<2`, `regen-compute@0.3.4`. `regen-update` now reports pinned vs published and documents how to bump a pin.
4. **Leave `registry-review` out** and say why in README, SETUP and the `regen-review` skill, which now explains the gap and points users with repo access to the upstream plugin.
5. **Skills use plugin-namespaced tool names**, each with a one-line note for hosts that add servers directly.
6. **Wallet safety:** `regen-footprint` defaults to the credit card link, confirms before any retirement call, and never suggests setting `REGEN_WALLET_MNEMONIC` / `ECOBRIDGE_EVM_MNEMONIC` or asks for a seed phrase; README replaces the env-var table with a Wallet keys section.

**Consequences:**
- The four ledger-backed skills (`regen-credits`, `regen-market`, `regen-governance`, `regen-portfolio`) work again
- Upstream releases no longer reach users silently; each bump is a plugin release with a CHANGELOG entry, and staying current depends on running `/regen-update`
- The `mcp<2` pin is a workaround with an exit: drop it once `regen-python-mcp` bounds `mcp` or moves to the 2.x `MCPServer` API
- Installing `koi@regen-ai` or `ledger@regen-ai` beside this plugin starts the same servers twice; README and SETUP now say so
- Users who added servers by hand under the old `regen-network` key keep working, but the skills' tool names no longer match theirs

**Alternatives Considered:**
- *Docs say two servers, ledger stays out* — rejected once the pinned server tested clean; four skills depend on it
- *Keep `@latest`* — rejected: it is what broke the ledger server, and an upstream rename would silently break skills
- *Pin `mcp` exactly (`mcp==1.30.0`)* — rejected for now; `mcp<2` matches upstream's PR and still takes 1.x patch releases
- *Skills-only variant that relies on upstream `koi`/`ledger` plugins* — not built; kept as an option in ROADMAP. It removes the double-loading risk and leaves server ownership with Gaia AI, but depends on upstream merging the `mcp<2` fix and on a cross-plugin tool prefix (`mcp__plugin_koi_regen-koi__*`), and still needs `regen-compute` from somewhere
