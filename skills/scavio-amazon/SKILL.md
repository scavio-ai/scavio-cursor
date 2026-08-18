---
name: scavio-amazon
description: Use when the user wants Amazon product data - searching Amazon by keyword, looking up a product by ASIN for price, rating, specs, images or availability, comparing sellers and finding the buy-box winner, price or stock tracking, or competitor research across any of the 22 Amazon marketplaces.
---

# Amazon via Scavio

Keyword search, full product detail by ASIN, and every seller offer on an ASIN with the buy-box winner, as normalized JSON across 22 Amazon marketplaces.

## When to use

- Find products on Amazon by keyword, with price, rating and review count
- Look up an ASIN: price, availability, images, specifications, variants, shipping
- Compare sellers on one product, find the cheapest offer, or see who holds the buy box
- Track a price, check stock, or watch a listing over time
- Research a marketplace other than the US (Germany, UK, Japan, ...)
- Mine best sellers rank, sales volume or badges for competitor research

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_amazon` - keyword search, one page per call
- `get_amazon_product` - full detail for one ASIN
- `get_amazon_offers` - seller offers on one ASIN, with the buy-box winner
- `get_amazon_options` - the marketplace list, free, no arguments

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: amazon`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. All data endpoints are `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/amazon/search` | 1 | Keyword search: product cards, filters, related searches |
| `POST /api/v1/amazon/product` | 1 | Full detail for one ASIN |
| `POST /api/v1/amazon/offers` | 1 | Every seller offer on one ASIN, with the buy-box winner |
| `GET /api/v1/amazon/options` | 0 | The marketplace list. No API key, no credit. |

Key parameters:

- `/search` - `query` (required, 1-500 chars), `country` (default `us`), `page` (1-based). `domain` and `start_page` are deprecated aliases for `country` and `page`.
- `/product` and `/offers` - the ASIN in `query` or `asin` (same parameter, send one), plus `country`.

Marketplaces: `us` `gb` `de` `fr` `it` `es` `nl` `be` `se` `pl` `tr` `ca` `mx` `br` `jp` `cn` `sg` `in` `au` `ae` `sa` `eg`

```bash
curl -s -X POST https://api.scavio.dev/api/v1/amazon/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "wireless headphones", "country": "de", "page": 2}'
```

## Notes

- **There is no sort.** Amazon accepts every sort value and ignores all of them, so no `sort_by` parameter exists. If the user wants "cheapest first", sort the returned `products[]` yourself on `price` and say you sorted one page locally - never claim Amazon ranked them.
- `country` is a two-letter country code, not a domain suffix: amazon.com is `us` (not `com`) and amazon.co.uk is `gb` (not `uk`). A two-letter code that is not a real marketplace does not error - it quietly returns the US storefront, so a typo looks like a successful search of the wrong country. Validate against `/api/v1/amazon/options` before sending.
- Search `reviews_count` is rounded whenever Amazon abbreviated it (`"1.3K"` parses to `1300`); `/product` and `/offers` return the exact integer. Do not mix the two in one comparison.
- `position` on a search card is Amazon's own grid index including ad and carousel slots, so it is neither the array index nor contiguous. Use array order for "the Nth result". `is_sponsored` marks paid placements.
- `total_results` is a floor parsed from page text ("over 100,000 results" -> `100000`). Never divide it by a page size to compute a page count - page until `products[]` comes back short.
- `price` and `rating` can be `null` on a card Amazon rendered without them. Guard before arithmetic.
- `availability` is free text in the marketplace's language ("In Stock", "Nur noch 3 auf Lager"). There is no boolean in-stock flag - quote it, do not derive from it.
- `reviews[]` on a product is metadata only (id, author, date, verified flag). There is no review body and no per-review rating, so never claim to have read a review's text.
- On offers, `price` is the item price and `shipping_price` is separate - landed cost is the sum. `is_buy_box_winner` is not always the cheapest offer. `/offers` returns page 1 only; if `has_more_pages` is `true`, say so rather than presenting the list as every seller.
- `other_sellers_count` on the product response is the cheapest way to decide whether an `/offers` call is worth a credit.
- Prices are numbers in `currency`, which changes with `country`. Never mix currencies in one comparison and never convert between them.
- A `200` with a top-level `warnings` array means the request carried a retired parameter that was ignored - the response is not filtered the way the request implied, and the call is still billed. Retired: `sort_by`, `pages`, `category_id`, `merchant_id`, `language`, `currency`, `device`, `zip_code`, `autoselect_variant`.
- `502` (Amazon temporarily unavailable or the ASIN could not be fetched) and `503` (upstream fetch never completed) are not billed. Retry once after a short backoff.
- Calls typically take 3-5 seconds. Set a client timeout of at least 60 seconds.
- Full reference: https://scavio.dev/docs/amazon-api
