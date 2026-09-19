# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- `docs/git-hardening-guidelines.md` — the repository git standard, distilled from the controls
  already in force across the other seven repositories in this account (`SignalSync`, `TASE-125`,
  `travel-planner`, `capital-intelligence-network`, `family-finance-organizer`, `referent-ai`,
  `STRIKE`). Covers branch and merge discipline (§1), commit and pull request conventions (§2),
  guarding `main` (§3), workflow hardening (§4), supply chain (§5), secret and data hygiene (§6),
  documentation in lockstep (§7), the server-side settings that cannot be committed (§8), a
  conformance matrix across all eight repositories (§9), and the adoption checklist (§10). Each
  rule names the repository it was taken from.

- `CONTRIBUTING.md` — the practical working agreement: branch naming, the four-section pull
  request body, Conventional Commits titles, squash merging, the no-direct-push and
  no-force-push rules, red-to-green bug fixes, and the secret and dependency rules.

- `SECURITY.md` — credential posture (Argus-Trader is not zero-credential: Polygon.io and an LLM
  provider are both keyed, and the design tracks order and position state in a public
  repository), private vulnerability reporting, the in/out-of-scope boundary, and an explicit
  split between the controls in force and the repository settings not yet enabled.

- `.gitignore` — the repository had none. Excludes credentials by pattern rather than by
  filename, plus local scratch, Python environments, caches, logs, and editor noise. Records in
  comments that `data/`, `artifacts/`, `test_results/` and `docs/index.html` are deliberately
  tracked, per design §10 and §11.

- `.github/pull_request_template.md` — the four sections CI requires (Context / Why, Summary of
  Changes, Test Steps, Doc Impact), the review checklist, and the bug-fix escape analysis.

- `.github/workflows/ci.yml` — the pull request gate. `pr-quality` checks the Conventional
  Commits title and the four required body sections; `changelog` requires a `CHANGELOG.md` entry
  unless the PR carries the `skip-changelog` label; `tests` becomes the merge gate once `src/`
  and `tests/` land and reports honestly until then; `audit` runs `pip-audit` advisorily once
  `requirements.txt` exists. Actions are pinned to commit SHAs verified against upstream, every
  job sets a timeout, permissions start from `contents: read`, and untrusted pull request text
  reaches every script through `env` rather than interpolation.

- `.github/workflows/main-guard.yml` — a tripwire that opens an incident issue for any push to
  `main` not associated with a merged pull request, and for any force-push. Ported from
  `referent-ai`, rewritten to use the preinstalled `gh` CLI so it depends on no third-party
  action. Exempts `github-actions[bot]`, which is the identity the future daily batch must
  commit under.

- `.github/dependabot.yml` — weekly `github-actions` and `pip` update PRs, with explicit
  Conventional Commits prefixes so Dependabot's titles satisfy the `pr-quality` gate by
  construction rather than by its style auto-detection.

- `docs/design/phase-1-product-design.md` — Phase 1 Product Design Document, the pre-implementation
  design of record. Covers the daily batch pipeline (§1), Polygon SIP data layer and quarterly
  Parquet partitioning (§2), the Tier 0 macro regime circuit breaker (§3), the Tier 1 algorithmic
  screener and setup archetypes (§4), the Tier 2 multi-agent committee and adversarial
  arbitration protocol (§5), portfolio risk and concentration boundaries (§6), the order state
  machine and CLI specification (§7), the GitHub Pages dashboard (§8), the retrospective audit
  taxonomy (§9), CI/CD backtest regression gates (§10), the target repository layout (§11), and
  open pre-implementation decisions (§12).

- `docs/design/argus_trader_phase1_design.pdf` — the authored PDF of the Phase 1 design,
  committed as the source of record for the Markdown transcription.

### Fixed

- Restored three formula blocks to `docs/design/phase-1-product-design.md` that were absent from
  the text originally transcribed into the repository. Verified against the authored PDF:
  - §4.2 ZigZag reversal threshold: `Reversal Threshold = 1.5 × ATR14`.
  - §5 Executive Decision Agent net-edge gate: `E_Net = C_Bull - R_Bear >= 25 where C_Bull >= 75
    and R_Bear < 50`. Only the `>= 25` component was previously present, and only in the §1
    pipeline diagram.
  - §6 position sizing: total dollar risk, risk per share, and the floored share count.
- Aligned §2 wording with the PDF ("via Developer/Starter tier", "safely below GitHub limits")
  and §4.2's coordinate-pair bullet.

- Corrected the §2 Window bullet in `docs/design/phase-1-product-design.md`, where a line wrap
  placed `>` at the start of a continuation line. CommonMark reads that as a blockquote, so the
  sentence rendered split across a quoted block. The thresholds now read `>$10` and `>1M`
  unbroken, matching the PDF, and the bullet renders as one sentence.

### Changed

- `README.md` — expanded from a one-line description to a project-status summary, a
  documentation index, and an architecture overview.

- `README.md` — documentation index now lists the guidelines, contributing guide and security
  policy; added a Contributing section with the CI job table and the public-repository warning.

[Unreleased]: https://github.com/yanivil/Argus-Trader/compare/main...HEAD
