# Incident log

Operational incidents that affected the live record. Entries are written after
the fact. They never edit past ledger files; the ledger stays exactly as it was
published.

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
