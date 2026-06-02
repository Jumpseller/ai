# Jumpseller REST API

Use this skill when integrating with the Jumpseller REST API to manage store data programmatically.

## Base URL

```
https://api.jumpseller.com/v1/
```

All endpoints use this central API domain regardless of the store's subdomain.

## Authentication

Basic Auth is the **recommended** method:

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN https://api.jumpseller.com/v1/products.json
```

Query parameters work but are **deprecated**:

```
?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

For third-party apps, use OAuth 2.0 — see https://jumpseller.com/support/oauth-2.

Retrieve credentials from: **Admin Panel → Account Settings → API Tokens**.

The same Login key + Auth Token pair is used by all Jumpseller interfaces — the REST API (Basic auth), the MCP server (`X-LOGIN-KEY`/`X-AUTH-TOKEN` headers), and the `jumpseller-cli` (stored in `~/.config/jumpseller/credentials`). You do not generate separate credentials per interface.

## URL Format

All endpoints require a `.json` extension:

```
GET https://api.jumpseller.com/v1/products.json?login=LOGIN_KEY&authtoken=AUTH_TOKEN
GET https://api.jumpseller.com/v1/orders/1234.json?login=LOGIN_KEY&authtoken=AUTH_TOKEN
```

## Request Format

- All request and response bodies are JSON.
- Set `Content-Type: application/json` on POST and PUT requests.
- Use standard HTTP verbs: `GET` (read), `POST` (create), `PUT` (update), `DELETE` (remove).

## Pagination

List endpoints accept `page` and `limit` query params:

| Param | Description | Default | Max |
|---|---|---|---|
| `page` | Page number, 1-indexed | `1` | — |
| `limit` | Results per page | `50` | `100` |

To retrieve all records: increment `page` until the number of returned items is less than `limit`.

Example:
```
https://api.jumpseller.com/v1/products.json?page=3&limit=100
```

## Rate Limiting

- **800 requests per minute**
- **20 requests per second**
- Rate limits apply per IP address AND per store
- After 2000 rate-limit hits, a temporary ban is applied

Response headers to monitor:

```
Jumpseller-PerMinuteRateLimit-Limit: 800
Jumpseller-PerMinuteRateLimit-Remaining: 799
Jumpseller-PerSecondRateLimit-Limit: 20
Jumpseller-PerSecondRateLimit-Remaining: 19
Jumpseller-BannedByRateLimit-Reset: 2024-05-23T16:13:47+00:00  (only when banned)
```

On `429`, check the headers and wait until the reset time before retrying.

## Resources

### Store

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/store/info.json` | Get store information |

Optional `fields` query param: comma-separated list of fields to return. Allowed values: `name`, `code`, `currency`, `country`, `timezone`, `email`, `hooks_token`, `url`, `subscription_plan`, `logo`, `weight_unit`, `fb_pixel_id`, `whatsapp_phone`, `mobile_app_version`, `subscription_status`, `checkout_version`, `address`.

### Products

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/products.json` | List products |
| `GET` | `/products/{id}.json` | Get a product |
| `POST` | `/products.json` | Create a product |
| `PUT` | `/products/{id}.json` | Update a product |
| `DELETE` | `/products/{id}.json` | Delete a product |
| `GET` | `/products/count.json` | Count all products |
| `GET` | `/products/{id}/variants.json` | List a product's variants |
| `POST` | `/products/{id}/variants.json` | Add a variant |
| `PUT` | `/products/{id}/variants/{variant_id}.json` | Update a variant |
| `DELETE` | `/products/{id}/variants/{variant_id}.json` | Delete a variant |
| `GET` | `/products/{id}/images.json` | List product images |
| `POST` | `/products/{id}/images.json` | Upload a product image |
| `DELETE` | `/products/{id}/images/{image_id}.json` | Delete an image |
| `GET` | `/products/search.json` | Search products |
| `GET` | `/products/status/{status}.json` | List products by status |
| `GET` | `/products/status/{status}/count.json` | Count products by status |

Valid product status values: `available`, `unavailable`

Key product fields: `name`, `description`, `price`, `compare_at_price`, `cost_per_item`, `stock`, `stock_unlimited`, `sku`, `brand`, `barcode`, `status` (`available` or `unavailable`), `type` (`physical` or `digital`), `weight`, `package_format` (`box`, `tube`, or `envelope`), `length`, `width`, `height`, `shipping_required`, `featured`, `categories`, `images`, `variants`, `permalink`.

### Orders

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/orders.json` | List all orders |
| `GET` | `/orders/{id}.json` | Get an order |
| `PUT` | `/orders/{id}.json` | Update order — **`status`, `shipment_status`, `additional_information`, `additional_fields` only** (NOT tracking) |
| `GET` | `/orders/count.json` | Count all orders |
| `GET` | `/orders/status/{status_enum}.json` | List orders by status |
| `GET` | `/orders/search.json` | Search/filter orders (supports `fulfillment_filters`, `status_filters[]`, `dateFilter` params) |
| `GET` | `/order/{id}/fulfillments.json` | List an order's fulfillments (note singular `/order/`) |

**Valid order status enums:**

| status_enum | Status name |
|---|---|
| `pending_payment` | Pending Payment |
| `paid` | Paid |
| `canceled` | Canceled |
| `abandoned` | Abandoned |

Order status (`status_enum`) tracks payment state. Shipment state is tracked separately via `shipment_status_enum` (e.g. `unfulfilled`, `fulfilled`).

Key order fields: `id`, `status`, `status_name`, `status_enum`, `currency`, `subtotal`, `tax`, `shipping`, `total`, `discount`, `fulfillment_status`, `shipment_status_enum`, `tracking_number`, `tracking_company`, `tracking_url`, `customer`, `shipping_address`, `billing_address`, `products`, `source`.

**Tracking is read-only on the order.** `tracking_number`, `tracking_company`, and `tracking_url` appear when you read an order, but they are populated *from* a Fulfillment — you cannot set them via `PUT /orders/{id}.json`. To add tracking you create a fulfillment. See the **Fulfillments** section below and `examples/fulfillments.md`.

### Fulfillments (shipping & tracking)

Fulfillments are a separate resource that carries shipping/tracking. Creating one is how you "ship" an order.

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/fulfillments.json` | List fulfillments |
| `POST` | `/fulfillments.json` | Create a fulfillment (adds tracking to an order) |
| `GET` | `/fulfillments/count.json` | Count fulfillments |
| `POST` | `/fulfillments/rates.json` | Get shipping rates |
| `GET` | `/fulfillments/{id}.json` | Get a fulfillment |
| `PUT` | `/fulfillments/{id}.json` | Modify a fulfillment |
| `GET` | `/fulfillments/{id}/label.json` | Shipping label (link valid ~24h) |

Create requires `type` (`manual`, `chilexpress`, `shipit`, `dhl`, `ctt`, `dpd`, `mrw`, `correos_chile`, `servientrega`, `starken`, `bluexpress`, `correos_express`) and `location_id`; pass `order_id` to attach it. Fulfillment-level `shipment_status` enum: `requested`, `in_transit`, `delivered`, `failed`. Full verified flow and field reference in `examples/fulfillments.md`.

### Customers

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/customers.json` | List customers |
| `GET` | `/customers/{id}.json` | Get a customer |
| `POST` | `/customers.json` | Create a customer |
| `PUT` | `/customers/{id}.json` | Update a customer |
| `GET` | `/customers/count.json` | Count customers |
| `GET` | `/customers/email/{email}.json` | Get a customer directly by email address |
| `GET` | `/customers/{id}/orders.json` | List all orders for a customer |
| `GET` | `/customers/search.json` | Search customers |

Customer objects include `shipping_addresses` and `billing_addresses` arrays.

### Categories

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/categories.json` | List all categories |
| `GET` | `/categories/{id}.json` | Get a category |
| `POST` | `/categories.json` | Create a category |
| `PUT` | `/categories/{id}.json` | Update a category |
| `DELETE` | `/categories/{id}.json` | Delete a category |

Category fields: `id`, `name`, `description`, `images`, `parent_id`, `permalink`, `products`.

### Pages

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/pages.json` | List custom pages |
| `GET` | `/pages/{id}.json` | Get a page |
| `POST` | `/pages.json` | Create a page |
| `PUT` | `/pages/{id}.json` | Update a page |
| `DELETE` | `/pages/{id}.json` | Delete a page |

Page fields: `id`, `title`, `body`, `status` (`public`/`hidden`), `legal`, `permalink`, `template`.

### Webhooks

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/hooks.json` | List all webhooks |
| `GET` | `/hooks/{id}.json` | Get a webhook |
| `POST` | `/hooks.json` | Register a webhook |
| `PUT` | `/hooks/{id}.json` | Update a webhook |
| `DELETE` | `/hooks/{id}.json` | Delete a webhook |

Webhook fields: `id`, `name`, `event`, `url`, `application_id`.

**Valid event values (from swagger spec):**

| Event | Trigger |
|---|---|
| `order_pending_payment` | New order created |
| `order_paid` | Order payment confirmed |
| `order_shipped` | Order shipped to customer |
| `order_canceled` | Order canceled |
| `order_abandoned` | Cart abandoned |
| `order_updated` | Any order state change |
| `product_created` | New product added |
| `product_updated` | Product details changed |
| `product_deleted` | Product removed |
| `product_stock_updated` | Product stock changed |
| `customer_created` | New customer registered |
| `customer_updated` | Customer profile changed |
| `customer_deleted` | Customer removed |

**Webhook payload:** Jumpseller POSTs JSON to your URL using the same structure as the REST API for that resource. Expects a `2xx` response within 15 seconds. Retries up to 4 times on failure (`N^4` minutes apart). After 8 failed attempts the webhook is paused and the store admin is notified.

**Request headers sent by Jumpseller:**

| Header | Example value | Description |
|---|---|---|
| `Content-Type` | `application/json` | Always JSON |
| `Jumpseller-Store-Code` | `your-store` | Store subdomain |
| `Jumpseller-Event` | `order_paid` | Event that fired |
| `Jumpseller-Hmac-Sha256` | `jg/4O1Tr...` | HMAC-SHA256 signature |
| `Jumpseller-Triggered-At` | `2025-01-01 10:18:51.829 UTC` | Timestamp |

**Verifying authenticity:** The `Jumpseller-Hmac-Sha256` header is a Base64-encoded HMAC-SHA256 of the raw request body, signed with your store's **hooks token** (found at Admin Panel → Config → Notifications/Webhooks — this is different from the API auth token).

See `examples/webhooks.md` for registration, payload, and HMAC verification examples.

### Promotions

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/promotions.json` | List promotions |
| `GET` | `/promotions/{id}.json` | Get a promotion |
| `POST` | `/promotions.json` | Create a promotion |
| `PUT` | `/promotions/{id}.json` | Update a promotion |
| `DELETE` | `/promotions/{id}.json` | Delete a promotion |

Promotion fields: `id`, `name`, `discount_target`, `discount_amount_fix`, `discount_amount_percent`, `begins_at`, `expires_at`, `max_times_used`, `times_used`, `cumulative`, `enabled`, `customers`, `coupons`, `categories`, `products`.

### Shipping Methods

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/shipping_methods.json` | List configured shipping methods |

Shipping method fields: `id`, `type`, `name`, `enabled`, `free_shipping`, `free_shipping_minimum_purchase`, `fee`, `services` (array of `{id, name, service_code}`).

### Payment Methods

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/payment_methods.json` | List configured payment methods |

Payment method fields: `id`, `type`, `name`.

## Error Handling

| HTTP Status | Meaning |
|---|---|
| `200` | Success |
| `201` | Created successfully |
| `400` | Bad request — malformed body, missing fields, or invalid parameter value |
| `401` | Unauthorized — invalid or missing credentials |
| `404` | Resource not found |
| `422` | Validation error — field-level errors in response body |
| `429` | Rate limit exceeded — wait 60 seconds |
| `500` | Server error |

## Examples

See `examples/products.md`, `examples/orders.md`, `examples/customers.md`, `examples/shipping.md`, `examples/promotions.md`, and `examples/webhooks.md` for complete request and response examples.
