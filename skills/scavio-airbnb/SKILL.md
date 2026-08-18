---
name: scavio-airbnb
description: Use when the user wants Airbnb data - searching listings for a destination and dates, reading a listing's detail and amenities, or pulling its guest reviews.
---

# Airbnb via Scavio

Airbnb listing search, listing detail and guest reviews, as structured JSON.

## When to use

- Search Airbnb listings by destination, dates and guests
- Read a listing's detail, amenities and house rules
- Pull guest reviews and ratings
- Analyse short-let supply and pricing in a market

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_airbnb`
- `get_airbnb_listing`
- `get_airbnb_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: airbnb`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/airbnb/search` | 1 | Search listings |
| `POST /api/v1/airbnb/listing` | 1 | A listing's detail |
| `POST /api/v1/airbnb/reviews` | 1 | Guest reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/airbnb/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Nightly prices depend on the dates and guest count in the query. Do not present a price without the stay it was quoted for.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/airbnb-search
