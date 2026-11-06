# Implementation backlog

Derived from `IMPLEMENTATION_PLAN.md`. This is the **source-of-truth issue seed list**, not a claim that GitHub issues or code already exist.

| ID | Priority | Dependency | Deliverable / acceptance |
| --- | --- | --- | --- |
| H-001 | P0 | none | Pin Cordis package/version; integration proof of service injection, events and unload |
| H-002 | P0 | H-001 | Set up TS/ESM/pnpm monorepo, strict typecheck, tests and CI |
| H-003 | P0 | H-001 | Runtime config, plugin DAG, atomic lifecycle and error recovery |
| H-004 | P0 | H-003 | Tool registry, versioned schemas, dynamic catalog and guarded handler dispatch |
| H-005 | P0 | H-004 | Stable remote MCP server with discovery/inspect/call and protocol smoke |
| H-006 | P0 | H-005 | OAuth2.1, principal scopes, protected metadata and denial tests |
| H-007 | P0 | H-005,H-006 | Real ChatGPT end-to-end catalog/inspect/hello invocation |
| H-008 | P1 | H-003 | Session event store, replay, IDs, owner isolation and handoffs |
| H-009 | P1 | H-004,H-006,H-008 | Approval, workspace grants and auditable operation events |
| H-010 | P1 | H-009 | Scoped file read/edit/diff plugin and path escape negative tests |
| H-011 | P1 | H-009 | Shell/job plugin with timeouts, process cleanup and isolation |
| H-012 | P1 | H-009,H-010 | Git status/diff/mutations plugin with exact approval |
| H-013 | P1 | H-012,H-008 | Worktree lifecycle and active-session ownership plugin |
| H-014 | P1 | H-004,H-009 | Restricted Code Mode interpreter, budgets and sandbox tests |
| H-015 | P1 | H-008,H-009 | Goal and plan plugin, blocked/resume/finish with evidence |
| H-016 | P2 | H-014,H-015 | Deterministic tool DAG orchestration with no external inference |
| H-017 | P1 | H-005,H-008 | MCP Apps resource, UI bridge and host compatibility smoke |
| H-018 | P2 | H-017 | Contributed declarative view registry, session timeline and panes |
| H-019 | P2 | H-003,H-009 | Trusted plugin manager, manifest lifecycle, version policies |
| H-020 | P2 | H-007,H-018 | ChatGPT thread/sidebar feature tests, UI fallback and accessibility |
| H-021 | P2 | all | Clean install, security audit, developer docs, first plugin tutorial |

## Suggested first PRs

- PR A (H-001/002): `hello-plugin` dynamic Cordis lifecycle and workspace CI.
- PR B (H-003/004): plugin loader, tool registry and conformance harness.
- PR C (H-005/006/007): minimal authorized MCP bridge + actual ChatGPT proof.
- PR D (H-008/009): session/audit/approval foundation.

Each PR should contain implementation, negative tests and evidence. No claims of host functionality based only on mocks.
