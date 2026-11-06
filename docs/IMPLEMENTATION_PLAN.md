# Implementation plan: ChatGPT-only Cordis harness

Status: planning baseline, 2026-11-06. Implement phases in order, with tests and explicit acceptance gates. Completing a phase does not imply features of later phases work.

## Target milestones

- **M0 — Architecture proven locally:** a Cordis Context mounts/unmounts a dynamically discovered sample plugin and all effects unwind.
- **M1 — ChatGPT can discover and invoke plugins:** a real ChatGPT connection exercises a stable MCP bridge without per-plugin MCP registration.
- **M2 — Secure development workspace:** session persistence, approved operations, files/shell/Git/worktree plugins and bounded Code Mode are fully testable.
- **M3 — Native ChatGPT agentic UX:** interactive MCP App with contributed panes, durable timelines, goals and workflows, plugin manager and verified handoff.

## Phase 0: feasibility / upstream contract spike

**Goal:** remove assumptions before committing to runtime packages and API surface.

Work:
- [ ] Inspect current Cordis core, loader, lifecycle and effect API and pin a compatible distribution/version. Choose between upstream `cordis` and the DeepSeek-published build; do not mix scopes.
- [ ] Verify Cordis license, dependency graph, Node version, ESM/TypeScript support, install/unload semantics.
- [ ] Prototype minimal `hello-plugin`: service injection, a dynamically registered tool and an event subscriber, with a clean teardown after unload.
- [ ] Verify current MCP SDK, MCP Apps SDK, OpenAI Plugin Extensions and OAuth2.1 schemas; pin versions.
- [ ] Test whether ChatGPT supports the planned stable tool surface, tool output structuring and native UI host callback methods.
- [ ] Record gaps as an ADR instead of simulating unsupported host features.

Acceptance: repeatable unit test asserting dynamic tool appears/disappears, no leaked effects, and a protocol-level MCP smoke test. A mock does **not** certify ChatGPT host compatibility.

## Phase 1: monorepo skeleton + runtime

Create:
- `apps/server` executable entrypoint and config.
- `packages/core` typed Cordis Context, health, startup/shutdown, plugin discovery.
- `packages/sdk` shared manifest, event, capability, schema and test helper types.
- `examples/hello-plugin` end-to-end executable demonstration.
- Basic `pnpm` workspace, TypeScript strict, lint/format, unit test and CI checks; package names/version policy.

Work:
- [ ] Validate plugins and dependencies before activation; reject duplicate IDs and invalid schema.
- [ ] Handle plugin startup failure transactionally; dispose previously registered effects.
- [ ] Offer deterministic dry-run config resolution and read-only plugin-tree inspection.
- [ ] Clean shutdown drains or cancels bounded work; no zombie processes.

Acceptance: isolated plugin lifecycle unit/integration tests, dependency failure and reload tests, hermetic build on Linux, no ambient credentials.

## Phase 2: generic registry and MCP bridge

Implement plugins:
- `plugin-tools`: namespaced tool registry, schema validation, annotations, scoped execution middleware.
- `plugin-mcp-bridge`: Streamable HTTP MCP server, static generic tool schemas, errors, output size limits.
- `plugin-approvals` + `plugin-auth`: bearer token verification, scopes, operation classification.
- `plugin-chatgpt-ui`: minimal MCP Apps resource with discovery viewer, capability-detected host communication.

Work:
- [ ] Implement `harness_catalog` with paging/filter/version and strong cache invalidation on plugin unload.
- [ ] Implement `harness_inspect` with JSON Schema and immutable capability handle.
- [ ] Implement `harness_call` validating principal/session/input/grants/version at invocation.
- [ ] Implement `harness_execute` only after secure sandbox design; initially return a documented unavailable error.
- [ ] Implement `harness_open` serving immutable versioned UI resource.
- [ ] Document exact OAuth2.1 and HTTPS transport setup, discovery metadata, PKCE, audience and scope verification.
- [ ] Test install/unload sample plugin without MCP bridge edits or restart (where supported by Cordis).
- [ ] Connect the independent MCP server to ChatGPT; observe real tool calls and result rendering.

Acceptance: real ChatGPT calls `harness_catalog`, `harness_inspect` and a read-only hello tool; uninstall removes it; unauthenticated/denied calls fail closed. No keys, shell, files or unsafe dynamic plugins exposed.

## Phase 3: durable session/event service

Implement `plugin-sessions` (initially SQLite or JSONL after ADR) with owner principal, session ID, workspace scope, monotonic event IDs and projections.

Work:
- [ ] Append durable created/requested/approved/denied/started/completed/failed/cancelled events.
- [ ] Restore sessions after restart without replaying side effects.
- [ ] Request idempotency keys for mutations and persist operation outcome.
- [ ] Provide session list/open/report/handoff, explicit context handover and scoped paging.
- [ ] Expose SSE/host-compatible notifications only when tested; polling fallback.
- [ ] Set retention, redaction, export and deletion policies.

Acceptance: restart, concurrent writes, replay, corrupted-tail and principal-separation tests. Cross-chat adoption works from explicit handle; no guessed ChatGPT thread identity.

## Phase 4: secure filesystem and execution plugins

Implement:
- `plugin-files`: canonical workspace roots, read/list/search, safe writes, diffs, controlled patches.
- `plugin-shell`: bounded subprocess execution, cwd allowlist, output and timeout limits.
- `plugin-jobs`: tracked long-running jobs with stop/recover.
- `plugin-git`: read-only Git status/history/diff plus separately approved writes.
- `plugin-worktrees`: branch/worktree discovery, isolation, guarded create/remove.

Work:
- [ ] Define operation capability grants separate from OAuth scopes.
- [ ] Implement explicit approvals and audit on writes, shell and Git mutations.
- [ ] Prevent traversal, symlink escapes, TOCTOU, cwd escape, shell argument injection and stream leaks.
- [ ] Enforce one worktree owner per active editing session; coordinate process and resource locks.
- [ ] Preserve existing user work; no destructive Git operations by default.
- [ ] Prove cancellation, retry idempotence, process exit cleanup and session principal isolation.

Acceptance: targeted negative security suite plus real Linux sandbox tests; no direct bypass via UI or dynamic tool invocation.

## Phase 5: bounded Code Mode

Implement `plugin-code-mode` as a sandboxed JavaScript-like interpreter with read-only capability discovery and explicit budget. Do not use unrestricted Node `vm` as a security boundary.

Work:
- [ ] Define grammar/interpreter package, allowed built-ins, no network/process/FS/ambient JS.
- [ ] Resolve tools solely from the current authorized registry snapshot, re-authorize each call.
- [ ] Enforce instruction/time/tool call/output/memory/concurrency budget; cancellation cascade.
- [ ] Reject nested execution privilege escalation and dynamic import or eval escape.
- [ ] Keep event log and per-call correlation IDs; partial tool failures are visible.

Acceptance: escape attempts denied; resource budgets enforced; three plugins callable from one Code Mode invocation; effects audited under caller identity.

## Phase 6: goals, workflows and safe delegation

Implement:
- `plugin-goals`: reusable goals, plan steps, checkpoints, evidence, approval gates, blocked/resumed/finished status.
- `plugin-orchestration`: explicit DAG of bounded tool steps and handoff records, not independent model reasoning.
- Optional adapters to GitHub/REA or other tools only as separately installed Cordis plugins later.

Work:
- [ ] A goal has a durable state machine and clear acceptance evidence.
- [ ] Support serialized owner claims, conflict detection and scoped worktrees.
- [ ] Separate tool job scheduling from host model turn generation.
- [ ] Create prompts/handovers requesting ChatGPT follow-up only via supported host UI and only with user action.
- [ ] Define what 'subagent' means without external model APIs: parallel **tool work**, not a separately inferred LLM.
- [ ] Expose lifecycle controls to ChatGPT and UI with same permission pipeline.

Acceptance: model-driven multi-turn workflow advances in ChatGPT with explicit resume after conversation switch; state survives restart and contention; no claims of unattended ChatGPT inference.

## Phase 7: plugin-composable ChatGPT UI

Implement `plugin-ui-registry` and `plugin-chatgpt-ui`:
- [ ] Core panels: sessions, events, tool calls, approvals and plugin inventory.
- [ ] Contributed panels: diffs, terminal/jobs, worktrees, goals, workflow DAG.
- [ ] MCP Apps lifecycle, `ui/initialize`, `ui/notifications/*`, `tools/call`, `ui/message`.
- [ ] Feature-detect ChatGPT conversation panel, global sidebar, fullscreen, file viewers and editor entrypoints.
- [ ] Versioned `ui://` resources, strict CSP, declarative view schema and safe cross-plugin composition.
- [ ] Accessibility, keyboard/mouse controls, theme changes, reduced motion, large session pagination.
- [ ] UI rehydration from durable backend, not browser local storage as authority.

Acceptance: tests with mocked host; real ChatGPT manual UI matrix for open, tool-call, approval, callback, reconnect, theme and host feature support. Unsupported extensions degrade to inline MCP App.

## Phase 8: distribution / reliability / developer experience

- [ ] One-command local install and run, secure HTTPS exposure procedure; no credential commits.
- [ ] Plugin templates, typed SDK, development-time reload, package manager, manifest inspection, permissions prompt.
- [ ] Linux support first; isolate any Windows/macOS implementation behind plugins.
- [ ] Open-source licensing decision and third-party notices before vendoring/copying upstream code.
- [ ] CI: build, unit/integration, security fuzz/sandbox, smoke, and manual host validation checklist.
- [ ] Migration policy and semver versioning; backups and recovery of session store.
- [ ] Docs for install, connect to ChatGPT, first plugin, permissions and troubleshooting.

Acceptance: independent clean install reproduces M3 workflow; users can add a custom Cordis plugin without editing/rebuilding MCP bridge; all security and lifecycle gates pass.

## MVP walkthrough (end-to-end acceptance)

1. User installs/links one ChatGPT plugin to the independent Harness MCP server.
2. ChatGPT discovers plugins using `harness_catalog` and inspects a registered tool.
3. A new Cordis plugin is loaded; ChatGPT invokes its capability without changing bridge source.
4. User approves a workspace-scoped operation; denial paths work.
5. A session persists events across restart and can be resumed via explicit ID.
6. ChatGPT uses files, shell, Git, worktree and goal plugins through the same bridge.
7. User opens the MCP App/panel to inspect tool history, edits and progress.
8. No remote model API is configured, called, required or represented in the dependency graph.

## Parallel work / ownership

After M1, workstreams can proceed in separate Git worktrees:
- Runtime and SDK owner: `core`, plugin contracts, schema.
- MCP/ChatGPT owner: bridge, OAuth, host probes.
- Security/execution owner: approvals, sandbox, shell, code-mode.
- State owner: sessions, event projections, goals.
- UI owner: UI registry and renderer.
- Integrator alone: shared workspace config, package boundaries, release integration.

Integration follows: source changes → typed tests → negative permission tests → ChatGPT host manual verification → documented evidence → PR review → merge.

## Work not authorized by this initial planning commit

No source packages, server process, plugin installations, third-party downloads, tunnel exposure, OAuth credentials, file/shell execution endpoint, or background agent are implemented in the initial documentation-only baseline.
