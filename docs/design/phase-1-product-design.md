# Argus Trader — Phase 1 Product Design Document

> **Status:** Draft — pre-implementation design of record.
> **Scope:** Phase 1 (architecture definition). No production code is specified by this document.
> **Last updated:** 2026-09-18

---

## System Overview

Argus Trader is an automated, multi-agent swing trading architecture for liquid U.S. equities
(~1,000 tickers). It couples deterministic market-structure pipelines with an adversarial LLM
decision committee, bound by quantitative risk constraints, an order state machine, a static
GitHub Pages operational interface, and automated CI/CD backtesting regression gates.

### Editorial notes on this transcription

- Mathematical notation from the source draft is rendered as plain inline text
  (`EMA50`, `SMA200`, `>= $10.00`) rather than LaTeX, because GitHub does not render math
  reliably inside tables. No values were altered.
- Two items were **left incomplete in the source draft** and are therefore *not* filled in
  here. They are marked inline with `> **UNSPECIFIED**` callouts and collected in
  [Appendix A](#appendix-a--unspecified-parameters-in-this-draft). They must be resolved before
  implementation begins.

---

## 1. System Architecture & Daily Batch Pipeline

The daily production cycle runs deterministically post-market close between 5:58 PM ET and
6:15 PM ET.

```
[ 5:58 PM ET: Polygon SIP Daily Aggregates ]
                    │
                    ▼
       [ Tier 0: Macro Regime Engine ] ──► RED REGIME ──► [ Halt Pipeline / 100% Cash ]
                    │                                     (Reconcile Active Only, Exit 0)
            GREEN / YELLOW REGIME
                    │
                    ▼
     [ 6:00 PM ET: Daily Order Reconciliation ]
     • PENDING Orders: Check Daily Low <= Limit X -> Move to ACTIVE
     • ACTIVE Positions: Check Daily High >= TP Y or Daily Low <= SL Z
                    │
                    ▼
      [ Retrospective Audit Engine ]
     • Run post-trade failure taxonomy on newly closed trades
                    │
                    ▼
     [ Tier 1: Algorithmic Market Screener ]
     • 15-Day Earnings Blackout
     • ATR-Adaptive ZigZag Extrema Detection
     • Multi-Archetype Union Filter -> Shortlist 30-35 Candidates
                    │
                    ▼
     [ Tier 2: Multi-Agent Analytical Committee ]
     • Technicals Lead (Trend/MTF, Liquidity/SR, ATR, 3 Pattern Sub-Agents)
     • Fundamentals Lead (Solvency, Margins, Growth, Altman Z)
     • Catalysts Lead (Filings, Revisions, Insider Action)
                    │
                    ▼
     [ Governance Layer: Adversarial Arena ]
     • Bull Thesis Agent vs. Bear Attack Agent -> Rebuttal
     • Executive Agent Arbitration (E_Net Gate >= 25, R:R >= 2:1)
                    │
                    ▼
     [ Output Generation & Publication ]
     • Write JSON Artifacts to agents_data/
     • Render docs/index.html (Jinja2)
     • Git Commit & Push to GitHub Pages CDN
```

---

## 2. Data Layer & Historical Ingestion

### Data Source

- **Provider:** Polygon.io Consolidated Tape (SIP) via the Developer/Starter tier.
- **Endpoint:** Grouped Daily Bars (`/v2/aggs/grouped/locale/us/market/stocks/{date}`).
- **Window:** 5-year rolling history covering ~1,000 liquid equities
  (> $10 share price, > 1M average daily volume), plus index benchmarks (SPY, QQQ, VIX).

### Partitioning & GitHub Storage Strategy

To comply with GitHub's 50 MB warning and 100 MB hard limit per file, raw ingestion data is
compressed with Snappy and partitioned quarterly:

- **Format:** Columnar Apache Parquet (`.parquet`).
- **Path schema:** `data/raw/polygon/{YYYY}/{YYYY}_Q{1-4}.parquet`
- **Quarter volume:** ~63 trading sessions per quarter × ~1,000 tickers ≈ 63,000 rows.
- **Storage footprint:** ~8 MB to 12 MB per partition file (well within GitHub limits).

---

## 3. Tier 0: Macro Regime Circuit Breaker

Evaluates broad market health at 5:58 PM ET before executing scans or allocating capital.

| Parameter | Green Regime (Risk-On) | Yellow Regime (Caution / Distribution) | Red Regime (Hostile / Crash) |
|---|---|---|---|
| SPY Moving Averages | Close > EMA50 **AND** > SMA200 | Close < EMA50 **BUT** > SMA200 | 2 consecutive daily closes < SMA200 |
| QQQ Trend | Close > SMA200 | Close within 2% of SMA200 | Close < SMA200 |
| VIX Volatility | VIX < 21.0 | 21.0 <= VIX <= 28.5 | VIX > 28.5 |
| Breadth (%S50) | >= 50% above 50 EMA | 30% <= %S50 < 50% | < 30% above 50 EMA |
| Risk Per Trade | 1.0% total equity | 0.5% total equity | 0.0% (trading halted) |
| Max Open Trades | 6 to 8 positions | 3 positions | 0 positions (100% cash) |
| Eligible Setups | All archetypes (A, B, C) | Archetypes A & B only | Pipeline skips Tiers 1 & 2 |

---

## 4. Tier 1: Algorithmic Market Screener

Screens ~1,000 tickers down to a shortlist of 30–35 actionable setups using deterministic
Python logic.

### 4.1 Mandatory Invalidation Gates

- **Liquidity:** 20-day average daily volume >= 1,000,000 shares; Close >= $10.00.
- **Earnings blackout:** Hard drop if corporate earnings occur within the next 15 calendar days.

### 4.2 Extrema Detection Engine: ATR-Adaptive ZigZag

Swing pivots are calculated via volatility-adjusted price reversals rather than fixed-width
time windows:

- Guarantees an alternating Peak → Trough → Peak sequence.
- Extracts exact `(x, y)` coordinate pairs `(Date, Price)` to feed Tier 2 geometric pattern
  agents.

### 4.3 Setup Archetypes & Scoring

- **Archetype A (Trend Pullback):** Daily Close > SMA200; 20 <= RSI14 <= 45; price touching or
  within 1.0 × ATR14 of a rising 50-day EMA.
- **Archetype B (Volatility Squeeze):** Bollinger Bands (20, 2.0) fully contract inside Keltner
  Channels (20, 1.5); volume contracting for >= 3 consecutive sessions.
- **Archetype C (Relative Strength / Momentum Consolidation):** 3-month relative performance >
  SPY; consolidation within 5% of the 52-week high; base duration >= 10 sessions.

---

## 5. Tier 2: Multi-Agent Committee & Adversarial Debate

Shortlisted candidates are analyzed across three independent analytical domains before entering
an adversarial debate.

```
                  [ Tier 1 Shortlist: 30-35 Candidates ]
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
[ Technicals Lead ]         [ Fundamentals Lead ]       [ Catalysts Lead ]
  • Trend & MTF Sub-Agent     • Capital Structure         • Earnings Quality
  • Liquidity & S/R Sub-Agent • Operational Margins       • SEC Form 4 Filings
  • Volatility (ATR Engine)   • Altman Z-Score            • Institutional Flows
  • 3 Pattern Sub-Agents:
    - Bullish Wolfe Wave
    - Inverse Head & Shoulders
    - Double Bottom
         │                           │                           │
         └───────────────────────────┼───────────────────────────┘
                                     ▼
                      [ Governance: Dialectical Arena ]
                      1. Bull Agent presents Thesis
                      2. Bear Agent presents Antithesis & Objections
                      3. Bull Agent offers Synthesis / Rebuttal
                                     │
                                     ▼
                          [ Executive Decision Agent ]
                      Arbitrates net edge: E_Net = C_Bull - R_Bear
```

### 5.1 Analytical Domains

**Technicals Domain**

- *Trend & MTF Sub-Agent:* Verifies weekly/daily moving average alignment and Stage 2 markup
  confirmation.
- *Liquidity & S/R Sub-Agent:* Maps Volume Profile Point of Control (POC), Value Area High
  (VAH), and Value Area Low (VAL).
- *Volatility Engine (pure Python):* Calculates ATR buffers and invalidation distances.
- *Pattern Sub-Agents (Wolfe Wave, Inverse H&S, Double Bottom):* Evaluate geometric symmetry,
  pivot alignments, and neckline breakout volumes.

**Fundamentals Domain**

Audits solvency, debt/equity ratios, and operating cash flow, and flags distress risk via the
Altman Z-score (Z < 1.8 triggers an automatic veto).

**Catalysts Domain**

Audits upcoming macro catalysts, earnings revisions, and Form 4 insider transactions.

### 5.2 Adversarial Arbitration Protocol

- **Bull Agent:** Formulates the optimal entry limit (X), structural target (Y), and base thesis
  score (`C_Bull ∈ [0, 100]`).
- **Bear Agent:** Attacks structural flaws, overhead resistance, and sector headwinds, returning
  an objection risk score (`R_Bear ∈ [0, 100]`).
- **Executive Decision Agent:** Approves trade tickets only when both of the following hold:
  - **Net edge gate:** `E_Net = C_Bull - R_Bear >= 25`.
    *(Threshold sourced from the pipeline diagram in [§1](#1-system-architecture--daily-batch-pipeline);
    the source draft's §5 bullet list omitted it.)*
  - **Reward-to-risk hurdle:** Minimum 2.0:1, calculated strictly against structural
    invalidation points.

### 5.3 Agent Artifact Hierarchy

All agent evaluations and debate transcripts are serialized to disk:

```
artifacts/agents_data/
├── technicals/
│   ├── parent_lead/{YYYY-MM-DD}/{TICKER}_technicals_dossier.json
│   ├── trend_mtf_sub_agent/{YYYY-MM-DD}/...
│   ├── liquidity_sr_sub_agent/{YYYY-MM-DD}/...
│   ├── wolfe_wave_sub_agent/{YYYY-MM-DD}/...
│   ├── inverse_hs_sub_agent/{YYYY-MM-DD}/...
│   └── double_bottom_sub_agent/{YYYY-MM-DD}/...
├── fundamentals/
│   └── fundamental_lead/{YYYY-MM-DD}/{TICKER}_fundamentals_dossier.json
├── catalysts/
│   └── catalyst_lead/{YYYY-MM-DD}/{TICKER}_catalysts_dossier.json
└── governance/
    ├── bull_bear_debates/{YYYY-MM-DD}/{TICKER}_debate.json
    └── executive_orders/{YYYY-MM-DD}/approved_tickets.json
```

---

## 6. Portfolio Risk, Sizing & Capital Allocation

### 6.1 Account Risk Allocation

- **Green Regime:** 1.0% total equity risk per trade.
- **Yellow Regime:** 0.5% total equity risk per trade.
- **Red Regime:** 0.0% (all new orders blocked).

### 6.2 Position Sizing Formulas

> **UNSPECIFIED — blocks implementation.**
> The source draft lists "Position Sizing Formulas" as a heading with no formula body.
> No formula has been inferred or invented here. The share-quantity calculation (and its
> rounding, minimum-size, and cap-interaction rules) must be defined before
> `manage_orders.py` or the backtester can be built. See
> [Appendix A](#appendix-a--unspecified-parameters-in-this-draft).

### 6.3 Concentration Boundaries

- **Max position cap:** No single position may exceed 20% of total account equity, regardless of
  how tight the stop loss is.
- **Sector exposure cap:** Maximum 25% of total portfolio equity in any single GICS sector.
- **Sub-industry cap:** Maximum 2 concurrent open positions within the same industry
  sub-sector.
- **Leverage:** Maximum gross exposure <= 100% (zero margin/leverage).

---

## 7. Execution Architecture & Order State Machine

Argus Trader uses a local CLI utility (`manage_orders.py`) to stage orders, while the evening
batch engine deterministically tracks order state transitions.

```
[ CLI: manage_orders.py set ]
             │
             ▼
      [ PENDING Queue ] ────────────► [ EXPIRED ]
      • Tracks ONLY:                  (If 3 days elapse without Low <= Limit X)
        Daily Low <= Limit X
             │
             ▼ (Condition Met: Fill at min(Open, Limit))
      [ ACTIVE HOLDING ]
      • NOW actively monitors:
        - Daily High >= TP Y ───────► [ CLOSED_TP ]
        - Daily Low <= SL Z  ───────► [ CLOSED_SL ]
```

### 7.1 State Machine Lifecycle Rules

**PENDING state**

- Holds orders staged via CLI: Limit Price (X), Take Profit (Y), Stop Loss (Z), and Share
  Quantity (N).
- Evaluates exactly one condition: `Daily Low <= Limit Price`.
- Take Profit and Stop Loss tracking are completely dormant while PENDING.
- If unfilled after 3 market sessions, transitions to `EXPIRED`.

**ACTIVE state**

- Entered automatically when a pending order is filled.
- Begins monitoring exits on subsequent sessions:
  - **Take Profit exit:** `Daily High >= TP Y` → exits at `max(Open, Y)`; logs `CLOSED_TP`.
  - **Stop Loss exit:** `Daily Low <= SL Z` → exits at `min(Open, Z)`; logs `CLOSED_SL`.

### 7.2 Execution Anomaly Tracking (Empirical Gap & Collision Metrics)

- **Same-bar collisions:** If a single bar has both `High >= Y` and `Low <= Z`, the conservative
  backtesting rule executes the Stop Loss first. The event is incremented in
  `same_bar_bracket_collisions`.
- **Down-gap slippage:** If a stock gaps down below the Stop Loss (`Open < Z`), execution occurs
  at the Open price. Realized slippage is logged in `slippage_drag_dollars`.

### 7.3 CLI Interface Specification

```bash
# Stage a new pending limit order
python scripts/manage_orders.py set NVDA --limit 128.50 --tp 142.50 --sl 122.10 --shares 150

# Inspect active pipeline status
python scripts/manage_orders.py status
```

---

## 8. Dashboard & Public Operations Log (GitHub Pages)

The user interface is a static HTML/CSS single-page dashboard compiled via Jinja2 templates
(`scripts/publish_dashboard.py`) and hosted directly on GitHub Pages (`docs/index.html`).

- **Macro Regime Banner:** Displays current market state (GREEN, YELLOW, RED), benchmark trends
  (SPY, QQQ, VIX), and breadth percentage.
- **Actionable Orders Table (tomorrow's session):** Displays valid PENDING orders with target
  entry, dynamic R:R, and remaining time-to-live (Day N of 3).
- **Active Portfolio Holdings:** Displays currently filled positions, unrealized P&L, days held,
  and distance to target/stop.
- **Historical Audit Ledger & Scorecard:** Displays completed trades, realized R-multiples, win
  rate, profit factor, and maximum drawdown.
- **Privacy Controls:** Shows risk allocation in risk units (R) and portfolio percentages, to
  allow public dashboard hosting without broadcasting raw personal dollar balances.

---

## 9. Retrospective Audit Engine

Executes immediately following daily order reconciliation whenever a trade transitions to
`CLOSED_TP`, `CLOSED_SL`, or `EXPIRED`.

### Root-Cause Taxonomy

| Classification | Outcome | Criteria |
|---|---|---|
| `THESIS_VALIDATED` | Win | Hit Take Profit within 15 sessions; volume confirmed the measured move. |
| `FALSE_BREAKOUT` | Loss | Stopped out within 3 sessions of entry; price failed to hold breakout level. |
| `OVERHEAD_ABSORPTION` | Loss | Stalled and reversed at overhead supply (POC / 200 SMA) flagged during the Bear debate. |
| `VOLATILITY_WHIPSAW` | Loss | Low pierced Stop Loss by < 0.25 × ATR before reversing directly toward the target. |
| `TIME_STOP_DECAY` | Scratch / Loss | Churns sideways for > 15 sessions without hitting target. |
| `MACRO_DRAG` | Loss | Stopped out during an intraday market-wide liquidity drop or Tier 0 regime downgrade. |

The engine writes audit notes to `data/audit_ledger.json` and outputs parameter-tuning
recommendations to calibrate sub-agent scoring weights.

---

## 10. Testing, Verification & CI/CD Regression Gates

Every pull request against `main` must pass an automated 5-year simulation run. Merges are
blocked if new logic degrades strategy performance.

### Test Results Directory Structure

```
test_results/
├── benchmark_latest.json                       # Production performance baseline
└── {YYYY-MM-DD}_PR-{ID}_{COMMIT_SHA}/
    ├── benchmark_summary.json                  # Candidate run metrics
    ├── closed_trades.parquet                   # Trade-by-trade ledger
    ├── retrospective_audit.md                  # Analysis of failures & recommendations
    └── anomaly_report.json                     # Same-bar collisions and gap counts
```

### Pull Request Regression Gate Rules

`scripts/verify_regression_gate.py` compares the candidate `benchmark_summary.json` against
`benchmark_latest.json`. A PR is blocked if any of the following occur:

| Gate | Blocking condition |
|---|---|
| Take Profit hit rate regression | `TP Rate(candidate) < TP Rate(baseline)` |
| Expectancy regression | `Expectancy R (candidate) < Expectancy R (baseline)` |
| Profit factor degradation | `Profit Factor(candidate) < Profit Factor(baseline)` |
| Drawdown expansion | `Max Drawdown(candidate) > Max Drawdown(baseline) + 1.0%` |

---

## 11. Complete Repository Layout

```
argus-trader/
├── .github/
│   └── workflows/
│       ├── daily_batch.yml                # Scheduled 5:58 PM ET daily cron
│       └── backtest_regression_gate.yml   # PR test gate & performance blocker
├── artifacts/
│   └── agents_data/                       # Daily serialized agent outputs
│       ├── technicals/
│       ├── fundamentals/
│       ├── catalysts/
│       └── governance/
├── data/
│   ├── raw/
│   │   └── polygon/                       # 5-year data in quarterly partitions (Parquet)
│   ├── pending_orders.json                # Staged CLI limit orders
│   ├── open_positions.json                # Active filled positions
│   ├── order_history.json                 # Realized trades and outcomes
│   └── audit_ledger.json                  # Retrospective classifications
├── docs/
│   ├── index.html                         # Live GitHub Pages dashboard
│   └── assets/                            # Styling and front-end scripts
├── scripts/
│   ├── fetch_historical_polygon.py        # 5-year backfill script
│   ├── manage_orders.py                   # Order registration CLI & daily reconciliation
│   ├── retrospective_audit.py             # Post-trade failure analysis engine
│   ├── publish_dashboard.py               # Jinja2 template rendering and git push
│   └── verify_regression_gate.py          # CI/CD PR evaluation script
├── src/
│   ├── tier0_macro/                       # SPY, QQQ, VIX regime engine
│   ├── tier1_screener/                    # Vectorized multi-archetype union
│   ├── tier2_agents/                      # Technical, fundamental, catalyst leads
│   └── governance/                        # Bull/Bear debate & Executive arbitration
├── templates/
│   └── dashboard.jinja                    # Dashboard HTML template
├── tests/
│   ├── run_full_backtest.py               # 5-year backtesting simulation runner
│   └── test_extrema_zigzag.py             # Unit tests for pivot math
├── test_results/
│   └── benchmark_latest.json              # Canonical production baseline
├── requirements.txt                       # Core dependencies
└── README.md                              # Repository overview and instructions
```

> **Note:** This layout is the *target* structure for the implementation phases. As of this
> commit the repository contains only `README.md`, `CHANGELOG.md`, and this design document —
> none of the directories above have been created yet.

---

## 12. Open Pre-Implementation Decisions

The following operational parameter will be settled prior to initiating codebase construction:

- **LLM candidate cap & batch ingestion:** Determine whether Tier 1 passes the top 15 or all
  30–35 candidates to Tier 2, and decide between standard synchronous API dispatch at 6:00 PM ET
  or queued asynchronous 24-hour Batch API processing.

---

## Appendix A — Unspecified Parameters in This Draft

These are gaps found in the source draft while transcribing it. They are **not** part of §12's
deliberate open decisions; they appear to be omissions. Nothing has been invented to fill them.

| # | Location | Gap | Impact if unresolved |
|---|---|---|---|
| A1 | [§6.2](#62-position-sizing-formulas) | "Position Sizing Formulas" is a heading with no formula. The share-quantity calculation, rounding rule, minimum position size, and the interaction between the risk-per-trade percentage and the 20% max-position cap are all undefined. | Blocks `manage_orders.py`, the backtester, and any risk validation. Highest-priority gap. |
| A2 | [§5.2](#52-adversarial-arbitration-protocol) | The Executive Decision Agent's approval criteria bullet list terminated after the R:R hurdle; the `E_Net` threshold was omitted. Recovered as `>= 25` from the §1 pipeline diagram and applied above. | Resolved by cross-reference, but should be confirmed by the author as intentional. |

### Additional questions worth settling alongside §12

These are observations, not defects in the draft — each is a design choice that the current
text leaves open:

- **Tier 0 parameter combination.** §3 gives four regime parameters (SPY MAs, QQQ trend, VIX,
  breadth) but not how they combine. Unanimous agreement, majority vote, and worst-of ("any
  single Red parameter forces Red") produce materially different halt frequencies.
- **Same-bar fill on entry day.** §7 states PENDING tracks only `Low <= Limit`, and that TP/SL
  are dormant while PENDING. It does not say whether TP/SL become live on the *fill bar itself*
  or only on the following session. This changes backtest results.
- **EXPIRED clock semantics.** "3 days" vs. "3 market sessions" are used in §1 and §7
  respectively; confirm sessions, and whether the staging day counts as Day 1.
- **Regime downgrade vs. open positions.** A Green → Red transition blocks *new* orders
  (§3, §6.1). Whether existing ACTIVE positions are force-closed, left to their brackets, or
  have stops tightened is not stated.
- **Baseline promotion.** §10 compares candidates to `benchmark_latest.json` but does not define
  when and by what process a merged PR's metrics become the new baseline.
