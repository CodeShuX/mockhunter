<p align="center">
  <img src="./assets/social-preview.png" alt="MockHunter — Find mock data, hardcoded values, and broken endpoints in your application" width="100%" />
</p>

# MockHunter

> **Find mock data, hardcoded values, and broken endpoints in your application.**

[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](./LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/CodeShuX/mockhunter?style=social)](https://github.com/CodeShuX/mockhunter/stargazers)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-d97757)](https://claude.com/claude-code)
[![Playwright](https://img.shields.io/badge/Playwright-MCP-2EAD33)](https://github.com/microsoft/playwright-mcp)

A Claude Code skill that opens your web app in a real browser, clicks every interactive element, traces every visible value to its actual source, and tells you — in plain English — what's real, what's mocked, and what's broken.

![MockHunter audits a Lovable admin dashboard in 60 seconds](./assets/demo.gif)

*A real Lovable admin dashboard. Looks polished. **Zero of those numbers are real** — every value is a string literal in the JS bundle. [See the full audit report →](./examples/lovable-realestate.md)*

---

## Table of Contents

- [Why this exists](#why-this-exists)
- [Who this is for](#who-this-is-for)
- [How it works](#how-it-works)
- [Installation](#installation)
- [Usage](#usage)
- [Sample output](#sample-output)
- [FAQ](#faq)
- [What MockHunter is NOT](#what-mockhunter-is-not)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)

---

## Why this exists

You shipped an app. Or your AI tool did. Or your contractor did.

Now you're not sure which numbers are real, which buttons actually work, and which "AI insights" are just placeholder text. AI tools like Lovable, Bolt, v0, Replit Agent, AI Studio, and Cursor Composer regularly ship UIs full of:

- Mock data displayed as if it were real (`Math.random()` values, hardcoded arrays of fake users)
- Hardcoded values masquerading as dynamic state (a "73% engagement" badge that's a string literal in JSX)
- LLM-fabricated metrics presented as analytics (a "viral probability score" that's actually just AI output)
- Broken endpoints the UI silently swallows (failed network calls hidden by empty states)
- Disconnected pipelines (UI shows data but the backend never populates it)

MockHunter answers one question: **For every value visible on this page, where does it actually come from?**

---

## Who this is for

- **Vibe-coders** who built an MVP with Lovable / Bolt / v0 / Replit / AI Studio and want a 5-minute reality check
- **Engineers** reviewing AI-generated code from teammates or contractors
- **Anyone** auditing a third-party app or deliverable before sign-off

If you have a working UI and you're not 100% sure what's wired up, MockHunter is for you.

---

## How it works

Five phases, all automated:

1. **Setup** — Asks you a few smart questions about the page (auth, DB access, suspicions)
2. **Catalog** — Opens the page in Playwright, screenshots it, inventories every element
3. **Test Interactivity** — Clicks every button, opens every modal, fills every form, captures console errors and network failures
4. **Trace Provenance** — For each visible value, follows it through the network → API → DB to determine: REAL / MOCK / LLM / HARDCODED / BROKEN / UNKNOWN
5. **Report** — Generates a markdown report with findings, severity, and recommended actions

[Read the full spec →](./SPEC.md)

---

## Installation

**Prerequisites:**
- [Claude Code](https://claude.com/claude-code) installed
- [Playwright MCP](https://github.com/microsoft/playwright-mcp) configured (most Claude Code users have this already)

### Option 1 — One-line install via Claude Code marketplace (recommended)

In any Claude Code session, run:

```
/plugin marketplace add CodeShuX/mockhunter
/plugin install mockhunter@mockhunter
```

Done. The skill is registered and `/mockhunter:mockhunter` is available.

### Option 2 — Manual symlink

```bash
# 1. Clone the repo
git clone https://github.com/CodeShuX/mockhunter.git ~/mockhunter

# 2. Symlink the skill into your Claude Code skills directory
mkdir -p ~/.claude/skills
ln -s ~/mockhunter/skill/SKILL.md ~/.claude/skills/mockhunter.md

# 3. (Optional) Restart Claude Code if it's running
```

Verify either way by typing `/mockhunter:mockhunter` in any Claude Code session.

---

## Usage

In any Claude Code session:

```
/mockhunter:mockhunter
```

The skill will ask:
- Which URL to audit
- How to handle auth (public, localhost, form login, or skip)
- Whether you have database access (optional but recommended)
- A few targeted questions about the page

Then it runs all five phases and writes `mockhunter-report.md` in your current directory.

### Example invocations

**Quickest — public page, no auth:**
```
/mockhunter:mockhunter
> URL: https://my-app.lovable.app
> Auth: skip
> DB: no
```

**With auth and DB:**
```
/mockhunter:mockhunter
> URL: https://staging.myapp.com/dashboard
> Auth: form
> Login URL: https://staging.myapp.com/login
> Email field: input[name="email"]
> Password field: input[name="password"]
> Submit: button[type="submit"]
> DB: psql "postgres://reader:pass@db.host/myapp"
```

**Localhost, frontend-only Lovable app:**
```
/mockhunter:mockhunter
> URL: http://localhost:5173
> Auth: localhost
> DB: no
```

---

## Sample output

```markdown
## Summary

| Verdict | Count |
|---|---|
| REAL | 4 |
| MOCK | 7 |
| LLM | 2 |
| HARDCODED | 5 |
| BROKEN | 1 |
| UNKNOWN | 0 |

## Findings — Dashboard

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 1 | "Engagement" badge | 73% | HARDCODED | String literal in JSX | P1 | Wire to GET /api/metrics |
| 2 | "Recent Activity" | (empty) | BROKEN | GET /api/activity → 404 | P0 | Implement endpoint |
| 3 | "Total Revenue" | $4,231 | REAL | Stripe → DB invoices.amount_total | — | None |
| 4 | "Viral score" badge | 75% | LLM | POST /api/ai/score → GPT-4 | P1 | Label as "AI estimate" |
```

[See full example reports →](./examples/)

---

## FAQ

### Does it work with Lovable apps?
Yes — Lovable is one of the primary targets. The README hero GIF is a real Lovable admin dashboard audit. MockHunter auto-detects `*.lovable.app` URLs and tunes its heuristics accordingly.

### Does it work with Bolt / v0 / Replit / AI Studio / Cursor Composer?
Yes. MockHunter auto-detects:
- Bolt (`*.bolt.new`, `*.stackblitz.io`)
- v0 (`*.v0.app`, `*.v0.dev`)
- Replit (`*.replit.app`, `*.repl.co`)
- Google AI Studio (`aistudio.google.com/*`)
- Cursor Composer (any URL — falls back to "Custom" stack)

Stack-specific heuristics are evolving — PRs welcome.

### Do I need a database to use MockHunter?
**No.** Database access is optional. Without it, MockHunter still detects HARDCODED, MOCK, LLM, and BROKEN values purely from network logs and DOM source. With a DB connection, it can additionally distinguish REAL from server-side-seeded values. See [docs/db-verification.md](./docs/db-verification.md).

### Does it work for authenticated pages?
Yes. Form-login (email + password) is supported in v0.1.0. OAuth, magic-link, and 2FA require a manual pre-login workaround (see [docs/auth-modes.md](./docs/auth-modes.md)).

### Will it modify my app?
No. MockHunter is read-only. It refuses to click destructive-looking buttons (delete, deactivate, transfer, etc.), never submits payment forms, never types real credentials, and only ever runs read-only DB SELECTs. See [docs/how-it-works.md](./docs/how-it-works.md#what-mockhunter-is-conservative-about).

### How is this different from Lighthouse / Axe / Applitools / browser-use / Momentic?
- **Lighthouse / Axe / Pa11y** — perf/SEO/a11y audits, not data provenance
- **Applitools / Percy** — visual regression, requires baselines
- **browser-use / Skyvern / LaVague** — task automators, not auditors
- **Momentic / QA Wolf** — enterprise test-suite tools

MockHunter does one thing: **for every visible value on a page, where does it actually come from?** None of the above answer that question.

### How long does an audit take?
~5–10 minutes for a typical dashboard with 25–50 elements. Page complexity drives the time. Future versions may add a fast-mode (~2 min) that skips Phase 3 interactivity.

### Does it support multi-page crawls?
Not in v0.1.0 — single page per run. Multi-page crawl is on the v0.2 roadmap.

### Can I run it in CI?
Not yet — v0.1.0 is interactive only. A GitHub Action that runs MockHunter on PR previews and posts the report as a PR comment is on the v0.2 roadmap.

### What's the license?
MIT. Use it commercially, modify it, redistribute it. See [LICENSE](./LICENSE).

---

## What MockHunter is NOT

| Tool | Use case |
|---|---|
| **Lighthouse** | Performance / SEO / a11y audits |
| **Axe / Pa11y** | Accessibility testing |
| **Applitools / Percy** | Visual regression |
| **Momentic / QA Wolf** | Enterprise test automation |
| **LaVague QA** | Spec → test conversion |
| **MockHunter** | **One-shot data provenance check on a live page** |

MockHunter doesn't replace any of these. It fills the gap they don't cover: "is this page actually wired up?"

---

## Roadmap

**v0.2** (planned)
- GitHub Action — run MockHunter on every PR, post report as PR comment
- Multi-page crawl
- JSON output format
- a11y signals (basic Axe integration)

**v1.0** (later)
- Diff mode — audit before/after a change
- Auto-fix suggestions
- Self-healing locators

---

## Contributing

PRs welcome. See [CONTRIBUTING.md](./CONTRIBUTING.md).

Areas where help is most needed:
- Stack-specific heuristics (every framework mocks differently)
- DB connection examples for less-common databases
- Real-world example reports we can include in `examples/`
- Edge cases the audit currently misses

---

## License

MIT — see [LICENSE](./LICENSE).

---

## Acknowledgments

Built on top of [Playwright MCP](https://github.com/microsoft/playwright-mcp) by Microsoft and [Claude Code](https://claude.com/claude-code) by Anthropic. Both projects do the heavy lifting; MockHunter just orchestrates them with opinions.

---

<p align="center">
  <img src="./assets/social-preview.png" alt="MockHunter" width="100%" />
</p>

<p align="center">
  <a href="https://github.com/CodeShuX/mockhunter">github.com/CodeShuX/mockhunter</a> · <a href="./LICENSE">MIT</a>
</p>
