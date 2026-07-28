# Jumpseller MCP & Gemini CLI Known Issues

## MCP Validation Bug (Issue #15375)
Gemini CLI fails to validate MCP responses that return a JSON Array in the `structuredContent` field.
- **Error:** `Invalid input: expected record, received array (path: structuredContent)`
- **Affected Tools:** `list_products`, `list_orders`, `list_categories`, `list_customers`, and all `search_*` tools.
- **Status:** Client-side bug in Gemini CLI; server is functional.

## Workflow Mandates
1. **List/Search Fallback:** Always use the Jumpseller REST API (via `curl` or shell) for listing or searching resources.
   - Use Basic Auth: `-u "${JUMPSELLER_LOGIN}:${JUMPSELLER_AUTH_TOKEN}"`.
   - Explicitly notify the user when using this fallback.
2. **MCP Usage:** Continue using MCP tools for:
   - Single resource retrieval: `get_product`, `get_order`, `get_customer`, `get_store_info`.
   - Write operations: `create_*`, `update_*`, `delete_*`.
