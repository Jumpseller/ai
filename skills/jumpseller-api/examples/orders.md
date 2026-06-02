# Orders API — Examples

## List all orders

```
GET https://api.jumpseller.com/v1/orders.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN&page=1&limit=50
```

**Ordering (verified):** orders are returned sorted by **`id` descending — newest first**. This is by id, not by `created_at` (abandoned carts can have out-of-order dates). So **the most recent N orders are simply `page=1&limit=N`** — do not compute the last page from `orders/count.json`. Page 1 is the newest; higher pages are older.

```bash
# The last 10 orders (most recent)
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  "https://api.jumpseller.com/v1/orders.json?limit=10&page=1"
```

Note: `orders/count.json` counts all orders including abandoned carts, so the total can be much larger than the highest order `id`.

## List orders by status

Valid status enums: `pending_payment`, `paid`, `canceled`, `abandoned`

```
GET https://api.jumpseller.com/v1/orders/status/paid.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:**
```json
[
  {
    "order": {
      "id": 5001,
      "status": "Paid",
      "status_name": "Paid",
      "status_enum": "paid",
      "created_at": "2026-05-01 10:30:00 UTC",
      "completed_at": "2026-05-01 10:30:00 UTC",
      "currency": "USD",
      "subtotal": 59.98,
      "tax": 0,
      "shipping_tax": 0,
      "shipping": 5.00,
      "total": 64.98,
      "discount": 0,
      "fulfillment_status": "unfulfilled",
      "shipment_status": "No Procesado",
      "shipment_status_enum": "unfulfilled",
      "shipping_method_name": "Standard Shipping",
      "payment_method_name": "Stripe",
      "payment_method_type": "stripe",
      "tracking_number": null,
      "tracking_company": null,
      "tracking_url": null,
      "source": { "created_from": "storefront" },
      "customer": {
        "id": 201,
        "email": "customer@example.com",
        "fullname": "Jane Doe",
        "phone": "+1-555-0100"
      },
      "shipping_address": {
        "name": "Jane",
        "surname": "Doe",
        "address": "123 Main St",
        "city": "New York",
        "region": "New York",
        "country": "United States",
        "country_code": "US",
        "postal": "10001"
      },
      "billing_address": {
        "name": "Jane",
        "surname": "Doe",
        "address": "123 Main St",
        "city": "New York",
        "country": "United States",
        "country_code": "US"
      },
      "products": [
        {
          "id": 1001,
          "variant_id": null,
          "sku": "TSH-001",
          "name": "Classic T-Shirt",
          "qty": 2,
          "price": 29.99,
          "tax": 0,
          "discount": 0,
          "weight": 0.3,
          "type": "physical"
        }
      ],
      "promotions": [],
      "coupons": null
    }
  }
]
```

## Get a single order

```
GET https://api.jumpseller.com/v1/orders/5001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

## Count orders

```
GET https://api.jumpseller.com/v1/orders/count.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:** `{ "count": 128 }`

## Update an order (status only — NOT tracking)

`PUT /orders/{id}.json` accepts **only** these fields (verified — sending anything else returns `400 "No se encontraron parámetros permitidos"`):

| Field | Notes |
|---|---|
| `status` | e.g. `"Paid"`, `"Canceled"` (capitalized) |
| `shipment_status` | order-level shipment state |
| `additional_information` | free text |
| `additional_fields` | custom fields |

**Tracking is NOT an order field — you cannot PUT `tracking_number` on an order.** Tracking lives on a separate **Fulfillment** resource. See `examples/fulfillments.md`.

```bash
# Mark an order as paid (verified)
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X PUT https://api.jumpseller.com/v1/orders/5001.json \
  -H "Content-Type: application/json" \
  -d '{"order":{"status":"Paid"}}'
```

## Ship an order with a tracking number

This is a two-step flow (verified against a live store):

1. Ensure the order is `Paid` (above).
2. Create a fulfillment with the tracking info — see **`examples/fulfillments.md`**. Once the fulfillment exists, the order's read-only `tracking_number`, `tracking_company`, `tracking_url`, `fulfillment_status: "fulfilled"`, and `shipment_status_enum: "fulfilled"` are populated **from** the fulfillment.
