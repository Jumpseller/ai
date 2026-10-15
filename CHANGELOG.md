# Changelog

## [Unreleased]

### Changed
- Codex signs in with OAuth (`codex mcp login jumpseller`); API token headers stay documented for CI.
- `jumpseller-mcp` skill lists all 27 tools: adds `search_support_docs`, `list_admin_routes` and `recommend_admin_route`.

### Added
- README: MCP setup for Grok CLI and Muse Code, both with API token headers.

## [1.1.0] - 2026-10-13

### Changed
- The MCP server connects with OAuth: Claude clients sign in with the user's Jumpseller account instead of reading `JUMPSELLER_LOGIN_KEY` / `JUMPSELLER_AUTH_TOKEN`. API token headers are still documented for CI and scripts, and the Gemini extension keeps using them.
- `jumpseller-mcp` skill: OAuth first, real scope names.
- Plugin description covers store management, not only development.

## [1.0.0] - 2026-05-29

### Added
- `jumpseller-api` skill: REST API reference covering authentication, all key resources, pagination, error handling, and filtering.
- `jumpseller-liquid` skill: Liquid templating reference covering global objects, theme file structure, component system, and common patterns.
- `jumpseller-mcp` skill: MCP server tools reference covering all 24 tools, authentication methods, and usage guidance.
- Claude Code plugin configuration.
- OpenAI Codex plugin configuration.
- Gemini CLI extension configuration.
- Pre-configured `.mcp.json` pointing to `https://mcp.jumpseller.com`.
