# CI/CD policy

> **Status: stub.** Awaiting RFC.

This file will codify cross-product CI/CD baselines: GitHub Actions
conventions, self-hosted runner naming, runner labels, secret
handling, deploy-job structure.

Tentative conventions (to be ratified by RFC):

- **Runner naming**: `<host>-<service>` (e.g. `tinsu-data-hub`,
  `tinsu-co`).
- **Runner labels**: `<service>-<tier>` (e.g. `data-hub-demo`,
  `data-hub-prod`). Each tier × service combination gets its own
  label so workflow files can target a specific environment.
- **One workflow file per repo** (`.github/workflows/ci-cd.yml`)
  with three jobs: `test`, `docker` (or build), `deploy`. `deploy`
  needs `[test, docker]` and is gated on `push:main` or
  `workflow_dispatch`.
- **Secrets** in `secrets.<NAME>` only; never inline.

Until this is ratified, each repo's existing workflow is the
authority for that repo. Cross-link from per-product
`docs/release-engineering.md`.
