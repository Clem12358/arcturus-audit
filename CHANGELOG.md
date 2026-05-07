# Methodology Changelog

## 2026-05 — methodology v6 (universe filter + data-quality guardrail)

This is hopefully the **last "non-live" rerun**. The official live launch is
pushed to **June 2026** (the next monthly cron). The intent is that the live
track record begins from a stable methodology, with no mid-flight strategy
changes once it is running.

### Why this change
An internal audit of the April 30 portfolio surfaced several anomalous
holdings where the optimizer had selected stocks with sparse valuation
coverage and corrupt input data (e.g. one ticker had a market_cap reported
at $1.99 trillion by the data feed — clearly an upstream unit error). The
cause traced back to two upstream issues:

1. The optimizer accepted stocks with insufficient valuation coverage,
   which led the Black-Litterman view-propagation to amplify spurious
   signals.
2. The data-ingestion pipeline did not validate month-over-month moves,
   so corrupt FMP values reached the model unchanged.

### What was shipped
1. **Universe filter** — stocks must have all of:
   - non-NULL `fair_value_composite`,
   - non-NULL `market_cap`,
   - ≥2 of 7 valid implied-price methods (PE, PB, EV/Sales, EV/EBITDA,
     EV/EBIT, DDM, RIM),
   - shares ∈ (0, 50B), market_cap ≤ $5T, cost_of_equity ∈ (0, 50%).

2. **Data-quality guardrail** — automated monthly cleanup of MoM market-cap
   and per-method implied-price outliers (>3× jumps) plus a backstop on
   composite swings. Now wired into the monthly cron and runs before
   portfolio optimization.

### Backtest metrics (2006-01 → 2026-04, vs S&P 500)
| Metric | Strategy | S&P 500 |
|---|---|---|
| CAGR | 11.33% | 8.86% |
| Sharpe Ratio | 0.56 | 0.48 |
| Sortino Ratio | 0.60 | 0.57 |
| Calmar Ratio (36M) | 1.22 | 0.67 |
| Max Drawdown | -52.40% | -37.03% |
| Annualized Vol | 19.07% | 16.28% |
| Beta | 0.94 | — |
| Alpha | 2.83% | — |
| Information Ratio | 0.24 | — |
| Win Rate | 65.98% | — |
| Avg Monthly Return | 1.02% | 0.80% |
| Best Month | 16.23% | 15.58% |
| Worst Month | -24.13% | -19.55% |
| Avg Turnover | 7.33% | — |

### Run identifier
`wf_20260506_1508_miqp_bl1.0_t40_mp0.002_v6_filter_no_mega`

This run will be extended monthly from June 2026 onward via the production
cron. Prior chain history (under earlier run_ids) remains in the audit log
for completeness — those entries documented what was live at the time of
publication and have not been rewritten.
