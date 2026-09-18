# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

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

[Unreleased]: https://github.com/yanivil/Argus-Trader/compare/main...HEAD
