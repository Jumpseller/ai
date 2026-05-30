# Promotions API — Examples

## List promotions

```
GET https://api.jumpseller.com/v1/promotions.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:**
```json
[
  {
    "promotion": {
      "id": 516289,
      "name": "Summer Sale 10%",
      "discount_target": "order",
      "quantity_x": 0,
      "condition_price": 0.0,
      "condition_qty": 0,
      "discount_amount_fix": 0.0,
      "discount_amount_percent": 10.0,
      "begins_at": "2026-06-01T00:00:00Z",
      "expires_at": "2026-08-31T23:59:59Z",
      "max_times_used": 0,
      "times_used": 0,
      "cumulative": true,
      "enabled": true,
      "customers": "all",
      "coupons": [
        { "id": 31051, "code": "SUMMER10", "usage_limit": 0, "times_used": 0 }
      ],
      "categories": [],
      "products": []
    }
  }
]
```

## Get a promotion

```
GET https://api.jumpseller.com/v1/promotions/516289.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

## Create a promotion

```
POST https://api.jumpseller.com/v1/promotions.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "promotion": {
    "name": "Welcome 5% off",
    "discount_target": "order",
    "discount_amount_percent": 5.0,
    "enabled": true,
    "customers": "all",
    "cumulative": false,
    "coupons": [
      { "code": "WELCOME5" }
    ]
  }
}
```

**Response:** `201 Created` with the full promotion object.

## Update a promotion

```
PUT https://api.jumpseller.com/v1/promotions/516289.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "promotion": {
    "enabled": false
  }
}
```

## `discount_target` values

| Value | Description |
|---|---|
| `order` | Discount applies to the entire order total |
| `product` | Discount applies to specific products (set in `products` array) |
| `category` | Discount applies to products in specific categories (set in `categories` array) |
