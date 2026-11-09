---
name: regen-update
description: >
  Check for updates across the regen-network plugin's upstream MCP dependencies
  and skill coverage. Use when the user asks to "check for regen updates",
  "update regen plugin", "what's new in regen MCPs", "are there new regen tools",
  or wants to know if any upstream packages have new versions or capabilities
  not yet reflected in this plugin's skills.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Check the three upstream MCP packages this plugin pins against their latest
published versions, and surface any MCP tools not yet covered by a skill. This
skill reports; it does not edit `.mcp.json`. Bumping a pin is a deliberate
release of this plugin (see "Updating a pin" below).

## Step 1: Read the pinned versions

Read `.mcp.json` at the plugin root (`${CLAUDE_PLUGIN_ROOT:-.}/.mcp.json`). Each
server's `args` carries its pin:

| Server | Package | Registry |
|--------|---------|----------|
| `regen-koi` | `regen-koi-mcp@<version>` | npm |
| `regen-ledger` | `regen-python-mcp==<version>` (plus `--with mcp<2`) | PyPI |
| `regen-compute` | `regen-compute@<version>` | npm |

## Step 2: Check published versions

```bash
npm view regen-koi-mcp version
npm view regen-compute version
curl -s https://pypi.org/pypi/regen-python-mcp/json | python3 -c 'import json,sys; print(json.load(sys.stdin)["info"]["version"])'
```

Without a shell, fetch `https://registry.npmjs.org/<package>/latest` (`.version`)
and `https://pypi.org/pypi/regen-python-mcp/json` (`.info.version`).

For `regen-python-mcp`, also check whether its `requires_dist` now bounds `mcp`
below 2, or whether it has moved to the `mcp` 2.x `MCPServer` API. Either one
means the `--with mcp<2` workaround can be dropped.

## Step 3: Check for tools not covered by skills

Tools the skills reference:

```bash
grep -rho 'mcp__plugin_regen-network_[a-z-]*__[a-z_]*' skills/ | sort -u
```

Tools the connected servers expose: list the tools available under
`mcp__plugin_regen-network_regen-koi__*`, `mcp__plugin_regen-network_regen-ledger__*`
and `mcp__plugin_regen-network_regen-compute__*`. Any exposed tool that no skill
references is a coverage gap. Flag it as a candidate for a new skill or an
existing one; do not flag `retire_via_ecobridge` as a gap to fill casually,
since it can move funds (see `regen-footprint`).

## Step 4: Read release notes for newer versions

- KOI MCP: `https://github.com/gaiaaiagent/regen-koi-mcp/releases`
- Ledger MCP: `https://github.com/gaiaaiagent/regen-python-mcp/releases`
- Compute: `https://github.com/regen-network/regen-compute/releases`

Summarise new tools, changed parameters, and anything touching signing or
payments in `regen-compute`.

## Step 5: Report

```
## Regen Plugin Update Report

### MCP package versions
| Package | Pinned | Latest | Status |
|---------|--------|--------|--------|
| regen-koi-mcp (npm) | x.y.z | x.y.z | up to date / update available |
| regen-compute (npm) | ... | ... | ... |
| regen-python-mcp (PyPI) | ... | ... | ... (+ mcp<2 still needed?) |

### Tools not yet covered by skills
[list, or "None found"]

### Notable changes in newer versions
[summary, or "No updates"]

### Recommended actions
[specific next steps, or "No action needed"]
```

Saying everything is up to date is a useful outcome too.

## Updating a pin

1. Change the version in `.mcp.json` (and the matching snippets in `SETUP.md`).
2. Start the server once and list its tools to confirm it initialises, e.g.
   `npx -y regen-koi-mcp@<new>` or `uvx --with 'mcp<2' regen-python-mcp==<new>`
   driven by an MCP client, or restart the host and check `/mcp`.
3. Re-run Step 3 so skills match the new tool surface.
4. Bump the plugin version in `.claude-plugin/plugin.json` and `marketplace.json`,
   and log the change in `CHANGELOG.md`.

`registry-review` is not checked: its server installs from a private GitHub
repo and is not part of this package (see `regen-review`).
