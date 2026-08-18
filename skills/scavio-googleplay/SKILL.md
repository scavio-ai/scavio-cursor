---
name: scavio-googleplay
description: Use when the user wants Google Play data - searching apps, reading an app's listing and install counts, or pulling its user reviews.
---

# Google Play via Scavio

Google Play search, app detail and user reviews, as structured JSON.

## When to use

- Search Google Play for apps
- Read an app's listing, installs and ratings
- Pull user reviews
- Compare an app's Android and iOS reception

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_google_play`
- `get_google_play_app`
- `get_google_play_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: googleplay`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/googleplay/search` | 2 | Search apps |
| `POST /api/v1/googleplay/app` | 2 | An app's listing |
| `POST /api/v1/googleplay/reviews` | 2 | User reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/googleplay/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Google Play endpoint costs 2 credits.
- Play search returns roughly 30 server-rendered apps and does not paginate. Reviews do paginate.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/google-play-search
