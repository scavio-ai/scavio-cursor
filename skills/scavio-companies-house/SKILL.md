---
name: scavio-companies-house
description: Use when the user wants UK Companies House data - searching companies, reading a company's registration, or listing its officers and filing history for UK corporate due diligence.
---

# Companies House via Scavio

UK Companies House company search, registration detail, officers and filing history, as structured JSON.

## When to use

- Search UK companies by name or number
- Read a company's registration, status and address
- List a company's officers and their appointments
- Walk a company's filing history
- Run UK corporate due diligence

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_companies_house`
- `get_companies_house_company`
- `get_companies_house_officers`
- `get_companies_house_filing_history`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: companieshouse`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/companieshouse/search` | 1 | Search companies |
| `POST /api/v1/companieshouse/company` | 1 | A company's registration |
| `POST /api/v1/companieshouse/officers` | 1 | A company's officers |
| `POST /api/v1/companieshouse/filing-history` | 1 | Filing history |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/companieshouse/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- UK registrants only. For US filings use the SEC EDGAR skill.
- Most calls take the company number, not the name. Search first to get it.
- Officer records include appointment and resignation dates - do not present a resigned officer as current.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/companies-house-search
