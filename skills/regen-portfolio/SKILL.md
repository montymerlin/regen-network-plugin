---
name: regen-portfolio
description: >
  Analyse a wallet's ecological portfolio on Regen Network. Use when the user
  asks about "regen portfolio", "wallet impact", "analyze address",
  "ecological impact of address", "my regen credits", "portfolio analysis",
  or provides a regen1... address for analysis.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Analyse the ecological impact and credit holdings of a Regen Network wallet address.

## Workflow

1. **Get the address** — the user must provide a `regen1...` address. If they say "my portfolio" without an address, ask for one.

2. **Portfolio impact analysis** — call `mcp__plugin_regen-network_regen-ledger__analyze_portfolio_impact` with the address. Use `analysis_type: "full"` for comprehensive analysis. This returns credit holdings, retirements, and ecological impact breakdown.

3. **Account balances** — call `mcp__plugin_regen-network_regen-ledger__get_all_balances` with the address for a complete view of token holdings (REGEN, ecocredits, basket tokens).

4. **Delegation and staking** — call `mcp__plugin_regen-network_regen-ledger__get_delegation_total_rewards` and `mcp__plugin_regen-network_regen-ledger__get_delegator_validators` to show staking positions and validator delegations.

5. **Sell orders** — call `mcp__plugin_regen-network_regen-ledger__list_sell_orders_by_seller` with the address to check if this wallet has any active marketplace listings.

6. **Basket holdings** — if the address holds basket tokens, call `mcp__plugin_regen-network_regen-ledger__list_basket_balances` to show the underlying credit composition.

## Presenting results

- Lead with the ecological impact summary: total credits held, total retired, credit types represented
- Break down holdings by credit type (carbon, biodiversity, etc.) with quantities
- Show retirement history separately — these represent completed ecological contributions
- Include staking position if the address has REGEN delegated
- If the address has active sell orders, mention marketplace activity
- Note that this is a point-in-time snapshot of on-chain state
