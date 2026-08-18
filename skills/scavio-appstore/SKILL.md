---
name: scavio-appstore
description: Use when the user wants Apple App Store data - searching apps, reading an app's listing and ratings, or pulling its user reviews.
---

# App Store via Scavio

Apple App Store search, app detail and user reviews, as structured JSON.

## When to use

- Search the App Store for apps
- Read an app's listing, ratings and version history
- Pull user reviews
- Track a competitor's app ratings and release cadence

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_app_store`
- `get_app_store_app`
- `get_app_store_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: appstore`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/appstore/search` | 1 | Search apps |
| `POST /api/v1/appstore/app` | 1 | An app's listing |
| `POST /api/v1/appstore/reviews` | 1 | User reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/appstore/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- App Store search cannot paginate - Apple returns a single page. Do not build a loop expecting more.
- Reviews do paginate.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/app-store-search
