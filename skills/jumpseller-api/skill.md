# Jumpseller REST API

Use this skill when integrating with the Jumpseller REST API to manage store data programmatically.

## Base URL

```
https://api.jumpseller.com/v1/
```

All endpoints use this central API domain regardless of the store's subdomain.

## Authentication

Append your credentials as query parameters to every request:

```
?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

Retrieve credentials from: **Admin Panel → Account Settings → API Tokens**.

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
| `limit` | Results per page | `50` | `200` |

To retrieve all records: increment `page` until the number of returned items is less than `limit`.

## Rate Limiting

100 requests per minute per account. On `429`, wait 60 seconds before retrying. For bulk operations, add a small delay (e.g. 100ms) between requests to avoid hitting the limit.

## Resources

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

Key product fields: `name`, `description`, `price`, `compare_at_price`, `cost_per_item`, `stock`, `stock_unlimited`, `sku`, `brand`, `barcode`, `status` (`available` or `unavailable`), `type` (`physical` or `digital`), `weight`, `package_format` (`box`, `tube`, or `envelope`), `length`, `width`, `height`, `shipping_required`, `featured`, `categories`, `images`, `variants`, `permalink`.

### Orders

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/orders.json` | List all orders |
| `GET` | `/orders/{id}.json` | Get an order |
| `PUT` | `/orders/{id}.json` | Update order status or tracking |
| `GET` | `/orders/count.json` | Count all orders |
| `GET` | `/orders/status/{status_enum}.json` | List orders by status |

**Valid order status enums:**

| status_enum | Status name |
|---|---|
| `pending_payment` | Pending Payment |
| `paid` | Paid |
| `canceled` | Canceled |
| `abandoned` | Abandoned |

Order status (`status_enum`) tracks payment state. Shipment state is tracked separately via `shipment_status_enum` (e.g. `unfulfilled`, `fulfilled`).

Key order fields: `id`, `status`, `status_name`, `status_enum`, `currency`, `subtotal`, `tax`, `shipping`, `total`, `discount`, `fulfillment_status`, `shipment_status_enum`, `tracking_number`, `tracking_company`, `tracking_url`, `customer`, `shipping_address`, `billing_address`, `products`, `source`.

### Customers

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/customers.json` | List customers |
| `GET` | `/customers/{id}.json` | Get a customer |
| `POST` | `/customers.json` | Create a customer |
| `PUT` | `/customers/{id}.json` | Update a customer |
| `GET` | `/customers/count.json` | Count customers |

Search customers by email: `GET /customers.json?email=user@example.com&login=...&authtoken=...`

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
| `GET` | `/hooks.json` | List webhooks |
| `POST` | `/hooks.json` | Register a webhook |
| `DELETE` | `/hooks/{id}.json` | Delete a webhook |

Webhook fields: `id`, `name`, `event`, `url`, `application_id`.

Events: `order_created`, `order_updated`, `order_paid`, `order_shipped`, `product_created`, `product_updated`, `product_deleted`, `customer_created`.

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

See `examples/products.md`, `examples/orders.md`, `examples/customers.md`, `examples/shipping.md`, and `examples/promotions.md` for complete request and response examples.
