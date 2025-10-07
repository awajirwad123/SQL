# Aggregate Window Functions

## Overview
Aggregate window functions apply standard aggregates (SUM, AVG, MIN, MAX, COUNT) over a window frame without collapsing rows. They power running totals, partition totals, moving averages, and comparative metrics.

## Patterns
- **Partition Total**: Entire partition (omit ORDER BY or use unbounded frame).
- **Running Total**: ORDER BY + frame up to current row.
- **Moving Window**: ORDER BY + bounded frame PRECEDING/FOLLOWING.
- **Percent Contribution**: Value / partition SUM window.

### Examples
```sql
-- Partition total (per customer lifetime spend)
SUM(amount) OVER (PARTITION BY customer_id) AS cust_total

-- Running cumulative
SUM(amount) OVER (
  PARTITION BY customer_id ORDER BY order_date
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS running_spend

-- 7-day moving average of daily totals
AVG(daily_total) OVER (
  PARTITION BY product_id ORDER BY date
  ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
) AS ma7

-- Percent of category total
amount * 1.0 / SUM(amount) OVER (PARTITION BY category) AS pct_of_cat
```

## Frame Choice
- Use `ROWS` for deterministic row-based increments.
- Use `RANGE` for value-based logical grouping; can merge peers.

## Interview-Focused Notes
1. Running vs moving difference? Running cumulative from start; moving bounded preceding range.
2. Why explicit frame for running totals? Avoid default RANGE peer grouping side effects.
3. Percent contribution calculation? value / SUM(value) OVER (partition).
4. MIN/MAX frame nuance? Without ORDER BY across entire partition; with ORDER BY and frame for sliding min/max.
5. COUNT distinction? COUNT(*) includes NULL rows; COUNT(column) excludes NULL.

## Quick Recall ✅
- Aggregates over window preserve rows.
- Explicit ROWS frame for strict running totals.
- Moving average uses bounded preceding frame.
- Combine with LAG for delta computations.
- Partition totals enable percent-of metrics.

## Interview Traps & Confusions ⚠️
- Forgetting frame leading to full-partition rather than running total.
- Using RANGE on non-unique order causing block jumps.
- Percent-of total integer division (cast to numeric).
- Overlapping moving window mis-specified (off by one boundary).
- High memory/sort cost if large unsorted partition.

## Bonus
### Cumulative Distinct Count (Requires Advanced Techniques)
Use window on row_number of first occurrences or hyperloglog approximate aggregates (engine-specific).

### Incremental Delta
```sql
amount - LAG(amount) OVER (PARTITION BY acct ORDER BY ts) AS delta
```

### Share of Running Total
```sql
amount / NULLIF(SUM(amount) OVER (
  PARTITION BY acct ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW),0) AS pct_running
```

### Bounded Future Look
```sql
SUM(amount) OVER (
  PARTITION BY acct ORDER BY ts
  ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
) AS next3_sum
```