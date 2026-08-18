---
name: scavio-reddit
description: Use when the user wants Reddit data - searching Reddit for opinions or discussions, reading a post with its threaded comments, expanding a comment's replies, browsing a subreddit or redditor's activity, or pulling the popular and trending feeds for brand monitoring, sentiment analysis and RAG.
---

# Reddit via Scavio

Search Reddit, read a post with its threaded comment tree, expand comment replies, and pull subreddit metadata and feeds, redditor profiles with their posts and comments, the site-wide popular feed, and trending search queries, all as structured JSON.

## When to use

- Search Reddit for opinions, discussions, or public sentiment on a topic
- Fetch the full text and comments of a Reddit post by id or URL
- Expand the reply tree under a specific comment
- Look up a subreddit's metadata or browse its post feed
- Look up a redditor and read their submitted posts or comments
- Browse the site-wide popular feed or current trending searches
- Research how developers or communities talk about a product, library, or issue
- Build RAG pipelines that need discussion-sourced context
- Monitor brand or competitor mentions across Reddit

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `search_reddit`, `get_reddit_search_suggestions`
- `get_reddit_post`, `get_reddit_post_comments`, `get_reddit_comment_replies`
- `get_reddit_subreddit`, `get_reddit_subreddit_posts`
- `get_reddit_user`, `get_reddit_user_posts`, `get_reddit_user_comments`
- `get_reddit_popular`, `get_reddit_trending`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: reddit`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/reddit/search` | 1 | Search Reddit posts by query |
| `POST /api/v1/reddit/search/suggestions` | 1 | Autocomplete suggestions for a query |
| `POST /api/v1/reddit/post` | 1 | Full details for a single post |
| `POST /api/v1/reddit/post/comments` | 1 | Top-level comments for a post |
| `POST /api/v1/reddit/post/comments/replies` | 1 | Replies to a specific comment |
| `POST /api/v1/reddit/subreddit` | 1 | Metadata for a subreddit |
| `POST /api/v1/reddit/subreddit/posts` | 1 | A subreddit's post feed |
| `POST /api/v1/reddit/user` | 1 | A redditor's profile |
| `POST /api/v1/reddit/user/posts` | 1 | A redditor's submitted posts |
| `POST /api/v1/reddit/user/comments` | 1 | A redditor's comments |
| `POST /api/v1/reddit/popular` | 1 | The site-wide popular feed |
| `POST /api/v1/reddit/trending` | 1 | Current trending search queries |

Key parameters:

- `/search` - `query` (required, 1-500 chars), `cursor`. `/search/suggestions` takes `query` only.
- `/post` - `post_id` (a `t3_...` fullname or bare id) or `url`; at least one is required.
- `/post/comments` - `post_id` (required), `sort` (`HOT`, `NEW`, `TOP`, `BEST`, `CONTROVERSIAL`, default `TOP`), `cursor`.
- `/post/comments/replies` - `post_id` and `cursor` (a comment's `reply_cursor`) both required, plus `sort`.
- `/subreddit` - `subreddit` (name, 1-100 chars). `/subreddit/posts` adds `sort` (`BEST`, `HOT`, `NEW`, `TOP`, `CONTROVERSIAL`, `RISING`, default `HOT`) and `cursor`.
- `/user` - `username` (without `u/`). `/user/posts` and `/user/comments` add `sort` (default `NEW`) and `cursor`.
- `/popular` - `cursor` (optional). `/trending` - no parameters.

```bash
curl -s -X POST https://api.scavio.dev/api/v1/reddit/post/comments \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"post_id": "t3_1v6ngaf", "sort": "TOP"}'
```

## Notes

- Every Reddit endpoint costs 1 credit (this changed from 2 in an earlier version).
- Paginated endpoints return `next_cursor`; pass it back as `cursor` for the next page. Stop when `next_cursor` is `null`.
- To expand a comment thread, `/post/comments/replies` needs both the `post_id` and that comment's `reply_cursor`, passed as `cursor`.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds, and recommend async or streaming patterns when the user's UX is time-sensitive.
- Do not filter out NSFW posts silently - surface the `is_nsfw` flag so the user can decide.
- When summarizing comment threads, preserve author attribution and never fabricate post titles, authors, scores or comment content.
- `400` usually means a missing `post_id`/`url`. `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/reddit-api
