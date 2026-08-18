---
name: scavio-tiktok
description: Use when the user wants TikTok data - looking up a creator's profile and follower count, searching videos or users by keyword, pulling a video's stats and comments, researching a hashtag's videos and view counts, or listing a creator's followers and followings.
---

# TikTok via Scavio

Search TikTok videos and users, look up profiles, read comments and replies, explore hashtags, and list followers/followings. Returns structured JSON with engagement stats, video metadata, and social graph data.

## When to use

- Look up a TikTok creator's profile, follower count, or bio
- Search TikTok for videos by keyword or topic, or search for users/creators
- Get details, stats, or engagement metrics for a specific TikTok video
- Read comments or replies on a TikTok video
- Explore a hashtag's stats or trending videos
- List a creator's followers or who they follow
- Analyze TikTok trends, influencer reach, or content performance

For the TikTok Shop catalog (products, prices, reviews, sellers) use the `scavio-tiktok-shop` skill instead.

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP - they handle auth and the request envelope:

`get_tiktok_profile`, `get_tiktok_user_posts`, `get_tiktok_video`, `get_tiktok_video_comments`, `get_tiktok_comment_replies`, `search_tiktok_videos`, `search_tiktok_users`, `get_tiktok_hashtag`, `get_tiktok_hashtag_videos`, `get_tiktok_user_followers`, `get_tiktok_user_followings`.

Hosted endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: tiktok`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`. Every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/tiktok/profile` | 1 | Get user profile by username or sec_user_id |
| `POST /api/v1/tiktok/user/posts` | 1 | List a user's videos (paginated, sortable) |
| `POST /api/v1/tiktok/video` | 1 | Get full details for a single video |
| `POST /api/v1/tiktok/video/comments` | 1 | List comments on a video |
| `POST /api/v1/tiktok/video/comments/replies` | 1 | List replies to a specific comment |
| `POST /api/v1/tiktok/search/videos` | 1 | Search videos by keyword |
| `POST /api/v1/tiktok/search/users` | 1 | Search users by keyword |
| `POST /api/v1/tiktok/hashtag` | 1 | Get hashtag details and stats |
| `POST /api/v1/tiktok/hashtag/videos` | 1 | List videos for a hashtag |
| `POST /api/v1/tiktok/user/followers` | 1 | List a user's followers |
| `POST /api/v1/tiktok/user/followings` | 1 | List accounts a user follows |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/tiktok/search/videos \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"keyword": "cooking recipe", "count": 10, "sort_type": "1", "publish_time": "7"}'
```

Key parameters:

- `/profile` - one of `username` (handle without @) or `sec_user_id`
- `/user/posts` - `sec_user_id` (required), `cursor` (default `"0"`), `count` (1-30, default 20), `sort_type` (`"0"` latest, `"1"` popular)
- `/video` - `video_id` (required)
- `/video/comments` - `video_id` (required), `cursor`, `count` (1-50)
- `/video/comments/replies` - `video_id` (required), `comment_id` (required, the `cid` from comments), `cursor`, `count` (1-50)
- `/search/videos` - `keyword` (required, 1-500 chars), `cursor`, `count` (1-30), `sort_type` (`"0"` relevance, `"1"` most likes), `publish_time` (`"0"`, `"1"`, `"7"`, `"30"`, `"90"`, `"180"`)
- `/search/users` - `keyword` (required, 1-500 chars), `cursor`, `count` (1-30)
- `/hashtag` - one of `hashtag_name` (without #) or `hashtag_id`
- `/hashtag/videos` - `hashtag_id` (required), `cursor`, `count` (1-30)
- `/user/followers`, `/user/followings` - `sec_user_id` (required), `count` (1-20), `page_token`, `min_time`

Every response uses the envelope `{ data, response_time, credits_used, credits_remaining }`.

Pagination differs by endpoint: `user/posts` pages with `cursor = data.max_cursor` and stops when `data.has_more === 0`; the search, hashtag/videos and comments endpoints page with `cursor = data.cursor` and stop when `data.has_more === 0`; followers/followings page with `page_token` plus `min_time` and stop when `data.has_more === false`.

## Notes

- Most endpoints need a `sec_user_id`, not a username. Resolve it first via `/tiktok/profile` and read `data.user.sec_uid`.
- All TikTok calls cost 1 credit each. Tell the user before paginating through many pages.
- `create_time` fields are Unix timestamps in seconds - multiply by 1000 for a JavaScript `Date`.
- Avatar and image fields return an object with a `url_list` array; use `.url_list[0]` for the URL.
- `401` is a bad or missing key, `429` a rate limit, `502`/`503` a temporary upstream outage - wait a few seconds and retry.
- If a search returns nothing, try different keywords, a broader `publish_time`, or a different `sort_type`.
- Never fabricate usernames, captions, follower counts, or comment text. Only return API data, and do not silently omit fields.
