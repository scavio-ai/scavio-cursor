---
name: scavio-indeed
description: Use when the user wants Indeed jobs data - searching job listings, reading a single posting, or looking up a company's profile and employee reviews.
---

# Indeed via Scavio

Indeed job search, job detail, company profiles and employee reviews, as structured JSON.

## When to use

- Search job listings by title, location and filters
- Read a single job posting in full
- Look up a company's Indeed profile
- Pull employee reviews
- Track hiring activity at a company or in a market

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_indeed_jobs`
- `get_indeed_job`
- `get_indeed_company`
- `get_indeed_company_reviews`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: indeed`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/indeed/search` | 2 | Search job listings |
| `POST /api/v1/indeed/job` | 2 | A single posting |
| `POST /api/v1/indeed/company` | 2 | A company profile |
| `POST /api/v1/indeed/company/reviews` | 2 | Employee reviews |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/indeed/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Indeed endpoint costs 2 credits.
- Job listings expire. A posting that 404s is usually filled or withdrawn, not an error - report it as closed.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/indeed-search
