# Incident log

Operational incidents that affected the live record. Entries are written after
the fact. They never edit past ledger files; the ledger stays exactly as it was
published.

## 2020-01 to 2025-12 (backtest): risk-free rate understated in the cost of equity

**What happened.** When the data was moved into the database in December
2025, part of the US 5-year Treasury history (2020-01 and 2020-09 to
2025-12) was stored as decimals (0.0145) while the rest was stored in
percent (1.45). The cost-of-equity step converts the whole series with a
single rule, so those rows were converted twice and the risk-free rate came
out close to zero. Seven months (2020-02 to 2020-08) were also missing and
reused January 2020's cost of equity.

**Effect.** For valuations dated 2020-01 to 2025-12, the cost of equity was
too low by about 0.5 percentage points in 2020, 0.8 in 2021, 2.8 in 2022
and about 4 in 2023–2025. This raised the discounted-cash-flow,
dividend-discount and residual-income values for those months. These
methods account for about 17% of the scoring model's feature importance
(36% in the model for the largest companies). The period is part of the
walk-forward backtest shown on the website, and the scoring model used for
the live portfolio was trained on data up to 2025-08 that includes it.
Valuations from 2026-01 onward, including every live rebalance since June
2026, use the correct rate.

**What was done (2026-10-25).** The stored rates now use a single unit and
the missing months were added. The cost-of-equity step rejects any rate in
the wrong unit, and the monthly job stops with an alert if one appears. The
published backtest, the live record and the current scoring model are left
exactly as they were. The next methodology version will be rebuilt and
retrained on the corrected data, and its results will be published
separately.

## 2026-08-31: rebalance skipped (data-fetch failure)

**What happened.** The monthly run of 2026-09-02 produced no portfolio for
2026-08-31. The pipeline reads each month's market capitalisations on the
first day of the month. 2026-08-01 was a Saturday, and the data request
covered only that non-trading day, so it returned no values. Without market
caps, the investable-universe filter excluded every stock and the optimiser
skipped the month. The job did not flag the empty result.

**Effect on the live record.** No rebalance took place in August. The
2026-07-31 portfolio stayed in place, drifting with prices, until the
2026-09-30 rebalance, which started from those July holdings. The performance
shown on the website already reflects this. No August portfolio has been
reconstructed after the fact, and no published entry has been changed.

**Fix (2026-10-25).** The market-cap request now covers the whole month, so a
weekend or holiday on the 1st no longer matters. The monthly job now stops
with an alert if a month's market caps are incomplete or if no portfolio gets
written. The missing August market-cap data was restored for completeness;
this does not change any published portfolio.
