# Argus-Trader

Systematic swing-trading engine for U.S. equities. Integrates deterministic Python market
structure pipelines with an adversarial multi-agent LLM committee, governed by automated CI/CD
performance regression gates.

## Project status

**Phase 1 — Design.** The architecture is specified; no implementation code exists in this
repository yet.

## Documentation

| Document | Purpose |
|---|---|
| [`docs/design/phase-1-product-design.md`](docs/design/phase-1-product-design.md) | Phase 1 Product Design Document — the design of record: daily batch pipeline, data layer, Tier 0/1/2 engines, governance, risk model, order state machine, dashboard, audit engine, and CI/CD regression gates. |
| [`docs/design/argus_trader_phase1_design.pdf`](docs/design/argus_trader_phase1_design.pdf) | Authored PDF of the Phase 1 design — the source of record the Markdown version is transcribed from. |
| [`docs/git-hardening-guidelines.md`](docs/git-hardening-guidelines.md) | The repository standard: branch and merge discipline, commit and PR conventions, workflow hardening, supply chain, secret hygiene. Each rule names the repository it was distilled from. |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | The practical how-to: branch, PR, review, and merge flow, and what CI checks. |
| [`SECURITY.md`](SECURITY.md) | Credential posture, the private reporting channel, and the in/out-of-scope boundary. |
| [`CHANGELOG.md`](CHANGELOG.md) | Change history, Keep a Changelog format. |

## Architecture at a glance

The daily production cycle runs deterministically post-market close, 5:58 PM ET – 6:15 PM ET:

1. **Tier 0 — Macro Regime Circuit Breaker.** SPY / QQQ / VIX / breadth determine a GREEN,
   YELLOW, or RED regime, which sets risk-per-trade, max open positions, and eligible setup
   archetypes. RED halts the pipeline at 100% cash.
2. **Daily Order Reconciliation.** Deterministic PENDING → ACTIVE → CLOSED_TP / CLOSED_SL /
   EXPIRED state transitions against the day's OHLC bar.
3. **Retrospective Audit Engine.** Post-trade root-cause taxonomy for newly closed trades.
4. **Tier 1 — Algorithmic Screener.** ~1,000 tickers reduced to a 30–35 candidate shortlist via
   liquidity gates, an earnings blackout, ATR-adaptive ZigZag extrema, and a union of three
   setup archetypes.
5. **Tier 2 — Multi-Agent Committee.** Technicals, Fundamentals, and Catalysts leads produce
   independent dossiers.
6. **Governance — Adversarial Arena.** Bull thesis vs. Bear attack vs. rebuttal, arbitrated by
   an Executive Decision Agent against a net-edge gate and a 2:1 reward-to-risk hurdle.
7. **Publication.** JSON artifacts plus a static Jinja2-rendered GitHub Pages dashboard.

See the design document for the full specification.

## Contributing

Every change — code, docs, configuration, workflow — lands on a branch and merges
through a reviewed pull request. `main` is never written to directly, and a push
that bypasses a PR opens an incident issue automatically.

Read [`CONTRIBUTING.md`](CONTRIBUTING.md) before opening a pull request. The
reasoning behind each rule, and the repository it was distilled from, is in
[`docs/git-hardening-guidelines.md`](docs/git-hardening-guidelines.md).

| Job | Runs on | Blocking |
|---|---|---|
| `pr-quality` | every PR — Conventional Commits title, four required body sections | yes |
| `changelog` | every PR — `CHANGELOG.md` entry, unless labelled `skip-changelog` | yes |
| `tests` | every PR — lint and tests once `src/` and `tests/` exist | yes |
| `audit` | every PR — `pip-audit` once `requirements.txt` exists | advisory |
| `Main guard` | every push to `main` — incident issue on a direct push or force-push | n/a |

> **This repository is public and the engine trades real capital through keyed,
> paid services.** Nothing that identifies an account, a position, or a
> credential belongs in a commit. A key that is pushed must be rotated, not
> deleted — git history is not a delete-capable store.
