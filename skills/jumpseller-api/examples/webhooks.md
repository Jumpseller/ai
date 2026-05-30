# Webhooks API — Examples

## Register a webhook

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X POST https://api.jumpseller.com/v1/hooks.json \
  -H "Content-Type: application/json" \
  -d '{
    "hook": {
      "event": "order_paid",
      "url": "https://your-app.com/webhooks/orders"
    }
  }'
```

**Response:**
```json
{
  "hook": {
    "id": 12345,
    "name": null,
    "event": "order_paid",
    "url": "https://your-app.com/webhooks/orders",
    "application_id": null
  }
}
```

## List all webhooks

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  https://api.jumpseller.com/v1/hooks.json
```

## Update a webhook URL

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X PUT https://api.jumpseller.com/v1/hooks/12345.json \
  -H "Content-Type: application/json" \
  -d '{ "hook": { "url": "https://your-app.com/webhooks/v2/orders" } }'
```

## Delete a webhook

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X DELETE https://api.jumpseller.com/v1/hooks/12345.json
```

## Incoming webhook payload (order_paid example)

When an order is paid, Jumpseller POSTs to your URL with these headers:

```
Content-Type: application/json
Jumpseller-Store-Code: your-store
Jumpseller-Event: order_paid
Jumpseller-Hmac-Sha256: jg/4O1TrK3fP7NX2ljNruHQsnhA9agn1TJu7lfn3R2s=
Jumpseller-Triggered-At: 2025-01-01 10:18:51.829 UTC
```

And a JSON body with the full order object (same structure as `GET /orders/{id}.json`):

```json
{
  "order": {
    "id": 1026,
    "status": "Paid",
    "status_enum": "paid",
    "currency": "USD",
    "subtotal": 399.0,
    "total": 369.2,
    "customer": { "id": "123", "email": "test@gmail.com" },
    "products": [{ "id": 10732902, "name": "Black", "qty": 1, "price": 399.0 }]
  }
}
```

## Verifying the HMAC signature

The `Jumpseller-Hmac-Sha256` header lets you confirm the request came from Jumpseller and was not tampered with.

**Important:** Use the store's **hooks token** (found at Admin Panel → Config → Notifications/Webhooks), NOT the API auth token.

**Node.js:**
```javascript
const crypto = require('crypto');

function verifyWebhook(rawBody, hmacHeader, hooksToken) {
  const computed = crypto
    .createHmac('sha256', hooksToken)
    .update(rawBody)
    .digest('base64');
  return crypto.timingSafeEqual(Buffer.from(computed), Buffer.from(hmacHeader));
}

// Express example
app.post('/webhooks/orders', (req, res) => {
  const rawBody = req.rawBody; // must be raw Buffer, not parsed JSON
  const valid = verifyWebhook(rawBody, req.headers['jumpseller-hmac-sha256'], process.env.HOOKS_TOKEN);
  if (!valid) return res.status(401).send('Unauthorized');
  const event = req.headers['jumpseller-event'];
  const data  = req.body;
  // handle event...
  res.status(200).send('OK');
});
```

**Ruby:**
```ruby
require 'base64'
require 'openssl'

HOOKS_TOKEN = ENV['HOOKS_TOKEN']

def verify_webhook(raw_body, hmac_header)
  digest = OpenSSL::Digest.new('sha256')
  computed = Base64.encode64(OpenSSL::HMAC.digest(digest, HOOKS_TOKEN, raw_body)).strip
  computed == hmac_header
end
```

**PHP:**
```php
$hooksToken = getenv('HOOKS_TOKEN');
$rawBody    = file_get_contents('php://input');
$hmacHeader = $_SERVER['HTTP_JUMPSELLER_HMAC_SHA256'];
$computed   = base64_encode(hash_hmac('sha256', $rawBody, $hooksToken, true));
$verified   = hash_equals($hmacHeader, $computed);
```

## Handling delivery failures

- Jumpseller retries up to **4 times** on non-2xx or timeout
- Retry schedule: `N^4` minutes (attempt 3 → 81 min, attempt 4 → 256 min)
- **15-second timeout** per attempt — respond fast, process async
- After **8 total failures** the webhook is paused and the store admin is notified by email

Best practice: return `200 OK` immediately, then process the payload in a background job.
