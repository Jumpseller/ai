# Jumpseller AI Toolkit

AI context for building Jumpseller themes and integrations. Install once and your AI assistant gains accurate, platform-specific knowledge of the Jumpseller REST API, Liquid templating system, and MCP server tools.

## What's included

- **`jumpseller-api` skill** — REST API reference: authentication, all resources, pagination, rate limits, and verified code examples.
- **`jumpseller-liquid` skill** — Liquid templating reference: global objects, filters, theme file structure, and the component settings system.
- **`jumpseller-mcp` skill** — MCP server reference: all 20 tools, authentication setup, and common multi-step patterns.
- **Pre-configured `.mcp.json`** — connects directly to `https://mcp.jumpseller.com`.

## Installation

### Claude Code

```
/plugin marketplace add Jumpseller/jumpseller-ai-toolkit
/plugin install jumpseller-plugin@jumpseller-ai-toolkit
```

### OpenAI Codex

Search for `jumpseller-ai-toolkit` in the Codex plugin interface, or run:

```bash
codex plugin install github:Jumpseller/jumpseller-ai-toolkit
```

### Gemini CLI

```bash
gemini extension install github:Jumpseller/jumpseller-ai-toolkit
```

## MCP server setup

The toolkit includes a pre-configured `.mcp.json` pointing to `https://mcp.jumpseller.com`. To activate it, set your store credentials as environment variables:

```bash
export JUMPSELLER_LOGIN_KEY=your-login-key
export JUMPSELLER_AUTH_TOKEN=your-auth-token
```

Retrieve these from **Admin Panel → Account Settings → API Tokens**.

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
