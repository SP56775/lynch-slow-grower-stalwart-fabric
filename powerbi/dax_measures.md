# Power BI: Semantic Model and DAX

**Lakehouse:** `lh_lynch_value`
**Storage:** Direct Lake on OneLake

---

## Tables to include

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

Do not add the obsolete combined `gold_lynch_candidates` table.

---

## Relationships

| From | To | Cardinality | Direction |
|---|---|---|---|
| `gold_slow_grower_candidates[symbol]` | `silver_lynch_features[symbol]` | Many to One | Single |
| `gold_stalwart_candidates[symbol]` | `silver_lynch_features[symbol]` | Many to One | Single |
| `gold_lynch_portfolio[symbol]` | `silver_lynch_features[symbol]` | Many to One | Single |
| `gold_lynch_transactions[symbol]` | `silver_lynch_features[symbol]` | Many to One | Single |

---

## Measures

### Screening counts

```dax
Slow BUY Count =
CALCULATE ( COUNTROWS ( 'gold_slow_grower_candidates' ) )
```

```dax
Slow WATCH Count =
CALCULATE (
    COUNTROWS ( 'gold_slow_grower_screen' ),
    'gold_slow_grower_screen'[final_status] = "SLOW_GROWER_WATCH"
)
```

```dax
Stalwart BUY Count =
CALCULATE ( COUNTROWS ( 'gold_stalwart_candidates' ) )
```

```dax
Stalwart WATCH Count =
CALCULATE (
    COUNTROWS ( 'gold_stalwart_screen' ),
    'gold_stalwart_screen'[final_status] = "STALWART_WATCH"
)
```

### Paper portfolio

```dax
Invested =
SUM ( 'gold_lynch_portfolio'[market_value] )
```

```dax
Cash Balance =
MAX ( 'gold_lynch_cash'[cash_balance] )
```

```dax
Total Account =
[Invested] + [Cash Balance]
```

```dax
Holdings Count =
COUNTROWS ( 'gold_lynch_portfolio' )
```

```dax
Unrealized PnL =
SUM ( 'gold_lynch_portfolio'[unrealized_pnl] )
```

```dax
Dividends Received =
CALCULATE (
    SUM ( 'gold_lynch_transactions'[amount] ),
    'gold_lynch_transactions'[transaction_type] = "DIVIDEND"
)
```

```dax
Buys Count =
CALCULATE (
    COUNTROWS ( 'gold_lynch_transactions' ),
    'gold_lynch_transactions'[transaction_type] = "BUY"
)
```

### Candidate averages

```dax
Avg Slow Yield =
AVERAGE ( 'gold_slow_grower_candidates'[ttm_dividend_yield] )
```

```dax
Avg Stalwart PEG =
AVERAGE ( 'gold_stalwart_candidates'[dividend_adjusted_peg] )
```
