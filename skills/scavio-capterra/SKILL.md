---
name: scavio-capterra
description: Use when the user wants Capterra software-review data - searching software, reading a product's profile, or pulling its user reviews for B2B vendor research.
---

# Capterra via Scavio

Capterra software search, product profiles and user reviews, as structured JSON.

## When to use

- Search Capterra's software catalogue
- Read a product's profile and ratings
- Pull user reviews
- Compare B2B software options alongside G2

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_capterra_software`
- `get_capterra_product`
- `get_capterra_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: capterra`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/capterra/search` | 2 | Search software |
| `POST /api/v1/capterra/product` | 2 | A product profile |
| `POST /api/v1/capterra/reviews` | 2 | User reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/capterra/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Capterra endpoint costs 2 credits.
- Capterra and G2 cover overlapping catalogues with different reviewer bases - for a fair read on a vendor, check both.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/capterra-search
