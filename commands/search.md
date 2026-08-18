---
name: search
description: Search Google through Scavio and summarize the results
---

Run a Google search through Scavio for: $ARGUMENTS

If the user gave no query, ask for one before calling anything.

Prefer the `search_google` tool from the scavio MCP server when that server is connected. Pass the whole string as `query`. Add optional parameters only when the user's request implies them: `gl` (country, e.g. `us`), `hl` (UI language), `location` (canonical location name), `start` (0 = page 1, 10 = page 2), `time_period` (`last_hour`, `last_day`, `last_week`, `last_month`, `last_year`), `device`, `safe`.

If the scavio MCP server is not connected, fall back to a direct call (JSON-escape the query before embedding it):

```
curl -s -X POST https://api.scavio.dev/api/v2/google \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"query":"<query>"}'
```

Then summarize the response for the user:

- Walk `organic_results` and list the top results, each as title, `link`, and a one-line takeaway from `snippet`. Do not paste raw JSON.
- If `ai_overview`, `related_questions`, `top_stories` or `knowledge_graph` are present and relevant, fold their substance into the summary.
- Close with a short direct answer to what the user was actually asking, and note `credits_used` and `credits_remaining`.
- If the response body carries an `error` field, report it verbatim instead of guessing at results.
