# FCL API Reference

## Endpoints

### Products

| Method | Path | Description |
|--------|------|-------------|
| GET | `/products` | List all products |
| GET | `/products/{product_id}` | Get a single product |

**Product fields:** `id`, `name`, `code`, `description`, `price_gbp_p`

Use `code` when creating a cart; `id` when fetching a single product. Always call `GET /products` to get current pricing. `VC001` (Virtual Card Credit) cannot be ordered via API.

---

### Drop-off points

| Method | Path | Description |
|--------|------|-------------|
| GET | `/dropoff_points` | List all drop-off/collection points |

**Fields:** `id`, `name`, `enabled`, `reduced_emissions`

Only `enabled: true` points can be used. Pass `null` as `dropoff_point_id` for standard postal delivery.

---

### Virtual cards

| Method | Path | Description |
|--------|------|-------------|
| GET | `/virtual_cards` | List cards owned by or shared with the account |

**Fields:** `id`, `owner_id`, `owner_email`, `label`, `balance_gbp_p`, `created_time`

Use `id` as `payment_virtual_card_id` when placing an order.

---

### Cart

| Method | Path | Description |
|--------|------|-------------|
| GET | `/cart` | Get current cart (404 if none) |
| POST | `/cart` | Create or replace cart |
| DELETE | `/cart` | Delete cart |

**POST /cart request body:**

```json
{
  "product_code": "FLPS001",
  "product_quantity": 2,
  "items": {
    "samples": [
      {"name": "Plasmid A", "length": 4500},
      {"name": "Plasmid B"}
    ]
  }
}
```

- `product_quantity` must equal `samples` array length
- `length` (bp) is optional but improves QC
- Always overwrites any existing cart

**Cart response fields:** `id`, `product_id`, `product_quantity`, `product_unit_price_gbp_p`, `subtotal_gross_gbp_p`, `total_gross_gbp_p`, `vat_rate`, `items`

---

### Orders

| Method | Path | Description |
|--------|------|-------------|
| GET | `/orders` | List orders (latest first) |
| POST | `/orders` | Place order from current cart |
| GET | `/orders/{order_number}` | Get order status and results |
| GET | `/orders/{order_number}/fasta` | Get assembled sequences |

**GET /orders query params:**
- `cursor` — pagination cursor from previous `next` field
- `fulfillment_status` — comma-separated: `unfulfilled,partially_fulfilled,fulfilled,on_hold,processing,canceled`
- `payment_virtual_card` — filter by card id
- `number` — partial order number match

**POST /orders request body:**

```json
{
  "cart_id": "CART12",
  "payment_method": "virtual_card",
  "payment_virtual_card_id": "CQ8CPF",
  "dropoff_point_id": null,
  "bag_code": null
}
```

Cart is deleted on success.

**Order fulfillment statuses:** `on_hold` → `unfulfilled` → `partially_fulfilled` / `fulfilled` (or `canceled`)

**`results_download_url`** is populated once status reaches `fulfilled` or `partially_fulfilled`. URL expires after 24 hours.

**GET /orders/{number}/fasta** — only available when `fulfilled` or `partially_fulfilled`:

Always use `curl -L` when fetching this endpoint.

```json
[
  {"sample_name": "Plasmid A", "length": 8415, "sequence": "TAAACA..."},
  {"sample_name": "Plasmid B", "length": 8414, "sequence": "ATGAAA..."}
]
```

---

## Error codes (HTTP 409)

| code | meaning |
|------|---------|
| `account_not_found` | API key not linked to an account |
| `product_not_found` | Unknown product code or id |
| `cart_not_found` | No active cart |
| `cart_id_conflict` | `cart_id` in order doesn't match current cart |
| `virtual_card_not_found` | Card id not found or not accessible |
| `insufficient_balance` | Card balance too low |
| `missing_billing_address` | Account has no billing address |
| `order_not_found` | Order number doesn't exist |
| `invalid_virtual_card_payment` | Tried to order VC001 via API (top up through website) |

## Authentication errors (HTTP 401)

Missing, invalid, or wrong-audience API key. Check `FCL_API_KEY` is set and correct.

## Pagination

All list endpoints return `{"items": [...], "next": "<cursor> | null"}`. Pass `next` as `cursor` on the next request to page through results.
