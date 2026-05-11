# MockHunter — Specification v0.1.0

> **Find mock data, hardcoded values, and broken endpoints in your application.**

This is the source-of-truth spec for MockHunter v0.1.0. Code, README, and docs all flow from here. If a question isn't answered in this file, it's an open question.

---

## 1. The Problem

Modern apps — especially those built with AI tools like Lovable, Bolt, v0, Replit Agent, Google AI Studio, and Cursor Composer — frequently ship with:

1. **Mock data** displayed as if it were real (`Math.random()` values, hardcoded arrays of fake users, lorem-ipsum content)
2. **Hardcoded values** masquerading as dynamic state (a "73% engagement" badge that's a string literal in JSX)
3. **LLM-fabricated metrics** presented as analytics (a "viral probability score" that's actually just AI output)
4. **Broken endpoints** the UI silently swallows (failed network calls hidden by empty states or fallback data)
5. **Disconnected pipelines** — UI shows data but the backend never populates it (create flows that don't trigger processing, schedulers that aren't registered)

The user — often a non-technical founder, a vibe-coder, or even an experienced engineer integrating AI-generated code — has no fast way to know what's real and what's theater. They click around, things look fine, they ship, and then customers see broken features.

**MockHunter answers one question:** *For every value visible on this page, where does it actually come from?*

---

## 2. The User

### Primary persona: The Vibe-Coder

- Built an MVP with Lovable / Bolt / v0 / Replit Agent / AI Studio / Cursor Composer
- Has a working UI but isn't sure what's wired up
- Wants a 5-minute reality check before showing it to a customer or investor
- Comfortable with terminal but not with reading source code line-by-line
- Has Claude Code installed (or willing to install it)

### Secondary persona: The Integrator

- Senior engineer reviewing AI-generated code from a teammate
- Doing due diligence on a third-party app
- Auditing a contractor's deliverable before sign-off
- Has access to the codebase + DB but wants automated coverage

### Non-goals (who this is NOT for)

- QA engineers building enterprise test suites — use Momentic, QA Wolf, Playwright direct
- Visual regression testing — use Applitools, Percy
- Performance/SEO/a11y audits — use Lighthouse, Axe
- Continuous CI testing — v0.1.0 is a one-shot interactive audit. CI integration is v0.2+

---

## 3. Scope of v0.1.0

### In scope

- Single Claude Code skill (`SKILL.md`)
- Five execution phases (see §6)
- Auth modes: public, localhost, form-login, skip
- Optional DB verification (any DB Claude Code can reach via shell)
- Markdown report output
- Smart questions baked into Phase 1
- Provenance classification: REAL / MOCK / LLM / HARDCODED / BROKEN / UNKNOWN
- Works against any web app reachable via URL

### Out of scope (v0.2+)

- GitHub Action / CI integration
- MCP server packaging
- Standalone CLI
- Visual regression / pixel diffing
- Performance/SEO/a11y deep audits
- Multi-page crawl (v0.1.0 = one page per run)
- OAuth / magic-link / SSO auth
- JSON output format
- Diff mode (audit before/after)
- Self-healing locators
- Auto-fix suggestions implemented as code patches

---

## 4. Inputs

The skill prompts the user for these at the start of Phase 1:

| Input | Required | Default | Notes |
|---|---|---|---|
| Target URL | Yes | — | The page to audit. Public, localhost, or behind login. |
| Auth mode | Yes | `skip` | One of: `public`, `localhost`, `form`, `skip` |
| Auth credentials | If `form` | — | URL, email field selector, password field selector, submit selector, email value, password value |
| Has database? | Yes (Y/N) | `N` | If Y, ask for connection details |
| DB connection | If has DB | — | Connection string OR docker exec command OR psql/mysql command |
| Stack hint | Optional | autodetect | Lovable / Bolt / v0 / Replit / AI Studio / Cursor / Custom — used to tune heuristics |
| Suspicions | Optional | — | Free-text: "I think the dashboard numbers are mocked" — focuses the audit |
| Money page | Optional | target URL | The single most important page; if different from target URL, audit prioritizes it |

### Smart questions (Phase 1, asked conversationally)

The skill asks 3–5 questions, picked from this pool based on context:

1. *"What is this page supposed to do for the user?"* (frames the audit)
2. *"Which numbers/badges/data points do you most suspect are mocked?"* (focuses scrutiny)
3. *"Do you have a way to query the database directly? If yes, what's the command?"* (enables Phase 4 deep-trace)
4. *"What stack is this built with?"* (tunes heuristics — Lovable apps mock differently than Bolt apps)
5. *"Are there any sections you know are 'AI-generated content' (not real data)?"* (avoids false-flagging intentional LLM output)

The skill asks no more than 5. If the user provides everything upfront, it skips to Phase 2.

---

## 5. Outputs

### Primary output: `mockhunter-report.md`

Written to the current working directory (or path the user provides). Sections:

```markdown
# MockHunter Report — {URL} — {timestamp}

## Summary
- Total elements scanned: N
- REAL: X
- MOCK: X
- LLM: X
- HARDCODED: X
- BROKEN: X
- UNKNOWN: X

## Findings

### {Section name}
| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 1 | "73% engagement" badge | 73% | HARDCODED | components/Card.tsx:42 — string literal | P1 | Replace with API call to /api/engagement |
| 2 | "Recent Activity" list | (empty) | BROKEN | GET /api/activity → 404 | P0 | Implement endpoint or hide section |
| 3 | "$4,231 revenue" | $4,231 | REAL | Stripe API → DB: invoices.amount | — | None |

## Console Errors (N)
- {error message} — {file:line}

## Network Failures (N)
- GET /api/foo → 404
- POST /api/bar → 500

## Smart Questions for the User
- You have a "Trending" section. Is the trend score a real calculation or AI-generated?
- The dashboard makes 8 parallel API calls on mount — is that intentional?
- Cold-start: when this page loads with zero data, what should the user see?

## Methodology
- Phases run: 1, 2, 3, 4, 5
- Auth mode: form
- DB verification: enabled
- Stack: Lovable
```

### Secondary outputs

- `mockhunter-screenshots/` directory with full-page + per-tab screenshots
- `mockhunter-trace.json` (raw Playwright network log + console log) — for debugging the audit itself

### Provenance verdict definitions

| Verdict | Meaning | Example |
|---|---|---|
| **REAL** | Value comes from a verified data source (DB query, real API, user input) | Stripe revenue, DB COUNT(*) |
| **MOCK** | Value comes from generated/random/placeholder data | `Math.random()`, lorem ipsum, faker.js |
| **LLM** | Value generated by an AI model (may be plausible but not data-backed) | "75% viral probability" from GPT-4 |
| **HARDCODED** | Value is a string literal or constant in source code | `<Badge>EMERGING</Badge>` |
| **BROKEN** | Endpoint returns error, missing data, or 404 | API call fails silently |
| **UNKNOWN** | Could not determine source within the audit's reach | Backend not accessible, no DB provided |

---

## 6. The Five Phases

### Phase 1 — Setup & Smart Questions

**Goal:** Gather just enough context to run a useful audit.

**Steps:**
1. Greet user, restate the skill's purpose in one line
2. Collect required inputs (§4)
3. Ask 3–5 smart questions (§4)
4. Detect stack from URL pattern if not provided (`*.lovable.app` → Lovable, `*.stackblitz.io` → Bolt, etc.)
5. Confirm plan with user before proceeding

**Tools:** AskUserQuestion (or plain prompt if not available)

**Output:** Internal config — passed to Phase 2

### Phase 2 — Navigate & Catalog

**Goal:** Get on the page and inventory everything visible.

**Steps:**
1. Open Playwright, navigate to URL
2. Handle auth per chosen mode:
   - `public` / `skip`: nothing
   - `localhost`: nothing (assume no auth)
   - `form`: fill form fields, submit, wait for nav
3. Wait for page to settle (network idle or 5s timeout)
4. Take full-page screenshot → `mockhunter-screenshots/01-initial.png`
5. Capture accessibility snapshot
6. Catalog:
   - Headings (h1–h6)
   - Buttons (with label, ref, disabled state)
   - Links (with href)
   - Form inputs (with type, name, value)
   - Tabs/navigation regions
   - Data displays (text content of cards, badges, stats, tables)
   - Empty state messages
   - Images (with src, alt, loaded/broken status)
7. Record initial console errors and network requests

**Tools:** Playwright MCP (browser_navigate, browser_snapshot, browser_take_screenshot, browser_console_messages, browser_network_requests)

**Output:** Element inventory + console log + network log

### Phase 3 — Test Interactivity

**Goal:** Find out what each interactive element actually does.

**Steps:**
1. For each tab/navigation element: click, wait, snapshot, check for new console errors, return to original
2. For each button (excluding nav): click, observe outcome (modal? toast? navigation? no-op?), close modal if opened, return to base state
3. For each form: identify required fields, attempt empty submit (capture validation), then attempt valid submit if safe
4. Record per-element behavior:
   - Did it trigger a network request? Which one?
   - Did it open a modal? Did the modal contain real data or placeholder?
   - Did it produce a console error?
   - Did it visibly change anything on the page?
   - Was it a no-op (silent button)?
5. Scroll to bottom of every section to catch below-the-fold content

**Safety rules:**
- Do not submit forms that look destructive (delete, deactivate, transfer, send) without explicit user confirmation
- Do not click buttons labeled "delete", "remove", "cancel subscription", etc.
- If a button triggers an external payment flow, abort and log

**Tools:** Playwright MCP (browser_click, browser_type, browser_fill_form, browser_evaluate, browser_wait_for)

**Output:** Behavior log — every interactive element + its observed behavior

### Phase 4 — Trace Provenance

**Goal:** For every visible value, classify its source.

This is the heart of MockHunter. Each visible value goes through this decision tree:

```
Visible value V
├── Was it fetched from a network request?
│   ├── Yes → trace to API endpoint
│   │   ├── Endpoint returns 4xx/5xx → BROKEN
│   │   ├── Endpoint returns data
│   │   │   ├── Response is a known mock library shape (faker, mockoon) → MOCK
│   │   │   ├── Response uniformity check (all values identical, all round numbers, all timestamps clustered) → MOCK or LLM (flag for review)
│   │   │   ├── If DB connection provided: query DB
│   │   │   │   ├── Value matches DB row → REAL
│   │   │   │   ├── Value not in DB but endpoint returned it → likely MOCK
│   │   │   │   └── Table doesn't exist → MOCK or BROKEN
│   │   │   └── No DB: classify based on heuristics → UNKNOWN with best-guess
│   │   └── Endpoint pattern matches LLM proxy (/api/openai, /api/generate, /api/ai/*) → LLM
│   └── No network request → look for the value in the DOM source
│       ├── Value is a string literal in JSX/HTML → HARDCODED
│       ├── Value computed from Math.random / Date.now / faker → MOCK
│       └── Can't determine → UNKNOWN
```

**Steps:**
1. Replay network log from Phase 2/3, build URL → response map
2. For each visible value (from inventory):
   a. Search network responses for the value
   b. If found, trace to endpoint URL pattern
   c. Apply decision tree above
   d. If DB access enabled, run verification query
   e. Apply uniformity heuristics (variance check on numeric series)
3. Detect LLM-generated content patterns:
   - Endpoint paths matching `/ai/`, `/openai/`, `/generate/`, `/llm/`, `/chat/`
   - Response shapes with `prompt`, `completion`, `model`, `tokens` keys
   - Values that are suspiciously well-formed prose for "data" fields
4. Detect hardcoded patterns:
   - Static badges (TRENDING, NEW, EMERGING, HOT) without a backing field in any API response
   - Round-number percentages (50%, 75%, 90%) appearing in cards labeled "score" or "probability"
   - Timestamps that match the deploy time exactly

**Heuristics for stack detection:**
- Lovable apps: `lovable.app` domain, often use Supabase + mock data fallbacks
- Bolt apps: `stackblitz.io` or `bolt.new` patterns, often raw frontend with no backend
- v0 apps: `v0.app` patterns, shadcn components, typically client-only with placeholder data
- Replit Agent: `replit.app` patterns
- AI Studio: `aistudio.google.com` patterns

**Tools:** Playwright MCP (browser_network_requests, browser_evaluate), Bash for DB queries

**Output:** Provenance map — every visible value labeled with verdict + source

### Phase 5 — Report

**Goal:** Produce a markdown report a human can read in 2 minutes.

**Steps:**
1. Aggregate findings from Phases 2–4
2. Apply severity:
   - **P0**: BROKEN endpoints, console errors that break functionality, fabricated data presented as real metrics
   - **P1**: HARDCODED values masquerading as dynamic, LLM data unlabeled, missing data with misleading empty states
   - **P2**: MOCK data clearly intentional but not labeled, suspicious uniformity in real data
   - **P3**: Cosmetic, expected mock data (e.g., placeholder avatars)
3. Write `mockhunter-report.md` to user's chosen path (default: `./mockhunter-report.md`)
4. Generate "Smart Questions" section — 3–5 follow-up questions tailored to findings
5. Print summary to user, ask if they want a deeper dive on any finding

**Tools:** Write tool

**Output:** `mockhunter-report.md` + `mockhunter-screenshots/` + `mockhunter-trace.json`

---

## 7. Skill Structure (Files)

```
mockhunter/
├── README.md                        # marketing + 3-step install + demo GIF
├── LICENSE                          # MIT (already exists)
├── CONTRIBUTING.md                  # how to contribute
├── CHANGELOG.md                     # v0.1.0 entry
├── SPEC.md                          # this file
├── skill/
│   └── SKILL.md                     # the actual Claude Code skill
├── examples/
│   ├── lovable-realestate.md        # demo run on Lovable app
│   ├── public-saas.md               # demo run on a public app
│   └── localhost-app.md             # demo run on a localhost app
├── docs/
│   ├── installation.md              # detailed setup
│   ├── how-it-works.md              # provenance decision tree explained
│   ├── auth-modes.md                # form login config, edge cases
│   └── db-verification.md           # DB connection examples (Postgres, MySQL, Supabase, MongoDB)
└── assets/
    └── demo.gif                     # hero demo (recorded by user)
```

---

## 8. Installation Flow (target: <5 minutes)

```bash
# Step 1: Clone or symlink the skill
git clone https://github.com/CodeShuX/mockhunter.git ~/.claude/skills/mockhunter-source
ln -s ~/.claude/skills/mockhunter-source/skill/SKILL.md ~/.claude/skills/mockhunter.md

# Step 2: Verify Playwright MCP is configured
# (already configured for most Claude Code users; doc'd in installation.md if not)

# Step 3: Run
# In any Claude Code session:
/mockhunter:mockhunter
```

The skill self-bootstraps from there: asks for URL, auth mode, etc.

---

## 9. Marketing Positioning

### One-liner
*"Find mock data, hardcoded values, and broken endpoints in your application."*

### Hero paragraph
*"You shipped an app. Or your AI tool did. Or your contractor did. Now you're not sure which numbers are real, which buttons actually work, and which 'AI insights' are just placeholder text. MockHunter is a Claude Code skill that opens your app in a real browser, clicks every button, traces every value to its source, and tells you — in plain English — what's real and what's theater."*

### Three audiences (in order of appearance in README)
1. Vibe-coders shipping AI-generated apps
2. Engineers reviewing AI-generated code from teammates
3. Anyone auditing a third-party app or contractor deliverable

### Three demo headlines (for HN / Twitter)
1. *"I built a tool to catch the lies in my own vibe-coded app"* (confessional)
2. *"Show HN: MockHunter — does your Lovable app actually work or does it just look like it does?"* (provocative)
3. *"A 5-minute reality check for AI-generated apps"* (utilitarian)

---

## 10. Success Criteria

v0.1.0 ships when:

- [ ] `SKILL.md` runs end-to-end on the Lovable demo app and produces a useful report
- [ ] Report identifies at least one MOCK or HARDCODED value with correct source citation
- [ ] Report identifies at least one BROKEN endpoint or zero false-broken
- [ ] README has a working demo GIF embedded
- [ ] At least 3 example reports in `examples/`
- [ ] Installation tested fresh (without prior MockHunter state) takes <5 minutes
- [ ] Zero references to private repos, internal tooling, or personal credentials
- [ ] PRs submitted to awesome-claude-code and antigravity-awesome-skills
- [ ] Hacker News post drafted (not yet posted)

v0.1.0 is **not** gated on:
- Star count
- Number of users
- HN frontpage
- Press coverage

---

## 11. Open Questions

These need answers before code:

1. **Q: Should the skill ask for permission before clicking destructive-looking buttons?**
   A (proposed): Yes — by default, skip any button matching `/delete|remove|cancel|deactivate|transfer|send/i`. User can override with a `--aggressive` flag (v0.2).

2. **Q: How does the skill handle SPAs that lazy-load content?**
   A (proposed): Wait for network idle (Playwright's built-in) up to 10s, then proceed. Document the limitation in `docs/how-it-works.md`.

3. **Q: What if the user's app has no backend at all (frontend-only Lovable app)?**
   A (proposed): That's fine — the audit will find lots of HARDCODED and zero REAL. The report will reflect this. Stack detection helps frame this honestly: "Detected: frontend-only Lovable app. All data is local; no backend was contacted."

4. **Q: What if the audit takes >10 minutes?**
   A (proposed): Acceptable for v0.1.0. The skill is interactive, not CI-driven. Report progress every phase. v0.2 can add streaming output.

5. **Q: What do we do about authenticated pages that require 2FA / magic-link / OAuth?**
   A (proposed): v0.1.0 supports form-login only. For 2FA/magic-link/OAuth, the user must log in manually first; the skill will detect existing auth state via Playwright's persistent context. Document workaround in `docs/auth-modes.md`.

---

## 12. Versioning & Release

- v0.1.0 = initial public release
- Semantic versioning: MAJOR.MINOR.PATCH
- Breaking changes to `SKILL.md` interface (e.g., renamed phases, changed input format) bump MINOR pre-1.0
- After 1.0, breaking changes bump MAJOR
- CHANGELOG.md follows [Keep a Changelog](https://keepachangelog.com/) format

---

## 13. License & Attribution

- License: MIT (see `LICENSE`)
- No third-party code redistributed in this repo
- Playwright MCP, Claude Code, and Anthropic SDK are user-installed dependencies, not bundled
- Contributors retain copyright on their contributions; project license applies to the combined work

---

## 14. Sign-off

This spec is locked when:
- [ ] Shubham reviews and approves
- [ ] All §11 open questions answered
- [ ] PR merged to `main` on `CodeShuX/mockhunter`

After sign-off, no changes to this spec without a new version (v0.1.1 spec amendment, etc.). Code MAY differ from spec only with a corresponding spec update committed in the same PR.
