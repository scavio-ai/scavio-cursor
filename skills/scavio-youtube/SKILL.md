---
name: scavio-youtube
description: Use when the user wants YouTube data - searching videos, Shorts or channels, pulling video metadata, transcripts and captions, reading comments and replies, getting related videos or download stream URLs, or looking up a channel's videos, Shorts and community posts.
---

# YouTube via Scavio

Search YouTube and retrieve videos, Shorts, search suggestions, video metadata, comments, transcripts, related videos, download streams, and full channel data (info, videos, Shorts, community posts). All endpoints return structured JSON.

## When to use

- Find YouTube videos, Shorts, or channels on a topic
- Get a video's metadata, view count, duration, or captions list
- Read a video's comments and comment replies
- Pull a full transcript or timed subtitles for a video
- Get related videos or direct download stream URLs
- Look up a channel's info, videos, Shorts, or community posts
- Resolve a channel handle or URL to a channel ID

## Using the MCP tools

If the Scavio MCP server is connected, prefer these tools over raw HTTP - they handle auth and the request envelope:

`search_youtube`, `search_youtube_shorts`, `youtube_search_suggestions`, `get_youtube_video`, `get_youtube_metadata`, `get_youtube_comments`, `get_youtube_comment_replies`, `get_youtube_transcript`, `get_youtube_related`, `search_youtube_channels`, `get_youtube_channel`, `get_youtube_channel_videos`, `get_youtube_channel_shorts`, `get_youtube_channel_community`, `resolve_youtube_channel`, `get_youtube_streams`.

Hosted endpoint: `https://mcp.scavio.dev/mcp` (streamable http), auth header `x-api-key`, tool allowlist header `x-scavio-platforms: youtube`.

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`. Every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`.

| Endpoint | Credits | Description |
|---|---|---|
| `POST /api/v1/youtube/search` | 2 | Search videos (and channels/playlists) |
| `POST /api/v1/youtube/shorts` | 2 | Search Shorts |
| `POST /api/v1/youtube/suggestions` | 1 | Search autocomplete suggestions |
| `POST /api/v1/youtube/video` | 1 | Full video metadata and captions list |
| `POST /api/v1/youtube/comments` | 1 | Top-level comments for a video |
| `POST /api/v1/youtube/comments/replies` | 1 | Replies to a specific comment |
| `POST /api/v1/youtube/transcript` | 8 | Full transcript or timed subtitles |
| `POST /api/v1/youtube/related` | 1 | Videos related to a video |
| `POST /api/v1/youtube/streams` | 3 | Direct playable/downloadable stream URLs |
| `POST /api/v1/youtube/channel/search` | 1 | Search channels |
| `POST /api/v1/youtube/channel` | 1 | Full channel info |
| `POST /api/v1/youtube/channel/videos` | 1 | A channel's videos |
| `POST /api/v1/youtube/channel/shorts` | 1 | A channel's Shorts |
| `POST /api/v1/youtube/channel/community` | 1 | A channel's community posts |
| `POST /api/v1/youtube/channel/resolve` | 1 | Resolve a handle or URL to a channel ID |

`POST /api/v1/youtube/metadata` still works as a deprecated alias of `/video`. Use `/video` in new code.

```bash
curl -s -X POST https://api.scavio.dev/api/v1/youtube/search \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"search": "langchain tutorial", "type": "video", "sort_by": "view_count"}'
```

Key parameters:

- `/search` - `search` (required), `upload_date`, `type`, `duration`, `sort_by`, `features[]`, `cursor`
- `/shorts` - `search` (required), `sort_by`, `cursor`
- `/suggestions` - `search` (required), `language` (default `en`), `region` (default `US`)
- `/video`, `/streams` - `video_id` (required; accepts a raw ID or a full watch URL)
- `/comments`, `/related` - `video_id` (required), `cursor`
- `/comments/replies` - `video_id` (required), `reply_cursor` (required), `cursor`
- `/transcript` - `video_id` (required), `language` (default `en`), `format` (`text` or `srt`, default `text`)
- `/channel/search` - `search` (required), `cursor`
- `/channel` - `channel_id` (required; accepts an ID, `@handle`, or channel URL)
- `/channel/videos`, `/channel/shorts`, `/channel/community` - `channel_id` (required), `cursor`
- `/channel/resolve` - `channel` (required; a `@handle` or channel URL)

Every response uses the envelope `{ data, response_time, credits_used, credits_remaining }`. Paginated endpoints return `next_cursor` and `has_more`; pass `next_cursor` back as `cursor`.

## Notes

- The search parameter is `search`, not `query` - this differs from other Scavio endpoints.
- Credits are not uniform: `search` and `shorts` cost 2, `streams` costs 3, `transcript` costs 8, everything else costs 1. Warn the user before paginating heavily, especially transcripts.
- `/metadata` is a deprecated alias of `/video`.
- Stream URLs from `/streams` expire (`expires_in_seconds`) - use them promptly.
- Legacy boolean search flags (`subtitles`, `hd`, `4k`, `live`, ...) are still accepted, but prefer the `features` array.
- If a transcript is unavailable in the requested `language`, retry with `en` or check `/video` `captions[]` for what exists.
- `400` is an invalid parameter, `401` a bad or missing key, `429` a rate or usage limit, `502`/`503` a temporary upstream outage - wait a few seconds and retry.
- Never fabricate video IDs, view counts, transcripts, comments, or channel data. Only return API data.
