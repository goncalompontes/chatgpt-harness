# Test strategy and evidence gates

## Layers

1. **Unit:** pure schema, plugin dependency, manifest, service lifecycle, event projection, policy classification.
2. **Integration:** actual Cordis runtime with dynamically loaded plugins, MCP protocol SDK, session store and sandbox engine.
3. **Security negative:** missing scopes, wrong principal, crafted inputs, workspace traversal, injection, sandbox and subprocess cancellation.
4. **Host simulation:** MCP Apps bridge mock with `ui/initialize`, `tools/call`, `ui/message`, multiple host capability combinations.
5. **Real ChatGPT host:** actual installed MCP plugin in ChatGPT, tool execution, UI rendering and click/approval handoff.
6. **Clean install:** new Linux environment, no local-shell-mcp process, no inference API keys, reproducible built-in plugin profile.

## Core lifecycle matrix

| Scenario | Requirement |
| --- | --- |
| Load hello plugin | New capability visible and schema valid |
| Unload hello plugin | Capability unavailable; event listeners and UI contributions removed |
| Plugin activation error | Transactional rollback; no leaked effects |
| Concurrent invoke + unload | Clean cancellation or completion, not wrong-version execution |
| Schema mismatch | Side effect never runs |
| Wrong principal/session | No capability or state leak |
| OAuth wrong audience | Request denied |
| Repeated mutation idempotency key | Side effect occurs at most once |
| Restart session store | Event history replays without re-executing prior tool calls |
| Code-mode nested calls | Same policy and grants; finite budgets |
| Worktree dirty/conflict | Destructive mutation blocked |
| UI unsupported host feature | Graceful fallback, no permissions bypass |

## Required evidence at each milestone

Record: date, source commit, Node/Cordis/MCP SDK versions, exact commands, pass/fail/skip counts, platform, ChatGPT client surface and which manual host behaviors were observed. Never equate mock MCP tests with real UI host support.

CI starts with clean install, typecheck, lint, unit/integration tests, dependency license report and secret scan; add Linux sandbox/security tests when a shell/code-mode implementation exists. Optional Windows/macOS jobs are separate adapters.

## Release rule

No claiming an installable or usable plugin until a user can connect from ChatGPT, discover a registered Cordis capability, invoke it through the real host, observe correct authorization, and unload it without editing the bridge.
