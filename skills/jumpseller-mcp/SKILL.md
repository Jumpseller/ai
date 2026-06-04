---
name: jumpseller-mcp
description: "Use when managing a Jumpseller store through MCP tools or conversational store operations."
---
# Jumpseller MCP Server

Use this skill when managing a Jumpseller store through the MCP server tools, or when setting up the MCP connection.

## What is the Jumpseller MCP Server

A live MCP (Model Context Protocol) server at `https://mcp.jumpseller.com` that lets AI agents act on a Jumpseller store in real time. It exposes 24 tools across seven domains.

## When to Use MCP vs the REST API

| Use MCP when… | Use the REST API when… |
|---|---|
| You want the AI to act directly on a store (conversational/agentic workflows) | You are building a programmatic integration or app |
| You need a quick one-off operation through a chat interface | You need full control over request/response handling |
| You are demoing store capabilities to a merchant | You are processing bulk data or running scheduled jobs |

## Authentication

### API Token (store owners)

Retrieve from **Admin Panel → Account Settings → API Tokens**.

This is the **same Login key + Auth Token** used by the REST API (`jumpseller-api`) and the CLI (`jumpseller-cli`) — one credential pair per store, used across all three interfaces.

Set as environment variables — the `.mcp.json` reads them automatically:

```bash
export JUMPSELLER_LOGIN_KEY=your-login-key
export JUMPSELLER_AUTH_TOKEN=your-auth-token
```

### OAuth 2.0 (third-party app developers)

Register your app in the Jumpseller Partner Panel to receive a `client_id` and `client_secret`. Merchants authorize your app via the OAuth flow.

Available OAuth scopes: `products`, `orders`, `customers`, `categories`, `pages`, `store`.

## Rate Limiting

100 requests per minute per account. If you are chaining many tool calls in a single workflow, add pauses for large batch operations to stay within limits.

## Available Tools

24 tools across seven domains. Names and parameters are verified against the live server's `tools/list`; `*` marks a required parameter.

### Products (6 tools)

| Tool | Parameters | Description |
|---|---|---|
| `list_products` | `page`, `limit`, `status`, `category_id`, `price_min`, `price_max` | List products. Pagination + filter by status, category, and price range. |
| `get_product` | `id*` | Single product — name, price, description, status, images, variants, categories. |
| `create_product` | `name*`, `price*`, `description`, `status`, `stock`, `sku`, `barcode`, `brand`, `categories`, `images`, `variants`, … | Create a product (many optional fields). |
| `update_product` | `id*`, plus any `create_product` field | Update a product; pass only changed fields. Can add/update variants. |
| `delete_product` | `id*` | Delete a product — irreversible. |
| `search_products` | `query*`, `page`, `limit` | Search products by name, description, or SKU. |

### Categories (3 tools)

| Tool | Parameters | Description |
|---|---|---|
| `list_categories` | (none) | List all product categories as a tree. |
| `create_category` | `name*`, `parent_id` | Create a category, optionally under a parent. |
| `delete_category` | `id*` | Delete a category — irreversible. |

> There is **no** `update_category` tool. To rename or restructure a category, delete and recreate it, or use the REST API (`jumpseller-api`).

### Orders (4 tools)

| Tool | Parameters | Description |
|---|---|---|
| `list_orders` | `page`, `limit`, `status`, `fulfillment_status`, `created_after`, `created_before` | List orders with filters. **No `customer_id` filter** — use `search_orders` for a customer's orders. |
| `get_order` | `id*` | Single order — products, customer, addresses, shipping, payment, status. |
| `search_orders` | `query*`, `page`, `limit` | Search by order ID, customer name, email, or product name. |
| `update_order` | `id*`, `status`, `tracking_number`, `tracking_url`, `tracking_company`, `additional_information` | Change status (`pending_payment`, `paid`, `canceled`, `abandoned`) and set tracking. |

> **Tracking via MCP:** `update_order` accepts `tracking_number` / `tracking_url` / `tracking_company` directly. (At the raw REST level tracking is carried by a Fulfillment — see `jumpseller-api` → Fulfillments — but the MCP tool wraps that for you.)

### Customers (3 tools)

| Tool | Parameters | Description |
|---|---|---|
| `list_customers` | `page`, `limit` | List customers (pagination). |
| `get_customer` | `id*` | Single customer — email, phone, shipping and billing addresses. |
| `search_customers` | `query*`, `page`, `limit` | Search customers by name or email. |

### Pages (6 tools)

| Tool | Parameters | Description |
|---|---|---|
| `list_pages` | `page`, `limit` | List store pages (About, Contact, Policies, blog posts…). |
| `create_page` | `title*`, `body`, `status`, `template`, `categories`, `page_title`, `meta_description` | Create a page. Use template `Post` + the `Blog` category for a blog post. |
| `update_page` | `id*`, plus any `create_page` field | Update a page; pass only changed fields. |
| `delete_page` | `id*` | Delete a page — irreversible. |
| `list_page_categories` | (none) | List page categories (e.g. `Blog`, `News`). |
| `list_page_templates` | (none) | List the current theme's page templates (e.g. `Default`, `Post`). |

### Store (1 tool)

| Tool | Parameters | Description |
|---|---|---|
| `get_store_info` | (none) | Store info: name, code, URL, country, currency, contact email. |

### Uploads (1 tool)

| Tool | Parameters | Description |
|---|---|---|
| `prepare_upload` | `filename*` | Returns a presigned POST upload target (valid 10 min, images only, max 10MB). POST the file to the returned S3 URL, then pass the `upload_id` to `create_product`/`update_product`. If you cannot make external HTTP requests, use base64 images instead. |

## Common Multi-Step Patterns

### Find and ship an order

1. `list_orders` with `status: "paid"` — find orders ready to fulfill.
2. `get_order` with the specific ID — confirm line items and shipping address.
3. `update_order` with `tracking_number`, `tracking_company`, and `tracking_url` to attach tracking. (The MCP tool records this on the order's fulfillment for you; the raw REST flow is in `jumpseller-api` → Fulfillments / `examples/fulfillments.md`.)

### Bulk price adjustment

1. `list_products` with `page: 1, limit: 100` — repeat incrementing `page` until the response is empty.
2. For each product, `update_product` with the new price.
3. Pause between requests to respect the 100 req/min limit.

### Set up a new product catalog

1. `create_category` for each top-level category — note the returned `id`.
2. `create_product` for each product, using the category `id` from step 1.

### Find a customer and review their orders

1. `search_customers` with `query: "customer@example.com"` — locate the customer.
2. `search_orders` with `query: "customer@example.com"` — find their orders (`list_orders` has no `customer_id` filter).
3. `get_order` for any order you want to inspect in detail.

## Connection Setup

The server is `https://mcp.jumpseller.com`, authenticated with `X-LOGIN-KEY` / `X-AUTH-TOKEN` headers read from the `JUMPSELLER_LOGIN_KEY` / `JUMPSELLER_AUTH_TOKEN` environment variables. **These env vars must be set before the client launches** — otherwise the server can't authenticate and exposes no tools.

**Claude Code** — reads the toolkit's `.mcp.json` automatically:

```json
{
  "mcpServers": {
    "jumpseller": {
      "type": "http",
      "url": "https://mcp.jumpseller.com",
      "headers": {
        "X-LOGIN-KEY": "${JUMPSELLER_LOGIN_KEY}",
        "X-AUTH-TOKEN": "${JUMPSELLER_AUTH_TOKEN}"
      }
    }
  }
}
```

**Gemini CLI** — does NOT read `.mcp.json`. The server is declared in the extension's `gemini-extension.json` (`mcpServers` with `httpUrl`); installing the extension wires it up. Export the env vars before running `gemini`, then verify with `/mcp`.

Credentials are never committed to the repo — only env-var references.

## Known limitation: Gemini CLI rejects list/search results

Gemini CLI validates every tool result's `structuredContent` as a JSON **object** and crashes when a tool's payload is a top-level JSON **array**, with:

```
Invalid input: expected record, received array   (path: structuredContent)
```

This affects the list/search tools (`list_products`, `list_orders`, `list_categories`, `list_customers`, `search_*`), whose payload is a JSON array. **This is a Gemini CLI client bug, not a server problem** — the Jumpseller server returns spec-valid MCP (the array sits in a `text` content block and `structuredContent` is never sent), and the same tools work in Claude clients. Tracking upstream: `google-gemini/gemini-cli#15375`. Single-resource tools (`get_product`, `get_order`, `get_customer`, `get_store_info`) return objects and are unaffected.

**Fallback (never silent):** if a list/search tool fails this way in Gemini, tell the user it's the known Gemini limitation, then retrieve the same data via the REST API (`jumpseller-api` skill) — the REST list endpoints return the identical payload and aren't subject to Gemini's structured-content validation.

## If the MCP tools are not available

If you (the assistant) were asked to use the MCP but the `get_*`/`create_*`/`update_*` tools are not in your toolset, **do not silently fall back to the REST API or shell.** The MCP simply isn't connected. Tell the user, and point them to the fix:

1. The MCP server config must be loaded by the client (Claude reads `.mcp.json`; Gemini needs the extension installed).
2. `JUMPSELLER_LOGIN_KEY` and `JUMPSELLER_AUTH_TOKEN` must be set **before the client started**.

Only after confirming with the user should you use the REST API (`jumpseller-api` skill) as an alternative — and say explicitly that you are doing so instead of the MCP.
