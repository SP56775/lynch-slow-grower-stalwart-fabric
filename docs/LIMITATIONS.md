
---

# 4. `docs/LIMITATIONS.md`

```markdown
# ⚠️ Technical & Methodological Limitations

Honesty about data and methodology constraints is essential in quantitative research. Below are the documented limitations of this platform.

---

## 1. Fundamental Data Depth Ceiling (Yahoo Finance)
* **The Constraint:** Yahoo Finance provides only a rolling window of **~4 annual statements** and **~4–6 quarters** for Indian equities (`.NS` tickers).
* **Analytical Impact:** While Peter Lynch evaluated 10-year earnings stability and recession resilience across multiple business cycles, our fundamental metrics (CAGRs, margin trends, ROE stability) are computed over a 3- to 4-year historical window.
* **Mitigation:** Long price and dividend histories (dating back 15+ years) are utilized to establish 10-year dividend consistency and historical median yield benchmarks.

---

## 2. Restatement & Point-in-Time Bias
* **The Constraint:** Financial statements accessed through free API wrappers reflect currently restated numbers rather than point-in-time disclosures originally filed with the exchange.
* **Analytical Impact:** Corporate restatements can retroactively modify historical metrics. While a 45-day publication lag is applied to mitigate look-ahead bias, it remains an approximation of exact filing dates.

---

## 3. Survivorship Bias in Universe Reference
* **The Constraint:** The screening universe is seeded with current constituents of the Nifty 500.
* **Analytical Impact:** Companies that were delisted, liquidated, or dropped from the index due to bankruptcy or severe underperformance over the past decade are absent from historical data. This inflates historical backtest results.

---

## 4. Execution vs. Simulation Variance (Paper Portfolio)
* **Slippage & Impact Costs:** The paper trading engine fills orders at the exact closing price of the screen date. Real-world execution is subject to bid-ask spreads, market impact, and liquidity constraints, particularly in mid-cap equities.
* **Corporate Actions & Dividends:** Cash accounting currently tracks cash dividends where reported, but complex corporate restructuring (demergers, rights issues, bonus stock distribution) requires manual oversight.
* **Tax Friction:** The simulation does not model Indian capital gains tax structures (Short-Term Capital Gains at 20% vs. Long-Term Capital Gains at 12.5% above ₹1.25L).