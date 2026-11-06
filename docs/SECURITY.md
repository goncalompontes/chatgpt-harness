# Security model

Status: design requirements. No deployment or authorization implementation exists in the planning baseline.

## Assets and threats

Assets: local source trees, files, Git history and worktrees, shell/process permissions, remote servers, tokens, session logs, user approvals, custom plugin code and uploaded content.

Threats: unauthenticated public endpoint, OAuth token replay, confused deputy from ChatGPT or UI principal mismatch, untrusted model-generated arguments, malicious files/instructions, plugin supply-chain compromise, sandbox escape, shell injection, traversal/symlink escape, leaked stdout/secrets, stale permission grants, races between sessions and unauthorized Git mutation.

## Trust boundaries

1. ChatGPT host → public HTTPS MCP endpoint (untrusted until bearer token validated).
2. MCP adapter → Cordis tool pipeline (authenticated principal, session ownership, workspace claims).
3. Tool pipeline → filesystem/process/git (canonicalized grants, OS sandbox and actual-operation enforcement).
4. Plugin packages → Cordis runtime (only approved trusted code; lifecycle effects and dependency audits).
5. Browser iframe → MCP host bridge (`ui/*` messages are not sufficient authorization).

No plugin may directly use a higher privilege merely because it runs in the same Node process. For third-party plugins this requires stronger process isolation, so **third-party execution remains disabled until a concrete isolation design passes tests**.

## OAuth and transport

Implement current MCP OAuth2.1 authorization (prefer managed IdP). Verify signed token on *every* tool invocation: issuer, audience/resource, expiration/not-before and scopes. Host `/.well-known/oauth-protected-resource` and authorization metadata, PKCE S256, support CIMD or DCR as selected. HTTPS only, except narrowly scoped local development. Protect introspection and open resources according to risk; no unprotected shell, file, session or Git tools.

Never log bearer tokens, auth codes, secrets or full request headers. Give UI minimal scoped data. Rate limit and bind body size, execution time and concurrency.

## Authorization model

Scopes: coarse `harness:read` / `harness:invoke` / `harness:admin`. Grants: per-session/per-workspace capability rights (`files:read`, `files:write`, `shell:execute`, `git:read`, `git:write`, `worktrees:create`, `jobs:manage`, `plugins:install`). Tool schemas declare the minimum. Owner principal and resource claims are checked at dispatch and again immediately before the side effect.

Tool and user requests are **not** implicit authorization for destructive operations. Present the exact cwd, command, path and Git operation to approve; scope grants narrowly, expire them, and reject stale approvals. A user's prior approval for one action does not grant arbitrary future commands.

## File/process controls

- Canonicalize granted roots, reject `..` and symlink escapes; revalidate around file opens to reduce TOCTOU.
- Paths outside all granted roots denied; no reads of private browser/session keys by default.
- Spawn argv via OS process API, not string shell concatenation. If `bash -c` is ever allowed, treat as high-risk and separate approval.
- Enforce OS-level containment where supported; remove network access from sandboxed Code Mode by default. Node `vm` is insufficient for untrusted JS.
- Kill process groups and descendants on cancellation; capture bounded stdout/stderr and redact secrets.
- No escalation through plugin loader or Code Mode: each nested call passes original principal and grants.

## Git and worktree safety

Read status/branches before mutations; preserve user dirt. Require approval for worktree addition, branch switching, commit, merge, rebase, push, force, reset or deletion. Verify base/branch and worktree ownership; avoid concurrent edits by unrelated sessions; never use `--force` as an error workaround.

## Plugin installation

Phase 1 only ships built-in plugins. Future installation requires allowlisted registries/sources, pinned integrity hashes, license and dependency review, requested-permission display, signature checks if available, time-bounded operation and reversible deployment. Do not execute untrusted published bundles inside the host Node process. Uninstall does not erase session/audit records.

## Sessions and logging

Separate authenticated users by principal. Session list requires ownership or delegated permission; events include actor, request/correlation ID, outcome, required grant and redacted operation metadata. Retention/export/delete are configurable; no automatic upload of local private data. Never commit runtime logs, stores or auth secrets.

## UI

Use strict Content Security Policy for MCP Apps. No untrusted scripts/frames or `postMessage` without origin checks and validated message schemas. The host UI bridge is not authority to execute. Use accessible approval dialogs that show the full requested side effect; no misleading 'success' before server confirmation.

## Attack-oriented acceptance tests

- Unknown/expired/wrong-audience token, wrong scope, wrong session principal and forged UI request are denied.
- Path traversal, symlinks, cwd escape and hardlinks where relevant cannot access another workspace.
- Failed plugin activation unwinds registrations; stale capabilities cannot be invoked.
- An installed plugin cannot give itself permissions or bypass audited dispatch.
- Sandbox cannot access JS host internals, network, FS or process primitives.
- Shell timeouts cancel descendants; concurrent worktree owner claims serialize.
- Sensitive stdout is redacted and audit streams are principal-scoped.

Security gating is **required before exposing write or shell capabilities to ChatGPT**. Never make a public endpoint writable merely to speed host testing.
