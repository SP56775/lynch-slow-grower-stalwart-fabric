# Pipeline Configuration

This file describes how the recurring Fabric pipeline for the Lynch Slow Grower and Stalwart screener is set up.

**Lakehouse:** `lh_lynch_value`

---

## Recurring pipeline

The pipeline runs seven notebook activities in strict sequence. Each activity starts only after the previous one succeeds.

```text
NB01 -> NB02 -> NB03 -> NB04a -> NB04b -> NB05 -> NB06
```

| Order | Notebook | Reads | Writes |
|---|---|---|---|
| 1 | `NB01_Universe` | Nifty 500 CSV, industry map, overrides | `bronze_universe` |
| 2 | `NB02_Yahoo_Bronze_Ingestion` | `bronze_universe`, config | Bronze Yahoo tables |
| 3 | `NB03_Lynch_Silver_Features` | Bronze Yahoo tables | `silver_lynch_features` |
| 4 | `NB04a_Slow_Growers` | `silver_lynch_features`, slow-grower config | `gold_slow_grower_screen`, `gold_slow_grower_candidates` |
| 5 | `NB04b_Stalwarts` | `silver_lynch_features`, stalwart config | `gold_stalwart_screen`, `gold_stalwart_candidates` |
| 6 | `NB05_Paper_Portfolio` | Split candidate tables, Bronze prices and dividends | `gold_lynch_portfolio`, `gold_lynch_cash`, `gold_lynch_transactions` |
| 7 | `NB06_Quarterly_Backtest` | Bronze prices, dividends and fundamentals | `backtest_qtr_trades`, `backtest_qtr_screens`, `backtest_qtr_summary`, `backtest_style_summary`, `backtest_screen_type_comparison` |

NB06 runs last. Nothing upstream depends on it, so the paper portfolio tables are already written before it starts.

---

## Requirements for every notebook activity

- Attach the project Lakehouse.
- Attach the Fabric Environment that has `yfinance` installed.
- Do not use inline `%pip install` inside pipeline notebooks. Install libraries through the Fabric Environment.

---

## Configuration files

Config files live in the Lakehouse Files area.

| File | Used by |
|---|---|
| `Files/config/lynch_parameters.csv` | NB02, NB03, NB05 and backtest parameters |
| `Files/config/slow_grower_parameters.csv` | NB04a |
| `Files/config/stalwart_parameters.csv` | NB04b |
| `Files/config/industry_classification.csv` | NB01 |
| `Files/config/symbol_overrides.csv` | NB01 |
| `Files/reference/nifty500_raw.csv` | NB01 |

Configuration files are authoritative. If this document and a config value differ, the config file and notebook code win.

---

## Checks after a run

- Review `bronze_yahoo_ingestion_log` for attempted, successful and failed symbols.
- Run the validation queries in the main README to confirm bucket counts, screen results, candidate counts and cash reconciliation.
- NB05 checks each candidate screen date against the corresponding screen table, so a stale candidate table is caught at that step.
