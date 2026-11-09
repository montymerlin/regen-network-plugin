---
name: regen-governance
description: >
  Query Regen Network governance activity. Use when the user asks about
  "regen governance", "proposals", "regen votes", "voting results",
  "community pool", "governance params", "active proposals", or wants
  to understand on-chain governance decisions.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Query and present Regen Network blockchain governance data.

## Workflow

1. **List proposals** — call `mcp__plugin_regen-network_regen-ledger__list_governance_proposals` to see all proposals. Filter by status if the user asks for active, passed, or rejected proposals.

2. **Proposal detail** — for a specific proposal, call `mcp__plugin_regen-network_regen-ledger__get_governance_proposal` with the proposal ID to get full description, timing, and status.

3. **Vote tallies** — call `mcp__plugin_regen-network_regen-ledger__get_governance_tally_result` for a proposal's current or final vote breakdown (Yes, No, Abstain, No With Veto).

4. **Individual votes** — call `mcp__plugin_regen-network_regen-ledger__list_governance_votes` to see who voted and how, or `mcp__plugin_regen-network_regen-ledger__get_governance_vote` for a specific voter's position.

5. **Deposits** — call `mcp__plugin_regen-network_regen-ledger__list_governance_deposits` or `mcp__plugin_regen-network_regen-ledger__get_governance_deposit` to check deposit status for proposals in deposit period.

6. **Governance parameters** — call `mcp__plugin_regen-network_regen-ledger__get_governance_params` three times with `params_type` set to `voting`, `tallying`, and `deposit` respectively, to get the full picture: voting period, quorum, threshold, veto threshold, and deposit requirements.

7. **Community pool** — call `mcp__plugin_regen-network_regen-ledger__get_community_pool` for the current community pool balance.

## Presenting results

- For active proposals, lead with the proposal title, what it does, and when voting ends
- Present vote tallies as percentages alongside raw numbers
- For passed proposals, summarise the outcome and margin
- Note quorum requirements when relevant (from governance params)
