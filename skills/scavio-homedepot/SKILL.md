---
name: scavio-homedepot
description: Use when the user wants Home Depot product data - searching the catalogue, reading a product with specifications and pricing, or pulling its customer reviews.
---

# Home Depot via Scavio

Home Depot search, product detail and customer reviews, as structured JSON.

## When to use

- Search Home Depot for products and prices
- Read a product's specifications and availability
- Pull customer reviews
- Compare hardware and building-supply pricing

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_home_depot`
- `get_home_depot_product`
- `get_home_depot_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: homedepot`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/homedepot/search` | 2 | Search products |
| `POST /api/v1/homedepot/product` | 2 | Full product detail |
| `POST /api/v1/homedepot/reviews` | 2 | Customer reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/homedepot/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Home Depot endpoint costs 2 credits.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/home-depot-search
