# Jumpseller MCP Server

Use this skill when managing a Jumpseller store through the MCP server tools, or when setting up the MCP connection.

## What is the Jumpseller MCP Server

A live MCP (Model Context Protocol) server at `https://mcp.jumpseller.com` that lets AI agents act on a Jumpseller store in real time. It exposes 20 tools across six domains.

## When to Use MCP vs the REST API

| Use MCP when... | Use the REST API when... |
|---|---|
| You want the AI to act directly on a store (conversational/agentic) | You are building a programmatic integration or app |
| You need a quick one-off operation through a chat interface | You need full control over request/response handling |
| You are demoing store capabilities to a merchant | You are processing bulk data or running scheduled jobs |

## Authentication

### API Token (store owners)

Retrieve from **Admin Panel → Account Settings → API Tokens**.

Pass as environment variables — the `.mcp.json` reads them automatically:

```bash
export JUMPSELLER_LOGIN_KEY=your-login-key
export JUMPSELLER_AUTH_TOKEN=your-auth-token
```

### OAuth 2.0 (third-party app developers)

Register your app in the Jumpseller Partner Panel to receive a `client_id` and `client_secret`. Merchants authorize your app via the OAuth flow. Scopes available: `products`, `orders`, `customers`, `categories`, `pages`, `store`.

## Available Tools

### Products (6 tools)

| Tool | Description |
|---|---|
| `get_products` | List products with optional filters (status, category, page, limit) |
| `get_product` | Get a single product by ID |
| `create_product` | Create a new product with name, price, stock, description |
| `update_product` | Update product fields (price, stock, status, name, description) |
| `delete_product` | Delete a product by ID |
| `search_products` | Search products by name or SKU keyword |

### Orders (3 tools)

| Tool | Description |
|---|---|
| `get_orders` | List orders with optional status filter |
| `get_order` | Get a single order by ID with full line items and shipping |
| `update_order` | Update order status or add tracking number and carrier |

### Customers (2 tools)

| Tool | Description |
|---|---|
| `get_customers` | List customers, search by email |
| `get_customer` | Get a single customer with addresses and order history |

### Categories (3 tools)

| Tool | Description |
|---|---|
| `get_categories` | List all categories |
| `create_category` | Create a new category |
| `update_category` | Update category name or description |

### Pages (2 tools)

| Tool | Description |
|---|---|
| `create_page` | Create a new custom store page |
| `update_page` | Update a page's content or title |

### Store (1 tool)

| Tool | Description |
|---|---|
| `get_store` | Get store configuration: name, currency, country, plan, contact info |

## Rate Limiting

100 requests per minute per account. The MCP server enforces this automatically. If you are chaining many tool calls in a workflow, add pauses between them for large batches.

## Common Multi-Step Patterns

### Find and update an order

1. `get_orders` with `status: "Paid"` to find orders ready to ship.
2. `get_order` with the specific order ID to confirm details.
3. `update_order` with `status: "Shipped"` and the tracking number.

### Bulk price update

1. `get_products` with `page: 1, limit: 200` — repeat incrementing page until empty.
2. For each product, `update_product` with the new price.

### Set up a new product catalog

1. `create_category` for each top-level category.
2. `create_product` for each product, assigning the correct category ID.

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

Set your credentials as environment variables before starting your AI tool.
