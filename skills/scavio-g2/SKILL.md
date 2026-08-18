---
name: scavio-g2
description: Use when the user wants G2 software-review data - searching the software catalogue, reading a product's profile, or pulling its verified user reviews for B2B vendor research.
---

# G2 via Scavio

G2 software search, product profiles and verified user reviews, as structured JSON.

## When to use

- Search G2's software catalogue
- Read a product's G2 profile and ratings
- Pull verified user reviews
- Research B2B software competitors
- Analyse what buyers say about a category

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_g2_software`
- `get_g2_product`
- `get_g2_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: g2`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/g2/search` | 5 | Search software |
| `POST /api/v1/g2/product` | 5 | A product profile |
| `POST /api/v1/g2/reviews` | 5 | Verified user reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/g2/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- G2 is the most expensive platform here at 5 credits per call - it sits behind a heavy upstream proxy profile. Do not loop it casually; fetch what you need.
- Sort options on search are relevance, popular and alphabetical.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/g2-search
