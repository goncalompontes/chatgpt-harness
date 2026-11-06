# ChatGPT Harness

**A plugin-first, Cordis-powered development harness for ChatGPT Chat.**

Status: **architecture and implementation planning only**. This repository does not yet ship an MCP server, a Cordis runtime, or an installable ChatGPT plugin. The public repository is an independent greenfield project.

## Mission

Expose a modular Cordis application to ChatGPT through **one stable MCP integration**. Tools, goals, shell access, Git worktrees, session persistence, code-mode, orchestration, and UI are independent Cordis plugins. ChatGPT Chat supplies all model inference, conversational reasoning and turn scheduling; the local harness supplies tools, durable state, policies and UI.

A plugin can be installed or unloaded without modifying the ChatGPT bridge. The bridge discovers, validates and dispatches capabilities at runtime.

## Non-negotiable boundaries

- **ChatGPT Chat only:** no external model APIs, direct OpenAI API, API keys for inference, local LLMs, model routing, or independent model-driven agent loop.
- **New project, from scratch:** not a fork of DeepSeek Harness, not an extension or dependency of `local-shell-mcp`. Cordis is an explicit dependency to evaluate and pin; other DeepSeek components are research references only unless specifically selected, reviewed and licensed.
- **Everything is a Cordis plugin:** each capability registers services, tools, event handlers, and UI contributions using lifecycle-managed effects.
- **One MCP surface:** use stable catalog/inspect/invoke/execute/session/UI tools; no per-feature handwritten MCP server changes.
- **No fictitious autonomy:** an MCP server cannot drive ChatGPT's internal model loop. Goals and workflows persist state and can request host follow-ups only where supported and user-approved; unattended agent execution is out of scope.
- **Security first:** authenticated principals, workspaces, approval gates, tool isolation, auditing, explicit scope, and no unauthenticated shell access.

## Architecture in one line

ChatGPT Chat ↔ authenticated ChatGPT MCP / MCP Apps bridge ↔ Cordis context + plugin loader ↔ independent tool, state, policy and UI plugins.

See [Architecture](docs/ARCHITECTURE.md), [Plugin contract](docs/specs/PLUGIN_CONTRACT.md), [MCP bridge contract](docs/specs/MCP_BRIDGE.md), and [Security](docs/SECURITY.md).

## Planned plugin ecosystem

| Package | Responsibility |
| --- | --- |
| `core` | Bootstrap Cordis, configuration profiles, health and lifecycle |
| `plugin-tools` | Registered tool schemas, discovery, dispatch and policy pipeline |
| `plugin-sessions` | Durable append-only events, session ownership, replay and handoffs |
| `plugin-approvals` | Authorization, consent, approval checks, audit |
| `plugin-mcp-bridge` | Stable remote MCP endpoint, OAuth, error normalization |
| `plugin-chatgpt-ui` | MCP Apps UI host and contributed panes |
| `plugin-files` | Scoped file reads, edits and diffs |
| `plugin-shell` | Allowlisted local shell execution and optional tracked jobs |
| `plugin-git` | Read-only Git inspection and explicit mutation gates |
| `plugin-worktrees` | Isolated worktree lifecycle and ownership |
| `plugin-code-mode` | Bounded, sandboxed multi-tool orchestration |
| `plugin-goals` | Goals, plans, acceptance checks and resumable progress |
| `plugin-orchestration` | Explicit cooperative workflows and delegated tasks (not independent LLM workers) |
| `plugin-extensions` | Plugin discovery, install/unload, config, compatibility and permissions |

The exact npm workspace names and Cordis versions are pinned only after the Phase 0 proof.

## Planned monorepo layout

```text
apps/
  server/                 # Cordis runtime entrypoint
  ui/                     # ChatGPT-hosted MCP Apps UI
packages/
  core/
  sdk/
  plugins/
    tools/ sessions/ approvals/ mcp-bridge/ chatgpt-ui/
    files/ shell/ git/ worktrees/ code-mode/ goals/
    orchestration/ extensions/
examples/
  hello-plugin/
tests/
  contracts/ integration/ e2e/
docs/
```

The directories above are **proposed**, not implemented.

## Development sequence

1. Prove a minimal Cordis plugin dynamically changes the catalog without a server patch.
2. Serve the catalog through a read-only MCP endpoint, then test ChatGPT connection and OAuth.
3. Add validated dispatch, scoped permissions, event log and lifecycle teardown.
4. Add sandboxed code-mode, files, shell, Git and worktrees as separate plugins.
5. Add goals/orchestration, plugin-manager and composite UI surfaces.
6. Demonstrate an end-to-end ChatGPT-only, multi-turn coding workflow with no external inference provider.

[Detailed staged backlog](docs/IMPLEMENTATION_PLAN.md) · [Verification](docs/TESTING.md) · [Design decisions](docs/adr/0001-chatgpt-only.md) · [References](docs/REFERENCES.md)

## Initial repository policy

Before changing source: read `AGENTS.md` and the linked architectural contracts. Feature work occurs on isolated Git branches/worktrees. Keep tool grants and operation logs separate from project source. Never commit tokens, tunnels, credentials, user workspaces or private session history.

## Attribution

Inspired by DeepSeek Harness's open-source Cordis architecture, but intentionally does **not** clone or vendor DeepSeek Harness. Any future reuse of MIT-licensed code requires preserving the upstream notices and identifying the exact imported components. This repository's code license is **not selected yet**; public visibility does not imply an open-source license.
