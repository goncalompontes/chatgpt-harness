# Upstream references and verified boundaries

Checked: 2026-11-06. These resources are **design references**, not vendored dependencies.

## Cordis and DeepSeek Harness

- DeepSeek Harness upstream: https://github.com/deepseek-ai/deepseek-harness
- Architecture and services: https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md
- Cordis primer: https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/cordis-primer.md
- Cordis vendor version/provenance: https://github.com/deepseek-ai/deepseek-harness/blob/master/vendor/README.md
- DeepSeek Harness MIT license: https://github.com/deepseek-ai/deepseek-harness/blob/master/LICENSE
- Cordis framework upstream: https://github.com/cordiverse/cordis

Upstream architecture establishes typed events, injectable services, reversible effects and dynamic plugin trees. The DeepSeek harness also implements LLM adapters and independent agent loops; **we do not include either**. We will implement only the Cordis runtime plus ChatGPT-facing capabilities.

At implementation time, confirm an exact Cordis package/version/commit and license, ideally using upstream distribution packages rather than copying the DeepSeek monorepo. If vendoring ever happens, copy and preserve upstream license and notices.

## ChatGPT and MCP

- ChatGPT plugin quickstart: https://developers.openai.com/plugins/build/app-quickstart
- MCP Apps UI resource and bridge: https://developers.openai.com/plugins/build/chatgpt-ui
- Plugin Extensions (sidebar and conversation panel): https://developers.openai.com/plugins/build/extensions
- OAuth resource server: https://developers.openai.com/plugins/build/auth
- MCP plugin packaging: https://developers.openai.com/plugins/build/plugins
- MCP server / client integration: https://developers.openai.com/api/docs/mcp

Current documentation recommends MCP Apps `_meta.ui.resourceUri`, `ui/initialize` and `tools/call` with optional ChatGPT `window.openai` extensions, plus OAuth2.1 PKCE for authenticated remote tools. Feature support may vary between ChatGPT clients and plans; test real connection rather than assuming availability.

## Explicit exclusions

- No dependency on `local-shell-mcp` or import from its source.
- No external model providers, model inference API calls or model adapter.
- No claim of invoking ChatGPT's hidden internal reasoning loop or model from an MCP server.
- No guarantee of unattended conversation continuation via `ui/message`.

## Attribution policy

This is an independent greenfield project inspired by general patterns in the upstream references. Any third-party source added later must have provenance, compatible licensing, notice retention and exact commit/version evidence. The repository license is undecided until the maintainer selects it.
