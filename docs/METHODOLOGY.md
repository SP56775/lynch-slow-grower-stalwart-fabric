

# 2. `docs/METHODOLOGY.md`

```markdown
# 🧠 Quantitative Peter Lynch Investment Methodology

This engine translates Peter Lynch’s qualitative lifecycle investing principles from *One Up On Wall Street* into mathematical, reproducible screening and portfolio allocation algorithms.

---

## Why Separate Slow Growers and Stalwarts?

Standard quantitative stock screeners use a single formula across all stocks (e.g., lowest P/E or highest ROE). This leads to **category errors**:
* A P/E of $25\times$ is expensive for a slow-growing utility, but a potential bargain for a consumer stalwart compounding at $15\%$.
* A dividend yield of $1.5\%$ is acceptable for a reinvesting compounder, but unacceptable for a mature slow grower whose entire thesis rests on cash return.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                           PETER LYNCH DUAL ENGINE                           │
├──────────────────────────────────────┬──────────────────────────────────────┤
│ 1. SLOW GROWERS                      │ 2. STALWARTS                         │
│ • Focus: High, Safe Dividends        │ • Focus: Earnings Compounding        │
│ • Growth: 0% – 12% CAGR              │ • Growth: 8% – 16% CAGR              │
│ • Hurdle: Yield ≥ 2.5%, FCF Cover    │ • Hurdle: P/E ≤ 1.5x Median, PEG     │
│ • Thesis: Cash generation & income   │ • Thesis: 30%–50% rotational gains   │
└──────────────────────────────────────┴──────────────────────────────────────┘