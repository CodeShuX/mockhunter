# Example Report — Lovable Real-Estate App

> **Note:** This is an illustrative example showing the format and content of a typical MockHunter report on a vibe-coded Lovable app. The values are representative, not from a real audit. For a real audit run, follow the install steps and run `/mockhunter` against your own app.

---

# MockHunter Report

**URL:** `https://example-realestate.lovable.app`
**Audit completed:** 2026-05-07 14:32 UTC
**Stack detected:** Lovable
**Auth mode:** skip
**DB verification:** disabled

## Summary

| Verdict | Count |
|---|---|
| REAL | 2 |
| MOCK | 14 |
| LLM | 3 |
| HARDCODED | 8 |
| BROKEN | 2 |
| UNKNOWN | 4 |

**Total elements scanned:** 33
**Console errors:** 5
**Failed network requests:** 2

---

## Findings — Hero / Landing

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 1 | Hero headline | "Find your dream home" | HARDCODED | String literal in `Hero.tsx` | — | None (intentional) |
| 2 | Hero subhead | "Browse 10,000+ verified listings" | HARDCODED | String literal | P1 | "10,000+" implies data — make dynamic or remove number |
| 3 | "Properties listed" stat | 1,247 | HARDCODED | Inline in `StatsCard.tsx`, no API call observed | P1 | Wire to GET /api/listings/count |
| 4 | "Active agents" stat | 89 | HARDCODED | Inline literal | P1 | Wire to real count |
| 5 | "Cities served" stat | 24 | HARDCODED | Inline literal | P1 | Same pattern as above |

## Findings — Featured Listings

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 6 | Listing cards (6 shown) | Various | MOCK | Static array in `mockListings.ts` | P1 | Replace with API call to /api/listings/featured |
| 7 | Property prices | $425k, $612k, $899k... | MOCK | From mock array | P1 | Real listings should have variance pricing |
| 8 | "View All" button | — | NO-OP | Click triggered no nav, no modal, no network call | P1 | Wire to /listings page or remove |

## Findings — Search Filters

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 9 | "City" dropdown | 24 cities | UNKNOWN | Populated client-side, source unclear without DB | P2 | Verify backend has 24 cities or label as sample |
| 10 | "Price range" slider | $0 – $5M | HARDCODED | Range hardcoded in component | P3 | Likely fine; consider data-driven max |
| 11 | "Search" button | — | BROKEN | Click → POST /api/search → 404 | P0 | Endpoint missing |

## Findings — Footer "Live Insights"

| # | Element | Value | Verdict | Source | Severity | Action |
|---|---------|-------|---------|--------|----------|--------|
| 12 | "Market Trend" badge | "Strong Buyer Market" | LLM | POST /api/insights/market → returns LLM prose | P1 | Label as "AI-generated insight" |
| 13 | "Confidence" score | 87% | LLM | Same response | P1 | No backing time-series; remove or label |
| 14 | "Predicted growth" | "+4.2% next quarter" | LLM | Same response | P1 | Misleading — no real prediction model |

---

## Console Errors

```
[error] Failed to load resource: net::ERR_FAILED — /api/search (404)
[error] Failed to load resource: 404 — /api/insights/regional (RegionalInsights.tsx:18)
[warn] React: each child in list should have a unique "key" prop (FeaturedListings.tsx:42)
[warn] Image failed to load: /images/hero-bg.jpg
[warn] Hydration mismatch: server rendered "1,247" but client wants "1,247 "
```

## Network Failures

| Method | URL | Status | Triggered by |
|---|---|---|---|
| POST | /api/search | 404 | "Search" button click |
| GET | /api/insights/regional | 404 | Page mount |

## NO-OP Buttons (clicked, nothing happened)

- "View All" — Featured Listings section
- "Save search" — Search filter bar
- "Notify me" — Footer

## Suspicious Patterns

- All 6 "Featured" listings have prices ending in 999 ($424,999, $611,999, $898,999) — looks templated
- All listing photos use the same Unsplash placeholder URL
- "Active agents" (89) hasn't changed across 3 page reloads despite a "Live counter" label

## Smart Questions for the User

1. The hero stats (1,247 properties, 89 agents, 24 cities) are all hardcoded. Should they be real-time, daily-refreshed, or static marketing copy?
2. Search button returns 404 — is the search backend planned for a future iteration, or did it break recently?
3. "Live Insights" section uses an LLM to generate market predictions. Is that the intended UX? Users may interpret "+4.2% next quarter" as a real forecast.
4. Featured listings are mocked. Do you have a real listings table to wire up, or is this still in design phase?

## Methodology

- Phases run: 1, 2, 3, 4, 5
- Auth mode: skip
- DB verification: disabled
- Stack: Lovable (auto-detected from `*.lovable.app` domain)
- Audit duration: 4 min 22 sec
- Total network requests captured: 38
- Total interactive elements tested: 12

## Severity Reference

- **P0** — Broken endpoints, data integrity failures, fabricated metrics presented as real data
- **P1** — Hardcoded values masquerading as dynamic, unlabeled LLM data, misleading empty states
- **P2** — Mock data clearly intentional but unlabeled, suspicious uniformity in real data
- **P3** — Cosmetic, expected mock data (placeholder avatars, lorem ipsum)

---

*Generated by MockHunter v0.1.0 — https://github.com/CodeShuX/mockhunter*
