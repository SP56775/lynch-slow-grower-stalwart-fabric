# Methodology

Lynch-inspired quantitative proxies for Slow Growers and Stalwarts. These are not complete implementations of Lynch's methods and are not investment advice.

> Configuration files are authoritative. If this document and a config value differ, the config file and notebook code win.

---

## Contents

1. [Universe and Classification (NB01)](#1-universe-and-classification-nb01)
2. [Yahoo Finance Ingestion (NB02)](#2-yahoo-finance-ingestion-nb02)
3. [Silver Feature Calculation (NB03)](#3-silver-feature-calculation-nb03)
4. [Slow Grower Screen (NB04a)](#4-slow-grower-screen-nb04a)
5. [Stalwart Screen (NB04b)](#5-stalwart-screen-nb04b)
6. [Paper Portfolio (NB05)](#6-paper-portfolio-nb05)
7. [Backtest (NB06)](#7-backtest-nb06)
8. [What These Screens Cannot Measure](#8-what-these-screens-cannot-measure)
9. [When Config and Documentation Disagree](#9-when-config-and-documentation-disagree)

---

## 1. Universe and Classification (NB01)

**Source:** Nifty 500 constituent CSV from Nifty Indices, mapped through `industry_classification.csv` and `symbol_overrides.csv`.

| Bucket | Meaning | Flows to screening? |
|---|---|---|
| ELIGIBLE | Non-financial, non-cyclical candidate | Yes |
| UTILITY | Regulated utility with different debt/coverage thresholds | Yes |
| CYCLICAL | Earnings depend on commodity or capex cycles | No |
| FINANCIAL | Banks, NBFCs, insurance; needs different metrics | No |
| REVIEW | Mixed business or unclear classification | No until manually resolved |
| UNMAPPED | Industry not in the classification file | No until added |

Only `in_scope = true` rows (ELIGIBLE + UTILITY) proceed to NB02.

---

## 2. Yahoo Finance Ingestion (NB02)

For each in-scope `.NS` ticker:

| Yahoo call | Stored in | Purpose |
|---|---|---|
| `Ticker.history(auto_adjust=False)` | `bronze_yahoo_prices` | Prices with both `close` (split-adjusted) and `adj_close` (split + dividend adjusted) |
| `Ticker.dividends` | `bronze_yahoo_dividends` | Full dividend history with ex-dates |
| `income_stmt`, `balance_sheet`, `cashflow` | `bronze_yahoo_fundamentals_long` (ANNUAL) | Annual statements |
| Quarterly variants | `bronze_yahoo_fundamentals_long` (QUARTERLY) | Quarterly statements |
| `Ticker.info` / `fast_info` | `bronze_yahoo_market_snapshot` | Market cap, shares, sector |

Statements are stored in long format:

```text
symbol, yahoo_symbol, period_type, statement_type,
period_end, source_field, value, ingested_at, run_id
```

Incremental price logic re-downloads a configurable overlap window (default 10 days) per symbol to capture revisions. Fundamentals are overwritten only when enough symbols succeed (configurable threshold).

---

## 3. Silver Feature Calculation (NB03)

One row per in-scope company. Key feature groups:

### Growth

| Metric | Calculation | Source |
|---|---|---|
| EPS CAGR (3-year) | Latest annual EPS vs. EPS 3 annual intervals earlier, requiring both positive | Annual statements |
| Revenue CAGR (3-year) | Same pattern for total revenue | Annual statements |
| Quarterly EPS YoY | Latest quarterly EPS vs. same quarter 4 quarters earlier | Quarterly statements |
| Quarterly revenue YoY | Same pattern for revenue | Quarterly statements |

### Profitability and quality

| Metric | Calculation |
|---|---|
| Annual ROE | Net Income / Equity for matched annual periods |
| Minimum ROE | Minimum annual ROE across available recent periods |
| Operating margin decline count | Count of year-over-year margin drops in available annual periods |

### Leverage and coverage

| Metric | Calculation |
|---|---|
| Debt-to-equity | Total Debt / Equity (prefer total debt; derive from long + short if absent) |
| Interest coverage | EBIT / absolute Interest Expense |

### Cash-flow quality

| Metric | Calculation |
|---|---|
| OCF/NI | Cumulative Operating Cash Flow / cumulative Net Income on matched periods with positive NI |
| FCF payout | absolute(Dividends Paid) / (OCF - absolute(CapEx)) or reported FCF, averaged over available years |
| Capex/Depreciation | absolute(CapEx) / absolute(Depreciation) over available years |

### Dividend metrics

| Metric | Calculation | History source |
|---|---|---|
| TTM dividend yield | Dividends with ex-date in last 365 days / latest close | Price + dividend tables |
| Own median yield | Median of fiscal-year DPS / price near fiscal-year-end over available history | Price + dividend tables |
| Dividend years | Count of fiscal years with DPS > 0 in measured window | Dividend table |
| No-cut check | No fiscal year in the last 5 observable years where DPS dropped below 95% of a prior positive year | Dividend table |
| DPS CAGR | Growth in fiscal-year DPS totals over available intervals | Dividend table |

### Valuation

| Metric | Calculation | Basis |
|---|---|---|
| Current P/E | Price / TTM EPS if four valid quarterly periods exist; else price / latest annual EPS | Labelled in `pe_basis` |
| Own median P/E | Median of annual price-near-period-end / annual EPS over available history | Short window |
| Peer median P/E | Industry median of current P/E across screened universe | Cross-sectional |
| P/E vs own median | Current P/E / own median P/E | Ratio |
| P/E vs peer median | Current P/E / industry peer median P/E | Ratio |

### Dividend-adjusted PEG

```text
dividend_adjusted_peg = (EPS CAGR % + TTM dividend yield %) / P/E
```

**Higher is better.** This is not Yahoo's PEG field. It must not be inverted.

### Data status

| Status | Meaning |
|---|---|
| `FEATURES_CALCULATED` | Price and fundamental data sufficient for calculations |
| `LIMITED_FUNDAMENTAL_HISTORY` | Fewer than 4 annual statement periods |
| `INSUFFICIENT_PRICE_DATA` | No price data for the symbol |

`data_notes` records specific coverage gaps (e.g., ROE available for 2 of 4 periods, P/E basis is annual EPS not TTM).

---

## 4. Slow Grower Screen (NB04a)

### Thesis

A Slow Grower is a mature company with modest earnings growth that returns a high, consistent dividend supported by a healthy balance sheet. The investment rationale is income and capital preservation.

**Config file:** `slow_grower_parameters.csv`

### Hard gates

| Gate | Default | Rationale |
|---|---|---|
| TTM dividend yield | >= 3% | Primary income filter; high current payout |
| Dividend years | >= 5 | Consistency; proven payout record |
| No dividend cut (last 5 years) | When required | Safety; cutting signals distress |
| FCF payout | <= 80% | Sustainability; earnings must cover dividends |
| Debt/Equity | <= 0.75 | Conservative leverage; Lynch's normal balance sheet preference |
| Interest coverage | >= 4.0x | Debt serviceability; can service obligations comfortably |
| Minimum ROE | >= 10% | Quality floor; business must generate returns |
| EPS CAGR | <= 12% | Category boundary; excludes faster growers |
| P/E | <= 50 | Valuation cap; avoid extreme multiples |

### What is NOT used

- **Dividend-adjusted PEG as a hard gate.** PEG is unstable when growth is near zero, which is common for mature dividend payers.
- **Quarterly EPS growth as a requirement.** Quarterly momentum may appear as a bonus or diagnostic, not a gate.

### Ranking

Candidates are ranked using a `dividend_score` combining yield, dividend years, FCF payout safety and leverage.

---

## 5. Stalwart Screen (NB04b)

### Thesis

A Stalwart is a large, established company with durable earnings growth that can be bought at a disciplined valuation relative to its own history and peers.

**Config file:** `stalwart_parameters.csv`

### Hard gates

| Gate | Default | Rationale |
|---|---|---|
| Market cap | >= Rs. 5,000 Cr | Stability and liquidity floor; established company |
| EPS CAGR | 8%–16% | Category boundary; durable but not explosive growth |
| Minimum ROE | >= 12% | Quality floor; strong return on equity |
| Debt/Equity | <= 0.50 | Conservative leverage; growth funded by earnings, not debt |
| Interest coverage | >= 5.0x | Debt serviceability; conservative threshold |
| OCF/NI | >= 0.80 | Cash-backed earnings; profits must be real |
| P/E vs own median | <= 1.5x | Valuation discipline; not paying far above own history |
| P/E vs peer median | <= 1.3x | Relative valuation; not overpaying vs industry |
| Dividend-adjusted PEG | >= minimum | Lynch-style value; higher is better |

### Ranking

Candidates are ranked primarily by dividend-adjusted PEG (descending), then P/E vs own median (ascending), then EPS CAGR (descending).

---

## 6. Paper Portfolio (NB05)

| Rule | Design |
|---|---|
| Starting capital | Rs. 10,00,000 (configurable) |
| Candidate sources | Latest `gold_slow_grower_candidates` and `gold_stalwart_candidates` with staleness check against screen tables |
| Duplicate prevention | A symbol appearing in both lists is bought only once |
| Existing holdings | Preserved across runs |
| New buys | Only symbols not already held |
| Position cap | Configured percentage of account value per stock |
| Industry cap | Configured maximum holdings per industry |
| Price used | Latest available close from `bronze_yahoo_prices` |
| Dividends | Accrued once per symbol and ex-date after the simulated buy date, credited to `gold_lynch_cash` |
| Automated sells | Not included in current version |

---

## 7. Backtest (NB06)

### Design

NB06 backtests Slow Growers and Stalwarts separately using their own entry criteria, then compares both styles and both rebalance frequencies (quarterly vs. annual).

### Point-in-time snapshot

Each screen date builds a snapshot using only statement periods with `period_end <= screen_date - publication_lag_days`.

### Slow Grower backtest criteria

- Dividend yield and dividend consistency (dividend years, no-cut check)
- Dividend safety (FCF payout, debt, coverage)
- Annual EPS CAGR cap (excludes fast growers)
- P/E cap

### Stalwart backtest criteria

- Annual EPS CAGR band
- Quarterly EPS and sales YoY where available
- ROE, leverage, cash-flow quality
- P/E vs own history
- Dividend-adjusted PEG

### Entry and exit

| Item | Method |
|---|---|
| Entry price | First available close on or after screen date (tolerance: 10 days) |
| Exit price | First available close on or after holding-period end (tolerance: 30–45 days) |
| Stop-loss | Configurable percentage below entry (Slow Grower and Stalwart use different defaults) |
| Transaction cost | Configurable percentage per round trip |
| Dividend income | Sum of dividends between screen date and exit date, multiplied by quantity |
| Benchmark | Nifty 50 same-window return |

### Output tables

`backtest_qtr_trades`, `backtest_qtr_screens`, `backtest_qtr_summary`, `backtest_style_summary`, `backtest_screen_type_comparison`.

---

## 8. What These Screens Cannot Measure

The notebooks automate quantitative factors available from Yahoo data. The following require manual research:

- Industry dynamics, competitive position, regulatory risk
- Management quality and capital-allocation decisions
- Hidden or mispriced assets and realization catalysts
- Customer concentration and dependency risk
- Promoter pledge and insider buying trends
- Dividend policy sustainability beyond historical pattern

Lynch's framework emphasised human judgment in these areas. The automated screens are quantitative proxies and starting points, not substitutes for research.

---

## 9. When Config and Documentation Disagree

Configuration files are authoritative. If a threshold in this document differs from the value in a config CSV file, the config file wins. Check `Files/config/` in the Lakehouse for current values before interpreting any result.
