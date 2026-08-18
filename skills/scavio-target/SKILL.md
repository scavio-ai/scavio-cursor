---
name: scavio-target
description: Use when the user wants Target.com product data - searching the catalogue, browsing a category, reading a product, or pulling its customer reviews.
---

# Target via Scavio

Target.com search, category browsing, product detail and customer reviews, as structured JSON.

## When to use

- Search Target for products with prices and ratings
- Browse a Target category
- Read full product detail
- Pull customer reviews for a product
- Compare Target pricing against other retailers

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_target_products`
- `get_target_category`
- `get_target_product`
- `get_target_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: target`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/target/search` | 1 | Search products |
| `POST /api/v1/target/category` | 1 | Browse a category |
| `POST /api/v1/target/product` | 1 | Full product detail |
| `POST /api/v1/target/reviews` | 1 | Customer reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/target/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Target is reachable only through a headless-browser fetch upstream, so calls are slower than a plain scrape. Set a generous client timeout.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/target-search
