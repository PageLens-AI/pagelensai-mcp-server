# PageLens AI — MCP Server

> AI agents can run senior-level website audits in seconds.

An [MCP (Model Context Protocol)](https://modelcontextprotocol.io) server that gives AI agents direct access to [PageLens AI](https://www.pagelensai.com) — automated website reviews covering UX, SEO, Performance, Accessibility, Security, and Conversion.

Plug it into Cursor, Claude, or any MCP-compatible client and your agent can read scan results, drill into findings, surface quick wins, and close the fix loop — all without leaving the IDE.

---

## MCP Endpoint

```
https://www.pagelensai.com/api/mcp
```

This is a **remote MCP server** — no local install required.

---

## Quick Start

### Cursor

Add to your `~/.cursor/mcp.json` (or workspace `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "pagelens": {
      "url": "https://www.pagelensai.com/api/mcp"
    }
  }
}
```

### Claude Desktop

Add to `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "pagelens": {
      "url": "https://www.pagelensai.com/api/mcp"
    }
  }
}
```

---

## Authentication

This server uses **OAuth 2.0**. Your MCP client will open a browser window to authorize with your PageLens account on first connect. No API keys to manage.

Scopes granted on authorization:

| Scope | What it unlocks |
|---|---|
| `read:scans` | List and read your scan results |
| `read:findings` | Read findings and quick wins |
| `write:feedback` | Submit false-positive reports |

---

## Available Tools

### `whoami`
Confirm which PageLens account this MCP session is operating on, plus granted OAuth scopes.

```json
{}
```

---

### `list_domains`
List the domains you've verified ownership of, along with badge tier and the scan currently anchored to the public badge.

```json
{
  "include_unverified": false
}
```

---

### `list_scans`
List your most recent PageLens scans. Filter by status, domain, or date. Returns a slim summary per scan (id, URL, score, grade, severity counts).

```json
{
  "limit": 20,
  "status": "COMPLETE",
  "domain": "example.com",
  "since": "2026-01-01T00:00:00Z"
}
```

**Status values:** `PENDING` · `RUNNING` · `COMPLETE` · `FAILED` · `CANCELLED`

---

### `get_scan`
Read the full summary for a single scan: score, grade, severity counts, executive summary, top-5 highest-priority findings, and per-persona reviews.

```json
{
  "scan_id": "clxxxxxxxxxxxxxxx"
}
```

---

### `list_findings`
Page through all findings for a scan. Filter by severity, category, persona, page URL, or rule ID. Use `format: "full"` to include descriptions, suggestions, and evidence.

```json
{
  "scan_id": "clxxxxxxxxxxxxxxx",
  "severity": "HIGH",
  "category": "SECURITY",
  "format": "full",
  "limit": 50
}
```

**Severity levels:** `CRITICAL` · `HIGH` · `MEDIUM` · `LOW` · `INFO`

**Categories:** `UX` · `SEO` · `PERFORMANCE` · `ACCESSIBILITY` · `SECURITY` · `CONTENT` · `HEADERS` · `DESIGN` · `ERROR`

**Personas:** `MARKETER` · `CRO` · `UX` · `ACCESSIBILITY` · `BRAND` · `EXECUTIVE` · `PERFORMANCE` · `SEO`

---

### `get_quick_wins`
Return the top N quick-win findings — high impact, low-to-moderate effort — ranked by the same Impact × Effort scorer used in the PageLens dashboard. Optionally override the scan's preset to re-rank under a different lens.

```json
{
  "scan_id": "clxxxxxxxxxxxxxxx",
  "limit": 5,
  "preset_override": "CONVERSION"
}
```

**Presets:** `PRE_SALES` · `PRE_LAUNCH` · `CONVERSION` · `INVESTOR` · `BRAND_POLISH`

---

### `report_finding_feedback`
Flag a finding as a false positive, wrong severity, wrong category, or not actionable. Requires a paragraph of reasoning and a concrete evidence snippet.

```json
{
  "finding_id": "clxxxxxxxxxxxxxxx",
  "kind": "FALSE_POSITIVE",
  "reason": "The selector .pointer-events-auto matches 100+ utility classes, not a single offending element.",
  "evidence": "<div class=\"pointer-events-auto ...\"> — Tailwind utility, not an event-handler.",
  "proposed_severity": "LOW"
}
```

**Feedback kinds:** `FALSE_POSITIVE` · `INCORRECT_SEVERITY` · `INCORRECT_CATEGORY` · `NOT_ACTIONABLE` · `OTHER`

---

## What PageLens Checks

Each scan runs a **deterministic rule engine + AI reviewer pipeline** across every page:

| Area | Examples |
|---|---|
| **SEO** | Title/meta length, canonical URL, Open Graph, heading hierarchy |
| **Performance** | Core Web Vitals (LCP, CLS, INP), page weight, render-blocking resources, DOM size, third-party load |
| **Security** | HTTP→HTTPS redirect, exposed `.env`/`.git` files, SSL expiry, XSS surfaces, source maps, exposed secrets in page source |
| **Accessibility** | Focus-visible CSS, reduced-motion support, ARIA patterns |
| **UX** | Hero hierarchy, mobile menu patterns, CTA structure |
| **Content** | Placeholder text, stale copyright year, mixed-content references |

Findings include severity, effort estimate, copy-pasteable evidence, and a one-line fix suggestion.

---

## Example Agent Workflows

**Pre-launch audit in Cursor:**
> "Run PageLens on my staging site, list all CRITICAL and HIGH findings, and create GitHub issues for the top 5."

**CRO review:**
> "Fetch the latest scan for example.com, get the CONVERSION quick wins, and summarise what to fix before the campaign launch."

**Security sweep:**
> "List all SECURITY findings with severity HIGH or above from my last scan and show me the evidence for each."

**Post-fix validation:**
> "After I've fixed the findings, start a new scan and compare the score to the previous one."

---

## Pricing

PageLens is pay-per-scan — no subscription required for one-off audits.

| Tier | Price | Pages per scan |
|---|---|---|
| **Starter** | $1 | 3 pages |
| **Professional** | $15 | 25 pages |
| **Monitor** | $5 / month | Weekly automated scans · 5 pages |

Every tier produces the same full report — Starter through Professional differ only by page-count cap, not report depth. The Monitor subscription runs weekly scans automatically and surfaces drift between runs.

[See full pricing →](https://www.pagelensai.com/#pricing)

---

## Links

- **Website:** https://www.pagelensai.com
- **MCP endpoint:** `https://www.pagelensai.com/api/mcp`
- **Dashboard:** https://www.pagelensai.com/dashboard
- **GitHub:** https://github.com/PageLens-AI/pagelensai-mcp-server
- **MCP docs:** https://www.pagelensai.com/mcp

---

## License

MIT © [PageLens AI](https://www.pagelensai.com)
