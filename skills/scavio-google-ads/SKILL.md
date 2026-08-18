---
name: scavio-google-ads
description: Use when the user wants Google's ad transparency data - resolving an advertiser, paging their entire ad library, or reading a single creative for competitor ad research.
---

# Google Ads Transparency via Scavio

Google Ads Transparency Center: advertiser resolution, ad library search and single creatives, as structured JSON.

## When to use

- Resolve an advertiser name to its transparency id
- Page an advertiser's entire ad library
- Read a single ad creative
- Research a competitor's live ad copy and formats
- Track when a competitor started or stopped running ads

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `resolve_google_ads_advertiser`
- `search_google_ads`
- `get_google_ads_creative`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: googleads`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/googleads/advertisers` | 1 | Resolve an advertiser to an id |
| `POST /api/v1/googleads/search` | 1 | Search an advertiser's ad library |
| `POST /api/v1/googleads/creative` | 1 | A single ad creative |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/googleads/advertisers \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Like every Google surface, this bills 1 credit per call.
- Search paginates by cursor, so an advertiser's full library can be walked. Resolve the advertiser id first.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/google-ads-search
