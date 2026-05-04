# Architecture overview — TinsuAI product portfolio

Cross-product picture of how the three products relate. Per-product
detail lives in each repo's `AGENTS.md`.

## Three products

1. **Data Hub** — master records management. Owns the shared
   HQ-data tier:
   - BCCT (customs declaration registry)
   - Danh Mục (material registry — NVL, SP, BTP)
   - BOM (bill of materials)

2. **BCQT-System** — annual settlement reports (Mẫu 15 / 15a / 16).
   Read-only consumer of Data Hub.

3. **CO-System** — per-shipment origin certificates. **Read-only
   consumer of Data Hub** for HQ-data in MVP (locked 2026-05-01).
   CO does not write back into Data Hub; it maintains its own
   per-shipment state in the `co` schema. A CO → Data Hub write
   path was discussed and deferred; see
   `BCQT-System/.ai/DECISIONS.md` 2026-05-01 entry.

## Storage ownership (provisional, MVP)

- **Postgres `hub` schema** — owned by Data Hub. Read-only access
  granted to BCQT and CO via separate roles.
- **Postgres `bcqt` schema** — owned by BCQT.
- **Postgres `co` schema** — owned by CO.
- **Per-project SQLite** — BCQT-only pattern. One `.db` file per
  BCQT project (DNCX × year). Holds project-truly-local data.
- **Files volume / object storage** — used by Data Hub for raw
  uploaded artifacts. LocalFS day 1; S3-compat phase 2.

## Three deployment shapes per agency (product-side, not ops-side)

This is a product-portfolio taxonomy, distinct from the ops-side
tier model in `policies/release-engineering.md` §3.

- **Data Hub only** — agency uses master records management only.
- **Data Hub + BCQT** — agency does settlement, no CO.
- **Data Hub + BCQT + CO** — full workflow; default expected shape.

BCQT solo (without Data Hub) is **not a supported shape**.

## SSO

A single SSO issuer hosted inside Data Hub serves all three
products. Issuer URL is environment-specific. Consumer apps verify
ed25519-signed JWTs against the issuer's published JWKS endpoint.

The decision to host SSO inside Data Hub (rather than introduce a
third-party identity provider) is recorded in
`BCQT-System/.ai/DECISIONS.md` under the 2026-04-30 PM entry.

## Source of truth for architectural decisions

Cross-repo decisions live in `BCQT-System/.ai/DECISIONS.md` until
Data Hub develops its own decision base. See that file's
"2026-04-30 PM — Data Hub 3-app architecture" entry for the
canonical history.
