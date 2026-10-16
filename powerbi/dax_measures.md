\## \`dax\_measures.md\`\`\`\`markdown# Power BI: Semantic Model and DAXLakehouse: \`lh\_lynch\_value\`Storage: Direct Lake on OneLake## Tables to include- \`gold\_slow\_grower\_screen\`- \`gold\_slow\_grower\_candidates\`- \`gold\_stalwart\_screen\`- \`gold\_stalwart\_candidates\`- \`gold\_lynch\_portfolio\`- \`gold\_lynch\_cash\`- \`gold\_lynch\_transactions\`- \`silver\_lynch\_features\`- \`bronze\_universe\`- \`bronze\_yahoo\_ingestion\_log\`Do not add the obsolete combined \`gold\_lynch\_candidates\` table.## Relationships| From | To | Cardinality | Direction ||---|---|---|---|| \`gold\_slow\_grower\_candidates\[symbol\]\` | \`silver\_lynch\_features\[symbol\]\` | Many to One | Single || \`gold\_stalwart\_candidates\[symbol\]\` | \`silver\_lynch\_features\[symbol\]\` | Many to One | Single || \`gold\_lynch\_portfolio\[symbol\]\` | \`silver\_lynch\_features\[symbol\]\` | Many to One | Single || \`gold\_lynch\_transactions\[symbol\]\` | \`silver\_lynch\_features\[symbol\]\` | Many to One | Single |## Measures\`\`\`daxSlow BUY Count =CALCULATE ( COUNTROWS ( 'gold\_slow\_grower\_candidates' ))

Slow WATCH Count =CALCULATE ( COUNTROWS ( 'gold\_slow\_grower\_screen' ), 'gold\_slow\_grower\_screen'\[final\_status\] = "SLOW\_GROWER\_WATCH")

Stalwart BUY Count =CALCULATE ( COUNTROWS ( 'gold\_stalwart\_candidates' ))

Stalwart WATCH Count =CALCULATE ( COUNTROWS ( 'gold\_stalwart\_screen' ), 'gold\_stalwart\_screen'\[final\_status\] = "STALWART\_WATCH")

Invested =SUM ( 'gold\_lynch\_portfolio'\[market\_value\] )

Cash Balance =MAX ( 'gold\_lynch\_cash'\[cash\_balance\] )

Total Account =\[Invested\] + \[Cash Balance\]

Holdings Count =COUNTROWS ( 'gold\_lynch\_portfolio' )

Unrealized PnL =SUM ( 'gold\_lynch\_portfolio'\[unrealized\_pnl\] )

Dividends Received =CALCULATE ( SUM ( 'gold\_lynch\_transactions'\[amount\] ), 'gold\_lynch\_transactions'\[transaction\_type\] = "DIVIDEND")

Buys Count =CALCULATE ( COUNTROWS ( 'gold\_lynch\_transactions' ), 'gold\_lynch\_transactions'\[transaction\_type\] = "BUY")

Avg Slow Yield =AVERAGE ( 'gold\_slow\_grower\_candidates'\[ttm\_dividend\_yield\] )

Avg Stalwart PEG =AVERAGE ( 'gold\_stalwart\_candidates'\[dividend\_adjusted\_peg\] )