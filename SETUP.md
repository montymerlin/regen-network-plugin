# Setup Guide

This package works anywhere MCP is supported. The host changes the setup flow and the UX, not the underlying capabilities.

It runs three MCP servers, each pinned to a tested version: `regen-koi` (knowledge commons), `regen-ledger` (Regen Ledger chain data) and `regen-compute` (AI footprint and credit retirement). `registry-review` is not bundled: its server installs from a private GitHub repo (see [README](README.md#not-bundled-registry-review)).

---

## Quick Install (for AI agents)

If you're an AI agent installing this plugin, **detect the host first**, then pick the matching path. Only ask the user the questions explicitly listed below.

```
1. Detect host:
   ├── Cowork (Claude Desktop, sandboxed)
   │     → Path A: Cowork upload (no questions needed)
   │     → MCPs auto-load. Skills appear under /regen-*.
   │
   ├── Claude Code CLI (terminal, has `claude` command)
   │     → Path B1: install from the marketplace (MCPs + skills + slash commands)
   │     → ASK USER: "Install for every project (user scope) or this project only (project scope)?"
   │       • user    → `claude plugin install regen-network@regen-network`
   │       • project → `claude plugin install regen-network@regen-network --scope project`
   │     → Path B2 (alternative): `claude mcp add` for MCPs only, if the user wants tools without skills
   │
   ├── Cursor (has Cursor MCP config)
   │     → Path C: Add MCP servers to .cursor/mcp.json (no skills loaded)
   │
   ├── VS Code (Continue.dev or Copilot Chat)
   │     → Path D: Add MCP servers to host's MCP config (no skills loaded)
   │
   └── Other MCP-compatible host
         → Path E: Reuse the .mcp.json server definitions in the host's MCP config location
```

**Host detection signals:**
- `$CLAUDE_COWORK == "1"` or `mount | grep virtiofs` matches → Cowork
- `command -v claude` succeeds and not Cowork → Claude Code CLI
- `~/.cursor/` exists → Cursor
- `~/.continue/` exists → Continue.dev (VS Code)

**Pre-install checks (always):**
- The servers reach external endpoints (KOI, Regen Ledger, Regen Compute). Verify the host has outbound HTTPS to `regen.gaiaai.xyz` and `regen.network` before troubleshooting tool failures.
- `npx` (Node) and `uvx` (uv) must be on PATH for the servers to start.
- Check whether `koi@regen-ai` or `ledger@regen-ai` is already installed. If so, ask the user which to keep; running both starts the same servers twice (see [README](README.md#using-this-beside-the-upstream-regen-ai-plugins)).
- Never set `REGEN_WALLET_MNEMONIC` or `ECOBRIDGE_EVM_MNEMONIC` as part of an install. With either set, `regen-compute` signs irreversible transactions itself (see [README](README.md#wallet-keys)).

---

## Compatibility Matrix

| Host | Status | Notes |
|------|--------|-------|
| **Cowork** | ✓ | One-click plugin upload, skills in `/` menu, auto MCP loading |
| **Claude Code CLI** | ✓ | Marketplace install, slash commands and natural-language invocation |
| **Cursor / VS Code** | ✓ MCP only | MCPs work; skills don't load (requires Continue/Copilot Chat) |
| **Codex (OpenAI)** | partial | Use global install script (not applicable here) |
| **Agent SDK** | ✓ | Standard plugin loader via `.mcp.json` |
| **Anthropic API direct** | partial | Manual tool wiring from MCP definitions |

---

## Cowork

Cowork provides the smoothest user experience: one-click plugin install, skill slash commands in the `/` menu, and automatic MCP loading.

You need a `.plugin` zip to upload. Get one of these three ways:

**Option 1 — pre-built release (preferred when available):**
Download `regen-network-<version>.plugin` from the GitHub Releases page of `montymerlin/regen-network-plugin`.

**Option 2 — built locally from this repo:**

```bash
git clone https://github.com/montymerlin/regen-network-plugin.git
cd regen-network-plugin
zip -r /tmp/regen-network-0.3.0.plugin . \
  -x "*.DS_Store" "*/__pycache__/*" "*.pyc" ".git/*" "node_modules/*" "*.log" "_dist/*"
```

The output `/tmp/regen-network-0.3.0.plugin` is your upload artifact.

**Option 3 — built by a packager skill** (e.g. the workspace `cowork-plugin-packager`):
The packaged file lives at `<workspace>/ops/plugins/_dist/regen-network-0.3.0.plugin` after the skill runs.

Then upload:

1. Open Claude Desktop → **Cowork** → **Plugins** in the sidebar.
2. Click **+ Add plugin** → **Upload a file** → select the `.plugin`.
3. Confirm install. Skills appear under `/regen-*` commands; MCPs auto-load.

**Pre-upload verification (recommended):**

```bash
# Confirm the manifest is at the zip root
unzip -l /tmp/regen-network-0.3.0.plugin | head -20

# Confirm size is under 50 MB (this plugin packages to tens of KB — metadata only; servers install via npx/uvx at runtime)
du -h /tmp/regen-network-0.3.0.plugin
```

**What gets installed:** Ten skills (`/regen-search`, `/regen-digest`, `/regen-credits`, `/regen-market`, `/regen-governance`, `/regen-portfolio`, `/regen-footprint`, `/regen-code`, `/regen-review`, `/regen-update`) + three MCP servers (regen-koi, regen-ledger, regen-compute).

---

## Claude Code CLI

Best for developers working in the terminal.

**Recommended path — marketplace install** (MCPs, skills and slash commands together). Inside Claude Code, one step:

```
/plugin install regen-network --marketplace montymerlin/regen-network-plugin
```

Or from a shell:

```bash
claude plugin marketplace add montymerlin/regen-network-plugin
claude plugin install regen-network@regen-network
```

Add `--scope project` to the install to record it in the current project's `.claude/settings.json` instead of for every project.

For local development of the plugin itself, run Claude Code against a checkout with `claude --plugin-dir /path/to/regen-network-plugin`.

**Alternative — MCP-only install** (tools without skills):

```bash
# KOI — knowledge commons
claude mcp add regen-koi \
  --env KOI_API_ENDPOINT=https://regen.gaiaai.xyz/api/koi \
  --env JENA_ENDPOINT=https://regen.gaiaai.xyz/api/koi/fuseki/koi/sparql \
  -- npx -y regen-koi-mcp@1.6.6

# Ledger — blockchain data (mcp<2: the package imports an API that mcp 2.x removed)
claude mcp add regen-ledger -- uvx --with 'mcp<2' regen-python-mcp==0.1.0

# Compute — footprint and credit retirement links
claude mcp add regen-compute -- npx -y regen-compute@0.3.4
```

**What gets installed via the marketplace:** Ten skills (slash commands or natural language) + three MCP servers (regen-koi, regen-ledger, regen-compute).
**What gets installed via `claude mcp add`:** Three MCP servers only. Skills are not loaded, and tool names use the bare server prefix (e.g. `mcp__regen-koi__search`).

---

## Cursor

Add the MCP servers to `.cursor/mcp.json` in your project root, or `~/.cursor/mcp.json` for global config:

```json
{
  "mcpServers": {
    "regen-koi": {
      "command": "npx",
      "args": ["-y", "regen-koi-mcp@1.6.6"],
      "env": {
        "KOI_API_ENDPOINT": "https://regen.gaiaai.xyz/api/koi",
        "JENA_ENDPOINT": "https://regen.gaiaai.xyz/api/koi/fuseki/koi/sparql"
      }
    },
    "regen-ledger": {
      "command": "uvx",
      "args": ["--with", "mcp<2", "regen-python-mcp==0.1.0"],
      "env": {
        "REGEN_MCP_LOG_LEVEL": "INFO"
      }
    },
    "regen-compute": {
      "command": "npx",
      "args": ["-y", "regen-compute@0.3.4"]
    }
  }
}
```

**What gets installed:** Three MCP servers (regen-koi, regen-ledger, regen-compute). Tools are available in Composer; skills don't load in Cursor.

---

## VS Code

**GitHub Copilot Chat** — add the same three server definitions to your `settings.json` under `"github.copilot.chat.mcp.servers"` (same `command` / `args` / `env` as the Cursor block above).

**Continue.dev** — add the servers to `~/.continue/config.json` under `"mcpServers"`, using the same format as Cursor.

**What gets installed:** Three MCP servers (regen-koi, regen-ledger, regen-compute). Tools are available in chat; skills don't load.

---

## Other MCP-Compatible Hosts

Any host supporting MCP stdio servers can use this package. Reuse the `command` / `args` / `env` definitions from `.mcp.json` and place them wherever that host stores MCP config.

**What gets installed:** Three MCP servers (regen-koi, regen-ledger, regen-compute). Skills availability depends on host skill loader support.

---

## Updating pinned versions

Server versions are pinned in `.mcp.json` and repeated in the snippets above. To move a pin, follow "Updating a pin" in `skills/regen-update/SKILL.md`: change the version, start the server once and list its tools, re-check skill coverage, then bump the plugin version and log it in `CHANGELOG.md`. `/regen-update` reports which pins are behind.

---

## Known Quirks

- **Cowork 50 MB file size limit:** The packaged `.plugin` file must stay under 50 MB. This plugin is manifests, skills and MCP definitions only; the server code installs via `npx`/`uvx` at runtime.
- **Cowork `.plugin` vs `.zip` extension:** Some Claude Desktop builds reject `.plugin` at the upload dialog despite accepting it in the docs. Workaround: rename the file to `.zip` (contents are identical). Tracked in [anthropics/claude-code#28337](https://github.com/anthropics/claude-code/issues/28337) and [#40414](https://github.com/anthropics/claude-code/issues/40414).
- **MCP server reachability:** The servers require outbound HTTPS to:
  - `https://regen.gaiaai.xyz/api/koi` (KOI semantic index and Jena SPARQL)
  - Regen Ledger public endpoints on `regen.network` (ledger queries)
  - `https://compute.regen.network` (Regen Compute API)

  If your network blocks these endpoints, MCP tools will time out or fail gracefully. Verify network access before troubleshooting tool failures.
- **Plugin manifest at top level:** When packaging the plugin, `.claude-plugin/plugin.json` must reside at the top level of the `.plugin` zip, not nested inside an extra parent directory. Mis-nesting breaks plugin detection. The `zip` command in the Cowork section above runs from inside the plugin dir specifically to avoid this.
- **First start is slower:** `npx` and `uvx` download each pinned package on first use and cache it afterwards, so the first session after install or a pin bump takes a few extra seconds.
- **`mcp<2` on the ledger server:** `regen-python-mcp` 0.1.0 declares `mcp>=1.0.0` without an upper bound and imports `mcp.server.fastmcp`, which `mcp` 2.x removed. Without `--with 'mcp<2'` it exits on start with `ModuleNotFoundError`.
