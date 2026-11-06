# ChatGPT ↔ Cordis MCP bridge contract (proposed v0)

The bridge is itself a Cordis plugin. It must be **generic**, thin, authenticated and model-agnostic in transport, while the *product* deliberately supports **ChatGPT Chat only**. It has no client for any inference API.

## Single MCP server

Remote Streamable HTTP at `/mcp` over HTTPS; authenticated with OAuth 2.1, protected-resource metadata, PKCE, audience and user scopes. Local development may use a loopback or secured tunnel but must not expose an anonymous write-capable endpoint.

MCP protocol and ChatGPT host integration use the official MCP SDK and MCP Apps specification. Do not implement the wire protocol manually where an audited SDK exists.

## Stable tools

The exact name/schema version is pinned after Phase 0. Proposed **minimum**:

| Tool | Input | Behavior |
| --- | --- | --- |
| `harness_catalog` | optional session ID, query, cursor, limit | Returns currently visible plugin capabilities and catalog epoch |
| `harness_inspect` | capability ID, epoch/generation, optional session ID | Returns full type schema, owner, scope/grants, safety annotations |
| `harness_call` | capability handle, validated JSON arguments, session ID, optional idempotency key | Authorizes and invokes one registered tool |
| `harness_execute` | bounded restricted program, session ID, max calls, output budget | Executes several registered tools in sandbox; invokes same policy pipeline |
| `harness_sessions` | action list/open/report/handoff, explicit IDs | Durable session discovery and handoff; write methods have their own grant |
| `harness_open` | view key or session ID | Opens a UI resource that renders available Cordis view contributions |
| `harness_health` | none | Read-only health/version/capabilities; minimal metadata only |

**No feature-specific MCP methods** (e.g. separate `goal_create`, `git_status`, `shell_run`). Such operations are capabilities listed in `harness_catalog` and dispatched via `harness_call` or bounded `harness_execute`. Each feature plugin remains separately installable. Static bridge schemas let ChatGPT learn one stable tool family.

### Why not map every Cordis tool to MCP tools dynamically?

The host may cache MCP tool discovery and may not pick up tools registered after connection. We will test this, but the default design avoids reliance on dynamic MCP `tools/list_changed` behavior. The server's internal catalog changes even when the MCP surface does not. Dynamic tool descriptions are obtained through `harness_catalog` / `harness_inspect`.

### Example invocation (illustrative)

```json
{
  "session_id": "s_explicit",
  "capability": "hello.ping@1",
  "generation": 2,
  "arguments": {},
  "idempotency_key": "req-opaque"
}
```

Service validates the principal and active scope, current registration generation, schema, consent, execution policy and session ownership. Result includes status, typed content/structured payload, bounded artifacts and correlation ID. Errors are normalized with stable codes, safe text and a retry indication; internal exceptions are not exposed to ChatGPT.

### Partial failures

- `UNAUTHENTICATED` / `PERMISSION_DENIED`: no effect, no private data leak.
- `CAPABILITY_NOT_FOUND` / `CAPABILITY_STALE`: re-inspect instead of rerunning a mutating request.
- `INVALID_INPUT`: schema errors, bounded diagnostic.
- `APPROVAL_REQUIRED`: return explicit pending approval handle without attempting operation.
- `CONFLICT`: worktree/session lock or idempotency conflict.
- `TIMED_OUT` / `CANCELLED` / `BUDGET_EXCEEDED`: terminate/cancel according to plugin contract.
- `INTERNAL`: redacted error; audit event contains internal diagnostic with restricted access.

## ChatGPT UI contract

- MCP App resources use a versioned `ui://` URI and MIME `text/html;profile=mcp-app`.
- Register app resource against `harness_open` using `_meta.ui.resourceUri`.
- UI initializes over `ui/initialize` and uses `tools/call`, `ui/notifications/*` and `ui/message`, checking the host's available capabilities.
- Optional ChatGPT Extensions (sidebar/thread/file editors) use documented metadata and **feature detection**. Do not depend on the extension host being available across all plans/clients.
- UI conveys consent or invokes a policy-gated tool. It is not an authorization authority.
- Components are versioned/cache-busted, CSP-restricted and do not directly fetch local resources from the user's machine.

## State and ChatGPT limitations

ChatGPT owns user and assistant messages, model sampling and tool scheduling. The MCP bridge can only see tool requests/results that ChatGPT sends. It cannot read the entire host transcript, tokens, hidden reasoning, conversation IDs, or automatically call ChatGPT's underlying model. Never infer session ownership from transport identity or thread ID.

Cross-turn persistence is explicit: ChatGPT is prompted to pass the session handle to each invocation; users can reopen sessions manually from UI. Any `ui/message` continuation is host-dependent and must not be labeled guaranteed background automation.

## Delivery & verification

Tests:
1. MCP Inspector protocol and JSON schema smoke.
2. OAuth token validation: missing/invalid issuer/audience/expiry/scope.
3. Dynamic Cordis tool load/unload reflected by catalog epoch, without MCP bridge edit.
4. Approval gate from both ChatGPT and UI.
5. Host manual test: ChatGPT actually runs catalog/inspect/call, opens UI and receives notifications; record client/platform/version.
6. MCP App fallback when UI APIs or Extensions are missing.

Sources: OpenAI documentation in [REFERENCES.md](../REFERENCES.md).
