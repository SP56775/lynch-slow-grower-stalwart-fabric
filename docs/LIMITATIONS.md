# Limitations

Read this before trusting any output from this project.

> **Disclaimer:** This is a research prototype. It is not investment advice and not a live trading system.

---

## Contents

1. [Yahoo Finance Data Constraints](#1-yahoo-finance-data-constraints)
2. [Classification Is Provisional](#2-classification-is-provisional)
3. [Survivorship Bias](#3-survivorship-bias)
4. [Restatement and Look-Ahead Risk](#4-restatement-and-look-ahead-risk)
5. [Coverage Varies by Symbol](#5-coverage-varies-by-symbol)
6. [Paper Portfolio Is a Simulation](#6-paper-portfolio-is-a-simulation)
7. [Backtest Constraints](#7-backtest-constraints)
8. [What the Screens Cannot Measure](#8-what-the-screens-cannot-measure)
9. [Cross-Project Contamination](#9-cross-project-contamination)
10. [Summary](#summary)

---

## 1. Yahoo Finance Data Constraints

### Short fundamental history

Yahoo annual financial statements cover approximately 4 years for most symbols. This means:

- 3-year EPS CAGR requires all 4 years to be present and positive.
- Own median P/E is computed from a small number of points.
- There is no true 10-year fundamental classification from Yahoo alone.
- Recession resilience (such as FY2020–21 earnings behaviour) cannot be tested from annual statements alone.

Dividend history and daily price history are longer (often 10–20 years) but still vary by company.

### Unofficial library

`yfinance` is an unofficial wrapper around Yahoo Finance web endpoints. Field names, coverage, rate limits and availability can change without notice. The project maps Yahoo source fields through a canonical mapping table, but new or renamed fields will appear as null until the mapping is updated.

### Price delay

Yahoo NSE prices are typically delayed (often around 15 minutes during market hours). This project does not use real-time or tick-level data.

### Statements update after publication

Financial statements appear on Yahoo only after companies publish them. The project uses period-end dates plus an assumed publication lag (configurable, default 45 days) as a proxy for filing dates. This is an approximation, not exact point-in-time disclosure timing.

---

## 2. Classification Is Provisional

Labels such as `PROVISIONAL_SLOW_GROWER` and `PROVISIONAL_STALWART` mean the company falls within the configured growth band on the available data. They do not mean the company has held that category for a decade. A full Lynch-style classification would require 10+ years of earnings, dividend and price history.

---

## 3. Survivorship Bias

The universe is based on the current Nifty 500 constituent list. Companies that were removed from the index or delisted in the past are absent. Any historical backtest or comparison built on this universe favours companies that survived. This inflates absolute return estimates.

---

## 4. Restatement and Look-Ahead Risk

Yahoo may show revised financial statements, not the figures as originally reported. The project uses period-end dates and an assumed publication lag rather than exact historical filing timestamps. Without actual filing dates from regulatory filings, small look-ahead risk remains in backtests.

---

## 5. Coverage Varies by Symbol

- Some symbols lack interest expense, inventory, dividends-paid, or quarterly EPS fields.
- Missing metrics fail the relevant gates with an explicit reason. They are not treated as passes.
- Quarterly TTM EPS falls back to latest annual EPS when four valid quarters are unavailable. The basis is labelled in the `pe_basis` column.
- Dividend history depth varies. Some newer listings have fewer than 5 observable dividend years.

---

## 6. Paper Portfolio Is a Simulation

- No broker connection and no real orders are placed.
- Buys use the latest available closing price. No slippage, brokerage fees, STT, taxes or market impact are modelled.
- Dividends are accrued from Yahoo ex-dates, gross of tax.
- The current NB05 version does buys and tracking only. Automated sell, stop-loss and review logic are not included.
- Position and industry caps are simulation assumptions, not risk-managed allocations.
- Checks run on the pipeline schedule, not continuously or in real time.

---

## 7. Backtest Constraints

- Point-in-time snapshots use statement periods available before the screen date after an assumed publication lag. Exact filing dates are not available.
- Quarterly fundamental coverage is uneven, especially for older quarters. Some Stalwart screens may fall back to annual data.
- Consecutive backtest screens can hold overlapping positions. Trades are not fully independent observations.
- A short backtest window, especially one covering a limited number of market regimes, does not establish a persistent statistical edge.
- Survivorship bias (see [section 3](#3-survivorship-bias)) inflates backtest results.
- No backtest performance figures are published in this README. Inspect the backtest tables after a run and apply these limitations before interpreting them.

---

## 8. What the Screens Cannot Measure

The following require manual research and are not captured by the notebooks:

| Factor | Why it is not automated |
|---|---|
| Industry dynamics and competitive position | Requires industry-specific knowledge and qualitative analysis |
| Management quality and capital-allocation decisions | Not available as structured data from Yahoo |
| Hidden or mispriced assets | Asset value and realization catalysts require reading footnotes and filings |
| Customer concentration | Requires annual report review |
| Promoter pledge and insider buying trends | Requires NSE/BSE shareholding filings |
| Regulatory risk and policy changes | Requires news and policy monitoring |
| Dividend sustainability beyond historical pattern | A past record does not guarantee future policy |

These are the areas where Lynch's own framework emphasised human judgment. The automated screens are quantitative proxies and starting points, not substitutes for research.

---

## 9. Cross-Project Contamination

This project must not share tables or notebooks with other Fabric projects. If tables or schemas overlap with another workspace or Lakehouse, results can silently mix. Keep `ws_lynch_value` and `lh_lynch_value` fully separate.

---

## Summary

This project automates data collection, metric calculation, rule-based screening, portfolio simulation and historical comparison for two Lynch-inspired investment categories. Its outputs are exploratory. They are not forecasts, not guarantees, and not financial advice.
