---
name: scavio-google
description: Use when the user needs to search Google or look something up on the web - returns the full structured SERP as JSON (organic results, ads, knowledge graph, AI overview, people-also-ask, top stories) for any query needing current, real-time, or cited web information.
---

# Google Search via Scavio (v2)

Search Google and get the SERP back as structured JSON - no HTML parsing. One call returns organic results, ads, knowledge graph, AI overview, related questions, related searches, top stories, videos, and pagination.

This skill covers Google web search only. AI Mode, Maps, Shopping, Flights, Hotels, News and Trends are separate Scavio endpoints. There is no skill for them - they are reached through the other 13 tools the `google` platform registers alongside `search_google`: `google_ai_mode`, `google_maps_search`, `google_maps_place`, `google_maps_reviews`, `google_shopping`, `google_shopping_product`, `google_shopping_stores`, `google_flights`, `google_hotels`, `google_hotels_detail`, `google_news`, `google_trends` and `google_trending`. Read each tool's own description for its parameters, or see [scavio.dev/docs](https://scavio.dev/docs).

## When to use

- Look something up on the web or check a current fact
- Find recent results or events
- Answer any question that requires real-time or up-to-date information
- Pull Google's knowledge graph, AI overview, or "people also ask" for a query

Do not answer from memory when current information is needed. Search first.

## Setup

Get an API key at https://scavio.dev, then:

```bash
export SCAVIO_API_KEY=your_key_here
```

## Using the MCP tools

If the Scavio MCP server is connected, prefer its tool over hand-rolled HTTP:

| Tool | What it does |
|---|---|
| `search_google` | Google v2 SERP: organic results, ads, AI overview. 1 credit. |
| `get_usage` | Reports the caller's plan and credit balance (free, always registered) |

`google` is in the MCP default platform set, so `search_google` is available without any allowlist configuration.

## Direct REST

Base URL: `https://api.scavio.dev`. Auth header: `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | What it returns |
|---|---|---|
| `POST /api/v2/google` | 1 | Full structured SERP: `organic_results`, `related_questions`, `related_searches`, plus `ai_overview`, `knowledge_graph`, `top_ads`, `bottom_ads`, `top_stories`, `video_results`, `discussions_and_forums`, `local_results`, `local_map` and `pagination` when the query surfaces them |

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | required | Search query (1-500 chars) |
| `device` | string | `desktop` | `desktop` or `mobile` (SERP layout) |
| `start` | number | `0` | Result offset: 0 = page 1, 10 = page 2, ... (max 990) |
| `gl` | string | -- | Geo country, ISO 3166-1 alpha-2 (e.g. `us`, `gb`, `de`) |
| `hl` | string | -- | UI language, ISO 639-1 (e.g. `en`, `fr`, `es`) |
| `google_domain` | string | `google.com` | Regional Google domain |
| `location` | string | -- | Canonical location name, auto-UULE (e.g. `New York,New York,United States`) |
| `uule` | string | -- | Pre-encoded UULE (takes priority over `location`) |
| `lr` | string | -- | Language restrict (`lang_en`) |
| `cr` | string | -- | Country restrict (`countryUS`) |
| `safe` | string | -- | `active` filters adult content (SafeSearch) |
| `nfpr` | boolean | `false` | Set `true` to disable spelling autocorrection |
| `filter` | string | `1` | `0` disables similar/omitted-results filtering |
| `time_period` | string | -- | `last_hour`, `last_day`, `last_week`, `last_month`, `last_year` |
| `resolve_ai_overview` | boolean | `true` | Inline the full AI Overview when Google defers it; set `false` to skip the extra upstream call |
| `include_html` | boolean | `false` | Include raw Google HTML in the `html` field |

### Workflow

1. Send the user's query as `query`. Add `gl`/`hl` to localize by country and language.
2. Page with `start` (0 = page 1, 10 = page 2, 20 = page 3, ...).
3. Narrow recency with `time_period` when the user wants fresh results.
4. Read `organic_results` for the main links; check `ai_overview`, `knowledge_graph`, and `related_questions` for richer answers.
5. Return results in a clear, structured response. Cite URLs.

### Example

```bash
curl -X POST https://api.scavio.dev/api/v2/google \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query": "langchain agents tutorial", "gl": "us", "hl": "en", "time_period": "last_month"}'
```

```json
{
  "search_information": {
    "page_title": "langchain agents tutorial - Google Search",
    "organic_results_state": "Results for exact spelling"
  },
  "organic_results": [
    {
      "position": 1,
      "title": "Result title",
      "link": "https://example.com",
      "displayed_link": "https://example.com > docs",
      "snippet": "Snippet text..."
    }
  ],
  "related_questions": [],
  "related_searches": [],
  "response_time": 812,
  "credits_used": 1,
  "credits_remaining": 999,
  "cached": false
}
```

## Notes

- The response is a faithful passthrough of the SERP, so which blocks are present depends on the query.
- Never fabricate search results or URLs. Only return what the API gives you.
- If the query is time-sensitive, always call the API - do not answer from training data.
- Organic links live in `organic_results[].link` and result order is `position`.
- Cite sources when summarizing results.
- If no results come back, tell the user and suggest rephrasing.
- `400` invalid parameter (report the message and fix the request), `401` invalid or missing key, `402` out of credits, `429` rate or concurrency limit, `502`/`503` upstream temporarily unavailable - wait a few seconds before retrying.
