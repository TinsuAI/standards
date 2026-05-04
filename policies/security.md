# Security policy

> **Status: stub.** Awaiting RFC.

This file will codify cross-product security baselines: secrets
handling, key rotation, the auth contract between products, JWT
issuer / verifier rules, vulnerability disclosure.

Until then, follow the per-product implementation in:

- `data-hub/deploy/runbook.md` (key rotation procedure for the SSO
  issuer keypair).
- Each product's `AGENTS.md` for product-specific auth notes.

When ready to fill this in, open an RFC under `proposals/`. See
[CONTRIBUTING.md](../CONTRIBUTING.md).
