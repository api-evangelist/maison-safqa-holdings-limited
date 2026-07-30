---
name: Sync inventory levels to Maison Safqa
description: Update variant inventory quantities for one product or many, and let the platform periodically sync to Shopify.
api: openapi/maison-safqa-holdings-limited-openapi-original.yml
operations:
- PATCH /products/{product_id}/inventory
- PATCH /inventory/bulk
---

# Sync inventory levels

Use this skill to keep a brand's on-hand quantities current on the Maison Safqa platform. Inventory
changes are stored locally and periodically synced to Shopify; **brand data always takes precedence
over Shopify** on conflict.

## Auth
- Header `X-MaisonSafqa-Api-Key: <key>` on every request (`ms_test_` in dev, `ms_live_` in prod).

## Steps
1. **Update one product** — `PATCH /products/{product_id}/inventory` with
   `{ "variants": [ { "sku": "...", "inventory_quantity": N } ] }`. The response returns a
   `results[]` array per SKU with `old_quantity` and `new_quantity`.
2. **Update many products** — `PATCH /inventory/bulk` with
   `{ "products": [ { "product_id": "...", "variants": [ ... ] } ] }`, up to 500 products. A `207`
   indicates partial success; inspect `results[].variants[].success`.

## Rules
- `inventory_quantity` must be a non-negative integer.
- Inventory endpoints are rate-limited (stricter than the default 300/min per the docs); honor
  `Retry-After` on `429`.
- A `404` means the `product_id` does not exist or does not belong to your brand — do not retry;
  fix the id.
- Errors use `{ "error": { "field", "message" } }`.
