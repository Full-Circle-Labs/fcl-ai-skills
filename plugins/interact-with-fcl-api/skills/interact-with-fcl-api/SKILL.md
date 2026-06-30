---
name: fcl-api
description: Place sequencing orders, check status, and retrieve results via the Full Circle Labs (FCL) API. Use when asked to order sequencing, submit samples, check order status, retrieve FASTA results, manage carts, or interact with the FCL API programmatically.
---

# FCL API

## API key

Before making any calls, check whether `FCL_API_KEY` is available:

```bash
echo $FCL_API_KEY
```

**If empty or unset**, do not ask the user to paste the key into the chat. Instead, tell them to add it to Claude Code's environment settings so it is available to the agent securely:

1. Open `.claude/settings.json` (or run `/config`)
2. Add under the `"env"` key:
   ```json
   { "env": { "FCL_API_KEY": "their-key-here" } }
   ```
3. Restart the session, then retry.

Once set, all requests use:

```
Authorization: Bearer $FCL_API_KEY
```

Base URL: `https://api.fcl.bio/v1`

## Typical workflow

```
1. GET /products              → find product_code and current price
2. GET /virtual_cards         → find card id with enough balance
3. POST /cart                 → create cart with samples
4. POST /orders               → place order (cart is consumed)
5. GET /orders/{number}       → poll until fulfilled/partially_fulfilled
6. GET /orders/{number}/fasta → retrieve assembled sequences
```

## Quick example (Python)

```python
import os, requests, time

BASE = "https://api.fcl.bio/v1"
H = {"Authorization": f"Bearer {os.environ['FCL_API_KEY']}"}

# 1. Pick a product
products = requests.get(f"{BASE}/products", headers=H).json()["items"]

# 2. Pick a virtual card
card = requests.get(f"{BASE}/virtual_cards", headers=H).json()["items"][0]

# 3. Create cart
samples = [{"name": "Plasmid A", "length": 4500}, {"name": "Plasmid B"}]
cart = requests.post(f"{BASE}/cart", headers=H, json={
    "product_code": "FLPS001",
    "product_quantity": len(samples),
    "items": {"samples": samples}
}).json()

# 4. Place order
order = requests.post(f"{BASE}/orders", headers=H, json={
    "cart_id": cart["id"],
    "payment_method": "virtual_card",
    "payment_virtual_card_id": card["id"],
    "dropoff_point_id": None,
    "bag_code": None,
}).json()

# 5. Poll for results
while order["fulfillment_status"] not in ("fulfilled", "partially_fulfilled"):
    time.sleep(300)
    order = requests.get(f"{BASE}/orders/{order['number']}", headers=H).json()

# 6. Get sequences
fasta = requests.get(f"{BASE}/orders/{order['number']}/fasta", headers=H).json()
```

## Key constraints

- `product_quantity` **must** equal `len(samples)`
- Only `virtual_card` payment is accepted via API; top up at fcl.bio
- `VC001` (Virtual Card Credit) **cannot** be ordered via API
- `POST /cart` always overwrites any existing cart
- `dropoff_point_id: null` = standard postal delivery (use `/dropoff_points` for alternatives)
- Prices are in GBP pence — divide by 100 for £. Always fetch from `GET /products`; account-specific pricing is reflected in the cart total.

See [REFERENCE.md](REFERENCE.md) for full endpoint details and error codes.

