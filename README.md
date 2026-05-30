# Jumpseller AI Toolkit

AI context for building Jumpseller themes and integrations. Install once and your AI assistant gains accurate, platform-specific knowledge of the Jumpseller REST API, Liquid templating system, and MCP server tools.

## What's included

- **`jumpseller-api` skill** — REST API reference: authentication, all resources, pagination, error handling, and code examples.
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

```
codex plugin install github:Jumpseller/jumpseller-ai-toolkit
```

### Gemini CLI

```
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

## Example prompts

Once installed, try asking your AI assistant:

- *"List all products with stock below 5 using the Jumpseller API."*
- *"Create a product card component in Liquid that shows price and a sold-out badge."*
- *"How do I paginate through all orders with status Paid?"*
- *"Use the MCP tools to find the last 10 orders and mark them as Shipped."*

## Contributing

Found an error or missing information? Open an issue — we review all suggestions. Pull requests are not accepted at this time.

## License

MIT
