---
name: scavio-instagram
description: Use when the user wants public Instagram data - a profile's follower count or bio, a creator's posts, reels, tagged posts or active stories, a single post's detail, comment threads and replies, follower or following lists, or a keyword search for accounts and hashtags - for influencer vetting, competitor tracking or creator discovery via the Scavio API.
---

# Instagram via Scavio

Public Instagram data as JSON: profiles, timeline posts, reels, tagged posts, active stories, single-post detail, comment threads and replies, follower and following lists, and user/hashtag search. 12 endpoints.

## When to use

- Look up an Instagram profile: follower count, bio, verified status, account type
- Pull a creator's recent posts, reels, or the posts they were tagged in
- Read one post in full: caption, media URLs, like and comment counts
- Mine the comments on a post, or the replies under one comment
- List who follows an account, or who it follows
- Find accounts or hashtags by keyword
- Influencer vetting, competitor tracking, creator-discovery research

## Using the MCP tools

If the Scavio MCP server is connected (`https://mcp.scavio.dev/mcp`, streamable http, auth header `x-api-key`, tool allowlist header `x-scavio-platforms`), prefer these tools over raw HTTP - they carry the auth and the parameter schemas already:

`get_instagram_profile`, `get_instagram_user_posts`, `get_instagram_user_reels`, `get_instagram_user_tagged`, `get_instagram_user_stories`, `get_instagram_post`, `get_instagram_post_comments`, `get_instagram_comment_replies`, `search_instagram_users`, `search_instagram_hashtags`, `get_instagram_user_followers`, `get_instagram_user_followings`

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`; every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/instagram/user/posts` | 2 | Timeline posts for a user. The cheap one - start here |
| `POST /api/v1/instagram/post` | 8 | Full detail for one post |
| `POST /api/v1/instagram/post/comments/replies` | 8 | Replies under one comment |
| `POST /api/v1/instagram/profile` | 10 | Profile: bio, counts, verified, account type |
| `POST /api/v1/instagram/user/reels` | 10 | A user's reels |
| `POST /api/v1/instagram/user/tagged` | 10 | Posts a user was tagged in |
| `POST /api/v1/instagram/user/stories` | 10 | Currently active stories |
| `POST /api/v1/instagram/post/comments` | 10 | Top-level comments on a post |
| `POST /api/v1/instagram/user/followers` | 10 | Who follows this account |
| `POST /api/v1/instagram/user/followings` | 10 | Who this account follows |
| `POST /api/v1/instagram/search/users` | 10 | Find accounts by keyword |
| `POST /api/v1/instagram/search/hashtags` | 10 | Find hashtags by keyword |

Key parameters. User endpoints take `username` (no `@`) or `user_id` (a string) - at least one is required, and `user_id` wins when both are sent. Feed endpoints also take `count` (default 12, 1-50; followers/followings allow up to 100) and `cursor`. `/post` takes `url`, `media_id` or `shortcode`. `/post/comments` takes `shortcode` or `url` plus `cursor` and `sort_order` (`popular` or `newest`). `/post/comments/replies` requires `media_id` and `comment_id`. The two search endpoints take `keyword` (not `query`) plus `cursor`, and have no `count`.

```bash
curl -X POST 'https://api.scavio.dev/api/v1/instagram/user/posts' \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"username": "instagram", "count": 24}'
```

## Notes

- No Instagram endpoint costs 1 credit - the range is 2 to 10 per call, so this is one of the pricier families in the API. State the credit cost to the user before starting any multi-call workflow, and prefer `/user/posts` (2 credits), which already carries captions, media and engagement counts for recent posts. Add `/profile` (10) only when follower counts, bio text, verified status or account type are actually needed.
- The three post endpoints take three non-overlapping identity sets: `/post` accepts `url`, `media_id` or `shortcode`; `/post/comments` accepts only `shortcode` or `url`; `/post/comments/replies` requires `media_id` plus `comment_id`. You cannot chain comments straight into replies - resolve `media_id` with `/post` first. That chain is 8 + 10 + 8 = 26 credits.
- Responses are a raw upstream passthrough, not a normalized Scavio shape. Field names are Instagram's: video is at `video_versions[].url` (there is no `video_url`), the cover is at `image_versions2.candidates[]` (there is no `thumbnail_url`), and `media_type` is the integer 1 (image), 2 (video) or 8 (carousel). Do not invent friendlier names when reporting results.
- Two upstream versions are raced and either may win, so top-level keys inside `data` vary between calls on the same endpoint. Probe defensively for `items` and for `data` and take whichever is present. On `/user/posts` with only `user_id`, items may arrive double-nested at `data.data`.
- `/profile` inlines the profile at the root of `data` - read `data.follower_count`, not `data.user.follower_count`. The post from `/post` is at `data.items[0]`; for a carousel, walk the children rather than expecting one media URL.
- Pagination field names vary by endpoint and by which upstream leg won: look for `next_max_id`, `pagination_token`, `next_min_id` or `rank_token` and pass whichever is present back as `cursor`. Stop when none is present or `has_more` / `more_available` is false. Raise `count` instead of making more calls - each call costs the same regardless of page size.
- `count` on `/user/posts` and `sort_order` on `/post/comments` are honored only when the newer upstream leg answers; treat both as requests, not guarantees. Search pagination is best-effort the same way - compare returned ids against what you already have rather than assuming forward progress.
- `/post/comments` needs a canonical post URL or a bare shortcode. A `/share/` link, a profile URL, or anything not of the form `instagram.com/p/<code>`, `/reel/<code>`, `/reels/<code>` or `/tv/<code>` fails as a 500, not a clean 400. Extract the shortcode and send that.
- Stories are not paginated and an account with no active stories returns an empty set - that is a normal result, not an error.
- Errors: 400 names the offending field in `details`; 401 is a bad key; 402 is out of credits (stop, do not retry); 429 means too many requests in flight (concurrency is capped per plan) so wait for one to finish; 502 is a transient upstream failure worth one retry. Every response carries `credits_used` and `credits_remaining`; `GET /api/v1/usage` checks the balance for 0 credits.
