---
name: regen-review
description: >
  Review carbon credit project documents for registry compliance. Use when
  the user asks for a "registry review", "review project documents",
  "document review", "credit registration review", "compliance check",
  or wants to verify project documentation against methodology requirements.
  NOTE: the Registry Review MCP is not bundled with this package (its source
  is private); this skill only explains how to get it.
metadata:
  version: "0.2.0"
  status: "unavailable"
---

> **Status: unavailable in this package.**
>
> The Registry Review MCP is not bundled here. Its install source is the private
> `regen-network/registry-review-mcp` GitHub repo (cloned over SSH), and the PyPI
> name `registry-review-mcp` is unpublished, so a public plugin cannot start it.

## What to tell the user

If the user asks for a registry or project-document review:

1. Say plainly that this package cannot run the review: the server it needs is not publicly installable.
2. If they have access to `regen-network/registry-review-mcp` on GitHub (Regen team members and partners), point them to the `registry-review` plugin in the upstream marketplace, [gaiaaiagent/regen-ai-claude](https://github.com/gaiaaiagent/regen-ai-claude), which installs it over SSH and needs an Anthropic API key for LLM extraction.
3. Otherwise, offer what this package can do: look up the methodology and credit class with `/regen-credits`, and search KOI with `/regen-search` for the methodology's documentation and past project reviews.

## What the server does (for context)

An 8-stage review of carbon credit project documents against a methodology checklist: initialise a session, discover and classify documents, map them to requirements, extract evidence with page citations, cross-validate dates, tenure and IDs, generate a Markdown + JSON report, flag items for human review, and finalise. LLM extraction incurs Anthropic API costs.

## Source

- Upstream plugin: [gaiaaiagent/regen-ai-claude](https://github.com/gaiaaiagent/regen-ai-claude) → `plugins/registry-review`
- Architecture: [Regen AI forum post](https://forum.regen.network/t/announcing-regen-ai/553)
