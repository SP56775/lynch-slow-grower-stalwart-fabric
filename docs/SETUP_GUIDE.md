# Setup Guide

Step-by-step instructions for deploying this project in a Microsoft Fabric workspace.

---

## Contents

- [Prerequisites](#prerequisites)
- [1. Create the workspace and Lakehouse](#1-create-the-workspace-and-lakehouse)
- [2. Create the Fabric Environment](#2-create-the-fabric-environment)
- [3. Create folder structure](#3-create-folder-structure)
- [4. Upload configuration files](#4-upload-configuration-files)
- [5. Download and upload the Nifty 500 constituent list](#5-download-and-upload-the-nifty-500-constituent-list)
- [6. Create the notebooks](#6-create-the-notebooks)
- [7. Run notebooks in order](#7-run-notebooks-in-order)
- [8. Create the data pipeline](#8-create-the-data-pipeline)
- [9. Create the Power BI semantic model](#9-create-the-power-bi-semantic-model)
- [10. Common problems](#10-common-problems)
- [11. What to run monthly](#11-what-to-run-monthly)

---

## Prerequisites

- Microsoft Fabric workspace with Lakehouse and Notebook support
- Basic familiarity with PySpark notebooks and Power BI
- A Fabric Environment with `yfinance` installed

---

## 1. Create the workspace and Lakehouse

1. Create a new workspace: `ws_lynch_value`
2. Create a new Lakehouse: `lh_lynch_value`
3. Keep this separate from any other project workspace

---

## 2. Create the Fabric Environment

Inline `%pip install` is blocked inside Fabric Data Pipelines. Install libraries through an Environment instead.

1. In the workspace, click **+ New item** → **Environment**
2. Name it: `env_lynch`
3. Under **Public libraries**, add `yfinance`
4. Click **Save**, then **Publish**
5. Wait for the Environment to finish publishing

Attach `env_lynch` to every notebook in this project.

---

## 3. Create folder structure

In the Lakehouse, under **Files**, create:

```text
Files/
├── config/
├── reference/
└── raw/
```

---

## 4. Upload configuration files

Upload these files to `Files/config/`:

| File | Source |
|---|---|
| `lynch_parameters.csv` | From this repository's `config/` folder |
| `slow_grower_parameters.csv` | From this repository's `config/` folder |
| `stalwart_parameters.csv` | From this repository's `config/` folder |
| `industry_classification.csv` | From this repository's `config/` folder |
| `symbol_overrides.csv` | From this repository's `config/` folder (header-only is valid) |

Verify they exist:

```python
import notebookutils
notebookutils.fs.ls("Files/config/")
```

---

## 5. Download and upload the Nifty 500 constituent list

1. Open your browser
2. Go to: `https://www.niftyindices.com/IndexConstituent/ind_nifty500list.csv`
3. Download the file
4. Rename it to `nifty500_raw.csv`
5. Upload it to `Files/reference/nifty500_raw.csv`

Expected columns: `Company Name, Industry, Symbol, Series, ISIN Code`

---

## 6. Create the notebooks

Create these 7 notebooks in the workspace:

| Notebook | Attached Lakehouse | Attached Environment |
|---|---|---|
| `NB01_Universe` | `lh_lynch_value` | `env_lynch` |
| `NB02_Yahoo_Bronze_Ingestion` | `lh_lynch_value` | `env_lynch` |
| `NB03_Lynch_Silver_Features` | `lh_lynch_value` | `env_lynch` |
| `NB04a_Slow_Growers` | `lh_lynch_value` | `env_lynch` |
| `NB04b_Stalwarts` | `lh_lynch_value` | `env_lynch` |
| `NB05_Paper_Portfolio` | `lh_lynch_value` | `env_lynch` |
| `NB06_Quarterly_Backtest_v2` | `lh_lynch_value` | `env_lynch` |

Paste each notebook's code from the `notebooks/` folder and save.

---

## 7. Run notebooks in order

```text
NB01 -> NB02 -> NB03 -> NB04a -> NB04b -> NB05
```

Run `NB06_Quarterly_Backtest_v2` separately when historical analysis is needed.

### Validation after NB01

```sql
SELECT bucket, COUNT(*) AS companies
FROM bronze_universe
GROUP BY bucket
ORDER BY companies DESC;

SELECT symbol, company_name, industry, bucket, reason
FROM bronze_universe
WHERE bucket IN ('UNMAPPED', 'REVIEW')
ORDER BY symbol;
```

Resolve any UNMAPPED or REVIEW rows by updating `industry_classification.csv` or `symbol_overrides.csv`, then re-run NB01.

### Validation after NB02

```sql
SELECT 'prices' AS src, COUNT(*) AS rows FROM bronze_yahoo_prices
UNION ALL SELECT 'dividends', COUNT(*) FROM bronze_yahoo_dividends
UNION ALL SELECT 'fundamentals', COUNT(*) FROM bronze_yahoo_fundamentals_long
UNION ALL SELECT 'market', COUNT(*) FROM bronze_yahoo_market_snapshot;

SELECT * FROM bronze_yahoo_ingestion_log ORDER BY run_timestamp DESC LIMIT 3;
```

### Validation after NB03

```sql
SELECT data_status, category_provisional, COUNT(*) AS stocks
FROM silver_lynch_features
GROUP BY data_status, category_provisional
ORDER BY data_status, stocks DESC;
```

### Validation after NB04a and NB04b

```sql
SELECT final_status, COUNT(*) AS stocks
FROM gold_slow_grower_screen
GROUP BY final_status;

SELECT final_status, COUNT(*) AS stocks
FROM gold_stalwart_screen
GROUP BY final_status;

SELECT COUNT(*) AS slow_candidates FROM gold_slow_grower_candidates;
SELECT COUNT(*) AS stalwart_candidates FROM gold_stalwart_candidates;
```

### Validation after NB05

```sql
SELECT cash_balance FROM gold_lynch_cash;

SELECT COUNT(*) AS holdings
FROM gold_lynch_portfolio
WHERE quantity > 0;

SELECT transaction_type, COUNT(*) AS n, SUM(amount) AS total
FROM gold_lynch_transactions
GROUP BY transaction_type;
```

---

## 8. Create the data pipeline

1. Click **+ New item** → **Data pipeline**
2. Name it: `PL_Lynch_Monthly_Screen`
3. Add 6 Notebook activities:

| Activity | Notebook | Timeout | Retry |
|---|---|---|---|
| `Act_01_Universe` | NB01_Universe | 00:15:00 | 1 |
| `Act_02_Yahoo_Bronze` | NB02_Yahoo_Bronze_Ingestion | 01:30:00 | 2 |
| `Act_03_Silver_Features` | NB03_Lynch_Silver_Features | 00:45:00 | 1 |
| `Act_04a_Slow_Growers` | NB04a_Slow_Growers | 00:20:00 | 1 |
| `Act_04b_Stalwarts` | NB04b_Stalwarts | 00:20:00 | 1 |
| `Act_05_Paper_Portfolio` | NB05_Paper_Portfolio | 00:20:00 | 1 |

4. For every activity, open **Settings** and set:
   - Lakehouse: `lh_lynch_value`
   - Environment: `env_lynch`
5. Connect the activities with green **On Success** arrows in order.
6. Click **Schedule** and set:
   - Run by schedule: On
   - Repeat: Monthly
   - Day: 1st
   - Time: 06:00
   - Time zone: (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi
7. Click **Apply**, then **Save**.

---

## 9. Create the Power BI semantic model

1. Open the Lakehouse and switch to **SQL analytics endpoint**
2. Click **Reporting** → **New semantic model**
3. Name it: `sem_lynch_value`
4. Storage mode: **Direct Lake on OneLake**
5. Select these tables:
   - `gold_slow_grower_screen`
   - `gold_slow_grower_candidates`
   - `gold_stalwart_screen`
   - `gold_stalwart_candidates`
   - `gold_lynch_portfolio`
   - `gold_lynch_cash`
   - `gold_lynch_transactions`
   - `silver_lynch_features`
   - `bronze_universe`
   - `bronze_yahoo_ingestion_log`
   - `backtest_qtr_trades`
   - `backtest_qtr_screens`
   - `backtest_screen_type_comparison`
6. Click **Confirm**

Do not create manual relationships between tables. Use DAX `CALCULATE` and `FILTER` instead. See `powerbi/dax_measures.md` for the complete measure and page specification.

---

## 10. Common problems

| Problem | Check |
|---|---|
| `bronze_universe` not found | Notebook attached to the wrong Lakehouse, or NB01 not run |
| Config file not found | File must be exactly `Files/config/<name>.csv` |
| `yfinance` import error | Environment `env_lynch` is not attached to the notebook |
| Yahoo returns 0 rows | Check `bronze_yahoo_ingestion_log`; wait and rerun |
| No candidates in screen | Query `status_reason` in the screen tables; do not relax rules blindly |
| NB05 drops holdings | Verify you are running NB05 v2 with persistence logic |
| `%pip install` blocked | Remove inline installs; use the Fabric Environment instead |
| Duplicate column in NB05 | Do not add `SELECT *, 'X' AS candidate_type` when the column already exists |

---

## 11. What to run monthly

The pipeline handles steps 1 through 6 above automatically on the 1st of each month.

Run manually:

- **NB06** when you want to compare quarterly vs. annual backtest results
- **NB01** whenever the Nifty 500 constituent list changes (update `nifty500_raw.csv` first)
- **Power BI refresh** after the pipeline completes (Direct Lake should pick it up automatically)
