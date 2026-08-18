# Scavio for Cursor

Structured web data for Cursor's agent. One API key, 31 platforms, structured
JSON rather than HTML you have to parse - search and page extraction, plus
first-party data from commerce, social, travel, jobs, real-estate, app-store,
software-review, ad-library and company-filing sources.

## Install

```bash
git clone https://github.com/scavio-ai/scavio-cursor.git
ln -s "$PWD/scavio-cursor" ~/.cursor/plugins/local/scavio
```

Get a key at [scavio.dev](https://scavio.dev), then export it **before starting
Cursor**:

```bash
export SCAVIO_API_KEY=sk_your_key_here
```

On macOS a shell export does not reach an app launched from the Dock, so put it
in your shell profile and restart Cursor. Then run **Developer: Reload Window**
and check **Cursor Settings -> MCP** for the `scavio` server.

Run `/setup` if you would rather be walked through it.

## Components

### Skills

| Skill | Covers |
|:------|:-------|
| `scavio-google` | Google SERP - organic, ads, knowledge graph, AI overview, PAA |
| `scavio-extract` | Any URL to markdown, text, or HTML |
| `scavio-amazon` | Product search, detail, and every seller offer on an ASIN |
| `scavio-walmart` | Product search, detail, reviews, offers |
| `scavio-ebay` | Live and SOLD listings - what buyers actually paid |
| `scavio-target` | Search, category, product, reviews |
| `scavio-homedepot` | Search, product, reviews |
| `scavio-youtube` | Videos, channels, transcripts, comments |
| `scavio-tiktok` | Videos, users, comments |
| `scavio-tiktok-shop` | Shop products and sellers |
| `scavio-instagram` | Posts, profiles, media |
| `scavio-threads` | Profiles, posts, replies, comments |
| `scavio-x` | Posts and profiles |
| `scavio-kuaishou` | Profiles, videos, hashtag feeds, trending |
| `scavio-reddit` | Posts, threaded comments, subreddits, user history |
| `scavio-linkedin` | Profiles, companies, jobs |
| `scavio-indeed` | Job search, postings, company reviews |
| `scavio-glassdoor` | Company profiles, employee reviews, salaries |
| `scavio-booking` | Hotel search, property detail, guest reviews |
| `scavio-airbnb` | Listing search, detail, guest reviews |
| `scavio-tripadvisor` | Hotels, restaurants, attractions, reviews |
| `scavio-yelp` | Local business search, profiles, reviews |
| `scavio-zillow` | Property search, detail, agent reviews |
| `scavio-redfin` | Listing search, property detail, market stats |
| `scavio-appstore` | App search, listings, user reviews |
| `scavio-googleplay` | App search, listings, user reviews |
| `scavio-g2` | Software search, profiles, verified reviews |
| `scavio-capterra` | Software search, profiles, reviews |
| `scavio-google-ads` | Google ad transparency - advertiser libraries, creatives |
| `scavio-meta-ads` | Meta ad library - ads, advertisers, creatives |
| `scavio-sec` | SEC EDGAR filings, XBRL facts, full-text search |
| `scavio-companies-house` | UK company registration, officers, filing history |
| `scavio-setup` | Key, MCP entry, and platform allowlist |

### Commands

| Command | Does |
|:--------|:-----|
| `/search` | Google search, summarized |
| `/extract` | A page as clean markdown |
| `/credits` | Current credit balance and usage |
| `/setup` | Walk through connecting Scavio |

### Rules

| Rule | Does |
|:-----|:-----|
| `scavio-live-data` | Reach for Scavio on time-sensitive questions instead of answering from memory |

### MCP

`mcp.json` registers the hosted server at `https://mcp.scavio.dev/mcp`. The
tool surface is chosen by the `SCAVIO_PLATFORMS` variable - unset gives 106
tools across 11 platforms, `all` gives 191. See `scavio-setup`.

## Related

The Claude Code plugin lives separately at
[scavio-ai/scavio-plugins](https://github.com/scavio-ai/scavio-plugins). The two
share skill content but differ in setup guidance and packaging, so a change to a
skill here should be mirrored there.

## License

MIT
