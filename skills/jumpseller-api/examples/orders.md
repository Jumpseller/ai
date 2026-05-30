# Orders API — Examples

## List all orders

```
GET https://api.jumpseller.com/v1/orders.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN&page=1&limit=50
```

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

## Update order — add tracking info

```
PUT https://api.jumpseller.com/v1/orders/5001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "order": {
    "tracking_number": "1Z999AA10123456784",
    "tracking_company": "UPS",
    "tracking_url": "https://www.ups.com/track?tracknum=1Z999AA10123456784"
  }
}
```

## Mark order as paid

```
PUT https://api.jumpseller.com/v1/orders/5001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "order": {
    "status": "paid"
  }
}
```
