# ADR-0001 — ChatGPT Chat owns the agent loop

Status: **accepted** · 2026-11-06

## Context

The product is a fully plugin-based harness **inside ChatGPT Chat**, modeled on Cordis extensibility. The maintainer explicitly rejects external model API providers and rejects building on local-shell-mcp. ChatGPT provides the model and native conversation.

## Decision

- Keep exactly one runtime reasoning host: ChatGPT Chat.
- Do not implement an LLM adapter, independent DeepSeek agent loop, auto-compaction runner or external model token billing.
- The Cordis app is a tool/session/policy/UI server called by ChatGPT over MCP. A plugin can provide deterministic/background *tool jobs* without making any new model requests.
- Goals, orchestration and delegation are plugins that coordinate tools and persisted tasks, not external LLM workers.
- ChatGPT follow-up via UI is best-effort and human/host driven. A session can persist but the model does not run after its turn ends unless the host explicitly starts another turn.

## Consequences

Pros: subscription-hosted ChatGPT interactions, no model credential setup, one tool/permission authority, extension design remains modular.

Limits: no guaranteed on-turn-end loop, no invisible ChatGPT calls, no true independent ChatGPT subagents or model-level scheduling from Cordis, no reading entire ChatGPT conversation from the server. Plans must expose explicit continuation/handoff affordances.

This is intentional. If the requirements ever demand unattended model-driven execution, that is a new product decision and **not** an implicit feature of this repository.
