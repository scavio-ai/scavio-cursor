---
name: extract
description: Pull a web page as clean markdown through the Scavio extract endpoint
---

Extract a web page through Scavio. Arguments: $ARGUMENTS

The first token is the URL. An optional second token is the fetch `mode`: `normal` (default), `advanced` (headless-browser rendering for JS-built pages), or `ultra` (residential proxy for hard anti-bot walls). If no URL was given, ask for one before calling anything.

Prefer the `extract_url` tool from the scavio MCP server when that server is connected. Call it with `url`, `format: "markdown"`, and `mode` only if the user named one.

If the scavio MCP server is not connected, fall back to a direct call:

```
curl -s -X POST https://api.scavio.dev/api/v1/extract \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url":"<url>","format":"markdown"}'
```

Handling the result:

- The markdown is in `data.content`; `data.url`, `data.format`, `data.mode` and `data.content_length` describe what came back.
- Start at `normal`. Only if `data.content` comes back empty, truncated, or is plainly a bot-wall or consent page, retry once with `mode: "advanced"`, then once with `mode: "ultra"`. Tell the user each time you escalate, and stop after `ultra`.
- Show the extracted markdown (or a faithful summary if it is long, saying so), then note `credits_used` and `credits_remaining`.
- Only http(s) URLs work; loopback and private hosts are rejected with a 400. Report any `error` field verbatim rather than retrying blindly.
