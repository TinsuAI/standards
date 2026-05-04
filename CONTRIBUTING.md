# Contributing

How to change TinsuAI standards.

## Two change paths

### Trivial change (no RFC, no cool-down)

Use this for: typos, broken links, formatting, clarifications that
don't shift behavior, examples that match the existing rule.

- Edit on `main` (or via a quick PR if you prefer review).
- No CHANGELOG entry needed.
- No tag.

### Substantive change (RFC + cool-down)

Use this for: anything that shifts what consumer repos must do.
Examples: changing the SemVer-vs-CalVer pick, adding a new
deployment tier, changing an RTO/RPO target, redefining a seed
category, adding a new `policies/*.md` file.

The flow:

1. **RFC**. Create `proposals/YYYY-MM-DD-<slug>.md` with a status
   header. Use this skeleton:

   ```markdown
   # RFC: <title>

   - **Status**: draft
   - **Author**: <name>
   - **Created**: YYYY-MM-DD
   - **Supersedes**: <RFC slug or "none">

   ## Problem
   ## Proposal
   ## Alternatives considered
   ## Migration path for existing repos
   ## Consequences
   ```

   Status moves through: `draft` → `in review` → `accepted` /
   `rejected` / `superseded`.

2. **Open a PR** for the RFC. Description should link to the
   conversation (chat / meeting notes) where the problem was
   raised.

3. **Cool-down**: after the RFC reaches `accepted`, wait at least
   **24 hours** before merging the policy change that implements
   it. This forces a re-read at a different time of day. The
   cool-down does not apply to the RFC itself, only to the policy
   file change.

4. **Implement**: a follow-up PR moves the accepted content into
   `policies/*.md`. The RFC stays in `proposals/` as the historical
   record (status: `accepted`, no further edits).

5. **CHANGELOG + tag**: add a CHANGELOG entry. Tag `vYYYY.MM.DD`
   matching the merge date.

## Initial baseline exception

The first tag of this repo (`v2026.05.04`) shipped a substantive
policy file (`policies/release-engineering.md`) without an RFC
under `proposals/`. That was intentional: the content was already
drafted in another repo, reviewed by an independent critic agent,
amended in response, and adopted by `data-hub` as part of the
same change.

The "RFC + cool-down" flow above applies to changes **after**
the baseline. If you're standing up this repo for the first time
or moving an already-reviewed draft from another location into
`policies/`, you can skip the RFC step provided:

- The CHANGELOG entry names where the draft came from and what
  review it went through.
- The cool-down rule still applies before the merge that
  promotes the baseline tag.

Subsequent substantive changes follow the RFC + cool-down flow.

## Solo maintainer realism

When the maintainer is the only reviewer:

- Self-merge after the 24-hour cool-down.
- Self-review out loud: open the PR, read the diff again, sleep on
  it, then merge.
- This is not theatre. The cool-down catches the kind of mistake
  that "looked right at the time" and reads as obviously wrong the
  next morning. Skipping it loses the benefit; faking it (commit
  with a backdated date) defeats the point.

## Deprecation

When a policy is replaced:

- Don't delete the old text. Move to `deprecated/<file>.md` with a
  header noting the replacement and a sunset date (typically 6
  months after replacement).
- Update consumer repos' `.standards-version` and per-product docs
  to reference the new policy.

## Adding a new policy file

1. RFC under `proposals/`.
2. New file under `policies/<slug>.md`.
3. Mention in `README.md` layout.
4. Add to `CHANGELOG.md`.
5. Tag.

Don't duplicate content across policies. Cross-link.

## Adding a template

Templates are bootstrap material for new repos. Add under
`templates/` with a sibling README explaining when to use it and
which placeholders need filling. Templates that diverge from the
policies they implement are bugs.

## What not to PR

- Decisions specific to one product (those go in that product's
  `.ai/DECISIONS.md`).
- Operational state (which deployment is up, which version is
  running) — not the standards repo's job.
- Customer data. Secrets. API keys.

## Style

- Markdown. CommonMark. Tables sparingly (and never for content
  someone would copy-paste mechanically — see
  `data-hub/.claude/...` lessons-learned about credentials in
  tables breaking on copy).
- No emoji unless the user explicitly asked.
- Short, contract-style sentences in policies. Prose acceptable in
  RFCs.
- Date format: ISO-8601 (`YYYY-MM-DD`).
