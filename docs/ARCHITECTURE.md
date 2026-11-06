# Architecture: ChatGPT-native Cordis Harness

Status: proposed (2026-11-06). This is a **new** Cordis application, not a DeepSeek Harness fork and not a local-shell-mcp extension.

## Conceptual boundaries

```text
ChatGPT Chat (reasoning + turn ownership)
   │ prompts / tool calls / tool results / user approvals
   ▼
ChatGPT MCP endpoint (one connected plugin)
   ├─ stable catalog/inspect/invoke/execute/session operations
   ├─ optional MCP Apps resource + ChatGPT Extensions entrypoints
   └─ OAuth resource server and principal verification
   ▼
Cordis application (Context, dependency injection, typed events, effect lifecycle)
   ├─ tool registry + validation / authorization pipeline
   ├─ sessions / append-only events + projections
   ├─ UI contribution registry
   ├─ configuration and plugin lifecycle
   └─ plugins: files, shell, git, worktrees, goals, code-mode, etc.
   ▼
Host resources (explicitly granted local workspace / process / Git repo)
```

**No model service exists below the MCP boundary.** ChatGPT is not called through an API; ChatGPT calls *us*. There is no server-owned agent loop, LLM adapter, multi-model router or hidden inference endpoint. Cordis 'orchestration' schedules verified tool workflows and records execution boundaries, but must not pretend it can produce new ChatGPT model turns.

## Top-level responsibilities

### 1. Minimal runtime

A TypeScript/Node monorepo starts a Cordis Context and mounts plugins described in a validated local profile. Bootstrap must contain as little product logic as possible: config parsing, an immutable identity for the running instance, startup/shutdown, health and plugin mounting. Choose/pin `cordis` versus `@deepseek-ai/cordis` only after compatibility and licensing checks; avoid mixing incompatible package scopes.

### 2. Core tools service

Each plugin contributes capabilities to `ctx.tools`. Required features: namespaced immutable IDs, structured metadata, JSON Schema input/output, optional UI resource, permission requirements, annotations (read-only, mutating, destructive), version, owner plugin, and lifecycle registration token. Tool lifecycle automatically unregisters and invalidates stale handles. Names are discoverable dynamically through a **stable** set of MCP operations. New tools do not require modifying/redeploying the bridge.

### 3. Sessions and events

Each session is durable with an owner principal, workspace scope, optional goal metadata and append-only events; project deterministic read projections for ChatGPT and UI. Events include `session.created`, `tool.requested`, `tool.approved` / `tool.denied`, `tool.started`, `tool.completed` / `tool.failed`, `goal.updated`, `plugin.loaded` / `plugin.unloaded`. Assign monotonic offsets and ids; idempotent replay must not repeat side effects. Since ChatGPT does not expose a guaranteed stable thread ID, associate sessions via explicit session ID/handoff rather than guessing conversation identity.

### 4. Policy and execution

Every invocation, including nested code-mode and UI-generated tool calls, passes identical authentication, authorization, resource lease and audit middleware. Tool plugins declare required grants; they never directly grant themselves privileges. Invocation includes principal, correlation ID, session ID, workspace grants, plugin ID, input and optional idempotency key. Callback checks happen at actual execution time, not only during discovery. No raw shell operations through generic 'eval' or plugin-manager escape routes.

### 5. Plugin lifecycle

Cordis owns install/load/update/unload and effect disposal. The runtime exposes a plugin-manager *service* but the external installation flow is a guarded plugin requiring explicit source trust, packages audit, capability approval and safe rollback. A plugin may register services, tools, events, view contributions, settings and commands; failed install must leave no partial registrations. Track plugin dependencies and semver contract range. Disallow untrusted third-party plugins in the MVP.

### 6. ChatGPT bridge

MCP adapter translates between host tool requests and Cordis service operations. The tool surface stays stable: discovery, inspection, single invocation, bounded Code Mode, session/goal handles and opening UI. This adapter cannot access internal ChatGPT messages or schedule its native agent loop. MCP Apps UI uses `ui/initialize`, `tools/call` and `ui/message` as supported; ChatGPT-specific extensions are feature-detected and optional.

### 7. Client-side UI

UI is itself a Cordis plugin providing a registry of panel contributions. The ChatGPT-hosted MCP App reads a signed/authorized view manifest, not arbitrary JavaScript injected from every installed package. Initial UI: session list, activity timeline, tool details, permissions/approvals, worktree view and progress. New plugins can contribute declarative views; executable custom frontend components require a separate, explicit trust/CSP design. UI is not a source of session authority.

## Plugin graph (proposed)

```text
bootstrap
  ├── sessions
  ├── tools  ─── approvals
  │     ├── files
  │     ├── shell ── jobs
  │     ├── git ─── worktrees
  │     ├── code-mode
  │     └── goals ── orchestration
  ├── ui-registry ── chatgpt-ui
  ├── plugin-manager
  └── mcp-bridge ── oauth
```

The graph shows dependency injection needs, not mandatory startup order. Tests must ensure a missing dependency suspends only the affected plugin, and removes its effects on unload.

## Modes and non-goals

- ChatGPT model remains the sole reasoning agent. Requests are synchronous/streamed tool work bounded by the host invocation.
- Tool-level background jobs may outlive a tool call when explicitly created and controllable; they do not contain an external model or reason independently.
- Plans may be resumed from another ChatGPT turn or conversation with explicit session IDs; no guaranteed unattended continuation.
- No external API inference keys, model adapters, direct OpenAI API, browser injection, desktop app replacement, general hosted multi-tenant SaaS, or dependency on local-shell-mcp.

## Cross-cutting concerns

- **Reliability:** restart-safe idempotency, cancellation, timeouts, concurrent invocation limits, bounded output, durable audit.
- **Security:** protected MCP endpoint, OAuth identity + scopes, workspace grants, path containment, subprocess isolation.
- **Portability:** Linux-first; optional Windows and macOS adapters as separately permissioned plugins.
- **Observability:** structured event streams with redaction, correlation IDs and bounded retention.
- **Compatibility:** protocol versions, plugin contract versions, schema migrations, host feature detection and regression tests.

See [MCP contract](specs/MCP_BRIDGE.md), [Plugin contract](specs/PLUGIN_CONTRACT.md), [Security](SECURITY.md), [Testing](TESTING.md), and [Phases](IMPLEMENTATION_PLAN.md).
