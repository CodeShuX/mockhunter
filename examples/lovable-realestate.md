# Example Report — Lovable Real-Estate App (Real Audit)

> **This is a real MockHunter audit run** against a Lovable preview app on 2026-05-07. The findings are not illustrative — every cited value, file, and behavior was observed live. Sensitive details (the preview token URL) have been redacted; all other findings are reproduced verbatim.

---

# MockHunter Report

**URL:** `https://preview--agenix-estate-forge-two.lovable.app/admin` (preview token redacted)
**Audit completed:** 2026-05-07 14:23 UTC
**Stack detected:** Lovable
**Auth mode:** skip
**DB verification:** disabled
**Audit duration:** ~7 minutes

## Summary

| Verdict | Count |
|---|---|
| REAL | 0 |
| MOCK | 0 |
| LLM | 0 |
| HARDCODED | 23 |
| BROKEN | 1 |
| UNKNOWN | 0 |
| NO-OP | 1 |

**Total elements scanned:** 25
**Console errors observed during audit:** 1 (404 on activity link)
**Console warnings:** 6 (Lovable preview postMessage warnings — not app bugs)
**Failed network requests during normal navigation:** 1 (the broken activity link)
**API calls observed:** **0**

> **Top-level finding:** This page contacts no backend whatsoever. Every number, badge, name, timestamp, and activity entry is a string literal in the client JavaScript bundle. The dashboard is functionally a static mockup.

---

## Findings — Stat Cards (top 6)

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 1 | Total Users | 12,453 | HARDCODED | String literal in `index-KPm_JlgQ.js` (1 occurrence). No `/api/` or `fetch()` to a data endpoint. | P1 | Wire to a real users count source |
| 2 | "+125 this week" delta on Total Users | +125 this week | HARDCODED | String literal in bundle | P1 | Compute from time-windowed user query |
| 3 | Active Listings | 8,721 | HARDCODED | String literal in bundle | P1 | Wire to listings count |
| 4 | "+42 today" delta on Active Listings | +42 today | HARDCODED | String literal in bundle | P1 | Compute from listings created today |
| 5 | Active Projects | 146 | HARDCODED | String literal in bundle (17 occurrences — also reused elsewhere) | P1 | Wire to projects count |
| 6 | "+3 this week" delta | +3 this week | HARDCODED | String literal in bundle | P2 | Compute from time-windowed project query |
| 7 | Pending Verifications | 57 | HARDCODED | String literal in bundle | P1 | Should sum the 4 queue counts below (28+15+9+5=57); verify whether sum is computed or also hardcoded |
| 8 | "Across all queues" subtitle | "Across all queues" | HARDCODED | Static label in `StatCard.tsx` (per `data-component-file` attribute) | — | Fine |
| 9 | Monthly Revenue | $42,850 | HARDCODED | String literal in bundle | P0 | Financial figures presented as live data are misleading. Wire to billing source or label as "Sample" |
| 10 | "+12% vs last month" | +12% vs last month | HARDCODED | String literal in bundle | P0 | Trend badge implies historical comparison — none exists |
| 11 | Open Tickets | 23 | HARDCODED | String literal in bundle | P1 | Wire to support ticket source |
| 12 | "5 high priority" | 5 high priority | HARDCODED | String literal in bundle | P1 | Wire to priority filter |

## Findings — Verification Queues

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 13 | Agent Verification count | 28 | HARDCODED | String literal in bundle | P1 | Wire to verification table count |
| 14 | Property Ownership count | 15 | HARDCODED | String literal in bundle | P1 | Same |
| 15 | SP Verification count | 9 | HARDCODED | String literal in bundle | P1 | Same |
| 16 | VAS Provider Verification count | 5 | HARDCODED | String literal in bundle | P1 | Same |

**Cross-page consistency check:** The Agent Verification queue link claims 28 pending. Following the link to `/admin/verification/agents` shows 3 rows in the "Pending Review" tab (AVR-001, AVR-002, AVR-004 — note the gap; AVR-003 is missing). The list page is also hardcoded, but the dashboard count (28) and the list size (3) do not match. Either the count is wrong or the list is incomplete. Both are HARDCODED.

## Findings — Recent Platform Activity (8 items)

| # | Activity | Verdict | Source | Severity | Action |
|---|----------|---------|--------|----------|--------|
| 17 | "New Builder 'Modern Homes LLC' registered — Just now" | HARDCODED | String literal in bundle (2 occurrences). "Just now" is a fixed string, not relative time math. | P1 | Wire to events feed |
| 18 | "Listing #45678 flagged for review — 10 minutes ago" | HARDCODED | String literal (2 occurrences); 45678 is the route param too | P1 | Wire to flagged listings query |
| 19 | "SP verification for 'Quality Photos Inc.' approved by admin John — 25 minutes ago" | HARDCODED | String literal | P1 | Wire to verification audit log |
| 20 | "Large payout batch of $15,450 processed for 25 SPs — 1 hour ago" | HARDCODED | String literal | P0 | **Critical — fake financial event in audit log.** Implies a batch processed; nothing happened. |
| 21 | "Agent 'Sarah Johnson' verification rejected — 2 hours ago" | HARDCODED | String literal | P1 | Wire to audit log |
| 22 | "System detected 5 potential duplicate listings — 3 hours ago" | HARDCODED | String literal | P1 | Implies a detection job ran; no such job exists |
| 23 | "New support ticket #4562: 'Payment processing issue' — 5 hours ago" | HARDCODED | String literal | P1 | Wire to support tickets |
| 24 | "Admin user changed fee structure for Agent subscriptions — Yesterday" | HARDCODED | String literal | P0 | Implies an admin action that never happened. Audit log integrity issue. |

## Findings — Interactive Elements

| # | Element | Behavior | Verdict | Severity | Action |
|---|---------|----------|---------|----------|--------|
| 25 | "Search users, listings..." textbox in header | Typed "sarah" → no autocomplete dropdown, no network request, no UI change | NO-OP | P1 | Either implement search or remove the field |
| 26 | "View Details" link on first activity item ("Modern Homes LLC") | Navigates to `/admin/users/builders/123` → 404 | BROKEN | P1 | Activity links point to nonexistent routes. Implement the routes or replace with no-op cards. |
| 27 | "View" link on Agent Verification queue | Navigates to `/admin/verification/agents` → renders a list page (also hardcoded; see cross-page note above) | (works, but downstream is HARDCODED) | P1 | Both pages are mocked end-to-end |
| 28 | Sidebar collapsible buttons (User Management, Verification Queues, Content Management) | Click expands inline submenu (no navigation, no network) | (works) | — | Fine |

---

## Console Errors

```
[ERROR] 404 Error: User attempted to access non-existent route: /admin/users/builders/123 @ /assets/index-KPm_JlgQ.js:596
```

(Triggered by clicking the first "View Details" link in Recent Platform Activity.)

## Console Warnings (not bugs)

```
[WARN] Failed to execute 'postMessage' on 'DOMWindow': target origin 'https://gptengineer.app' does not match recipient origin (×6)
```

These come from Lovable's preview infrastructure (`cdn.gpteng.co/lovable.js`), not the user's app. They appear on every Lovable preview and are not actionable for the app owner.

## Network Requests Captured

The full network log during the audit shows only 5 unique requests:

1. `GET /admin` (200) — the page HTML
2. `GET /assets/index-KPm_JlgQ.js` (200) — the app bundle (2.7 MB minified)
3. `GET /assets/index-ACntSNxu.css` (200) — the stylesheet
4. `GET https://cdn.gpteng.co/lovable.js` (200) — Lovable preview helper
5. `GET https://preview--agenix-estate-forge-two.lovable.app/admin` (second hit during navigation)

**Zero requests to a `/api/` endpoint. Zero requests to Supabase, Firebase, or any third-party data API. Zero WebSocket connections. Zero `react-query`/`axios`/`supabase-js` references in the bundle.**

## Bundle inspection (Phase 4 deep trace)

Pulled the 2.7 MB JavaScript bundle and grep'd for the visible values + common data-fetching patterns:

| Pattern | Hits | Interpretation |
|---|---|---|
| `12,453` (Total Users) | 1 | Single string literal |
| `8,721` (Active Listings) | 1 | Single string literal |
| `"146"` (Active Projects) | 17 | Reused as constant |
| `+12% vs last month` | 1 | Hardcoded delta |
| `$42,850` | 1 | Hardcoded revenue |
| `+125 this week` | 1 | Hardcoded delta |
| `Modern Homes LLC` | 2 | Hardcoded activity entry |
| `AVR-001`, `AVR-002` | 1 each | Hardcoded queue rows |
| `Quality Photos` | 2 | Hardcoded activity |
| `15,450` | 1 | Hardcoded "payout" amount |
| `Math.random` | 4 | Used in non-data contexts (likely IDs/animations) |
| `axios` / `supabase` / `/api/` literals | **0** | No data layer wired |
| `fetch(` | 6 | All internal (router/asset preloading) |
| `localStorage` | 2 | App state, not data |

**Verdict:** the entire admin dashboard is server-side-renderable static content. There is no backend integration, no DB, no remote state. This matches the Lovable scaffolding pattern.

## Suspicious Patterns

- All 6 stat cards have rounded "trend" suffixes (+125, +42, +3, +12%, "5 high priority") — none are computed
- Activity timestamps are all fixed strings ("Just now", "10 minutes ago", "1 hour ago") that never update on refresh — confirmed by reloading the page; same strings persist
- Verification queue counts (28, 15, 9, 5) sum to 57, which matches the "Pending Verifications" stat card. Either the sum was hardcoded to match, or the dashboard sums hardcoded children — either way, no live data
- The 4 Lovable component-file attributes leak (`data-component-file="ActivityItem.tsx"`, `data-component-file="StatCard.tsx"`, etc.) — useful for the audit, but reveals source structure to anyone

---

## Smart Questions for the User

1. **What's the intended state?** This appears to be an early-stage Lovable scaffold with no backend yet. Is connecting the data layer the next milestone, or is this UI being kept as a static demo?
2. **Financial values** — the Monthly Revenue ($42,850, +12%) and the Recent Activity item "$15,450 payout batch" are P0 concerns. Even in a demo, hardcoded financial figures shouldn't be presented without a "Sample data" label. Should we add the label or wait until the data layer lands?
3. **Activity links 404** — the Recent Platform Activity items link to `/admin/users/builders/123`, `/admin/content/listings/45678`, etc., none of which exist. Plan: implement the detail routes, or temporarily make these non-clickable until they do?
4. **Search box NO-OP** — the header search box accepts input but does nothing. Plan: implement (with what backend?), or remove until ready?
5. **Verification list mismatch** — the dashboard says 28 pending agent verifications, the list page shows 3. Easy fix when wiring real data; flagging it now so it doesn't ship that way.

---

## Methodology

- Phases run: 1 (smart questions), 2 (catalog), 3 (interactivity test), 4 (provenance trace via DOM + bundle grep), 5 (this report)
- Auth mode: skip (admin dashboard renders fully without login on the Lovable preview)
- DB verification: not applicable (no backend)
- Stack: Lovable (auto-detected from `*.lovable.app` domain)
- Pages visited: `/admin`, `/admin/verification/agents`, `/admin/users/builders/123` (404 confirmation)
- Interactive elements tested: 4 (verification View link, search box typing, activity View Details link, sidebar collapsibles)
- Bundle inspected: `index-KPm_JlgQ.js` (2,772,407 bytes) — fetched via in-page fetch with explicit `Accept: application/javascript` (Lovable preview's SSR route returns HTML for default Accept)

## Severity Reference

- **P0** — Misleading financial data, fake audit-log entries, fabricated metrics presented as real
- **P1** — Hardcoded data presented as dynamic, broken navigation, NO-OP buttons
- **P2** — Cosmetic delta values that should be computed
- **P3** — Cosmetic, expected for a scaffolded preview

---

*Generated by MockHunter v0.1.0 — https://github.com/CodeShuX/mockhunter*

*This is the actual output of running MockHunter against a Lovable preview app. Reproducible by anyone with the skill installed.*
