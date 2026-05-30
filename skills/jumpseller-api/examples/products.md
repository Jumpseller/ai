# Products API — Examples

## List products (first page)

**Request:**
```
GET https://your-login.jumpseller.com/api/v1/products?page=1&limit=50
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

**Response:**
```json
[
  {
    "product": {
      "id": 1001,
      "name": "Classic T-Shirt",
      "price": 29.99,
      "stock": 100,
      "sku": "TSH-001",
      "status": "available",
      "categories": [{ "id": 5, "name": "Clothing" }],
      "images": [{ "id": 1, "url": "https://cdn.jumpseller.com/products/tshirt.jpg", "position": 1 }]
    }
  }
]
```

## Get a single product

**Request:**
```
GET https://your-login.jumpseller.com/api/v1/products/1001
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

## Create a product

**Request:**
```
POST https://your-login.jumpseller.com/api/v1/products
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

{
  "product": {
    "name": "Classic T-Shirt",
    "description": "<p>100% cotton, available in multiple colors.</p>",
    "price": 29.99,
    "stock": 100,
    "sku": "TSH-001",
    "status": "available",
    "weight": 0.3
  }
}
```

**Response:** `201 Created` with the created product object.

## Update a product

**Request:**
```
PUT https://your-login.jumpseller.com/api/v1/products/1001
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

{
  "product": {
    "price": 24.99,
    "status": "available"
  }
}
```

## Delete a product

**Request:**
```
DELETE https://your-login.jumpseller.com/api/v1/products/1001
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token
```

**Response:** `200 OK`

## Add a variant

**Request:**
```
POST https://your-login.jumpseller.com/api/v1/products/1001/variants
Content-Type: application/json
X-LOGIN-KEY: your-login-key
X-AUTH-TOKEN: your-auth-token

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

## Bulk price update (iterate all products)

```javascript
const headers = {
  'X-LOGIN-KEY': process.env.JUMPSELLER_LOGIN_KEY,
  'X-AUTH-TOKEN': process.env.JUMPSELLER_AUTH_TOKEN,
  'Content-Type': 'application/json'
};
const base = `https://your-login.jumpseller.com/api/v1`;

let page = 1;
while (true) {
  const res = await fetch(`${base}/products?page=${page}&limit=200`, { headers });
  const products = await res.json();
  if (products.length === 0) break;

  for (const { product } of products) {
    await fetch(`${base}/products/${product.id}`, {
      method: 'PUT',
      headers,
      body: JSON.stringify({ product: { price: product.price * 1.1 } })
    });
  }
  page++;
}
```
