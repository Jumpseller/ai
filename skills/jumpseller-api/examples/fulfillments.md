# Fulfillments API — Examples

Fulfillments are the resource that carries **shipping and tracking** information. They are **separate from orders**: an order's `tracking_number` / `tracking_company` / `tracking_url` are read-only and populated *from* a fulfillment. To add tracking to an order, you create a fulfillment — you cannot PUT tracking onto the order itself.

> All of the below is verified against a live store.

## Endpoints

| Method | Path | Description |
|---|---|---|
| `GET` | `/fulfillments.json` | List fulfillments |
| `POST` | `/fulfillments.json` | Create a fulfillment (this is how you add tracking) |
| `GET` | `/fulfillments/count.json` | Count fulfillments |
| `POST` | `/fulfillments/rates.json` | Get shipping rates |
| `GET` | `/fulfillments/{id}.json` | Get a fulfillment |
| `PUT` | `/fulfillments/{id}.json` | Modify a fulfillment |
| `GET` | `/fulfillments/{id}/label.json` | Get the shipping label (link valid ~24h) |
| `GET` | `/order/{id}/fulfillments.json` | List fulfillments for an order (note: singular `/order/`) |

## Create a fulfillment with manual tracking

Required fields: **`type`** and **`location_id`**. Pass `order_id` to attach it to an order.

`type` enum: `manual`, `shipit`, `chilexpress`, `ctt`, `dpd`, `mrw`, `correos_chile`, `dhl`, `servientrega`, `starken`, `bluexpress`, `correos_express`.

For a manually-entered tracking number, use `type: "manual"`. The `tracking_url` is only honored when `type: "manual"` **and** `tracking_company: "other"`.

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X POST https://api.jumpseller.com/v1/fulfillments.json \
  -H "Content-Type: application/json" \
  -d '{
    "fulfillment": {
      "order_id": "2346",
      "type": "manual",
      "location_id": "269115",
      "tracking_company": "other",
      "tracking_number": "TRK-12345",
      "tracking_url": "https://www.chilexpress.cl/seguimiento?codigo=TRK-12345",
      "send_email": false
    }
  }'
```

**Response (trimmed):** returns `order_fulfillment` with the new fulfillment `id` and the full embedded `order`, which now shows:

```json
{
  "order_fulfillment": {
    "id": 12474228,
    "tracking_number": "TRK-12345",
    "tracking_company": "other",
    "type": "manual",
    "order": {
      "id": 2346,
      "status": "Paid",
      "fulfillment_status": "fulfilled",
      "shipment_status": "Procesado",
      "shipment_status_enum": "fulfilled",
      "tracking_number": "TRK-12345",
      "tracking_company": "other",
      "tracking_url": "https://www.chilexpress.cl/seguimiento?codigo=TRK-12345"
    }
  }
}
```

### Useful create fields

| Field | Notes |
|---|---|
| `order_id` | Order to attach the fulfillment to |
| `type` | **Required.** Carrier/service (see enum above); `manual` for hand-entered tracking |
| `location_id` | **Required.** Origin location the shipment leaves from |
| `tracking_number` / `tracking_company` | Tracking info (for `manual` type) |
| `tracking_url` | Only used when `type: manual` and `tracking_company: "other"` |
| `send_email` | `true` emails the customer the shipping notification; set `false` to stay silent |
| `receiver_email`, `phone`, `phone_prefix` | Customer contact (optional if `order_id` given) |
| `packages_dimensions` | Array of package dimensions; defaults to order dimensions if empty |
| `message`, `observations` | Notes; `message` is included in the shipped-order email |
| `expected_arrival_from` / `expected_arrival_to` | Delivery window |

## Update a fulfillment

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  -X PUT https://api.jumpseller.com/v1/fulfillments/12474228.json \
  -H "Content-Type: application/json" \
  -d '{"fulfillment":{"shipment_status":"in_transit","tracking_number":"TRK-99999"}}'
```

`shipment_status` enum (fulfillment-level): `requested`, `in_transit`, `delivered`, `failed`.

## List fulfillments for an order

```bash
curl -u YOUR_LOGIN_KEY:YOUR_AUTH_TOKEN \
  https://api.jumpseller.com/v1/order/2346/fulfillments.json
```

## Full "ship an order" flow (verified)

1. `PUT /orders/{id}.json` with `{"order":{"status":"Paid"}}` — ensure the order is paid.
2. `POST /fulfillments.json` with `order_id`, `type`, `location_id`, and tracking fields.
3. The order is now `fulfillment_status: "fulfilled"` with tracking populated. Use `send_email: true` if you want the customer notified.
