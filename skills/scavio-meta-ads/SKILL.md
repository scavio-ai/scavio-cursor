---
name: scavio-meta-ads
description: Use when the user wants Meta's ad library data - searching ads, looking up an advertiser, or reading a single ad for Facebook and Instagram competitor research.
---

# Meta Ads Library via Scavio

Meta Ads Library: ad search, advertiser lookup and single ads, as structured JSON.

## When to use

- Search the Meta ad library by keyword or advertiser
- Look up an advertiser's page and ad activity
- Read a single ad's creative and run dates
- Research competitor campaigns on Facebook and Instagram

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_meta_ads`
- `get_meta_ads_advertiser`
- `get_meta_ad`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: metaads`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/meta-ads/search` | 1 | Search the ad library |
| `POST /api/v1/meta-ads/advertiser` | 1 | An advertiser's page |
| `POST /api/v1/meta-ads/ad` | 1 | A single ad |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/meta-ads/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Note the hyphen: these paths are `/api/v1/meta-ads/...`, not `/api/v1/metaads/...`. The MCP tool names use `meta_ads`.
- Political and issue ads carry more disclosure than commercial ones - field coverage differs by ad category.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/meta-ads-search
