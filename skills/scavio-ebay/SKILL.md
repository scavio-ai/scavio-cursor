---
name: scavio-ebay
description: Use when the user wants eBay data - searching live or SOLD listings for real transaction prices, reading a listing, or paging a seller's entire catalogue for price research and resale analysis.
---

# eBay via Scavio

Search live or completed eBay listings, read a single item, and look up a seller, as structured JSON.

## When to use

- Research what buyers actually paid, using sold listings
- Search current listings with price, condition and format filters
- Enumerate a seller's whole inventory
- Read one listing's full detail
- Price a used or collectible item from real transactions

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_ebay`
- `get_ebay_product`
- `get_ebay_seller`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: ebay`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/ebay/search` | 1 | Search live or sold listings |
| `POST /api/v1/ebay/product` | 1 | A single listing by item id |
| `POST /api/v1/ebay/seller` | 1 | A seller profile |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/ebay/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Set `sold=true` to search COMPLETED listings that actually sold. That is the reason to reach for eBay over a retail catalogue - it shows what buyers paid, not what sellers ask.
- On the sold view eBay publishes no headline count, so `total_results` is null. `count` still reports how many rows came back.
- Either `query` or `seller` is required. Passing `seller` alone with no keyword pages that seller's entire catalogue - the only way to enumerate inventory, since the seller endpoint is a profile card.
- `per_page` accepts only 60, 120 or 240; eBay silently falls back to 60 for anything else.
- An unrecognised `category_id` returns the UNFILTERED set under a 200 - verify the filter took effect.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/ebay-search
