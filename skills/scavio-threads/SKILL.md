---
name: scavio-threads
description: Use when the user wants Meta Threads data - a profile, a user's posts or replies, a single post with its comment tree, or searching for Threads users.
---

# Threads via Scavio

Public Meta Threads data as JSON: profiles, a user's posts and replies, a single post with its comments, and user search.

## When to use

- Look up a Threads profile and its follower counts
- Read a user's posts or their replies
- Pull a single post with its comment thread
- Find Threads users by name or handle
- Track how a brand or person posts on Threads

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `get_threads_profile`
- `get_threads_user_posts`
- `get_threads_user_replies`
- `get_threads_post`
- `get_threads_post_comments`
- `search_threads_users`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: threads`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/threads/profile` | 2 | A profile by handle or user id |
| `POST /api/v1/threads/user/posts` | 2 | A user's posts |
| `POST /api/v1/threads/user/replies` | 2 | A user's replies |
| `POST /api/v1/threads/post` | 2 | A single post |
| `POST /api/v1/threads/post/comments` | 2 | A post's comments |
| `POST /api/v1/threads/search/users` | 2 | Search Threads users |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/threads/profile \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Every Threads endpoint costs 2 credits.
- Upstream's username-keyed profile endpoint is dead, so a handle has to be resolved through search first - a second billable call. Endpoints that accept either key cost double when given a username. Pass a numeric user id when you have one and pay the base rate.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/threads-profile
