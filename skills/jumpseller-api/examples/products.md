# Products API — Examples

## List products

```
GET https://api.jumpseller.com/v1/products.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN&page=1&limit=50
```

**Response:**
```json
[
  {
    "product": {
      "id": 1001,
      "name": "Classic T-Shirt",
      "page_title": "Classic T-Shirt",
      "description": "<p>100% cotton t-shirt.</p>",
      "price": 29.99,
      "compare_at_price": null,
      "cost_per_item": null,
      "weight": 0.3,
      "stock": 100,
      "stock_unlimited": false,
      "stock_threshold": 0,
      "stock_notification": false,
      "back_in_stock_enabled": true,
      "sku": "TSH-001",
      "brand": null,
      "barcode": null,
      "featured": false,
      "reviews_enabled": true,
      "status": "available",
      "shipping_required": true,
      "type": "physical",
      "package_format": "box",
      "length": 0.0,
      "width": 0.0,
      "height": 0.0,
      "categories": [{ "id": 5, "name": "Clothing", "parent_id": null }],
      "images": [{ "id": 1, "url": "https://images.jumpseller.com/store/your-store/1001/image.jpg", "position": 1 }],
      "variants": [],
      "fields": [],
      "permalink": "classic-t-shirt",
      "discount": "0.0",
      "currency": "USD"
    }
  }
]
```

## Get a single product

```
GET https://api.jumpseller.com/v1/products/1001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

## Count products

```
GET https://api.jumpseller.com/v1/products/count.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:**
```json
{ "count": 42 }
```

## Create a product

```
POST https://api.jumpseller.com/v1/products.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "product": {
    "name": "Classic T-Shirt",
    "description": "<p>100% cotton t-shirt.</p>",
    "price": 29.99,
    "stock": 100,
    "sku": "TSH-001",
    "status": "available",
    "weight": 0.3,
    "type": "physical",
    "package_format": "box"
  }
}
```

**Response:** `201 Created` with the full product object.

## Update a product

```
PUT https://api.jumpseller.com/v1/products/1001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "product": {
    "price": 24.99,
    "status": "available",
    "stock": 85
  }
}
```

## Delete a product

```
DELETE https://api.jumpseller.com/v1/products/1001.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
```

**Response:** `200 OK`

## Add a variant

```
POST https://api.jumpseller.com/v1/products/1001/variants.json?login=YOUR_LOGIN_KEY&authtoken=YOUR_AUTH_TOKEN
Content-Type: application/json

{
  "variant": {
    "sku": "TSH-001-RED-M",
    "price": 29.99,
    "stock": 25,
    "options": [
      { "name": "Color", "value": "Red" },
      { "name": "Size", "value": "M" }
    ]
  }
}
```

## Bulk price update — iterate all products

```javascript
const LOGIN = 'YOUR_LOGIN_KEY';
const TOKEN = 'YOUR_AUTH_TOKEN';
const BASE  = 'https://api.jumpseller.com/v1';
const AUTH  = `login=${LOGIN}&authtoken=${TOKEN}`;

let page = 1;
while (true) {
  const res      = await fetch(`${BASE}/products.json?${AUTH}&page=${page}&limit=100`);
  const products = await res.json();
  if (products.length === 0) break;

  for (const { product } of products) {
    await fetch(`${BASE}/products/${product.id}.json?${AUTH}`, {
      method:  'PUT',
      headers: { 'Content-Type': 'application/json' },
      body:    JSON.stringify({ product: { price: product.price * 1.1 } }),
    });
    // Small delay to respect rate limits
    await new Promise(r => setTimeout(r, 100));
  }
  page++;
}
```
