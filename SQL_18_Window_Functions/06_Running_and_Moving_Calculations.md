# Running and Moving Calculations

## Overview
Running (cumulative) and moving (sliding window) calculations provide sequential analytics: cumulative sums, rolling averages, rolling counts, forward/backward look windows.

## Running Calculations
- Use unbounded preceding frame.
- Deterministic form:
```sql
SUM(metric) OVER (PARTITION BY key ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_sum
```

## Moving Calculations
- Bounded frame relative to current row or current value/time.

### Fixed Row Count Window
```sql
AVG(metric) OVER (
  PARTITION BY key ORDER BY ts
  ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
) AS ma5
```

### Time-Based Rolling Window
```sql
SUM(metric) OVER (
  PARTITION BY key ORDER BY ts
  RANGE BETWEEN INTERVAL '29 day' PRECEDING AND CURRENT ROW
) AS rolling_30d_sum
```

### Forward Looking (Lead Window)
```sql
SUM(metric) OVER (
  PARTITION BY key ORDER BY ts
  ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
) AS next3_sum
```

## Edge Handling
- Start of partition: fewer preceding rows; averages computed over available frame (no padding) unless you COALESCE.
- Use CASE to require full frame:
```sql
CASE WHEN ROW_NUMBER() OVER (PARTITION BY key ORDER BY ts) >= 5
     THEN AVG(metric) OVER (...)
END AS ma5_full
```

## Interview-Focused Notes
1. Running vs moving? Running accumulates from start; moving slides fixed frame.
2. Why explicit ROWS frame? Avoid peer grouping anomalies; guarantee row-based increments.
3. Time window caution? RANGE requires engine support & may not handle calendar gaps consistently.
4. Forward-looking window usage? Forecast comparisons, smoothing future events (may require business justification).
5. Partial frame acceptance? Decide if early rows produce partial averages or return NULL until filled.

## Quick Recall ✅
- Running: UNBOUNDED PRECEDING.
- Moving: N PRECEDING.
- Time-based: RANGE INTERVAL.
- Forward windows allow look-ahead.
- Guard for full-frame requirement.

## Interview Traps & Confusions ⚠️
- Using moving average without deterministic ORDER BY.
- Expecting fixed denominator when frame not full yet.
- Large frames causing memory & sort pressure.
- Failing to convert integer division for averages.
- Assuming RANGE works identically across engines.

## Bonus
### Cumulative Distinct Count Approximation
Use HyperLogLog extension or pre-aggregated summary joined.

### Rolling Standard Deviation
`STDDEV_SAMP(val) OVER ( ... ROWS BETWEEN 6 PRECEDING AND CURRENT ROW )`

### Multi-Window Combo
Return running_sum, ma7, ma30 in same SELECT with repeated window definitions (consider named window base). 

### Gap-Aware Rolling
Pre-aggregate daily buckets then window for performance vs raw event rows.
