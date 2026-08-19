# Lekta MCP server

[Lekta](https://lekta.dev) audits how machines read a page: robots permissions for AI crawlers, JavaScript dependency, citability and freshness, graded **A+ to F** with a spec or a dated measurement behind every finding.

This repository documents the **official hosted MCP server** at `https://lekta.dev/mcp`, so an AI assistant (Claude Code, Cursor, or any MCP client) can audit a URL, read the ranked fixes, apply them and re-audit — until the page reaches A+.

> The server is hosted; there is nothing to install from this repository. This page exists so you can see exactly how the connection works, which tools exist, and what data they touch — before you paste a key anywhere.

## Quickstart

1. Sign in at [lekta.dev](https://lekta.dev/en/login) (free during beta) and generate an API key on **[Panel → API key](https://lekta.dev/en/panel/api)**. Keys look like `lekta_…` and are shown **once**.
2. Add the server to your client. Replace `lekta_YOUR_KEY` with your key.

### Claude Code

**macOS / Linux** (bash, zsh)

```bash
claude mcp add --transport http lekta https://lekta.dev/mcp \
  --header "Authorization: Bearer lekta_YOUR_KEY"
```

**Windows** (PowerShell or cmd — one line)

```powershell
claude mcp add --transport http lekta https://lekta.dev/mcp --header "Authorization: Bearer lekta_YOUR_KEY"
```

Verify on any OS with `claude mcp list` — `lekta` should show as connected. Remove with `claude mcp remove lekta`.

### Cursor

Add to the MCP config file and restart Cursor:

| OS | File |
|---|---|
| macOS / Linux | `~/.cursor/mcp.json` |
| Windows | `C:\Users\YOU\.cursor\mcp.json` |

```json
{
  "mcpServers": {
    "lekta": {
      "url": "https://lekta.dev/mcp",
      "headers": { "Authorization": "Bearer lekta_YOUR_KEY" }
    }
  }
}
```

The same JSON works for a single project under `.cursor/mcp.json` in the project root, and for other editors that read an `mcpServers` map (Windsurf: `~/.codeium/windsurf/mcp_config.json`).

### Any other MCP client

Anything that speaks **streamable HTTP with custom headers** works: endpoint `https://lekta.dev/mcp`, header `Authorization: Bearer lekta_YOUR_KEY`. Without a valid key the server returns `401` — not even the tool list is served.

A quick connectivity check from the terminal (should return the tool list):

**macOS / Linux**

```bash
curl -s https://lekta.dev/mcp \
  -H "Authorization: Bearer lekta_YOUR_KEY" -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

**Windows (PowerShell)**

```powershell
Invoke-RestMethod -Uri "https://lekta.dev/mcp" -Method Post `
  -Headers @{ Authorization = "Bearer lekta_YOUR_KEY" } `
  -ContentType "application/json" `
  -Body '{"jsonrpc":"2.0","id":1,"method":"tools/list"}' | ConvertTo-Json -Depth 6
```

3. Ask your assistant something like:

> Audit https://example.com/pricing with lekta, apply the fixes it suggests, then re-audit and show me the diff.

## Tools

| Tool | What it does | Cost |
|---|---|---|
| `lekta_audit` | Audits a URL the way answer engines fetch it. Returns the grade, top issues ranked by point impact, and complete fixes. | 1 fresh-audit slot (cache hits are free) |
| `lekta_report` | Full structured report as JSON — every check with evidence and fix. | Same as above |
| `lekta_fix_plan` | Ordered "path to A+" from the most recent audit, with expected point impact per step. | Free |
| `lekta_diff` | Compares the last two audits: score movement, checks fixed, checks regressed. | Free |
| `lekta_my_sites` | Lists the saved sites on your account with their latest grade. | Free |

All outputs are English, with a fixed heading scheme (`## Verdict / ## Top issues / ## Fixes`) so agents can parse them reliably.

## Limits (beta)

- **10 fresh audits per day, 1 site per day** — separate from your normal member quota; resets 00:00 UTC.
- Results are cached for 15 minutes; reading a cached result never spends a slot.
- Per-target rate limits (1/min · 5/hour per audited domain) protect the sites being audited — see the [crawler policy](https://lekta.dev/en/bot).

## Security and data

- **Your key is your account.** Anyone holding it can run audits under your quota and read your saved-site list. Keep it out of committed files; rotate it any time on [Panel → API key](https://lekta.dev/en/panel/api) — rotation invalidates the old key immediately.
- The server authenticates every request; unauthenticated requests receive `401` with no tool metadata. Browser cross-origin calls are rejected (DNS-rebinding guard).
- Audits fetch **public pages only**, identified honestly as `LektaBot` — never impersonating a browser or another crawler. Raw page HTML is processed in memory and not stored; only the derived report is kept, served with `X-Robots-Tag: noindex`.
- MCP responses contain your audit findings and fixes. They do not contain other users' data.

Found a security issue? Write to **hello@lekta.dev** — it lands in front of the people who build the engine.

## License

The contents of this repository are MIT licensed. The Lekta service, engine and scoring methodology are proprietary — see [lekta.dev/en/legal](https://lekta.dev/en/legal).
