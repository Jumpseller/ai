# Customers API — Examples

## Search customer by email

**Request:**
```
GET https://your-login.jumpseller.com/api/v1/customers?email=customer@example.com
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

**Response:**
```json
[
  {
    "customer": {
      "id": 201,
      "email": "customer@example.com",
      "name": "Jane Doe",
      "phone": "+1-555-0100",
      "orders_count": 3,
      "total_spent": 149.97
    }
  }
]
```

## Create a customer

**Request:**
```
POST https://your-login.jumpseller.com/api/v1/customers
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

{
  "customer": {
    "email": "newcustomer@example.com",
    "name": "John Smith",
    "phone": "+1-555-0101"
  }
}
```

**Response:** `201 Created` with the created customer object.

## Add an address to a customer

**Request:**
```
POST https://your-login.jumpseller.com/api/v1/customers/201/addresses
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

{
  "address": {
    "name": "Jane Doe",
    "address": "456 Oak Ave",
    "city": "Valparaíso",
    "region": "Valparaíso",
    "country": "CL",
    "postal_code": "2340000"
  }
}
```
