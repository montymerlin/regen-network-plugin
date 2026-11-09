# AGENTS.md — regen-network

Canonical repo instructions for `regen-network-plugin`.

## Project Identity

- **Name:** regen-network
- **Type:** Host-agnostic MCP + skills package with Claude plugin packaging compatibility
- **Version:** 0.3.0
- **Stack:** Markdown skills + `.mcp.json` server definitions + setup docs
- **Repository:** https://github.com/montymerlin/regen-network-plugin
- **Maintainer:** Monty Bryant. The MCP servers are upstream work by Gaia AI and Regen Network (credited in README); this repo owns only the skills, docs and packaging.

## Canonical Structure

```
regen-network-plugin/
├── .claude-plugin/
│   ├── plugin.json          # Claude plugin packaging metadata
│   └── marketplace.json     # Claude marketplace listing
├── .mcp.json                # MCP server definitions (pinned versions)
├── skills/
│   ├── regen-search/
│   ├── regen-digest/
│   ├── regen-credits/
│   ├── regen-market/
│   ├── regen-governance/
│   ├── regen-portfolio/
│   ├── regen-footprint/
│   ├── regen-code/
│   ├── regen-review/
│   └── regen-update/
├── AGENTS.md                # Canonical repo instructions
├── CLAUDE.md                # Claude compatibility wrapper
├── CHANGELOG.md
├── DECISIONS.md
├── ROADMAP.md
├── LICENSE                  # Apache-2.0
├── README.md
└── SETUP.md
```

## Canonical Rules

- `AGENTS.md` is the canonical instruction file for this repo.
- `CLAUDE.md` is a thin compatibility layer that points Claude-family hosts back to `AGENTS.md`.
- `skills/` and `.mcp.json` are the product.
- `.claude-plugin/` is Claude-specific packaging metadata, not the source of truth for skills or MCP configuration.
- Keep host recommendations in docs where they reflect real UX differences, but do not position the repo itself as single-host.

## Packaging for Cowork

Cowork install is via a `.plugin` zip uploaded through Claude Desktop. Two paths:

1. **Use a packager script** (e.g. the `cowork-plugin-packager` skill if installed in your workspace) — runs validation, packaging, and verification. Produces `regen-network-<version>.plugin`.
2. **Build by hand** — see `SETUP.md` § "Cowork" for the raw `zip` command and verification steps.

`SETUP.md` is the single source of truth for install pathways across Cowork, Claude Code CLI, Cursor (MCP), VS Code (MCP), and other MCP-compatible hosts — read it before changing install or packaging behavior.

## Runtime Conventions

- Probe MCP availability before invoking skills that depend on specific servers.
- Three MCP servers are declared: `regen-koi`, `regen-ledger`, `regen-compute`. Do not assume all three are connected in every host. `registry-review` is deliberately not bundled (private install source; Decision 005).
- In skills, reference tools by their plugin-namespaced names, `mcp__plugin_regen-network_<server>__<tool>`, which is what Claude Code exposes for plugin servers. Each skill carries a one-line note for hosts that add the servers directly.
- Server versions are pinned in `.mcp.json` and mirrored in `SETUP.md`. Bump them deliberately, following "Updating a pin" in `skills/regen-update/SKILL.md`; never switch back to `@latest` or an unpinned `uvx`.
- Use a portable intra-repo root variable if future skill references need filesystem lookups:
  `REGEN_NETWORK_ROOT="${REGEN_NETWORK_ROOT:-${CLAUDE_PLUGIN_ROOT:-$PWD}}"`

## Documentation Rules

- README.md is human-facing overview.
- AGENTS.md is the canonical agent-facing document.
- CLAUDE.md exists for compatibility only and should stay short.
- SETUP.md is the per-host setup guide.
- DECISIONS.md logs major structural choices before implementation.
- ROADMAP.md holds future ideas until they become decisions.
- CHANGELOG.md records narrative milestones after significant work.

## Design Principles

1. **Host-agnostic package, host-specific guidance** — the package should run anywhere MCP works, while docs can still recommend the best host UX.
2. **Skills route through MCPs** — do not improvise capabilities outside the declared MCP surface without checking availability.
3. **Progressive disclosure** — keep the repo-level instructions concise and push setup detail into `SETUP.md`.
4. **Safety around transactions** — never execute on-chain actions without explicit confirmation. Skills default to the link-based retirement path and never suggest setting `REGEN_WALLET_MNEMONIC` or `ECOBRIDGE_EVM_MNEMONIC`; with either set, `regen-compute` signs irreversible transactions itself.
5. **Ecosystem clarity** — docs should distinguish knowledge commons, ledger, compute, and registry-review roles cleanly.

## Boundaries

### Do
- Keep MCP dependencies explicit in skill docs
- Update `.mcp.json`, README, and SETUP together when install paths change
- Preserve host recommendations only where they are operationally justified
- Log major structural choices in DECISIONS.md before implementation

### Don't
- Reintroduce `CLAUDE.md` as the canonical repo instruction file
- Hardcode Claude-specific runtime paths as if they define the package
- Assume one host is mandatory when the MCP layer is portable
- Expose mnemonics, API keys, or other sensitive credentials, or ask a user for a seed phrase
