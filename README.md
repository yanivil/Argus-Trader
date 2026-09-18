# Argus-Trader

Systematic swing-trading engine for U.S. equities. Integrates deterministic Python market
structure pipelines with an adversarial multi-agent LLM committee, governed by automated CI/CD
performance regression gates.

## Project status

**Phase 1 — Design.** The architecture is specified; no implementation code exists in this
repository yet. The repository currently contains documentation only.

## Documentation

| Document | Purpose |
|---|---|
| [`docs/design/phase-1-product-design.md`](docs/design/phase-1-product-design.md) | Phase 1 Product Design Document — the design of record: daily batch pipeline, data layer, Tier 0/1/2 engines, governance, risk model, order state machine, dashboard, audit engine, and CI/CD regression gates. |
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

## Open items before implementation

Implementation is blocked on the items tracked in
[Appendix A](docs/design/phase-1-product-design.md#appendix-a--unspecified-parameters-in-this-draft)
and [§12](docs/design/phase-1-product-design.md#12-open-pre-implementation-decisions) of the
design document — most critically the **position sizing formula (§6.2)**, which the current
draft leaves undefined.
