# Cordis plugin contract (proposed v0)

All product features are Cordis plugins, including the ChatGPT bridge itself. This document defines a design contract, not a validated implementation API.

## Plugin package shape

A plugin exports a stable ID, semver version, required service dependencies, granted capability requests, configuration schema and a Cordis `apply(ctx, config)` entry. Prefer native Cordis `inject`/`Service` conventions and reversible effects. Do not invent Cordis-specific method signatures: validate against the pinned upstream version in Phase 0.

Conceptual package manifest:

```json
{
  "id": "dev.chatgpt-harness.worktrees",
  "version": "0.1.0",
  "entry": "./dist/index.js",
  "requires": ["tools@^1", "git@^1", "sessions@^1", "approvals@^1"],
  "permissions": ["git:read", "git:mutate", "workspace:manage"],
  "configSchema": "schemas/config.json",
  "contributions": ["tools", "events", "ui"]
}
```

This is a **proposed shape**. Standard Cordis loader manifests may differ; our wrapper adds only what is missing.

## Contribution categories

- **Services:** declared keys with public versioned interface and dependency injection; no direct concrete imports across plugins unless explicitly shared SDK.
- **Tools:** namespace/name/version, JSON Schema input/output, metadata, required grant, mutability, required workspace and cancellation policy. Register through the guarded `tools` service.
- **Events:** typed namespaced event map; ephemeral plugin lifecycle events vs durable session facts explicitly differentiated.
- **UI:** typed view descriptors (e.g. session-detail pane, tool-log renderer, diff renderer) with declarative props and host capability declarations; custom code loading requires explicit signing/CSP/trust review.
- **Commands:** explicit user-facing non-model operations; never bypass the authorization gate.
- **Settings:** typed, versioned and validated; store secrets in a protected secret store, not manifests.

## Tool definition

Each registered tool must have:
- `id` — immutable namespace/tool@major identifier.
- `ownerPlugin` — plugin ID + installed version.
- `description` / `annotations` — truthful readOnly/mutation/destructive/idempotence classification.
- `inputSchema` / `outputSchema` — JSON Schema with bounded input size and rejected unknown fields when applicable.
- `requiredScopes` — OAuth-level access.
- `requiredGrants` — per-session/per-workspace grants.
- `timeout` and `budget` — enforced limits.
- `handler` — receives authorized invocation context with immutable principal, session, workspace, abort signal, logging and trace ID.

Never allow the plugin to declare a permissive annotation and thereby skip policy; the policy engine may strengthen classifications.

## Lifecycle invariants

1. Validate package origin, manifest, version and dependencies.
2. Resolve services and configuration before activation.
3. Begin a transactional effect scope.
4. Register all contributions atomically or unwind completely.
5. On unload, revoke handles, stop accepting new calls, await/cancel in-flight calls, dispose effects, and remove UI contributions/events.
6. In-flight requests must finish with a stable result or explicit cancellation; not execute against a new plugin version inadvertently.
7. Reload makes a new immutable capability generation; stale handles fail with a typed `CAPABILITY_STALE` response.

Dependencies form a DAG; detect cycles and downgrade optional dependencies by explicit contract, not silent fallback.

## Discovery semantics

Catalog changes increment a version/epoch. The MCP bridge must not predeclare new per-plugin tools; it returns dynamic descriptors. A client obtains a concrete capability handle from inspect, invokes via generic `harness_call` with version. The server **re-evaluates** grants at invocation, even when the catalog said the tool was available.

Discovery never exposes tools from a different principal, session, workspace or installation trust level.

## Plugin safety levels

- **Built-in trusted:** shipped, tested, signed/pinned with harness distribution. Still subject to runtime grants.
- **Locally authored:** explicit approval to activate, sandbox and capability scope; source reviewed.
- **Downloaded third-party:** disabled until installation policy, dependency audit, signature/provenance and isolation have been implemented. Never arbitrarily npm-install from model output.

## Example expected lifecycle (illustrative pseudocode)

```ts
export const name = "hello";
export const inject = ["tools", "sessions"];

export function apply(ctx) {
  // Adapt to the exact pinned Cordis tool registry API.
  const dispose = ctx.tools.register({
    id: "hello.ping@1",
    inputSchema: { type: "object", properties: {} },
    async handler(invocation) {
      return { pong: true, sessionId: invocation.sessionId };
    }
  });
  ctx.effect(() => dispose());
}
```

The `ctx.tools` interface above is **ours to implement**, not an upstream Cordis API promise.

## Contract tests

Every plugin must pass:
- load → discover → validate input → execute → unload → unavailable;
- missing dependency, invalid manifest, config update and activation rollback;
- stale capability handle, exception, cancellation and duplicate ID;
- cross-principal access denial, missing grant denial and event redaction;
- unload with running task, restart recovery where applicable;
- hostless MCP behavior (UI optional) and contribution compatibility.

The SDK should generate a reusable conformance suite from this contract.
