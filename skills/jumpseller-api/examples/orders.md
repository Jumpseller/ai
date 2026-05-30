# Orders API — Examples

## List paid orders

**Request:**
```
GET https://your-login.jumpseller.com/api/v1/orders?status=Paid&page=1&limit=50
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

**Response:**
```json
[
  {
    "order": {
      "id": 5001,
      "status": "Paid",
      "total": 59.98,
      "currency": "USD",
      "customer": { "id": 201, "email": "customer@example.com", "name": "Jane Doe" },
      "line_items": [
        { "product_id": 1001, "name": "Classic T-Shirt", "quantity": 2, "price": 29.99 }
      ],
      "shipping_address": {
        "name": "Jane Doe",
        "address": "123 Main St",
        "city": "Santiago",
        "country": "CL"
      },
      "created_at": "2026-05-01T10:30:00Z"
    }
  }
]
```

## Update order status to Shipped

**Request:**
```
PUT https://your-login.jumpseller.com/api/v1/orders/5001
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

{
  "order": {
    "status": "Shipped",
    "tracking_number": "1Z999AA10123456784",
    "tracking_company": "UPS"
  }
}
```

## Count orders by status

**Request:**
```
GET https://your-login.jumpseller.com/api/v1/orders/count?status=Pending
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

**Response:**
```json
{ "count": 12 }
```
