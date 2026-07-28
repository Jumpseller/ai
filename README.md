# Jumpseller AI Toolkit

AI context for building Jumpseller themes and integrations. Install once and your AI assistant gains accurate, platform-specific knowledge of the Jumpseller REST API, Liquid templating system, and MCP server tools.

## What's included

- **`jumpseller-api` skill** — REST API reference: authentication, all resources, pagination, rate limits, and verified code examples.
- **`jumpseller-liquid` skill** — Liquid templating reference: global objects, filters, theme file structure, and the component settings system.
- **`jumpseller-mcp` skill** — MCP server reference: all 24 tools, authentication setup, and common multi-step patterns.
- **`jumpseller-cli` skill** — CLI reference: credential management, store resolution, and local theme development (export, import, watch, apply).
- **Pre-configured `.mcp.json`** — connects directly to `https://mcp.jumpseller.com`.

The REST API, MCP server, and CLI all use the **same credentials** — one Login key + Auth Token pair per store, from **Admin Panel → Account Settings → API Tokens**. The separate [Jumpseller CLI](https://github.com/Jumpseller/jumpseller-cli) (`npm i -g @jumpseller/cli`) is the official tool for syncing themes between your machine and a store.

## Installation

### Claude Code

Claude Code supports installing plugins directly from GitHub repos. Two commands and you're done.

> **Prerequisite:** run these in the **Claude Code CLI** (`claude` in your terminal, or the desktop app). They do not work in the claude.ai web interface.

**Step 1 — register the GitHub repo as a plugin source** (once per machine):

```
/plugin marketplace add Jumpseller/ai
```

**Step 2 — install the plugin from that source:**

```
/plugin install jumpseller-plugin@ai
```

`@ai` is the marketplace alias from step 1 (the repo name), not a version number. After both steps, open a new session and the `jumpseller-api`, `jumpseller-liquid`, `jumpseller-mcp`, and `jumpseller-cli` skills load automatically.

### OpenAI Codex

Install the Codex CLI, then wire up the MCP server (see [MCP server setup](#mcp-server-setup) → OpenAI Codex):

```bash
npm install -g @openai/codex   # or: brew install --cask codex
```

Codex manages plugins through its interactive `/plugins` browser (run `codex`, then type `/plugins`) and registers marketplace sources with `codex plugin marketplace add`. There is no `codex plugin install <github>` shell command. The most reliable way to get the live Jumpseller tools is the MCP config below.

### Gemini CLI

```bash
gemini extensions install https://github.com/Jumpseller/ai
```

You'll be asked to trust the workspace and confirm the third-party extension. Then run `gemini` and the skills load automatically. Verify with `gemini extensions list`.

## MCP server setup

The MCP server lives at `https://mcp.jumpseller.com`. It authenticates with your store credentials sent as the `X-LOGIN-KEY` / `X-AUTH-TOKEN` headers, which every client reads from environment variables. **Export these before launching your client** — if the client starts without them, the MCP server can't authenticate and its tools never load (the assistant will silently fall back to the REST API or shell instead):

```bash
export JUMPSELLER_LOGIN_KEY=your-login-key
export JUMPSELLER_AUTH_TOKEN=your-auth-token
```

Retrieve these from **Admin Panel → Account Settings → API Tokens**.

> ⚠️ **These credentials are full-access — keep them server-side.** Never embed the Login key / Auth Token in theme code, HTML, client-side JavaScript, or any public storefront output. A leaked key gives anyone full read/write over the store. For storefront features, render data with server-side Liquid objects instead of client-side API calls.

### Claude Code

The toolkit ships a pre-configured `.mcp.json` pointing at the server. Claude Code reads it automatically — approve the server when prompted, then the MCP tools are available.

### Gemini CLI

Gemini does **not** read `.mcp.json` (that file is Claude-only). The MCP server is declared in the extension's `gemini-extension.json`, so installing the extension wires it up. Steps:

1. `export` the two env vars **before** starting Gemini (above).
2. `gemini` → `/extensions install https://github.com/Jumpseller/ai`
3. Verify the server connected with `/mcp` — you should see `jumpseller` and tools like `list_orders`. No tools listed usually means the env vars weren't set when Gemini launched.

> ⚠️ **Known Gemini CLI limitation:** the list/search tools (`list_orders`, `list_products`, `list_categories`, `list_customers`, `search_*`) currently fail in Gemini CLI with `Invalid input: expected record, received array (path: structuredContent)`. This is a Gemini CLI client-side validation bug ([gemini-cli#15375](https://github.com/google-gemini/gemini-cli/issues/15375)), **not** a server issue — the same tools work in Claude clients. Until it's fixed upstream, use the REST API for listing/searching; the single-resource tools (`get_product`, `get_order`, …) and all writes are unaffected.

### OpenAI Codex

Codex reads MCP servers from `~/.codex/config.toml` (not `.mcp.json`). Add:

```toml
[mcp_servers.jumpseller]
url = "https://mcp.jumpseller.com"
env_http_headers = { "X-LOGIN-KEY" = "JUMPSELLER_LOGIN_KEY", "X-AUTH-TOKEN" = "JUMPSELLER_AUTH_TOKEN" }
```

`env_http_headers` maps a header name to an environment variable name; Codex injects the value at runtime. With `JUMPSELLER_LOGIN_KEY` / `JUMPSELLER_AUTH_TOKEN` exported, run `codex` and ask it to use the Jumpseller tools. You can also configure this interactively with `codex mcp add`.

For third-party app developers, use OAuth 2.0 instead. See the [MCP server documentation](https://jumpseller.com/support/mcp-server/) for OAuth setup.

## REST API authentication

The Jumpseller REST API supports two authentication methods. **Basic Auth is recommended:**

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  https://api.jumpseller.com/v1/products.json
```

Query parameters also work but are deprecated:

```bash
curl "https://api.jumpseller.com/v1/products.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN"
```

## Example prompts

Once installed, try asking your AI assistant:

- *"How do I authenticate with the Jumpseller REST API?"*
- *"List all products with stock below 5."*
- *"Create a product card component in Liquid that shows a sale badge."*
- *"How do I paginate through all paid orders?"*
- *"Use the MCP tools to find the last 10 orders and add tracking numbers."*
- *"What's the correct URL for the store info endpoint?"*

## API reference

The skill content is verified against the official Jumpseller OpenAPI specification at `https://api.jumpseller.com/swagger.json`. If you find a discrepancy between this toolkit and the spec, please open an issue.

## Contributing

Found an error or missing information? Open an issue — we review all suggestions. Pull requests are not accepted at this time.

## License

MIT
