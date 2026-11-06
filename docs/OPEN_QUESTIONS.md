# Open questions / decision gates

These are deliberately unresolved; no implementation should silently turn them into assumptions.

1. **Cordis distribution:** use upstream `cordis` + loader or DeepSeek-published `@deepseek-ai/cordis`? Confirm current API/ESM/lifecycle compatibility, package versions, notices, and release maintenance. Gate: Phase 0 tests.
2. **Local deployment:** choose a stable HTTPS endpoint for a user-owned local daemon (reverse tunnel, domain, auth service), balancing portability and protected device access. Gate: OAuth2.1 demo with no public anonymous write operations.
3. **ChatGPT availability:** which Plugin Extensions entrypoints (thread panel, sidebar, composer, file view) are enabled for the user's ChatGPT client? Gate: in-host proof with capability detection.
4. **Core persistence:** SQLite vs JSONL; adopt append-only facts, migrations, durability, replay and concurrency evidence before selecting backend. Gate: Phase 3 test matrix.
5. **Code Mode interpreter:** select a safe, maintainable interpreter with strict budgets; **do not** treat Node VM as a sandbox. Gate: independent threat test review.
6. **Dynamic UI plugin contributions:** start with declarative schema only; decide whether custom per-plugin JS bundles are ever allowed and how to constrain privileges. Gate: CSP and trusted package loader design.
7. **Public repository license:** choose a license with maintainer approval; public repository currently has no license. Record third-party attributions before source reuse.
8. **Remote platform support:** Linux-first, decide whether macOS/Windows backends are worthwhile after local MVP.
9. **OAuth provider:** select managed IdP or hardened existing provider. Do not build a production authorization server from scratch without strong justification.
10. **User consent UX:** decide how Cordis tool approvals map to ChatGPT's own confirmation UI without duplicate approval or bypass. Gate: real-host integration tests.

## Decision process

Use numbered ADRs in `docs/adr/`. Add measured observations, provider/protocol versions, alternatives, security impact, and tests that informed each acceptance. Avoid feature implementation on an unproven assumption.
