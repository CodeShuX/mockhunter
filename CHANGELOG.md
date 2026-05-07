# Changelog

All notable changes to MockHunter are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.1.0] — 2026-05-07

Initial public release.

### Added
- `SPEC.md` — full v0.1.0 specification (problem, user, scope, 5-phase model, provenance taxonomy, success criteria)
- `skill/SKILL.md` — the Claude Code skill that runs the audit
- `README.md` — install + usage + positioning
- `CONTRIBUTING.md` — contribution guidelines
- `docs/installation.md` — detailed install steps
- `docs/how-it-works.md` — the provenance decision tree explained
- `docs/auth-modes.md` — public, localhost, form-login, skip
- `docs/db-verification.md` — how to provide DB access for deep tracing
- `examples/` — sample reports

### Capabilities
- Five-phase audit: Setup → Catalog → Test Interactivity → Trace Provenance → Report
- Provenance verdicts: REAL / MOCK / LLM / HARDCODED / BROKEN / UNKNOWN
- Auth modes: public, localhost, form-login, skip
- Optional DB verification (any DB reachable via shell command)
- Stack auto-detection: Lovable, Bolt, v0, Replit, AI Studio, Custom
- Markdown report output
- Conservative interactivity (skips destructive-looking buttons)
- Console error + network failure capture
- Uniformity heuristics for detecting seeded/templated data

### Known limitations
- Single-page audit per run (no multi-page crawl)
- Form-login only (no OAuth, magic-link, 2FA)
- Caps at 30 most-prominent buttons per page
- No streaming output (full audit runs to completion before report)
- Report is markdown only (no JSON)
