---
name: scavio-zillow
description: Use when the user wants Zillow real-estate data - searching listings in a market, reading a property's detail and price history, or pulling an agent's reviews.
---

# Zillow via Scavio

Zillow property search, property detail and agent reviews, as structured JSON.

## When to use

- Search for-sale or rental listings in a market
- Read a property's detail, price history and tax record
- Pull an agent's reviews
- Analyse residential pricing and inventory

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_zillow`
- `get_zillow_property`
- `get_zillow_agent_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: zillow`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/zillow/search` | 1 | Search listings |
| `POST /api/v1/zillow/property` | 1 | A property's detail |
| `POST /api/v1/zillow/reviews` | 1 | Agent reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/zillow/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Zillow is US-only. For a non-US market this returns nothing useful - say so rather than presenting an empty result as an answer.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/zillow-search
