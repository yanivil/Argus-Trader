# Security Policy

## Overview

**Argus-Trader** is a systematic swing-trading engine for U.S. equities. This
repository is **public** and, as of Phase 1, contains design documentation only —
no implementation code, no dependencies, and no workflows that touch market data.

Unlike the other scanners in this account, Argus-Trader is **not** a
zero-credential system. The Phase 1 design specifies two keyed, paid
dependencies:

- **Polygon.io** (design §2) — SIP market data, via an API key.
- **An LLM provider** (design §5) — the Tier 2 agent committee.

The design also specifies that order and position state (`data/pending_orders.json`,
`data/open_positions.json`, `data/order_history.json`) is tracked in git. That
state describes a real book in a public repository, which makes the boundary
below a live concern rather than a formality.

## Reporting a vulnerability

1. **Do not open a public GitHub issue.**
2. Use GitHub's private vulnerability reporting: the repository's
   [Security tab](https://github.com/yanivil/Argus-Trader/security) →
   **Report a vulnerability**.
3. Include a description, steps to reproduce or a proof of concept, and the
   impact you believe it has.

You will get an acknowledgement within a few days. Fixes ship as ordinary pull
requests and are recorded under **Security** in `CHANGELOG.md`.

## Scope

**In scope**

- Credential handling for the Polygon.io and LLM provider keys.
- The GitHub Actions workflows in `.github/workflows/`, and their permission
  scopes and action pins.
- Supply-chain integrity: the dependency pins and their hashes.
- Any path by which account, order, or position data could reach a public
  artifact — a commit, a Pages build, a workflow log, or a CI annotation.
- The design documents in `docs/design/`, where the design itself creates one of
  the above.

**Out of scope**

- Trading outcomes, strategy profitability, and the correctness of any signal.
  Argus-Trader is an analytical tool, not financial advice, and a losing trade is
  not a vulnerability.
- Availability and correctness of third-party providers (Polygon.io, the LLM
  provider, GitHub). Report those upstream.

## Controls

### In force in this repository

These ship as files in the repository and are active as soon as they are on
`main`:

- **Secrets never enter the repository.** `.gitignore` excludes credentials by
  pattern, not by filename. Real values live in GitHub Actions secrets or the
  local environment; required variables are documented in `.env.example`.
- **Least-privilege workflows.** Every workflow declares `permissions`, starting
  from `contents: read`, and elevates only the scope a specific job needs.
- **Actions pinned to commit SHAs.** A tag is a movable pointer; a pinned SHA is
  not. Dependabot opens the PRs that move the pins forward, weekly.
- **Untrusted input is never interpolated into a shell script.** PR titles and
  bodies reach workflow scripts through `env`, as data.
- **Dependency audit in CI.** `pip-audit` runs against the pinned set on every
  pull request, from the moment `requirements.txt` exists.
- **`main` has a tripwire.** The *Main guard* workflow opens an incident issue
  for any push to `main` that did not arrive through a merged pull request, and
  for any force-push.

### Not yet enabled — repository settings

These are GitHub settings, not files. They cannot be committed, and they are
**not** currently configured on this repository. Until they are, the controls
above are the only ones operating:

- **Branch ruleset on `main`** — require a pull request, require the CI checks,
  block force pushes, restrict deletions. `main` is currently unprotected.
- **Secret scanning and push protection.** Push protection is the only control
  anywhere in this policy that stops a leak *before* it reaches the remote;
  everything else only limits the damage afterwards. Its absence is the largest
  open gap.
- **Dependabot security updates** (distinct from the version-update PRs that
  `.github/dependabot.yml` configures).
- **Private vulnerability reporting**, so the channel this policy points at
  above actually exists.

The exact settings paths are in
[`docs/git-hardening-guidelines.md`](docs/git-hardening-guidelines.md) §8, which
also gives the full standard and the provenance of each control.

## A note on credential rotation

Git history is not a delete-capable store. A key that has been pushed to this
repository — even on a branch that was later deleted, even in a commit that was
later reverted — must be **rotated**, not removed. Treat any such push as a
disclosed credential.
