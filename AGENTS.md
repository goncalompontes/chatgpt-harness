# Contributor / agent instructions

This is **chatgpt-harness**, an independent Cordis application for ChatGPT Chat. The current branch starts at **planning-only** status.

## Product contract

- Use ChatGPT Chat as the **sole inference and reasoning host**. No model API integrations, no API-key model invocation, no local/remote LLM runner, no separate background agent loop.
- Cordis is the plugin runtime. Every feature beyond minimal bootstrap is a Cordis plugin, including MCP bridge, tools, security policy, session store, UI, shell, Git, worktrees, goals and orchestration.
- Do **not** depend on or import `local-shell-mcp`. It may be studied as a reference for proven API interactions, never as runtime code or a required service.
- Keep MCP entrypoints generic; do not add a new MCP tool for each newly installed plugin.
- Distinguish host-verified capabilities from speculative ChatGPT extension behavior. Feature-detect UI bridges and fall back safely.
- No claim of autonomously resuming ChatGPT turns, reading the host's private model state, spawning a ChatGPT subagent, or bypassing the host's approval flow.

## Before making changes

1. Read `README.md`, `docs/ARCHITECTURE.md`, `docs/IMPLEMENTATION_PLAN.md`, both `docs/specs` and `docs/SECURITY.md`.
2. Inspect status, branches, worktrees and dirty files. Avoid other agents' work.
3. Select a task/phase and make its acceptance contract explicit. Create an isolated worktree/branch with the user's approval before Git mutations.
4. Review current OpenAI docs when changing protocol/host integration; review upstream Cordis interfaces before code changes. Keep exact references and dependency versions.
5. Implement the smallest end-to-end vertical slice. Tests must distinguish mocked host behavior from verified in-ChatGPT behavior.
6. Recheck diff, tests, security boundaries and docs before any commit or PR.

## Guardrails

- Authentication and authorization are mandatory before exposing the shell or filesystem over a remote MCP endpoint.
- No direct plugin bypass of the tool policy pipeline. Tools and Code Mode both use the same principal-scoped authorization path.
- Never execute arbitrary installed plugin code as trusted unless explicitly authorized with dependency/source review and capability grant. External plugin loading is disabled by default.
- No plugin writes outside its granted workspace; resolve canonical paths and reject symlink escapes.
- Commands, Git mutations, remote operations, branch switches, uploads and deletions require appropriate grants or interactive approvals. Report exact requested operations.
- Do not expose tokens, keys, OAuth codes, tunnel credentials, machine fingerprint or private session logs in commits, issues, PRs or diagnostics.
- Do not assume ChatGPT UI message APIs guarantee unattended agent continuation.
- Preserve reversibility: hot-unload tears down event listeners, tools, UI contributions, timers and subprocess resources.

## Definition of done

Each plugin requires: clear contracts and versioning, schema validation, dependency lifecycle tests, unload/reload tests, principal isolation, denial tests, auditability, docs, and a user-facing example. Host-specific UI claims need a real ChatGPT manual gate in addition to mock-protocol integration tests.

## Commit discipline

No force push, rebase, reset, or history rewriting without explicit approval. Preserve unrelated work; prefer scoped changes. When a plugin is not implemented, label documentation as proposed instead of describing it as functional.
