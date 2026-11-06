# ADR-0002 — Independent Cordis application and stable MCP bridge

Status: **accepted in principle; dependency version pending Phase 0** · 2026-11-06

## Context

We want the plugin model of DeepSeek Harness, including dependencies, reversible effects and extensible tools, without inheriting its desktop client, bundled model adapters, backend agent loop or monorepo.

## Decision

- Create an independent greenfield TypeScript/Node Cordis application.
- Take upstream Cordis as a dependency after version, package and license verification; do not copy all of DeepSeek Harness.
- Implement our tool registry, session/event service, authorization pipeline and MCP bridge as Cordis plugins.
- Expose a small stable MCP tool family (`harness_catalog` / `harness_inspect` / `harness_call` / `harness_execute` / `harness_open` / `harness_sessions`) to avoid needing to rebuild a bridge for each new plugin.
- Use official OpenAI MCP Apps and Plugin Extensions contracts for ChatGPT UI. Feature-detect optional surfaces.
- Remain completely independent of `local-shell-mcp`, although prior knowledge may inform compatibility tests.

## Consequences

Cordis plugin authors get an application-level typed SDK with interoperable service contracts. One installed ChatGPT plugin can access any approved tool plugin. The core tool service is purposely a shared abstraction: plugins do not import the MCP server.

Unknowns: exact Cordis version/package scope, whether host supports all desired UI entrypoints, how to expose secure HTTPS locally, and exact permission UX. These must be empirically validated before implementation choices are frozen.
