---
name: Onboard a brand product catalog to Maison Safqa
description: Authenticate and create products (single or bulk) in draft status on the Maison Safqa platform for sync to Shopify.
api: openapi/maison-safqa-holdings-limited-openapi-original.yml
operations:
- POST /products
- POST /products/bulk
- GET /products/{product_id}
---

# Onboard a brand product catalog

Use this skill to push a brand's product catalog into the Maison Safqa platform. Products are created
in `draft` status and are synced to Shopify only when the Maison Safqa team activates them.

## Auth
- Send every request with header `X-MaisonSafqa-Api-Key: <key>`.
- Use an `ms_test_...` key against `https://api.maisonsafqa.com/v1` while developing; switch to
  `ms_live_...` for production. Keys are environment-specific and are shown only once at creation.

## Steps
1. **Create a single product** — `POST /products` with a JSON body containing `title` (required) and
   a `variants[]` array (at least one; each variant needs `title`, `sku`, `price`,
   `inventory_quantity`). SKUs must be unique within your brand. On success you get `201` and a
   `product.id` in the `product_XXXXXXXXXXXXXXXX` format.
2. **Or create in bulk** — `POST /products/bulk` with `{ "products": [ ... ] }`, up to 500 items.
   Partial success is supported: a `207` response returns `created[]` and `failed[]`, where each
   failure carries its `index`, `title`, and `errors[]`. Re-submit only the failed items.
3. **Verify** — `GET /products/{product_id}` to confirm status and the `synced` flag.

## Rules
- Respect the rate limit: 300 requests/minute. On `429`, wait the `Retry-After` seconds before
  retrying.
- There is **no idempotency key**; a duplicate SKU returns `400` rather than de-duplicating, so do
  not blindly retry a create that may have partially succeeded — reconcile via `GET` first.
- Errors use `{ "error": { "field", "message" } }`; read `error.field` to locate the bad input.
