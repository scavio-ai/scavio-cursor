---
name: scavio-yelp
description: Use when the user wants Yelp data - searching local businesses, reading a business's profile, or pulling its customer reviews for local-market and reputation research.
---

# Yelp via Scavio

Yelp business search, business detail and customer reviews, as structured JSON.

## When to use

- Search local businesses by term and location
- Read a business's profile, hours and categories
- Pull customer reviews and ratings
- Build local lead lists
- Track a local brand's reputation

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_yelp`
- `get_yelp_business`
- `get_yelp_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: yelp`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/yelp/search` | 2 | Search businesses |
| `POST /api/v1/yelp/business` | 2 | A business profile |
| `POST /api/v1/yelp/reviews` | 2 | Customer reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/yelp/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Yelp endpoint costs 2 credits.
- Yelp bot-walls aggressively. An empty payload under a 200 usually means the wall, not a business with no data - retry rather than reporting zero results.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/yelp-search
