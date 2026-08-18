---
name: scavio-linkedin
description: Use when the user wants LinkedIn data - a member's profile, about section or post feed, what a member commented on or reacted to, a company profile or its posts, job listings by keyword and location, a single job listing, or a post with its comment thread - for B2B prospecting, recruiting or market research via the Scavio API.
---

# LinkedIn via Scavio

Look up LinkedIn people and companies, read their posts, search job listings, and read a single job or post with its comments. 9 endpoints, all structured JSON.

## When to use

- Look up a LinkedIn member's profile, about section, or recent posts
- See what a member engages with rather than what they publish - the posts they commented on or reacted to
- Look up a company profile or its recent posts
- Search for job listings by keyword and location
- Read a single job listing, or a single post with its comments
- Build B2B prospecting, recruiting, or market-research pipelines

## Using the MCP tools

If the Scavio MCP server is connected (`https://mcp.scavio.dev/mcp`, streamable http, auth header `x-api-key`, tool allowlist header `x-scavio-platforms`), prefer these tools over raw HTTP - they carry the auth and the parameter schemas already:

`get_linkedin_person`, `get_linkedin_person_about`, `get_linkedin_person_posts`, `get_linkedin_company`, `get_linkedin_company_posts`, `search_linkedin_jobs`, `get_linkedin_job`, `get_linkedin_post`, `get_linkedin_post_comments`

## Direct REST

Get an API key at https://scavio.dev, then `export SCAVIO_API_KEY=...`. Base URL `https://api.scavio.dev`; every call is a `POST` with a JSON body and `Authorization: Bearer $SCAVIO_API_KEY`. Cost is not uniform - check the table before planning a run.

| Endpoint | Credits | What it returns |
|---|---|---|
| `POST /api/v1/linkedin/person` | 1 | Full profile: about, experience, education, links |
| `POST /api/v1/linkedin/person/about` | 1 | Just the narrative sections of a profile |
| `POST /api/v1/linkedin/person/posts` | 10 per page | A member's posts, or the posts they commented on or reacted to. 50 per page, paginated |
| `POST /api/v1/linkedin/company` | 1 | Company profile, locations, related companies |
| `POST /api/v1/linkedin/company/posts` | 10 per page | A company's posts, 50 per page, paginated |
| `POST /api/v1/linkedin/search/jobs` | 10 per page | Job listings by keyword and location, 25 per page, paginated |
| `POST /api/v1/linkedin/job` | 30 | Full detail for one job listing |
| `POST /api/v1/linkedin/post` | 1 | Full detail for one post |
| `POST /api/v1/linkedin/post/comments` | 10 per page | Comments on a post, paginated by page number |

Key parameters. `/person` and `/person/about` take `username` (the vanity handle) or `url`. `/person/posts` adds `type` (`posts` default, `comments`, `reactions`) and `cursor`. `/company` and `/company/posts` take `company` (a slug) or `url`, plus `cursor` on the feed. `/search/jobs` takes `search` (required), optional `location`, and `cursor`. `/job` takes `job_id` or `url`. `/post` and `/post/comments` take `post_id` (bare id or activity urn) or `url`; comments also take a 1-based `page`. Every reference param also accepts a full LinkedIn URL - `{"username": "williamhgates"}` and `{"url": "https://www.linkedin.com/in/williamhgates/"}` are equivalent.

```bash
curl -X POST 'https://api.scavio.dev/api/v1/linkedin/search/jobs' \
  -H "Authorization: Bearer $SCAVIO_API_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"search": "software engineer", "location": "United States"}'
```

## Notes

- Cost varies sharply: the four single-object reads (`/person`, `/person/about`, `/company`, `/post`) are 1 credit, the four paginated endpoints are 10 credits per page, and `/job` is 30. A ten-page job search is 100 credits. Cap any paging loop and tell the user the spend before starting.
- `/job` is the most expensive call here, and `/search/jobs` already returns title, company, location, posted time, workplace type and salary for every hit. Only call `/job` for listings the user actually picked out - never in a loop over search results.
- `/person` already contains the about text, experience and education. Do not also call `/person/about` for the same member - it is the same upstream fetch billed twice.
- Cursor paging covers `/person/posts`, `/company/posts` and `/search/jobs`: while `has_more` is true, send `next_cursor` back verbatim as `cursor`. Cursors are opaque and endpoint-specific - do not parse, build or increment one, and do not carry a cursor across endpoints or across a changed `username`, `company`, `search` or `type`. An unreadable cursor does not error; it silently returns page one, which looks like a loop that never advances.
- `/post/comments` pages by 1-based `page` number, and only an empty page means the end. Page size varies (10, 10, 8 and 9 comments were observed on one thread) so a short page is not the last page, and `total` is returned on page 1 only - it is `null` afterwards. `has_more` means this page carried at least one comment; `next_page` carries the number to ask for.
- `/search/jobs` is not a snapshot: upstream rotates its result set, so pages overlap and two identical searches return different listings. Dedupe by job `id`, do not present the union as the complete set of matching jobs, and do not diff two runs to infer new postings.
- Five paths were retired upstream and always return HTTP 410 with `{"code": "endpoint_retired", "reason": ...}`, never billed: `/person/contact`, `/company/people`, `/company/jobs`, `/search/people`, `/search/posts`. The data is gone, not temporarily unavailable - do not retry. Closest substitutes: `/company` returns `featured_employees` (a sample of 4-6 staff) in place of `/company/people`, and `/search/jobs` with the company name as `search` replaces `/company/jobs`. There is no substitute for the other three.
- 404 on `/job` means the listing has no detail record upstream and is not billed. Roughly one job id in five from `/search/jobs` answers this way - expired postings linger in search after their detail page is gone. Skip that id; retrying will not produce a record.
- LinkedIn upstream can be slow. Set a client timeout of at least 60 seconds and be ready to retry.
- Other errors: 400 is a missing or invalid parameter, such as neither `username` nor `url`, or a `type` outside `posts`/`comments`/`reactions` (a bad `cursor` is the exception - it quietly returns page one instead); 401 is a bad or missing key; 429 is a rate or usage limit (see https://scavio.dev/docs/rate-limits); 502 and 503 are transient and worth a few retries.
- This is public profile data. Never fabricate names, titles, employers, listings, post text or counts, and do not infer private details.
