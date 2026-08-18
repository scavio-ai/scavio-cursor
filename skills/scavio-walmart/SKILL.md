---
name: scavio-walmart
description: Use when the user wants Walmart data - searching walmart.com, walmart.ca or walmart.com.mx by keyword, reading a product by item id, pulling customer reviews and the rating breakdown, browsing a category, checking the buy-box seller, or looking up a marketplace seller's storefront and catalog.
---

# Walmart via Scavio

Search Walmart, read a product in full, page its customer reviews, list a category, check the buy-box offer on an item, and read a marketplace seller's storefront and catalog, all as structured JSON.

## When to use

- Search Walmart for products by keyword, price band or sort order
- Look up a Walmart item by its item id (usItemId)
- Read customer reviews and the rating breakdown for a Walmart product
- List the products inside a Walmart category
- Check who holds the buy box on a Walmart listing and at what price
- Look up a Walmart marketplace seller: rating, review count, Pro Seller badge
- See what a Walmart marketplace seller lists
- Compare Walmart pricing against another retailer
- Search the Canadian (walmart.ca) or Mexican (walmart.com.mx) marketplace

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_walmart` - keyword search
- `get_walmart_product` - full detail by item id
- `get_walmart_reviews` - reviews plus the rating breakdown
- `get_walmart_category` - products in a category, same shape as search
- `get_walmart_offers` - the buy-box seller for an item
- `get_walmart_seller` - marketplace seller storefront
- `get_walmart_seller_products` - a seller's catalog

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: walmart`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/walmart/search` | 1, or 2 when `domain` is `com.mx` | Keyword search: `products[]`, `products_count`, `location` |
| `POST /api/v1/walmart/product` | 1 | Full product detail by item id |
| `POST /api/v1/walmart/reviews` | 1 | Customer reviews plus the rating breakdown |
| `POST /api/v1/walmart/category` | 1, or 2 when `domain` is `com.mx` | Products in a category, same shape as search |
| `POST /api/v1/walmart/offers` | 1 | The buy-box seller for an item |
| `POST /api/v1/walmart/seller` | 1 | Marketplace seller storefront |
| `POST /api/v1/walmart/seller-products` | 1 | A seller's catalog (path is hyphenated) |

Key parameters:

- `/search` - `query` (required, 1-500 chars), `page`, `sort_by` (`best_match`, `price_low`, `price_high`, `best_seller`, `rating_high`, `new`), `min_price`, `max_price`, `fulfillment_speed` (`today` or `tomorrow`), `fulfillment_type` (`in_store`), `domain` (`com`, `ca`, `com.mx`). `start_page` is a deprecated alias for `page`.
- `/category` - `category_id` (required; leaf id `1095191` or full path `3944_133251_1095191`), plus `page`, `limit`, `sort_by`, `min_price`, `max_price`, `fulfillment_speed`, `domain`.
- `/product` and `/offers` - `product_id` (required), the usItemId.
- `/reviews` - `product_id` (required), `page` (10 per page), `sort` (`relevancy`, `submission-desc`, `submission-asc`, `rating-desc`, `rating-asc`, `helpful-desc`).
- `/seller` and `/seller-products` - `seller_id` (required), the NUMERIC catalog id from `seller_catalog_id`.

```bash
curl -s -X POST https://api.scavio.dev/api/v1/walmart/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "wireless headphones", "sort_by": "price_low", "max_price": 100}'
```

## Notes

- **Cost is a function of the request body, not a constant.** `domain` is the only price-bearing parameter: `com` (US, default) and `ca` cost 1 credit, `com.mx` costs 2. Only `/search` and `/category` accept `domain`, so only those two can ever cost 2. Read `credits_used` on the response rather than assuming a cost, and never quote a flat price for search or category without stating the domain rule.
- Search rows carry the item id as `id`, not `product_id` - that is the value the id-keyed endpoints take as `product_id`.
- `/offers` returns the BUY-BOX SELLER ONLY. It is not the full offer list and must never be described as one. If the user wants every seller on an item, say this API cannot enumerate them.
- `/seller-products` returns roughly the first 40 items, server-rendered, with no pagination. `total_count` reports the seller's real catalog size, so the two numbers disagree by design.
- `seller_id` must be the NUMERIC catalog seller id from `seller_catalog_id`. The GUID form of `seller_id` returns 404 - that is what a 404 on `/seller` or `/seller-products` almost always means.
- `domain` is accepted on `/search` and `/category` only. walmart.ca product pages could not be fetched in testing, so the id-keyed endpoints are US-only.
- `limit` on `/category` trims the response after fetching. It does not reduce the credit cost.
- `search`, `reviews` and `category` paginate with `page` (1-based). `product`, `offers`, `seller` and `seller-products` do not paginate at all.
- `sort_by`, `fulfillment_speed`, `fulfillment_type` and `domain` are closed enums - a value outside them is a `400`. To mean "anytime" for `fulfillment_speed`, omit the parameter entirely.
- Exactly three retired parameters answer with a `warnings[]` entry rather than an error: `device`, `delivery_zip` and `store_id`. A request carrying one looks successful but was silently unfiltered. `2_days` and `anytime` are NOT in that group - `fulfillment_speed` is a closed enum of `today` and `tomorrow`, so either value is a `400`; omit the parameter to mean "anytime". Results always come back against Walmart's default store, which is reported in `data.location`.
- Transient `502`s happen on Walmart. Wait a few seconds and retry once before reporting failure.
- Surface any `warnings[]` on a response - it means part of the request was ignored.
- Full reference: https://scavio.dev/docs/walmart-api
