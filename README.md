# Scavio for Cursor

Structured web data for Cursor's agent. One API key gets you Google search,
page extraction, and first-party data from Amazon, YouTube, Reddit, TikTok,
LinkedIn, Instagram, Walmart and X - all as structured JSON rather than HTML
you have to parse.

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
| `scavio-youtube` | Videos, channels, transcripts, comments |
| `scavio-reddit` | Posts, threaded comments, subreddits, user history |
| `scavio-tiktok` | Videos, users, comments |
| `scavio-tiktok-shop` | Shop products and sellers |
| `scavio-instagram` | Posts, profiles, media |
| `scavio-linkedin` | Profiles, companies, jobs |
| `scavio-walmart` | Product search, detail, reviews, offers |
| `scavio-x` | Posts and profiles |
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
