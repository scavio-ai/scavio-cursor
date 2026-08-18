---
name: scavio-redfin
description: Use when the user wants Redfin real-estate data - searching listings, reading a property, or pulling market-level statistics for a region.
---

# Redfin via Scavio

Redfin listing search, property detail and regional market statistics, as structured JSON.

## When to use

- Search Redfin listings in a market
- Read a property's detail and history
- Pull market statistics for a region
- Compare housing inventory and pricing trends

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_redfin`
- `get_redfin_property`
- `get_redfin_market`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: redfin`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/redfin/search` | 1 | Search listings |
| `POST /api/v1/redfin/property` | 1 | A property's detail |
| `POST /api/v1/redfin/market` | 1 | Regional market statistics |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/redfin/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Redfin is US-only.
- The market endpoint is the one to reach for on 'is the market up or down' questions - do not try to infer a trend by averaging search results.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/redfin-search
