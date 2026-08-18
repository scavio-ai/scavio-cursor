---
name: scavio-glassdoor
description: Use when the user wants Glassdoor data - resolving a company, reading its profile, or pulling employee reviews and salary reports for employer research.
---

# Glassdoor via Scavio

Glassdoor company resolution, company profiles, employee reviews and salary data, as structured JSON.

## When to use

- Resolve a company name to its Glassdoor id
- Read a company's profile and ratings
- Pull employee reviews
- Pull salary reports by role
- Research an employer's reputation and pay bands

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `resolve_glassdoor_company`
- `get_glassdoor_company`
- `get_glassdoor_reviews`
- `get_glassdoor_salaries`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: glassdoor`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/glassdoor/companies` | 1 | Resolve a company name to an id |
| `POST /api/v1/glassdoor/company` | 1 | A company profile |
| `POST /api/v1/glassdoor/reviews` | 1 | Employee reviews |
| `POST /api/v1/glassdoor/salaries` | 1 | Salary reports |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/glassdoor/companies \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Most calls need a Glassdoor company id. Resolve the name first rather than guessing.
- Salary figures are self-reported and skew by seniority and location. Present them as ranges with their sample size, never as a single authoritative number.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/glassdoor-companies
