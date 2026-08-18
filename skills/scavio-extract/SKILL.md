---
name: scavio-extract
description: Use when the user pastes a link or asks to read, summarize, quote, or convert a web page - fetches any http(s) URL through Scavio and returns it as clean Markdown, plain text, or raw HTML, including pages that block a plain HTTP request or render client-side.
---

# Extract any URL via Scavio

The read-a-page primitive: one URL in, page content out. Extract is not a platform, it is a core endpoint - no namespace, no per-site parser, no site-specific parameters. It works on any http(s) URL.

## When to use

- Read, summarize, or quote a specific web page
- Turn a URL into Markdown or plain text for an LLM prompt or a RAG chunk
- Pull an article, docs page, changelog, pricing page, or blog post into the conversation
- Fetch a page that blocked a plain HTTP request
- Grab the raw HTML of a page to parse it yourself

This is usually the right first tool whenever a user pastes a link and asks a question about what is on it.

## Setup

Get an API key at https://scavio.dev, then:

```bash
export SCAVIO_API_KEY=your_key_here
```

## Using the MCP tools

If the Scavio MCP server is connected, prefer its tool over hand-rolled HTTP:

| Tool | What it does |
|---|---|
| `extract_url` | Reads a URL and returns `{ url, format, mode, content, content_length }` |
| `get_usage` | Reports the caller's plan and credit balance (free, always registered) |

`extract` is in the MCP default platform set, so `extract_url` is available without any allowlist configuration.

## Direct REST

Base URL: `https://api.scavio.dev`. Auth header: `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | What it returns |
|---|---|---|
| `POST /api/v1/extract` | 1, 1 or 2 by `mode` | `{ url, format, mode, content, content_length }` |

No pagination - one URL, one response.

### Parameters

| Parameter | Type | Default | Description |
|---|---|---|---|
| `url` | string | required | The page to read (1-2048 chars). http(s) only; a bare host is upgraded to `https`. |
| `format` | string | `markdown` | `html`, `markdown`, `text` |
| `mode` | string | `normal` | `normal`, `advanced`, `ultra`. This is the price-bearing parameter. |

### Cost is a function of the body

| `mode` | What it does | Credits |
|---|---|---|
| `normal` (default) | Plain datacenter fetch | 1 |
| `advanced` | Renders JavaScript before reading | 1 |
| `ultra` | Heaviest fetch, for the hardest bot walls | 2 |

`advanced` costs the same as `normal`, so reach for it freely on any page that renders client-side. `ultra` is the only step that doubles the price - escalate to it only after `normal` or `advanced` came back empty or blocked.

### Formats

- `markdown` (default) - readability extraction: the article content, cleaned of navigation and chrome. This is what you want for an LLM prompt or a RAG chunk.
- `html` - the raw page exactly as fetched. Use it when you intend to parse the DOM yourself.
- `text` - that same readability Markdown flattened to plain text. The flattener is conservative about CommonMark, so `snake_case` identifiers, `__dunders__` and inline code survive intact.

### Example

```bash
curl -X POST https://api.scavio.dev/api/v1/extract \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/pricing", "format": "markdown", "mode": "normal"}'
```

Response envelope is `{ data, response_time, credits_used, credits_remaining }`, and `data` is:

```json
{
  "url": "https://example.com/pricing",
  "format": "markdown",
  "mode": "normal",
  "content": "# Pricing\n\nSimple, usage-based pricing...",
  "content_length": 4821
}
```

`url`, `format` and `mode` echo what was actually used, so you can log which tier paid for the result. `credits_used` tells you whether the call cost 1 or 2.

## Notes

- Billing is charge-on-success. Only a `2xx` extraction is billed - a dead link, a bot wall, or a timeout costs nothing, so escalating a failed fetch to a higher tier does not stack charges for the failures.
- Never quote a flat price for this endpoint. Price it by mode: 1 credit for `normal` and `advanced`, 2 for `ultra`.
- Start at `normal`. Escalate to `advanced` for a client-rendered page, and to `ultra` only when the cheaper tiers came back blocked or empty.
- Do not retry a failed fetch at the same tier more than once. Failures are free but usually deterministic; change the tier instead.
- Use `markdown` or `text` for anything going into a model prompt. `html` is for parsing, and it burns context.
- A short `content_length` on a `200` usually means a bot wall or a client-rendered shell, not an empty page. Escalate the tier before reporting the page as blank.
- Never invent page content. If the extraction is empty, say the page could not be read rather than answering from memory about what the URL probably says.
- Keep the source URL alongside any content you surface.
- This reads publicly reachable pages only. It does not log in, submit forms, or bypass a paywall.
- URL guard: `http` and `https` only. Loopback, private, link-local and cloud-metadata hosts are rejected with a `400` - a higher `mode` will not fix that.
- `400` bad or rejected URL (not billed), `401` invalid or missing key, `404` page does not exist upstream (not billed), `429` rate or concurrency limit, `402` out of credits, `502`/`503` fetch failed or upstream unavailable (not billed - retry once, then change tier).
