---
name: scavio-sec
description: Use when the user wants SEC EDGAR filings data - resolving a company to its CIK, listing filings, reading XBRL facts and concepts, or full-text searching filings for financial research.
---

# SEC EDGAR via Scavio

SEC EDGAR company resolution, filing lists, XBRL facts and concepts, and full-text filing search, as structured JSON.

## When to use

- Resolve a company or ticker to its CIK
- List a company's filings by form type
- Pull a specific XBRL concept across periods
- Pull all reported facts for a company
- Full-text search filings for a phrase
- Build financial research and monitoring pipelines

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `resolve_sec_company`
- `get_sec_company`
- `get_sec_filings`
- `get_sec_concept`
- `get_sec_facts`
- `search_sec_filings`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: sec`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/sec/lookup` | 1 | Resolve a company or ticker to a CIK |
| `POST /api/v1/sec/company` | 1 | A company's profile |
| `POST /api/v1/sec/filings` | 1 | A company's filings |
| `POST /api/v1/sec/concept` | 1 | One XBRL concept over time |
| `POST /api/v1/sec/facts` | 1 | All reported XBRL facts |
| `POST /api/v1/sec/search` | 1 | Full-text filing search |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/sec/lookup \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Almost everything is keyed on CIK. Resolve the ticker or name first - do not guess a CIK.
- This is US registrants only. For UK companies use the Companies House skill.
- XBRL concepts use us-gaap taxonomy names. Quote the concept and period alongside any figure so it can be checked.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/sec-edgar-search
