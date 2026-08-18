---
name: credits
description: Report the current Scavio credit balance and usage for this period
---

Report the user's current Scavio credit balance and usage.

Prefer the `get_usage` tool from the scavio MCP server when that server is connected. It takes no arguments and costs no credits.

If the scavio MCP server is not connected, fall back to a direct call:

```
curl -s https://api.scavio.dev/api/v1/usage \
  -H "Authorization: Bearer $SCAVIO_API_KEY"
```

Summarize the response in a few plain lines, using only these fields:

- `plan` - the current plan
- `credit_balance` - credits available now
- `purchased_credits` - the separately tracked purchased bucket
- `searches_used` - API calls made in the current period, which started at `period_start`
- `auto_recharge` - report `enabled`, and when it is on, its `threshold`, `amount` and `cost`

Then add one line of judgement: whether the balance looks comfortable relative to `searches_used` so far this period, and mention auto-recharge only if it is off and the balance is running low.

This endpoint returns period totals, not a per-call history. Do not invent one and do not estimate per-endpoint spend; if the user wants call-level detail, point them at the dashboard on https://scavio.dev. If the call returns an `error`, report it verbatim and suggest running `/scavio:setup`.
