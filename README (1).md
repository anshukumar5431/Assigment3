# Trader Performance vs Market Sentiment (Hyperliquid × Bitcoin Fear/Greed Index)

Data Science / Analytics Intern — Round-0 Assignment (Primetrade.ai)

This project looks at how Bitcoin market sentiment (the Fear/Greed Index) relates to how
traders actually behave and perform on Hyperliquid, and tries to turn that into a couple
of practical trading rules of thumb.

## Repo structure

```
.
├── data/
│   ├── historical_data.csv          # raw Hyperliquid trade-level export (~211k rows)
│   └── fear_greed_index.csv         # raw Bitcoin Fear/Greed Index (daily, 2018-2025)
├── notebooks/
│   ├── Trader_Sentiment_Analysis.ipynb   # main deliverable - full analysis, already run
│   ├── 01_data_prep.py              # script version of Part A
│   ├── 02_metrics.py                # script version of the metric-building step
│   ├── 03_analysis.py               # script version of Part B
│   └── 04_bonus_model_clustering.py # script version of the bonus section
├── outputs/
│   ├── charts/                      # PNG charts (same ones shown in the notebook)
│   ├── table_*.csv                  # summary tables used in the write-up
│   ├── stat_tests.txt               # Mann-Whitney U test results
│   └── bonus_model_report.txt       # classification reports for the bonus model
├── WRITEUP.md                       # 1-pager: methodology, insights, strategy
└── README.md                        # this file
```

## How to run it

1. Set up an environment (Python 3.10+) and install the requirements:
   ```bash
   pip install pandas numpy matplotlib scipy scikit-learn jupyter
   ```
2. Run `notebooks/Trader_Sentiment_Analysis.ipynb` top to bottom. If you'd rather run
   plain scripts instead of the notebook, the numbered `.py` files do the same thing —
   just run them in order (`01` → `02` → `03` → `04`) from inside `notebooks/`, since
   they read from `../data/` and write to `../outputs/`.
3. Charts/tables land in `outputs/`, and they're also shown inline in the notebook.

## Datasets

1. **Bitcoin Fear/Greed Index** (`fear_greed_index.csv`) — daily sentiment label
   (Extreme Fear through Extreme Greed), 2,644 days, Feb 2018 to May 2025.
2. **Hyperliquid trade data** (`historical_data.csv`) — 211,224 individual trade
   executions, 32 accounts, 246 coins, May 2023 to May 2025. Includes execution price,
   size, side, PnL, fees, and leverage-related fields.

Both were already clean going in (no nulls, no dupes). I aligned them at daily
granularity — each trade gets mapped to its calendar date (IST) and joined to that day's
sentiment label. Only 6 out of 211,224 trades fell outside the Fear/Greed Index's date
range, so I just dropped those rather than trying to fill them in.

## Methodology, briefly

- **Part A** — load and sanity-check both datasets, merge them, and build out the
  account-day and market-wide daily metric tables (PnL, win rate, trade count, avg size,
  leverage proxy, long/short ratio, drawdown-from-peak).
- **Part B** — compare Fear vs Greed regimes: descriptive stats, Mann-Whitney U tests,
  and three account segmentations (by leverage, frequency, and consistency).
- **Part C** — turn the findings into two concrete trading rules of thumb.
- **Bonus** — a next-day profitability classifier (Logistic Regression + Random Forest)
  plus a KMeans clustering of trader types.

Full write-up with all the numbers and reasoning is in `WRITEUP.md`. All the code, charts
and tables live in the notebook.

## A note on the leverage proxy

There's no actual leverage/margin column in the raw Hyperliquid data, so I approximated
it: each trade's USD size relative to that same account's own median trade size (so a
value above 1 means the trader sized up beyond what's typical for them that day). It's a
within-account relative measure, not real position leverage — just want to be upfront
about that here and in the notebook so it's not read as more precise than it is.
