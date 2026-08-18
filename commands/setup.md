---
name: setup
description: Connect Scavio in Cursor - API key, MCP server entry, and platform allowlist
---

Walk the user through connecting Scavio in Cursor.

1. Ask whether they have a Scavio API key. If not, send them to
   <https://scavio.dev> to create one. Keys are `sk_` plus 64 hex characters.

2. Have them export it before starting Cursor:

   ```bash
   export SCAVIO_API_KEY=sk_your_key_here
   ```

   Stress the ordering: Cursor reads the variable from the environment it was
   launched in, so on macOS a export in an already-open terminal will not reach
   a Cursor started from the Dock. Put it in the shell profile and restart.

3. Confirm the `scavio` server appears under **Cursor Settings -> MCP**. This
   plugin ships `mcp.json`, so it should register on install. If it is missing,
   run **Developer: Reload Window**.

4. Verify with a `get_usage` call and report the balance back.

5. Mention the platform allowlist: the server picks its tool surface from the
   `x-scavio-platforms` header, filled from `SCAVIO_PLATFORMS`. Unset means
   `default` - 11 platforms plus `get_usage`, 106 tools. It is additive
   (`default,zillow,sec`), and `all` exposes 191 tools - warn that this is a
   large payload in every session.

6. If a call returns an auth error, the likeliest cause is that
   `SCAVIO_API_KEY` was unset when Cursor started, so the server got the
   literal `${env:SCAVIO_API_KEY}` string. Export and restart, then retry.
