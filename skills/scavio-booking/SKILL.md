---
name: scavio-booking
description: Use when the user wants Booking.com hotel data - searching hotels for dates and occupancy, reading a property, or pulling its guest reviews.
---

# Booking.com via Scavio

Booking.com hotel search, property detail and guest reviews, as structured JSON.

## When to use

- Search hotels by destination, dates and occupancy
- Read a property's detail, facilities and pricing
- Pull guest reviews and scores
- Compare accommodation pricing across sites

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_booking`
- `get_booking_hotel`
- `get_booking_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: booking`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/booking/search` | 1 | Search hotels |
| `POST /api/v1/booking/hotel` | 1 | A property's detail |
| `POST /api/v1/booking/reviews` | 1 | Guest reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/booking/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Prices are date- and occupancy-dependent. Always pass the check-in/check-out dates the user actually means; a price without its dates is meaningless.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/booking-search
