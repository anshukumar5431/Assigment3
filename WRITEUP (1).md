# Write-Up: Trader Performance vs Market Sentiment

Primetrade.ai Data Science Intern — Round-0 Assignment

## Methodology

I merged two datasets at daily granularity: the Hyperliquid trade-level export (211,224
trades, 32 accounts, May 2023 to May 2025) and the Bitcoin Fear/Greed Index (2018-2025,
trimmed down to the overlapping window). Both datasets were already pretty clean — no
missing values, no duplicates — which made life easier. Each trade got mapped to its
calendar date (I used IST) and joined to that day's sentiment label. Almost everything
matched (99.997%), so I just dropped the handful of trades that fell outside the sentiment
index's coverage rather than trying to patch them.

From there I built two metric tables:

- Account-day level: daily PnL, win rate, trade count, avg trade size, a leverage proxy,
  long/short ratio, and a running drawdown-from-peak.
- Market-wide daily level: same metrics but aggregated across all active accounts for
  that day.

Quick note on the leverage proxy — the raw data doesn't have an actual leverage/margin
column, so I approximated it using each trade's size relative to that account's own
median trade size. It's not "real" leverage, more of an aggressiveness signal, but it's
flagged as such everywhere it's used so it doesn't get over-interpreted.

For comparing Fear vs Greed I used Mann-Whitney U tests instead of a t-test, since PnL is
heavily right-skewed (a few big winning days pull the mean around a lot) and Mann-Whitney
doesn't assume normality. I also split accounts three ways using median splits — high vs
low leverage-proxy, frequent vs infrequent traders, and consistent-winners vs everyone
else (by average win rate) — to see if the sentiment effect was uniform or concentrated
in a subset of traders.

## Key Insights

**1. The "Fear days are more profitable" thing isn't really a market-wide effect — it's
mostly a high-leverage, high-frequency trader thing.**

At the aggregate level, mean daily PnL is a bit higher on Fear days (\$5,185) than Greed
days (\$4,144), but honestly that gap doesn't clear significance overall (p ≈ 0.06, so
borderline). It's only once you split by leverage that the picture gets clearer:
high-leverage accounts pull in ~\$10.8k/day on Fear days vs ~\$4.8k on Greed days, while
low-leverage accounts barely make anything on Fear days (~\$209/day) and are actually
better off on Greed days (~\$3.8k/day). Frequent traders show basically the same split
(~\$7.4k Fear vs ~\$2.7k Greed); infrequent traders barely show a gap at all.

**2. Traders are taking on more risk during Fear, but their actual skill/edge doesn't
go up to match.**

The leverage proxy is noticeably higher on Fear days (7.58 vs 5.02, p ≈ 0.001), average
trade size is bigger (\$8,530 vs \$5,955), and people are trading more often (105 vs 77
trades/account/day). But win rate barely moves — 84.2% vs 85.6%, not significant
(p ≈ 0.26). So people are placing bigger, more frequent bets during Fear without actually
getting better at picking them. That's more variance, not more edge.

**3. This particular trader group is buying dips into Fear, not panic-selling.**

Market-wide, the buy:sell ratio is more long-skewed on Fear days (2.24) compared to Greed
days (1.63). That's kind of the opposite of what I expected going in — I assumed Fear
would mean people are dumping positions, but it looks more like contrarian/dip-buying
behavior. This also ties back into point 1 — it's why the high-leverage active traders
can actually turn Fear-day volatility into profit while the rest of the base just eats
the extra risk.

**4. This isn't one homogeneous group of traders.**

I ran KMeans (k=4, checked with an elbow plot) on PnL, frequency, win rate, leverage
proxy, trade size, and PnL volatility at the account level. It splits out a large cluster
of steady, moderate-size traders and a much smaller group of high-volume, high-PnL,
high-volatility "power accounts." Basically confirms that any single sentiment-based rule
is going to help some accounts and hurt others — there's no one-size-fits-all here.

## Strategy Recommendations

1. Only scale up exposure during Fear regimes for traders who already have a track
   record of high-leverage/high-frequency trading during Fear — and actually scale down
   for everyone else. The Fear-day premium is real, but it's segment-specific, so
   applying it as a blanket rule would probably hurt more accounts (the low-leverage,
   infrequent ones) than it helps.

2. Treat elevated leverage/long-bias on Fear days as a risk flag, not a green light to
   buy. Since win rate doesn't actually improve when position sizes go up during Fear, a
   safer default is to widen stop-losses and cap how much position size can increase per
   trade (something like +20-30% over baseline) unless the account has a proven history
   of being profitable on Fear days specifically.

## Bonus

I also threw together a quick next-day profitability classifier using sentiment plus
that day's behavior features. Random Forest got to ROC AUC ≈ 0.66, Logistic Regression
only ≈ 0.57. Not great, but not nothing either — there's a modest signal there, with
today's PnL and win rate being the strongest features. Given this is a Round-0
assignment, I'm treating this as "the pipeline works end-to-end," not "here's a
production trading signal." Would need a lot more validation before I'd trust it beyond
that.
