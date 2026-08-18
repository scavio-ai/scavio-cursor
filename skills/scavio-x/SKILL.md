---
name: scavio-x
description: Use when the user wants X (Twitter) data - searching tweets or people for a keyword, reading a tweet with its replies and retweeters, looking up a handle's profile, pulling a user's tweets, replies, media, followers or followings, or getting trending topics by country - for brand and competitor monitoring, sentiment work or social RAG pipelines via the Scavio API.
---

# X via Scavio

Search X, read a single tweet with its replies and retweeters, pull a user's profile plus their tweets, replies, media, followers and followings, and get trending topics by country. 11 endpoints, all structured JSON.

## When to use

- Search tweets or people on X for a topic or keyword
- Read a tweet's full details, its replies, or who retweeted it
- Look up a user's profile by handle
- Pull a user's tweets, replies, or media
- List a user's followers or the accounts they follow
- Get trending topics for a country
- Monitor brand, competitor, or campaign mentions
- Build RAG or sentiment pipelines that need social context

## Using the MCP tools

If the Scavio MCP server is connected (`https://mcp.scavio.dev/mcp`, streamable http, auth header `x-api-key`, tool allowlist header `x-scavio-platforms`), prefer these tools over raw HTTP - they carry the auth and the parameter schemas already:

`search_x`, `get_tweet`, `get_tweet_comments`, `get_tweet_retweeters`, `get_x_user`, `get_x_user_tweets`, `get_x_user_replies`, `get_x_user_media`, `get_x_user_followers`, `get_x_user_followings`, `get_x_trending`

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`; every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/x/search` | 1 | Search tweets and people |
| `POST /api/v1/x/tweet` | 1 | Full details for a single tweet |
| `POST /api/v1/x/tweet/comments` | 1 | Replies to a tweet (ranked or chronological) |
| `POST /api/v1/x/tweet/retweeters` | 1 | Users who retweeted a tweet |
| `POST /api/v1/x/user` | 1 | Profile details for a user |
| `POST /api/v1/x/user/tweets` | 1 | A user's tweets |
| `POST /api/v1/x/user/replies` | 1 | A user's tweets and replies |
| `POST /api/v1/x/user/media` | 1 | A user's media tweets |
| `POST /api/v1/x/user/followers` | 1 | A user's followers |
| `POST /api/v1/x/user/followings` | 1 | Accounts a user follows |
| `POST /api/v1/x/trending` | 1 | Trending topics for a country |

Key parameters. `/search` takes `search` (required, 1-500 chars), `search_type` (`Top` default, `Latest`, `People`, `Photos`, `Videos`) and `cursor`. `/tweet`, `/tweet/comments` and `/tweet/retweeters` take `tweet_id`; comments also take `rank` (`top` default, or `latest`). The six user endpoints take `screen_name`, a handle without `@`. `/trending` takes `country` (default `UnitedStates`). Paginated endpoints return `next_cursor` - pass it back as `cursor` and stop when it is `null`.

```bash
curl -X POST 'https://api.scavio.dev/api/v1/x/search' \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"search": "AI agents", "search_type": "Latest"}'
```

Responses use the envelope `{ data, response_time, credits_used, credits_remaining }`. Search and the feed endpoints return `timeline[]` with `tweet_id`, `screen_name`, `text`, `created_at`, `favorites`, `retweets`, `replies`, `quotes`, `views` plus `next_cursor`; `/user` returns `rest_id`, `screen_name`, `name`, `description`, `followers_count`, `friends_count`, `blue_verified`; `/tweet/retweeters` returns `retweeters[]`; `/user/followers` returns `followers[]` and `/user/followings` returns `following[]`; `/trending` returns `trends[]`.

## Notes

- The search field is `search`, not `query` - this differs from some other Scavio endpoints.
- `screen_name` is a handle without the leading `@`.
- Every X endpoint costs 1 credit. Tell the user the cost before paginating through many pages.
- Never fabricate tweet ids, handles, metrics or replies, and surface engagement counts exactly as returned - do not round or invent them.
- Errors: 400 is an invalid or missing parameter (usually no `tweet_id` or `screen_name`); 401 is a bad or missing key; 429 is a rate or usage limit, so wait before retrying (see https://scavio.dev/docs/rate-limits); 502 and 503 are transient upstream failures worth a retry after a few seconds.
- If a search returns nothing, suggest different keywords or another `search_type` rather than retrying the same call.
