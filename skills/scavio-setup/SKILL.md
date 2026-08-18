---
name: scavio-setup
description: Use when Scavio is not yet connected in Cursor, when a Scavio tool returns an auth error, or when the user wants to change which Scavio platforms are exposed - covers the API key, the MCP server entry, and the platform allowlist.
---

# Connect Scavio in Cursor

## Trigger

- "Set up Scavio" / "connect the Scavio MCP" / "add Scavio to Cursor"
- Any Scavio tool call fails with an authentication error
- The user wants more (or fewer) Scavio platforms available

## 1. Get a key

Keys come from the dashboard at <https://scavio.dev>. A real key is `sk_`
followed by 64 hex characters. If the user has not got one, send them there
first - nothing below works without it.

## 2. Export the key

Cursor reads the key from the environment it was launched in.

```bash
export SCAVIO_API_KEY=sk_your_key_here
```

On macOS this must be set **before Cursor starts**. Exporting it in a terminal
that is already open does not reach an app launched from the Dock - the user
has to put it in their shell profile and restart Cursor, or launch Cursor from
a shell that has the variable.

## 3. Confirm the MCP server

This plugin ships `mcp.json`, so the `scavio` server registers automatically on
install. Check it under **Cursor Settings -> MCP**, or in `~/.cursor/mcp.json`
for a manual install:

```json
{
  "mcpServers": {
    "scavio": {
      "url": "https://mcp.scavio.dev/mcp",
      "headers": {
        "x-api-key": "${env:SCAVIO_API_KEY}",
        "x-scavio-platforms": "${env:SCAVIO_PLATFORMS}"
      }
    }
  }
}
```

Reload with **Developer: Reload Window** after any change.

## 4. Verify

Run a cheap call - `get_usage` - and report the credit balance back. A working
key returns a balance; a bad or missing one returns an auth error.

## 5. Choose the platform surface

The full tool surface is too large to register every session, so the server
takes an allowlist from the `x-scavio-platforms` header, filled above from
`SCAVIO_PLATFORMS`.

- unset or `default` - 11 platforms plus `get_usage`, 106 tools
- additive, e.g. `export SCAVIO_PLATFORMS=default,zillow,sec`
- `all` - every platform, 191 tools. Warn the user: that is a large payload in
  every session.

Unknown keys are skipped with a warning rather than breaking the connection.
Changing the variable means restarting Cursor, not editing a header by hand.

## Troubleshooting

An auth error almost always means `SCAVIO_API_KEY` was unset when Cursor
started, so the server received the literal text `${env:SCAVIO_API_KEY}` as its
`x-api-key` header rather than a key. Export it, restart Cursor, retry.
