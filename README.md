# PageLens AI - MCP Server

> Connect PageLens AI reports to Claude, Cursor, Codex, and other MCP clients so your AI assistant can understand what failed, why it matters, and what to fix next.

An [MCP (Model Context Protocol)](https://modelcontextprotocol.io) server that gives AI agents direct access to [PageLens AI](https://www.pagelensai.com) - the independent launch reviewer for AI-built websites. PageLens AI turns launch risk across UX, SEO, Performance, Accessibility, Security, Conversion, and QA journey audits into plain-English priorities and agent-ready fix context.

Plug it into Cursor, Claude, Codex, or any MCP-compatible client and your agent can read scan results, drill into findings, surface quick wins, record owner decisions, and help close the fix loop without leaving the IDE.

---

## MCP Endpoint

```
https://www.pagelensai.com/api/mcp
```

This is a **remote MCP server** — no local install required.

---

## Which PageLens Path Is This?

- **Launch Pack** is for founders, marketers, and AI builders who want a one-off repair loop: owner-first verdict, fix prompts for their AI builder, Markdown export, desktop + mobile review, and a re-scan.
- **CLI/API/deploy hooks** are for Solo+ users who want PageLens AI to create scans from CI, release scripts, and client workflows.
- **MCP** is for AI assistants that need to read a PageLens report, understand the evidence, and help you patch or triage findings.

If you do not want to read technical detail, start in the PageLens web report and copy the fix prompt into Lovable, Bolt, Replit, Cursor, Codex, Claude, Copilot, or Windsurf. If you do want your coding agent to work directly with the report data, connect this MCP server.

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
      "type": "http",
      "url": "https://www.pagelensai.com/api/mcp"
    }
  }
}
```

### Codex CLI

Add to `~/.codex/config.toml`:

```toml
[mcp_servers.pagelens]
url = "https://www.pagelensai.com/api/mcp"
```

### Generic / other clients

Use the streamable HTTP endpoint:

```text
https://www.pagelensai.com/api/mcp
```

---

## Authentication

This server uses **OAuth 2.0**. Your MCP client will open a browser window to authorize with your PageLens account on first connect. No API keys to manage.

Scopes granted on authorization:

| Scope | What it unlocks |
|---|---|
| `read:scans` | List and read your scan results |
| `read:findings` | Read findings and quick wins |
| `write:feedback` | Submit finding feedback and owner decisions |

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
List your most recent PageLens scans. Filter by status, domain, or date. Returns a slim summary per scan (id, URL, score, grade, severity counts). Scans may include `launchContext` with `builderPlatform` and `launchMoment` so agents can frame fixes for the owner's workflow without changing evidence or severity. QA Audit scans include a compact `qaAudit` summary with confidence, journey-step count, pages reviewed versus page budget, blocked/needs-review step counts, auth-profile status, and the reason fewer than the page budget were captured when the safe same-origin link graph is exhausted.

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
Read the full summary for a single scan: score, grade, severity counts, launch context, executive summary, top-5 highest-priority findings, and per-persona reviews. For QA Audit scans, `get_scan` also returns a `qaAudit` block containing the product-flow synthesis, journey replay, safe blocked actions, confidence, authenticated-route context, and page-budget coverage.

```json
{
  "scan_id": "clxxxxxxxxxxxxxxx"
}
```

---

## Available Resources

### `pagelensai://scan/{id}/markdown`
Fetch the same agent-flavoured Markdown report available from the PageLens UI. Standard reports include launch-context front matter such as `ai_builder` and `launch_moment` when captured. For QA Audit scans, this includes front matter such as `qa_journey_event_count`, `qa_confidence`, and `qa_needs_review_step_count`, followed by the application interpretation, journey replay, blocked/risky paths, safe actions, and next QA tests.

Use this when an agent needs rich context to reason about a QA Audit:

```text
pagelensai://scan/clxxxxxxxxxxxxxxx/markdown
```

### `pagelensai://scan/{id}/summary.json`
Fetch compact JSON for a scan. For QA Audit scans, the `qaAudit` object includes journey metadata, synthesis, page-budget coverage, and any `queue_exhausted` reason explaining why fewer than the purchased page count were discoverable.

```text
pagelensai://scan/clxxxxxxxxxxxxxxx/summary.json
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

### `acknowledge_finding_decision`
Attach owner-controlled context to a finding when the issue is real, but reflects an intentional architecture, security, or product tradeoff.

This does **not** hide the finding, edit the report, or change the PageLens score. It records the rationale so the report can show that the owner has acknowledged the tradeoff, and future scans can recognise the same finding as previously acknowledged.

```json
{
  "finding_id": "clxxxxxxxxxxxxxxx",
  "decision": "INTENTIONAL_TRADEOFF",
  "reason": "Next.js Cache Components and PPR currently prevent us from using per-request CSP nonces safely.",
  "evidence": "proxy.ts documents the CSP tradeoff: script-src uses 'unsafe-inline' because cached HTML shells cannot vary nonces per request.",
  "expires_at": "2026-10-01T00:00:00Z"
}
```

**Decision kinds:** `ACKNOWLEDGED` · `ACCEPTED_RISK` · `INTENTIONAL_TRADEOFF` · `WONT_FIX_NOW`

---

### `clear_finding_decision`
Clear a previously acknowledged decision so it stops appearing on current and future reports. The audit history is preserved.

```json
{
  "decision_id": "clxxxxxxxxxxxxxxx",
  "reason": "We have migrated to a nonce-compatible rendering path."
}
```

You can also clear by `finding_id` when you do not have the `decision_id`.

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
| **QA Audit** | Journey replay, safe form/input exploration, blocked risky actions, app-flow synthesis, page-budget coverage |

Findings include severity, effort estimate, copy-pasteable evidence, and a one-line fix suggestion.

QA Audit scans are different from standard technical scans: the primary artifact is the agentic journey report. PageLens will attempt to review the purchased page budget (for example, up to 10 pages on QA Audit) and should only return fewer pages when the safe same-origin link graph is exhausted. It can use validated auth profiles for scoped post-login routes, while still avoiding SSO, CAPTCHA, MFA, signup, payment, destructive changes, and other committing actions.

---

## Example Agent Workflows

**Fix with my AI builder:**
> "Read the latest PageLens Launch Pack report for example.com, list the top 3 owner-risk fixes, and turn each one into a patch plan I can apply in this repo."

**Respect launch context:**
> "Open the latest PageLens report. If it was built with Codex or Cursor and the launch moment is Product Hunt, prioritise the fixes that make it safe to post publicly, then give me exact patches."

**Pre-launch audit in Cursor:**
> "Run PageLens on my staging site, list all CRITICAL and HIGH findings, and create GitHub issues for the top 5."

**CRO review:**
> "Fetch the latest scan for example.com, get the CONVERSION quick wins, and summarise what to fix before the campaign launch."

**Security sweep:**
> "List all SECURITY findings with severity HIGH or above from my last scan and show me the evidence for each."

**Accepted architecture decision:**
> "For the CSP unsafe-inline finding, acknowledge this as an intentional Next.js/PPR tradeoff with the rationale from our security docs. Do not mark it as a false positive."

**Post-fix validation:**
> "After I've fixed the findings, start a new scan and compare the score to the previous one."

**QA Audit review:**
> "Fetch the latest QA Audit for example.com, read the markdown resource, and tell me which user journeys were verified, which actions were blocked safely, and whether PageLens reviewed the full page budget."

---

## Pricing

PageLens AI supports one-off launch reviews and ongoing plan automation.

| Product | Price | Best for |
|---|---:|---|
| **Launch Scan** | $1 | Quick 3-page pre-launch check |
| **QA Audit** | $10 | Agentic public-app journey review across up to 10 pages |
| **Full Site Scan** | $15 | Broader 25-page site review |
| **Launch Pack** | $29 | Serious AI builders who want desktop + mobile, up to 15 pages, AI-builder fix prompts, Markdown export, and one re-scan |

MCP access is included with paid reports so an agent can work from the same evidence the owner sees. Continuous Health Watch monitoring, API keys, CLI scans, GitHub Actions, deploy hooks, competitors, alerts, and team/client seats are plan features:

| Plan | Price | Why upgrade |
|---|---:|---|
| **Solo** | $19/mo | 3 sites, Health Watch, larger monthly scan allowance, Solo+ automation |
| **Pro** | $49/mo | More sites, Slack/webhook alerts, competitors, and team workflow scale |
| **Agency** | $149/mo | Many client sites, higher quotas, more seats, priority support |

There is no separate new-user Monitor product in the current pricing model. Health Watch is included with paid account plans; legacy monitor rows may still exist for existing customers.

[See full pricing ->](https://www.pagelensai.com/pricing)

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
