# Customers API — Examples

## List customers

```
GET https://api.jumpseller.com/v1/customers.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN&page=1&limit=50
```

**Response:**
```json
[
  {
    "customer": {
      "id": 201,
      "email": "customer@example.com",
      "phone": "+1-555-0100",
      "phone_prefix": null,
      "status": "approved",
      "fullname": "Jane Doe",
      "accepts_marketing": false,
      "accepted_marketing_at": null,
      "shipping_addresses": [],
      "billing_addresses": [],
      "customer_categories": [],
      "customer_additional_fields": []
    }
  }
]
```

## Search customer by email

```
GET https://api.jumpseller.com/v1/customers.json?email=customer@example.com&login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

## Create a customer

```
POST https://api.jumpseller.com/v1/customers.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "customer": {
    "email": "newcustomer@example.com",
    "fullname": "John Smith",
    "phone": "+1-555-0101"
  }
}
```

**Response:** `201 Created` with the full customer object.

## Update a customer

```
PUT https://api.jumpseller.com/v1/customers/201.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "customer": {
    "accepts_marketing": true
  }
}
```

## Count customers

```
GET https://api.jumpseller.com/v1/customers/count.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:** `{ "count": 1432 }`
