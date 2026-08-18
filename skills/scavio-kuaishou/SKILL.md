---
name: scavio-kuaishou
description: Use when the user wants Kuaishou (Kwai) data - a creator profile, their posts or live status, a video with its comments, hashtag feeds, trending, or searching videos, users and live rooms.
---

# Kuaishou via Scavio

Kuaishou (Kwai) profiles, videos, comments, hashtag feeds, trending and search, as structured JSON.

## When to use

- Look up a Kuaishou creator and their posts
- Check whether a creator is live
- Read a video with its comments and reply threads
- Batch-fetch several videos in one call
- Browse a hashtag feed or the trending list
- Search videos, users or live rooms

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP:

- `get_kuaishou_profile`
- `get_kuaishou_user_posts`
- `get_kuaishou_user_live`
- `resolve_kuaishou_user`
- `get_kuaishou_video`
- `get_kuaishou_video_comments`
- `get_kuaishou_comment_replies`
- `get_kuaishou_videos_batch`
- `search_kuaishou`
- `search_kuaishou_videos`
- `search_kuaishou_users`
- `search_kuaishou_live`
- `get_kuaishou_tag_feed`
- `get_kuaishou_trending`

Hosted MCP endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: kuaishou`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`, auth header `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is a `POST` with a JSON body.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/kuaishou/profile` | 1-40 | A creator profile |
| `POST /api/v1/kuaishou/user/posts` | 1-40 | A creator's posts |
| `POST /api/v1/kuaishou/user/live` | 1-40 | A creator's live status |
| `POST /api/v1/kuaishou/user/resolve` | 1-40 | Resolve a handle to a user id |
| `POST /api/v1/kuaishou/video` | 1-40 | A single video |
| `POST /api/v1/kuaishou/video/comments` | 1-40 | A video's comments |
| `POST /api/v1/kuaishou/video/sub-comments` | 1-40 | Replies under a comment |
| `POST /api/v1/kuaishou/videos/batch` | 1-40 | Several videos in one call |
| `POST /api/v1/kuaishou/search` | 1-40 | General search |
| `POST /api/v1/kuaishou/search/videos` | 1-40 | Video search |
| `POST /api/v1/kuaishou/search/users` | 1-40 | User search |
| `POST /api/v1/kuaishou/search/live` | 1-40 | Live-room search |
| `POST /api/v1/kuaishou/tag/feed` | 1-40 | A hashtag feed |
| `POST /api/v1/kuaishou/trending` | 1-40 | The trending feed |

```bash
curl -s -X POST https://api.scavio.dev/api/v1/kuaishou/profile \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Notes

- Cost varies sharply by endpoint - most are 1 credit, some 2, some 10, and the batch endpoint is 40. Check before looping.
- The batch endpoint is capped at 20 ids per call because a single request can otherwise fan out to an unbounded upstream spend.
- Kuaishou hides some failures inside an HTTP 200 - a response can carry a non-success result field. Treat a 200 with no payload as an error, not as an empty result.
- This is Kuaishou proper. Kwai International is a different product and is not covered.
- Requests take 5-15 seconds. Set a client timeout of at least 30 seconds.
- Never fabricate values. If a field is absent, say it is absent rather than filling it in.
- `502` / `503` mean upstream is temporarily unavailable - wait a few seconds before retrying.
- Full reference: https://scavio.dev/docs/kuaishou-profile
