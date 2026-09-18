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
- Appendix A to the design document, recording two gaps found in the source draft: the missing
  position-sizing formula, and the `E_Net >= 25` executive gate threshold recovered by
  cross-reference from the §1 pipeline diagram.

### Changed

- `README.md` — expanded from a one-line description to a project-status summary, a
  documentation index, an architecture overview, and a pointer to the open pre-implementation
  items.

[Unreleased]: https://github.com/yanivil/Argus-Trader/compare/main...HEAD
