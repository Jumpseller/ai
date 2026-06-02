# Jumpseller MCP Server

Use this skill when managing a Jumpseller store through the MCP server tools, or when setting up the MCP connection.

## What is the Jumpseller MCP Server

A live MCP (Model Context Protocol) server at `https://mcp.jumpseller.com` that lets AI agents act on a Jumpseller store in real time. It exposes 20 tools across six domains.

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

### Products (6 tools)

| Tool | Description |
|---|---|
| `get_products` | List products with optional filters: `status`, `category_id`, `page`, `limit` |
| `get_product` | Get a single product by ID — returns full field set including variants and images |
| `create_product` | Create a product with `name`, `price`, `stock`, `description`, `status`, `type` |
| `update_product` | Update product fields: `price`, `stock`, `status`, `name`, `description` |
| `delete_product` | Delete a product by ID — irreversible |
| `search_products` | Search products by name or SKU keyword |

### Orders (3 tools)

| Tool | Description |
|---|---|
| `get_orders` | List orders with optional `status_enum` filter (`pending_payment`, `paid`, `canceled`, `abandoned`) |
| `get_order` | Get a single order by ID with full line items, addresses, and shipping |
| `update_order` | Update order status or add `tracking_number`, `tracking_company`, `tracking_url` |

### Customers (2 tools)

| Tool | Description |
|---|---|
| `get_customers` | List customers — supports search by `email` |
| `get_customer` | Get a single customer with addresses and order history |

### Categories (3 tools)

| Tool | Description |
|---|---|
| `get_categories` | List all categories |
| `create_category` | Create a new category with `name` and optional `description`, `parent_id` |
| `update_category` | Update category `name` or `description` |

### Pages (2 tools)

| Tool | Description |
|---|---|
| `create_page` | Create a new custom store page with `title`, `body`, `status` |
| `update_page` | Update a page's `title`, `body`, or `status` |

### Store (1 tool)

| Tool | Description |
|---|---|
| `get_store` | Get store configuration: name, currency, country, plan, contact info |

## Common Multi-Step Patterns

### Find and ship an order

1. `get_orders` with `status_enum: "paid"` — find orders ready to fulfill.
2. `get_order` with the specific ID — confirm line items and shipping address.
3. `update_order` with `tracking_number`, `tracking_company`, and `tracking_url`.

### Bulk price adjustment

1. `get_products` with `page: 1, limit: 200` — repeat incrementing page until response is empty.
2. For each product, `update_product` with the new price.
3. Pause between requests to respect the 100 req/min limit.

### Set up a new product catalog

1. `create_category` for each top-level category — note the returned `id`.
2. `create_product` for each product, using the category `id` from step 1.

### Find a customer and review their orders

1. `get_customers` with `email: "customer@example.com"` — get the customer ID.
2. `get_orders` with `customer_id` filter — list their orders.
3. `get_order` for any order you want to inspect in detail.

## Connection Setup

The `.mcp.json` in this toolkit pre-configures the connection:

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

Set `JUMPSELLER_LOGIN_KEY` and `JUMPSELLER_AUTH_TOKEN` as environment variables before starting your AI tool. The credentials are never committed to the repo.
