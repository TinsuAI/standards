# Changelog

All notable changes to TinsuAI engineering standards. CalVer tags:
`vYYYY.MM.DD`. See [CONTRIBUTING.md](CONTRIBUTING.md) for the change
flow.

## v2026.05.04 — initial release

### Added

- `policies/release-engineering.md` — versioning (SemVer 0.x →
  1.0.0), GitHub Flow branching, three-tier deployment shape
  (Demo / Staging / Production), five-category seed-data
  taxonomy, three-tier backup model, post-1.0.0 hardening list.
  Lifted from a data-hub draft after independent review.
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
