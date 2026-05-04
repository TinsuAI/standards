# Release engineering policy

> Status: **v2026.05.04** — initial policy, lifted from a data-hub draft
> after independent review. Adopted by data-hub. CO + BCQT pending review.

This is **portable policy**. It does not name a product, a repo, a
file path, an environment variable, a port, or a runner. Each consuming
repo maintains a `docs/release-engineering.md` that maps these
policies onto its concrete instances.

Each section follows the same shape:

- **Standard** — short industry context.
- **Policy** — what TinsuAI products do.
- **Per-product mapping** — what the consuming repo's own
  `docs/release-engineering.md` must cover.

Contents:

1. Versioning
2. Branching
3. Deployment shapes
4. Seed-data taxonomy
5. Backup & restore
6. Open items
7. Required sections of per-product mapping
8. Post-1.0.0 hardening (deferred, not yet required)

---

## 1. Versioning

### Standard

Two dominant schemes:

- **SemVer (`MAJOR.MINOR.PATCH`)** — implies an API stability promise.
  Most-used default for libraries and APIs.
- **CalVer (`YYYY.MM.PATCH` or `YY.MM`)** — used by Ubuntu, GitLab,
  Twisted. Better for continuous-delivery products with no API
  stability promise.

Build identification beyond the version number:

- Git short SHA (7 chars) — pin to an exact commit.
- Build timestamp (ISO-8601 UTC).
- Conventional Commits + automated bumps (`commitizen`,
  `release-please`, `semantic-release`) — useful at scale, optional
  at small scale.

### Policy

**SemVer.** Tag format `vMAJOR.MINOR.PATCH`. Pre-1.0.0 means
"unstable, breaking changes expected per SemVer." Promote to `1.0.0`
when the first paying customer goes live, not before.

Rationale: SemVer is the de-facto industry default; the version
number is a low-cost convention. We are honest that it does not
solve a real coordination problem while consumer apps share the same
maintainer. The convention buys us a clean external story for free.

**Image tag scheme** (only meaningful once a container registry is in
play):

| Tag                  | Semantics                | Promoted to             |
|----------------------|--------------------------|-------------------------|
| `<service>:vX.Y.Z`   | Immutable, exact version | Production              |
| `<service>:main`     | Latest mainline          | Demo                    |
| `<service>:sha-<7>`  | Pinned to a commit       | (debug / rollback)      |

A registry is **not required** for tier-D demo deployments — building
on the deploy host is acceptable. Add a registry when (a) deploying
to more than one host, or (b) needing to pin production to a known
artifact independently of Git.

**Runtime version exposure.** Each service SHOULD expose
`GET /version` returning `{version, commit, built_at,
deployment_env}` to make "what is actually running" answerable
without ssh. Implementation is per-product. Not a hard requirement
for tier-D demos; required for tier-S and tier-P (see §3).

**Where the version lives at deploy time:**

1. **Most authoritative**: the running app's `/version` response.
2. **Build trail**: the deploy job's log (Git ref + image digest, if
   any).
3. **Filesystem** (fallback): `pyproject.toml` / equivalent in the
   deployed source tree.

### Per-product mapping

Each consuming repo's `docs/release-engineering.md` must record:

- Current version + how it's bumped.
- Whether `/version` is implemented; if not, the date by which it
  will be (or an explicit waiver for tier-D).
- Image tag scheme actually in use today (often "build-on-host, no
  registry" while pre-MVP).

---

## 2. Branching

### Standard

- **Trunk-based development** — everything on `main`, feature flags
  for in-progress work. Used at Google, Meta scale.
- **GitHub Flow** — `main` + short-lived feature branches → PR →
  merge. Simple, fits small teams.
- **GitLab Flow with environment branches** — `main` plus
  environment branches (`production`, `staging`); merges flow
  forward.
- **Git Flow** (Driessen 2010) — `develop`, `release/*`, `master`,
  `hotfix/*`. Heavy. Out of fashion for continuous-delivery.

### Policy

**GitHub Flow + tagged production releases.**

- `main` is **always deployable**. CI must be green for a merge.
- Branch naming:
  - `feature/<short-slug>` — new functionality
  - `fix/<short-slug>` — bug fix
  - `chore/<short-slug>` — tooling, refactors, no behavior change
  - `docs/<short-slug>` — docs only
- Branches are short-lived (target ≤ 5 days). Long-running work uses
  feature flags, not long branches.
- Merge style: **rebase + fast-forward** OR squash-merge for
  feature branches with messy history. No merge commits on `main`.
- No `develop` branch. No `release/*` for routine work. No
  `hotfix/*`.

**Direct commits to `main`** are allowed for trivial edits (typos,
comment fixes, doc clarifications) by the maintainer. PRs preferred
for everything that touches code or policy.

**Branch protection on `main`** (required for any tier-P-bearing
repo): no force-push, linear history, CI must pass before merge.
Approvals are not required for solo maintainers; required when
contributors > 1.

### Hotfix flow

A hotfix is needed when production is broken and `main` cannot be
released as-is (it has unreleased work).

**Default flow** (cherry-pick path):

1. Cut a release branch from the production tag:
   `git checkout -b release/v0.5 v0.5.0`
2. Branch a fix off it: `git checkout -b fix/<slug>`
3. Fix + tests + PR → `release/v0.5`
4. Tag from the release branch: `v0.5.1`
5. Cherry-pick the fix to `main` if relevant there too.

**Lucky path** (tag-on-main): if `main` has no unreleased changes,
fix on a `fix/<slug>` branch off `main`, merge, tag a patch from
`main`. Verify by `git log v<latest-prod>..main` showing only the
fix.

The cherry-pick path is the common case once continuous merging into
main is underway. Plan for it.

### Per-product mapping

Each repo's per-product doc records:

- Current branch protection state on `main` (enabled / not).
- Approval policy.
- Whether the maintainer signs commits or tags (optional —
  see §8).

---

## 3. Deployment shapes

Three tiers. Each is a **contract**: what the deployment must
guarantee. Implementation (Docker Compose, K8s, systemd, bare metal)
is per-product.

### Tier D — Demo

**Audience**: internal show-and-tell, sister-app integration,
investor walkthroughs.

| Property | Required |
|----------|----------|
| Hosting | Single host acceptable |
| Public access | Optional; if public, behind TLS (tunnel or reverse proxy) |
| Auth | May be relaxed for demos |
| Deploy trigger | Auto-deploy `main` on every push, OR manual |
| Backup | Tier 1 (see §5) |
| Monitoring | Optional |
| RTO / RPO | Best-effort / 24h |
| Customer data | Forbidden — demo only |

### Tier S — Staging

**Audience**: rehearse production deploys, validate release
candidates before tagging.

| Property | Required |
|----------|----------|
| Hosting | Mirrors production topology in shape (not necessarily scale) |
| Public access | TLS required; auth strict |
| Deploy trigger | RC tags only (`vX.Y.Z-rc.N`), `workflow_dispatch` |
| Backup | Tier 2 (see §5) |
| Monitoring | Health probe |
| Demo data | Forbidden — staging uses curated, prod-shaped data |
| RTO / RPO | 8h / 4h |

### Tier P — Production

**Audience**: paying agency customers in regulated workflows.

| Property | Required |
|----------|----------|
| Hosting | Per customer OR shared with documented isolation (see below) |
| Public access | TLS, auth strict |
| Deploy trigger | Tagged stable releases only (`vX.Y.Z`), `workflow_dispatch` |
| Backup | Tier 2 minimum at go-live; Tier 3 after second customer |
| Monitoring | External health probe (UptimeRobot / Healthchecks.io) |
| Demo data | Forbidden |
| RTO / RPO | 4h / 1h |

### Multi-tenant on a shared host

Two distinct co-tenancy patterns. Don't conflate them.

**Pattern A — same product portfolio, schema-per-app on a single
database.** Members of one TinsuAI product portfolio (e.g. Data
Hub + BCQT + CO) MAY share a single Postgres database with one
schema per app and one Postgres role per app. The role-per-app
pattern is the isolation contract. Required:

- Each app has a dedicated Postgres role.
- Each role has DDL+DML privileges on **its own** schema only.
- Each role has explicit `usage` + `select` grants on the schemas
  it consumes; no `insert/update/delete` on schemas it doesn't
  own.
- Application code never connects with a superuser role.
- Backup runs per-database (one logical dump covers all schemas).
- Restore drill verifies cross-schema reads still resolve after a
  full restore.

This pattern is allowed at any tier (D / S / P). It is what the
TinsuAI product portfolio architecture commits to.

**Pattern B — unrelated products co-tenanted on one host.** A
single host may run multiple products that are not in one
portfolio (e.g. an unrelated SaaS sharing the same VPS as a
TinsuAI Tier-D demo).

- **Tier D + S (allowed freely)**: separate Compose project,
  separate named volumes, separate published ports. No further
  requirement.
- **Tier P co-tenanted with unrelated workloads (only with explicit
  approval)**: every tenant gets its own database (not just its
  own schema), its own Postgres role, volumes mounted with
  non-overlapping host paths, per-tenant backup, documented in the
  tenant's per-product doc with a date and a reviewer name.
- **Forbidden**: two unrelated tier-P workloads sharing one
  Postgres database (even with separate schemas) — they must use
  separate databases.

### Promotion path

```
feature/* ──┐
            │ PR (CI green)
            ▼
          main ──────► Tier D (auto, every push)
            │
            │ tag vX.Y.Z-rc.N
            ▼
          Tier S ──── (manual workflow_dispatch)
            │
            │ tag vX.Y.Z (after RC sign-off)
            ▼
          Tier P ──── (manual workflow_dispatch, per customer)
```

### Per-product mapping

Each repo records:

- Which tiers exist today.
- For each live tier: hosting, public URL (if any), deploy trigger,
  current backup tier, current monitoring state.
- Which tier the per-product doc commits to standing up next, with
  date.

---

## 4. Seed-data taxonomy

Five disjoint categories. Don't mix them. Mixing causes one of three
incidents: customer data overwritten by a re-seed, reference data
drifting between environments, or tests passing locally and failing
in CI.

### The five categories

| Category | Where it lives (in concept) | Lifecycle |
|----------|------------------------------|-----------|
| **C1 schema (DDL)** | Migration files, append-only | Same lifetime as the app version that introduced it |
| **C2 reference / system data** | Code- or YAML-defined seed in repo | Lives forever; updates with code |
| **C3 demo data** | Code- or script-defined; only seeded when target environment declares itself a demo | Tier D + tier S only |
| **C4 customer (operational) data** | Created via UI / API in a tier-P deployment | Customer's lifetime; survives app upgrades only via C1 migrations |
| **C5 test fixtures** | Per-test setup in repo's test tree | Test runtime only |

### Category-level rules

**C1 schema:**

- Append-only. Once a migration is applied in any tier-S or tier-P
  environment, the file is **frozen forever**. To reverse its
  effect, write a new migration that compensates.
- One logical change per file.
- Numbered sequentially (or strictly orderable by some deterministic
  scheme — choice is per-product).
- DDL + same-migration data fix-ups are allowed and often correct.
- The migration runner must be **idempotent on filename** (skip
  already-applied) and **transactional per file** (a failed
  migration leaves no partial state).

**C2 reference:**

- Each repo declares the seed's behavior model:
  - **One-shot**: seeded only when target table is empty;
    subsequent changes flow via C1 migrations. Simpler.
  - **Re-converging**: re-runs every boot with `INSERT ... ON
    CONFLICT DO UPDATE`. More forgiving, requires every column
    to be derivable from the seed source.
- The chosen model is per-product. The policy here is: pick one,
  document which, and behave consistently. Do **not** mix models
  silently.

**C3 demo data:**

- Gated by an explicit "this is a demo" environment flag. Never
  seeded into tier-P.
- Code-defined or script-defined preferred over binary dumps
  (`pg_dump` artifacts). Code seeds diff cleanly in PRs; binary
  dumps go stale silently when the schema evolves.
- If a binary dump is the chosen vehicle, it MUST carry the schema
  version it was generated against (in filename or sidecar
  metadata), and the restore tool MUST refuse to load a dump whose
  schema version is older than the current migrations.

**C4 customer:**

- Never serialized into the repo. Never shipped in a container
  image. Never included in a demo seed.
- Schema-affecting changes migrate forward via C1 only. Hand-edited
  customer data outside a migration is a bug.

**C5 test fixtures:**

- Test-suite setup runs C1 migrations + C2 reference seed against a
  scratch database before tests touch it.
- Per-test fixtures use UPSERT, never `truncate`, to compose with
  each other.
- **C3 demo data may double as a test fixture** if the test suite
  declares it. When that happens, the per-product doc says so
  explicitly. (Example: a product whose pagination tests rely on
  thousands of seeded rows.)

### Per-product mapping

Each repo records:

- File / module names that hold each category's seed.
- Which behavior model C2 follows (one-shot vs re-converging).
- Whether C3 doubles as C5 (and the test-suite consequence).
- The flag / environment variable that gates C3.

---

## 5. Backup & restore

### Standard

- **3-2-1 rule** (Peter Krogh, *The DAM Book*, 2005): three copies
  of data, on two different storage classes, with one off-site.
- **RTO** (Recovery Time Objective): max acceptable downtime to
  restore service.
- **RPO** (Recovery Point Objective): max acceptable data loss
  measured in time.
- **Test restores periodically** — un-tested backups are not
  backups. Quarterly minimum once tier-P is live.
- **Postgres options:**
  - `pg_dump --format=custom` — logical, portable, slow on very
    large DBs.
  - `pg_basebackup` + WAL archiving — physical, point-in-time
    recovery.
  - Streaming replica — hot-standby, near-zero RPO/RTO at the
    cost of a second host.

### Three tiers

#### Tier 1 — Demo

| Property | Required |
|----------|----------|
| Method | Daily logical dump |
| Retention | ≥ 7 days local |
| Off-site | Not required |
| File volumes | Not required |
| Test restore | Verify each dump opens (`pg_restore --list`) |
| RPO | 24 h |
| RTO | best-effort |

#### Tier 2 — First production go-live (minimum bar)

| Property | Required |
|----------|----------|
| Method | Daily logical dump |
| Retention | 30 days local + 90 days off-site |
| Off-site | S3-compatible (Cloudflare R2, MinIO, AWS S3), encrypted at rest |
| File volumes | Daily archive shipped off-site |
| Secrets / keys | Backed up SEPARATELY (different storage account or vault), once at generation |
| Test restore | Manual drill **once before go-live** + quarterly thereafter |
| RPO | 24 h |
| RTO | 4 h (single-host) |

#### Tier 3 — Mature production (after second paying customer, or when SLA demands)

| Property | Required |
|----------|----------|
| Method | Daily base + WAL archive (≤ 5 min cadence) OR streaming replica |
| Retention | 7 daily + 4 weekly + 12 monthly |
| Off-site | Same as Tier 2 + a second region or provider |
| Test restore | Automated monthly via CI: scratch host, restore latest, run smoke suite |
| RPO | 1 h |
| RTO | 1-4 h (depends on whether replica or restore-from-dump) |

### Restore drill — required content

Every per-product doc must include a **concrete, copy-pasteable
restore procedure** for its current tier. The procedure:

- Names the exact backup location.
- States preconditions (must app be stopped first?).
- Lists the commands to issue, with placeholders for the only
  values that change between drills (timestamp, dump filename).
- States how to verify the restore worked (smoke endpoint, row
  counts, expected user list).
- States what is NOT restored from the data dump (e.g. encryption
  keys, files volume, JWKS).

If the procedure ever drifts from the actually-current deployment
shape, the procedure is wrong. Treat it as a P1 doc bug.

### Per-product mapping

Each repo records:

- Current tier (1 / 2 / 3) and date of last review.
- Backup location (path / bucket / endpoint).
- Restore procedure (or link to the file containing it).
- Date of the last successful restore drill.

---

## 6. Open items

Each open question lists who decides, when (deadline or trigger),
and the default if no decision is made.

| # | Question | Owner | Decide by / trigger | Default if undecided |
|---|----------|-------|---------------------|----------------------|
| O1 | Push images to GHCR / Docker Hub, or build on the deploy host? | Maintainer | Before deploying to a second host | Build on host (status quo) |
| O2 | Adopt Conventional Commits + automated CHANGELOG? | Maintainer | After first `1.0.0` tag | Manual CHANGELOG, free-form commits |
| O3 | Require signed commits / signed tags on `main`? | Maintainer | When team > 1 contributor | Unsigned (status quo) |
| O4 | Require ≥ 1 PR approval on `main`? | Maintainer | When team > 1 contributor | Solo merge OK |
| O5 | Stand up a dedicated Tier S (staging) host, or share with Tier D? | Maintainer | When cutting first `vX.Y.Z-rc.N` | Share with Tier D, separate Compose project |
| O6 | Customer-supplied LLM keys vs vendor-supplied for Tier P? | Sales / Maintainer | Before first paying customer signs | Customer-supplied |

When a question is decided, move the row out of this table and into
the relevant policy section, then bump the policy version (CalVer).

---

## 7. Required sections of the per-product mapping

Each consuming repo MUST keep a `docs/release-engineering.md` file
that covers the items below. Treat it as a contract. CI may grow
checks for missing sections later.

1. **Standards version pinned** — the CalVer tag of this policy
   that the project last reconciled with. Stored in
   `.standards-version` at the repo root and reflected here.
2. **Versioning** — current version, bump policy, `/version` status.
3. **Branching** — protection state on `main`, approval policy.
4. **Deployment shapes** — for each live tier: hosting, URL (if
   any), deploy trigger, current backup tier, monitoring state.
5. **Seed taxonomy** — file names per category, C2 behavior model,
   whether C3 doubles as C5, the flag that gates C3.
6. **Backup & restore** — current tier, backup location, link to
   the restore drill procedure, date of last restore drill.
7. **Open product-specific decisions** — analogous to §6 but for
   per-product items not covered here.

---

## 8. Post-1.0.0 hardening

These are good practices that **do not justify their cost** for a
1-2 dev pre-MVP shop. Adopt after `1.0.0`, not before. Listed so
nobody re-derives them under deadline pressure.

- GPG / SSH-signed commits and tags on `main`.
- Conventional Commits + `release-please` automated CHANGELOG +
  version bumps.
- Image push to GHCR; image signing (Sigstore / cosign).
- SBOM generation per release.
- Branch protection requiring N reviewers.
- Automated monthly restore drill via CI on a scratch host.
- WAL archive shipping every 5 min (Tier 3).
- Multi-region off-site backup.
- External monitoring + on-call rotation.

A repo that has shipped to two paying customers should adopt at
least 5 of these. A repo with one customer and 6 months of stable
operation should adopt 3.

---

## Appendix A — Industry references

- [Semantic Versioning 2.0.0](https://semver.org/)
- [Conventional Commits 1.0.0](https://www.conventionalcommits.org/)
- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [GitLab Flow](https://docs.gitlab.com/topics/gitlab_flow/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [PostgreSQL Continuous Archiving and PITR](https://www.postgresql.org/docs/current/continuous-archiving.html)
- Peter Krogh, *The DAM Book*, 2nd ed. (O'Reilly, 2009) — origin of
  the 3-2-1 backup rule.

## Appendix B — Glossary

- **CalVer** — Calendar Versioning (`YYYY.MM.PATCH`).
- **PITR** — Point-In-Time Recovery. Restore Postgres to any moment
  within the WAL archive window.
- **RPO** — Recovery Point Objective. Maximum acceptable data loss
  expressed in time.
- **RTO** — Recovery Time Objective. Maximum acceptable downtime
  before service is restored.
- **SemVer** — Semantic Versioning 2.0.0 (`MAJOR.MINOR.PATCH`).
- **Tier D / S / P** — Deployment tiers defined in §3 (Demo /
  Staging / Production).
- **Tier 1 / 2 / 3** — Backup tiers defined in §5.
- **WAL** — Write-Ahead Log (Postgres). Archiving these enables PITR.
- **3-2-1 rule** — 3 copies of data, on 2 storage classes, with 1
  off-site. Coined by Peter Krogh.
