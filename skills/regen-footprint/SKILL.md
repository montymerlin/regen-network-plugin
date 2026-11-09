---
name: regen-footprint
description: >
  Estimate and offset the ecological footprint of an AI session. Use when the
  user asks about "footprint", "offset my session", "offset AI usage",
  "environmental cost", "retire credits", "carbon offset", "session impact",
  "eco footprint", or wants to understand and compensate for the energy cost
  of their AI usage.
metadata:
  version: "0.2.0"
---

> Tool names here are the ones Claude Code gives this package's servers when it is installed as the `regen-network` plugin (`mcp__plugin_regen-network_<server>__<tool>`). Where the servers were added to a host directly, the prefix differs (e.g. `mcp__regen-koi__search`): match on server and tool name.

Estimate the energy and CO2 footprint of the current AI session and offer to offset it by retiring ecological credits on Regen Network.

## Workflow

1. **Estimate footprint** — call `mcp__plugin_regen-network_regen-compute__estimate_session_footprint` with:
   - `session_minutes`: approximate duration of the current session
   - `tool_calls`: approximate number of tool calls made (improves accuracy)

   Present the estimate clearly: energy in kWh, CO2 in kg, and the suggested credit retirement quantity.

2. **Browse matching credits** — call `mcp__plugin_regen-network_regen-compute__browse_available_credits` to show what's available. Let the user choose between carbon and biodiversity credits if both are available.

3. **Offer to retire** — only if the user says they want to. Before calling `mcp__plugin_regen-network_regen-compute__retire_credits`, restate the credit, quantity and approximate cost and wait for a clear yes. Use the **credit card** path: it returns a Regen Marketplace purchase link, and the user completes (or abandons) the purchase in their own browser. Nothing is spent until they do.

4. **Verify retirement** — after retirement, call `mcp__plugin_regen-network_regen-compute__get_retirement_certificate` to retrieve the on-chain proof.

## Monthly perspective

If the user asks about their broader AI usage impact, call `mcp__plugin_regen-network_regen-compute__estimate_monthly_footprint` for a personalised monthly estimate with location and product multipliers.

## Important notes

- Estimates are heuristic-based and clearly approximate — say so
- Read-only tools (estimate, browse) work with zero config
- Credit retirement requires a Regen Compute subscription — mention this if the user wants to retire. Direct them to [compute.regen.network](https://compute.regen.network) for pricing
- All retirements are permanent and recorded on-chain — this is real ecological action, not a simulation
- Present the network impact summary (`mcp__plugin_regen-network_regen-compute__get_impact_summary`) for context on the broader community effort

## Wallet keys and signing

`regen-compute` changes behaviour when a wallet mnemonic is in its environment: with `REGEN_WALLET_MNEMONIC` set, `retire_credits` signs and broadcasts a buy-and-retire transaction itself, and with `ECOBRIDGE_EVM_MNEMONIC` set, `retire_via_ecobridge` can send tokens on another chain. Both are irreversible and spend real funds without a browser step.

- Never suggest setting either variable, never ask for a mnemonic or seed phrase, and never write one into chat, a config file, `.mcp.json` or a shell command. If the user offers one, tell them not to share it and carry on with the link-based path.
- If the user asks about direct on-chain retirement, explain the trade-off and point them to the [regen-compute README](https://github.com/regen-network/regen-compute) to configure it themselves, ideally with a dedicated low-balance wallet.
- If a mnemonic already appears to be configured (a retirement tool returns a transaction hash rather than a link), treat every `retire_credits` and `retire_via_ecobridge` call as a payment: confirm credit, quantity and cost first, and make one call per confirmation.
