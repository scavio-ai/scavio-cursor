---
name: scavio-tripadvisor
description: Use when the user wants Tripadvisor data - resolving a place to its location id, searching hotels, restaurants or attractions, reading a location, or pulling its reviews.
---

# Tripadvisor via Scavio

Tripadvisor location resolution, search, location detail and reviews, as structured JSON.

## When to use

- Resolve a place name to a Tripadvisor location id
- Search hotels, restaurants or attractions in a destination
- Read a location's detail and ranking
- Pull traveller reviews
- Compare hospitality reputation across sites

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `resolve_tripadvisor_location`
- `search_tripadvisor`
- `get_tripadvisor_location`
- `get_tripadvisor_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: tripadvisor`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/tripadvisor/locations` | 2 | Resolve a place to a location id |
| `POST /api/v1/tripadvisor/search` | 2 | Search a destination |
| `POST /api/v1/tripadvisor/location` | 2 | A location's detail |
| `POST /api/v1/tripadvisor/reviews` | 2 | Traveller reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/tripadvisor/locations \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Tripadvisor endpoint costs 2 credits.
- Most calls need a location id, not a place name. Resolve first, then query - do not guess ids.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/tripadvisor-search
