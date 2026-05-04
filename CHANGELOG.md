# Changelog

All notable changes to TinsuAI engineering standards. CalVer tags:
`vYYYY.MM.DD`. See [CONTRIBUTING.md](CONTRIBUTING.md) for the change
flow.

## v2026.05.05 — corrections after second review pass

### Changed

- `policies/release-engineering.md` §3 multi-tenant rules
  rewritten. Two patterns are now distinguished: (A) members of a
  single TinsuAI product portfolio sharing one Postgres database
  with schema-per-app + role-per-app — explicitly **allowed at
  any tier** (matches the locked Data Hub + BCQT + CO
  architecture); (B) unrelated workloads co-tenanted on a host —
  separate databases required at Tier P. The previous wording
  inadvertently forbade pattern A at Tier P.
- `reference/glossary.md` and `reference/architecture.md` CO
  write-boundary entries: replaced the "varies by MVP scope"
  hedge with the locked 2026-05-01 answer (CO does not write
  BCCT to Data Hub in MVP; CO is read-only on the `hub` schema).
- `README.md` "How AI agents should use this repo": dropped the
  WebFetch-the-private-URL guidance (which was unworkable
  because the repo is private). Added local-checkout-first +
  `gh api` fallback resolution order.
- `CONTRIBUTING.md`: added an "Initial baseline exception"
  section to document why v2026.05.04 shipped without an RFC and
  to constrain the exception to the first tag.

### Adopted by

- `data-hub` — bumps `.standards-version` to `v2026.05.05`.

## v2026.05.04 — initial release

### Added

- `policies/release-engineering.md` — versioning (SemVer 0.x →
  1.0.0), GitHub Flow branching, three-tier deployment shape
  (Demo / Staging / Production), five-category seed-data
  taxonomy, three-tier backup model, post-1.0.0 hardening list.
  Lifted from a data-hub draft after independent review by a
  critic agent (5 BLOCKER + 10 MAJOR + 3 NIT findings) and
  amended in response. The pre-review draft is not preserved
  here; it lives in the `data-hub` repo's git history.
- `README.md`, `CONTRIBUTING.md`.
- Stub `policies/{security,code-style,ci-cd,ai-collaboration}.md`
  to be filled in by future RFCs.
- Stub `reference/{glossary,architecture}.md`.
- Empty `templates/` and `proposals/` directories.

### Adopted by

- `data-hub` — sets `.standards-version` to `v2026.05.04` and
  rewrites `docs/release-engineering.md` as a per-repo instance.

### Pending

- CO and BCQT review and adoption.

[See the full release notes via `git show v2026.05.04`.]
