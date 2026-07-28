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

## Create a "tables" shipping method (rate table by weight/location)

```
POST https://api.jumpseller.com/v1/shipping_methods.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json
```

```json
{
  "shipping_method": {
    "type": "tables",
    "name": "Tarifario CL",
    "enabled": false,
    "tables": [
      {
        "basedon": "weight",
        "values": [
          { "amount": 0,   "price": 3990 },
          { "amount": 2,   "price": 4990 },
          { "amount": 9,   "price": 6990 },
          { "amount": 20,  "price": 8990 },
          { "amount": 45,  "price": 11990 },
          { "amount": 100, "price": 11990 }
        ],
        "locations": [
          { "country": "CL", "region": "13", "municipality": "13101" }
        ]
      }
    ]
  }
}
```

- One `tables[]` entry = one rate table for one `locations[]` zone (e.g. one row of a rate sheet). A shipping method can carry many tables, one per zone.
- `values[]` is a list of **breakpoints**, not buckets: `{amount, price}` means "starting at this weight, this price applies until the next breakpoint." If your source data is organized as columns that are range *ceilings* (e.g. a column named `2` meaning "price for 0–2kg", a column `9` meaning "price for 2–9kg"), you must shift it: prepend `{amount: 0, price: <price of the first column>}`, then pair each subsequent column's price with the *previous* column's weight as the `amount`, and finally repeat the last column's price at `amount: <last weight>` as the "or heavier" catch-all. Getting this backwards produces a table that looks valid but charges the wrong bracket's price.
- `region` in `locations[]` is Chile's 2-digit region code as a zero-padded **string** (`"02"`, not `"2"`).
- Creating with `enabled: false` and enabling only after checking the table in the admin is a reasonable safety practice — a bad range-shift silently produces wrong live pricing rather than an error.
- A `tables` entry doesn't have to be a multi-bracket weight scale — `basedon: "price"` (or `"weight"`) with a single `{amount: 0, price: X}` value is a valid, common shape: a flat per-zone rate with no brackets at all.

## Update an existing "tables" shipping method

```
PUT https://api.jumpseller.com/v1/shipping_methods/{id}.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json
```
```json
{ "shipping_method": { "tables": [ /* full replacement tables[] array */ ] } }
```

Confirmed working (2026-07-28, live store): `PUT` with a `tables[]` body updates the tables **in place** — no need to delete/recreate the shipping method to change prices. The array you send **replaces** the existing one wholesale, it is not a per-zone patch — `GET` the shipping method first, build the full new `tables[]` (unchanged zones included verbatim), then `PUT` it back. Sending only the zones you're changing silently drops shipping rates for every zone you omitted.

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
