# Shipping Methods API — Examples

## List shipping methods

```
GET https://api.jumpseller.com/v1/shipping_methods.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:**
```json
[
  {
    "shipping_method": {
      "id": 779781,
      "type": "starken",
      "name": "Starken",
      "enabled": true,
      "free_shipping": false,
      "free_shipping_minimum_purchase": 0.0,
      "fee": null,
      "services": [
        { "id": 122060, "name": "Agencia Normal",   "service_code": "10" },
        { "id": 122061, "name": "Agencia Expreso",  "service_code": "11" },
        { "id": 122062, "name": "Domicilio Normal",  "service_code": "20" },
        { "id": 122063, "name": "Domicilio Expreso", "service_code": "21" }
      ]
    }
  },
  {
    "shipping_method": {
      "id": 779782,
      "type": "custom",
      "name": "Free Shipping",
      "enabled": true,
      "free_shipping": true,
      "free_shipping_minimum_purchase": 50.0,
      "fee": 0.0,
      "services": []
    }
  }
]
```

The `services` array lists the specific service options available for carrier-based shipping methods (e.g. Starken, Blue Express). For custom/flat-rate methods, `services` is empty and `fee` is the flat charge.

## Payment Methods

```
GET https://api.jumpseller.com/v1/payment_methods.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:**
```json
[
  { "payment_method": { "id": 1001, "type": "stripe",       "name": "Stripe" } },
  { "payment_method": { "id": 1002, "type": "webpay_plus",  "name": "Webpay Plus" } },
  { "payment_method": { "id": 1003, "type": "oneclick",     "name": "Oneclick" } }
]
```
