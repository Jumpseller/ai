# Jumpseller REST API

Use this skill when integrating with the Jumpseller REST API to manage store data programmatically.

## Authentication

Every request must include two HTTP headers:

| Header | Value |
|---|---|
| `X-LOGIN-KEY` | Your store's Login Key |
| `X-AUTH-TOKEN` | Your store's Auth Token |

Retrieve credentials from: **Admin Panel → Account Settings → API Tokens**.

## Base URL

```
https://{login}.jumpseller.com/api/v1/
```

`{login}` is your store's login name — the subdomain of your Jumpseller store URL.

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

## Resources

### Products

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/products` | List products |
| `GET` | `/products/{id}` | Get a product |
| `POST` | `/products` | Create a product |
| `PUT` | `/products/{id}` | Update a product |
| `DELETE` | `/products/{id}` | Delete a product |
| `GET` | `/products/count` | Count all products |
| `GET` | `/products/{id}/variants` | List a product's variants |
| `POST` | `/products/{id}/variants` | Add a variant to a product |
| `PUT` | `/products/{id}/variants/{variant_id}` | Update a variant |
| `DELETE` | `/products/{id}/variants/{variant_id}` | Delete a variant |
| `POST` | `/products/{id}/images` | Upload a product image |
| `DELETE` | `/products/{id}/images/{image_id}` | Delete a product image |

Key product fields: `name`, `description`, `price`, `stock`, `sku`, `status` (`available` or `unavailable`), `weight`, `categories` (array of category IDs), `images`.

### Orders

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/orders` | List orders |
| `GET` | `/orders/{id}` | Get an order |
| `PUT` | `/orders/{id}` | Update order status or tracking |
| `GET` | `/orders/count` | Count orders |

Order status values: `Pending`, `Paid`, `Shipped`, `Delivered`, `Canceled`.

Filter orders: `GET /orders?status=Paid&page=1&limit=50`

### Customers

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/customers` | List customers |
| `GET` | `/customers/{id}` | Get a customer |
| `POST` | `/customers` | Create a customer |
| `PUT` | `/customers/{id}` | Update a customer |
| `GET` | `/customers/count` | Count customers |
| `GET` | `/customers/{id}/addresses` | List a customer's addresses |
| `POST` | `/customers/{id}/addresses` | Add an address |

Search customers: `GET /customers?email=user@example.com`

### Categories

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/categories` | List all categories |
| `GET` | `/categories/{id}` | Get a category |
| `POST` | `/categories` | Create a category |
| `PUT` | `/categories/{id}` | Update a category |
| `DELETE` | `/categories/{id}` | Delete a category |

### Pages

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/pages` | List custom pages |
| `GET` | `/pages/{id}` | Get a page |
| `POST` | `/pages` | Create a page |
| `PUT` | `/pages/{id}` | Update a page |
| `DELETE` | `/pages/{id}` | Delete a page |

### Store

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/store` | Get store configuration and info |

Returns: store name, currency, country, contact email, plan, languages.

### Webhooks

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/hooks` | List webhooks |
| `POST` | `/hooks` | Register a webhook |
| `DELETE` | `/hooks/{id}` | Delete a webhook |

Events: `order/created`, `order/updated`, `order/paid`, `order/shipped`, `product/created`, `product/updated`, `product/deleted`, `customer/created`.

### Promotions

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/promotions` | List promotions |
| `POST` | `/promotions` | Create a promotion |
| `PUT` | `/promotions/{id}` | Update a promotion |
| `DELETE` | `/promotions/{id}` | Delete a promotion |

## Error Handling

| HTTP Status | Meaning |
|---|---|
| `200` | Success |
| `201` | Created successfully |
| `400` | Bad request — malformed body or missing required fields |
| `401` | Unauthorized — invalid or missing credentials |
| `404` | Resource not found |
| `422` | Validation error — field-level errors in the response body |
| `429` | Rate limit exceeded — wait 60 seconds before retrying |
| `500` | Server error |

Error response body:
```json
{ "error": "Description of what went wrong" }
```

## Rate Limiting

100 requests per minute per account. When you receive `429`, wait 60 seconds before retrying. For bulk operations, add a small delay between requests (e.g., 100ms) to avoid hitting the limit.

## Examples

See `examples/products.md`, `examples/orders.md`, and `examples/customers.md` for complete request and response examples.
