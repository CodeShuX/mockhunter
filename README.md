# MockHunter

> **Find mock data, hardcoded values, and broken endpoints in your application.**

A Claude Code skill that opens your web app in a real browser, clicks every interactive element, traces every visible value to its actual source, and tells you — in plain English — what's real, what's mocked, and what's broken.

<!-- Demo GIF goes here once recorded -->
<!-- ![MockHunter demo](./assets/demo.gif) -->

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

**Three steps:**

```bash
# 1. Clone the repo
git clone https://github.com/CodeShuX/mockhunter.git ~/mockhunter

# 2. Symlink the skill into your Claude Code skills directory
mkdir -p ~/.claude/skills
ln -s ~/mockhunter/skill/SKILL.md ~/.claude/skills/mockhunter.md

# 3. (Optional) Restart Claude Code if it's running
```

That's it. Verify by typing `/mockhunter` in any Claude Code session.

---

## Usage

In any Claude Code session:

```
/mockhunter
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
/mockhunter
> URL: https://my-app.lovable.app
> Auth: skip
> DB: no
```

**With auth and DB:**
```
/mockhunter
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
/mockhunter
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
