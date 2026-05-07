# Contributing to MockHunter

Thanks for considering a contribution. MockHunter is small, opinionated, and aims to stay that way — so please read the spec before opening a large PR.

## Before you start

1. Read [SPEC.md](./SPEC.md) — the source of truth for v0.1.0. Anything not in the spec is out of scope unless we agree to a spec amendment.
2. Skim [skill/SKILL.md](./skill/SKILL.md) — the actual skill the user invokes.
3. Check open issues and PRs to avoid duplicate work.

## Areas where help is most welcome

| Area | Why it matters |
|---|---|
| Stack-specific heuristics | Lovable, Bolt, v0, Replit, AI Studio each mock differently. Tune detection per stack. |
| DB connection examples | Add examples for MySQL, MongoDB, Supabase HTTP, PlanetScale, etc. |
| Real example reports | Run MockHunter on apps you control and PR the report (sanitize first). |
| Auth modes | Document workarounds for OAuth, magic-link, 2FA flows. |
| False-positive triage | If MockHunter mislabels a value, file an issue with a repro. |
| Edge cases | SPAs with weird lazy-loading, infinite scroll, virtualized tables, iframes. |

## What to NOT contribute (yet)

- Multi-page crawl, GitHub Action, JSON output → roadmap'd for v0.2. Wait until we open those tracks.
- Auto-fix code patches → out of scope; MockHunter recommends, doesn't patch.
- Visual regression / pixel diffing → use Applitools or Percy; not our space.

## How to contribute

### Filing an issue

Use one of:

- **Bug** — MockHunter misclassified, crashed, or gave wrong output. Include the URL audited (or a redacted version), the report, and what you expected.
- **Feature request** — keep it narrow. "Add support for X stack" is good. "Make it do everything" is not.
- **Spec amendment** — propose a change to SPEC.md before changing code.

### Opening a PR

1. Fork the repo, create a branch off `main`: `feature/short-description` or `fix/short-description`.
2. Make your change. Update `SPEC.md` in the same PR if you change phase semantics or input format.
3. Update `CHANGELOG.md` under `[Unreleased]`.
4. If your change affects user-visible behavior, update `README.md` examples.
5. Open a PR with:
   - What changed
   - Why
   - How to verify (ideally: a sample report before/after)

### PR review criteria

- Stays in scope of v0.1.0 unless explicitly amending the spec
- Doesn't add new dependencies (skill is meant to be a single markdown file)
- Doesn't add tracking, telemetry, or any kind of external call beyond what the user opts into
- Keeps the audit conservative (no destructive actions, no real credential typing)
- Documentation reflects the change

## Style

- Markdown for docs, plain English in the skill prompt
- Tables for anything with multiple dimensions (verdict / severity / etc.)
- No emojis in code/spec; sparingly in README/CHANGELOG if at all
- Honest UNKNOWN beats false REAL — the skill must never inflate certainty

## Sensitive content policy

If you submit example reports, **sanitize them first**:
- Remove URLs that expose internal infrastructure
- Redact API keys, tokens, real user names, real emails
- Replace real DB connection strings with placeholders

We will not accept PRs that include unredacted credentials or production secrets, even if they're "old."

## Code of conduct

Be kind, direct, and concrete. Critique the code, not the contributor. Disagreement is fine; rudeness is not.

## License

By contributing, you agree your contributions are licensed under MIT (the project license). You retain copyright on your contributions; the project license applies to the combined work.

## Questions

Open a GitHub Discussion or file an issue tagged `question`. No private channels yet.
