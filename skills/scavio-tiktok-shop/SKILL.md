---
name: scavio-tiktok-shop
description: Use when the user wants TikTok Shop e-commerce data - searching products with exact prices, ratings and sold counts, reading product detail and variants, mining product reviews, browsing the category tree or a category's and a seller's catalog, or resolving a TikTok Shop link to a product or shop id.
---

# TikTok Shop via Scavio

Search the TikTok Shop catalog, read full product detail, pull paginated reviews with a star histogram, walk the global category tree, list a category's or a seller's products, and resolve any TikTok Shop URL or share link to a `product_id` / `shop_id`. All endpoints return structured JSON.

## When to use

- Search TikTok Shop for products by keyword, with prices, ratings and sold counts
- Get autocomplete or keyword expansion for a TikTok Shop search term
- Read a product's description, images, variants, stock, shipping and seller profile
- Pull a product's reviews, filter by rating or media, or get the star distribution
- Browse the TikTok Shop category tree or list products in a category
- List a seller's / shop's product catalog
- Turn a TikTok Shop link (canonical page, affiliate share link, `vt.tiktok.com` short link) into an id
- Do price research, competitor catalog tracking or review mining on TikTok Shop

This covers the shop side of TikTok. For creator profiles, videos, comments and hashtags use the `scavio-tiktok` skill. There is no video-to-product join: you cannot ask which products are attached to a TikTok video.

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP - they handle auth and the request envelope:

`search_tiktok_shop`, `get_tiktok_shop_search_suggestions`, `get_tiktok_shop_product`, `get_tiktok_shop_product_reviews`, `get_tiktok_shop_categories`, `get_tiktok_shop_category_products`, `get_tiktok_shop_shop_products`, `resolve_tiktok_shop_url`.

Hosted endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: tiktok-shop`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`. Every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/tiktok-shop/search` | 1 | Search products by keyword, with exact prices (US catalog) |
| `POST /api/v1/tiktok-shop/search/suggestions` | 1 | Keyword autocomplete and expansion, 8 regions |
| `POST /api/v1/tiktok-shop/product` | 1 | Full product detail (no price - see Notes) |
| `POST /api/v1/tiktok-shop/product/reviews` | 1 | Paginated reviews, up to 200 per call |
| `POST /api/v1/tiktok-shop/categories` | 1 | The global category tree (28 top-level, 240 nodes) |
| `POST /api/v1/tiktok-shop/category/products` | 1 | Products in a category, with exact prices |
| `POST /api/v1/tiktok-shop/shop/products` | 1 | A shop's catalog, 30 per page, with exact prices |
| `POST /api/v1/tiktok-shop/resolve` | 1 | Resolve a TikTok Shop URL to a `product_id` / `shop_id` |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/tiktok-shop/product/reviews \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"product_id": "1729508370969629931", "page": 1, "page_size": 200, "sort": "relevant"}'
```

Key parameters (regions are `US`, `GB`, `SG`, `MY`, `PH`, `TH`, `VN`, `ID` unless noted):

- `/search` - `search` (required, 1-200 chars), `cursor`. US catalog only, takes no `region`; there is no sort or price/rating filter.
- `/search/suggestions` - `search` (required, 1-100 chars), `region` (default `US`)
- `/product` - `product_id` (required, 6-25 digits), `region` (default `US`)
- `/product/reviews` - `product_id` (required), `page` (1-500, default 1), `page_size` (1-200, default 20), `sort` (`relevant` or `recent`), `rating` (1-5), `has_media`, `verified_only`, `region`
- `/categories` - no parameters, send `{}`
- `/category/products` - `category_id` (required, from `/categories`; level 1 or 2), `cursor`, `region` (`US` or `GB` only)
- `/shop/products` - `shop_id` (required, 6-25 digits), `cursor`, `region`
- `/resolve` - `url` (required, max 2000 chars)

Every response uses the envelope `{ data, response_time, credits_used, credits_remaining }`. `search`, `category/products` and `shop/products` return the same product card and are the only place exact prices appear. Paginate by passing `next_cursor` back as `cursor` until it is `null` or `has_more` is `false`.

## Notes

- `/product` returns no price: `price.current` and `price.original` are `null` there by design. Read the price from `/search`, `/shop/products` or `/category/products`, and never compute one from `discount_percent` or `savings`.
- A `404` from `/product` is expected for roughly 56% of ids taken from `/search` (measured: 11 of 25 resolved) and is a permanent upstream gap, not an error to retry or report. The 404 body is `error` plus the credit fields with no `data` key, so branch on the HTTP status. Skip the item; do not describe search to product as a dependable pipeline.
- `/product/reviews` is the fallback when detail 404s: on 8 measured unresolvable ids, 8 of 8 returned HTTP 200 and 7 of 8 carried at least one review. That is a small measured sample, not a guarantee - handle an empty `reviews[]`.
- Every endpoint costs 1 credit, including calls that answer `404`. Warn the user before looping detail lookups over a whole search page.
- Prefer one large `page_size` on reviews over many small pages: 200 rows and 20 rows both cost 1 credit.
- `sort: "recent"` is fresher but text-sparse (many stars-only rows); `sort: "relevant"` is text-complete and image-heavy.
- `has_media` and `verified_only` share one upstream filter slot - if both are `true`, `has_media` wins. Read `filters_applied` in the response for what was actually applied.
- Coverage is uneven: `search` is US-only and takes no `region`, `category/products` serves `US` and `GB` only, `categories` is global and takes no parameters. The category tree is two levels deep.
- Category listings are shallow: after a few pages `has_more` turns `false`, which is the end of the listing, not an error. Page size varies upstream (15-20), so always paginate with `next_cursor`.
- Cursors are opaque - pass `next_cursor` back verbatim, never decode, edit or reuse one across endpoints. A `400 "Invalid cursor."` means restart from page 1.
- Products repeat across search pages; dedupe by `product_id` when merging.
- `total_reviews` drifts between calls - page with `has_more`, never compute a page count from it.
- Reviewer names arrive pre-masked by TikTok (`"C**"`) - leave them. Reviewer avatars and review images are signed short-lived URLs - do not cache them as identifiers. Review text is genuinely multilingual, including in US reviews - do not strip or translate it silently.
- If `search` returns `"degraded": true`, the list is incomplete - say so rather than presenting it as the full result set.
- `400` on `/resolve` means an unsupported URL form and is not billed. `404` on `/category/products` or `/shop/products` means the id has no products - check the id rather than retrying. `502` means TikTok Shop is temporarily unavailable; the request already spent an internal retry budget, so retry slowly. Set a client timeout of at least 60 seconds.
- Never fabricate product titles, prices, ratings, sold counts, review text or shop names. If a field is `null`, say it is unavailable.
