# TinsuAI engineering standards

Authoritative cross-product engineering policy for TinsuAI products,
plus templates that bootstrap new projects to those policies.

## Scope

In scope:

- Cross-product policy (versioning, branching, deployment shapes,
  seed taxonomy, backup/restore, code style, security baselines,
  CI/CD conventions, AI-collaboration norms).
- Templates that new projects copy at scaffold time
  (`AGENTS.md.tmpl`, CI workflow, `pyproject.toml` skeleton, etc.).
- Cross-product reference material (glossary, architecture
  overview).

Out of scope:

- Per-product decisions (those live in each repo's
  `.ai/DECISIONS.md`).
- Per-product runbooks, deploy commands, customer data, secrets.
- Operational state (which container is running where, etc.).

## Layout

```
.
├── README.md                # this file
├── CONTRIBUTING.md          # how to propose changes (RFC + cool-down)
├── CHANGELOG.md             # significant changes per CalVer tag
├── policies/
│   ├── release-engineering.md   # versioning, branching, deploy, seed, backup
│   ├── security.md              # secrets, key rotation, auth contract
│   ├── code-style.md            # python / JS / SQL baseline
│   ├── ci-cd.md                 # GHA conventions, runner labels
│   └── ai-collaboration.md      # rules for AI agents working in repos
├── templates/                   # bootstrap material for new projects
│   ├── AGENTS.md.tmpl
│   ├── pyproject.toml.tmpl
│   ├── .github/workflows/ci.yml.tmpl
│   └── deploy/docker-compose.yml.tmpl
├── reference/
│   ├── glossary.md              # cross-product domain terms
│   └── architecture.md          # 3-product picture
└── proposals/                   # in-flight RFCs before becoming policy
```

`policies/` is what consumers reference. Everything else supports it.

## Versioning

This repository uses **CalVer** tags: `vYYYY.MM.DD`. Each tag is
"what the standards said on this date." Standards aren't an API;
SemVer would over-promise.

Tags:

- Cut a tag when a substantive policy change ships.
- Trivial fixes (typos, clarifications, broken links) merge to
  `main` without a tag.
- The current `main` is always considered the live policy. Pinning
  to a tag is for projects that don't want to track `main` (rare).

## How to consume this from a product repo

1. Each consuming repo's `AGENTS.md` (or equivalent) has a
   "Standards" section that links to specific policy files by URL.
2. Each consuming repo has a `.standards-version` file at the root
   containing one CalVer tag — the version the project last
   reconciled with.
3. Each consuming repo has a `docs/release-engineering.md` that
   maps these policies onto its concrete instances (paths, env
   vars, ports, runner labels). Required sections of that
   per-product doc are listed in
   [policies/release-engineering.md §7](policies/release-engineering.md).

When a policy file changes:

- Maintainer reviews the diff against `.standards-version` in each
  consuming repo.
- If the change is substantive, update the consuming repo's
  per-product doc and bump `.standards-version`.
- If the change is trivial, leave the pin alone.

## How AI agents should use this repo

This repo is **private**. WebFetch against the GitHub URL returns
404 for unauthenticated agents — don't rely on it. Resolution
order:

1. **Local checkout (preferred)**. Most maintainer machines have
   this repo at `~/workspace/client/tinsu-standards`. Read the
   policy file directly off disk. Pin to the tag in the consumer
   repo's `.standards-version` via:
   ```
   git -C ~/workspace/client/tinsu-standards show \
       <tag>:policies/<file>.md
   ```
2. **GitHub CLI fallback**. If the local checkout isn't present
   but the user has `gh` authenticated:
   ```
   gh api repos/TinsuAI/standards/contents/policies/<file>.md \
       --ref <tag> --jq .content | base64 -d
   ```
3. **Worst case**: ask the user to paste the relevant section.
   Don't guess from training data.

Behaviour when working in a consumer repo:

- Read the consumer's `.standards-version` first; that pins which
  tag of policy applies to that repo.
- If the consumer's per-product doc disagrees with the policy
  doc, prefer the per-product doc but flag the disagreement to
  the user — the per-product doc may be stale OR the policy may
  need amending.
- Don't edit policy files from a consumer-repo session. Surface
  the question to the user; they'll open a session in this repo
  to amend.

## Maintenance

- Default branch: `main`. Branch protection: linear history, no
  force-push, CI passing where applicable.
- Substantive policy changes go through a written RFC under
  `proposals/`. See [CONTRIBUTING.md](CONTRIBUTING.md).
- Solo maintainer: self-merge is allowed after a 24-hour
  cool-down for substantive changes (forces re-read).
- Trivial changes (typos, clarifications, broken-link fixes) merge
  immediately.

---

Maintainer: TinsuAI engineering. Issues / proposals: open a PR.
