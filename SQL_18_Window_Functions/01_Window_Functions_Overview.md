# Window Functions Overview

## Overview
Window (analytic) functions perform calculations across a set of rows related to the current row without collapsing result cardinality (unlike GROUP BY). They enable ranking, running totals, moving averages, gap/lag analysis, percentiles, and intra-partition comparisons in a single pass.

## Core Concepts
- **Preserve Rows**: Output row count equals input row count.
- **OVER Clause**: Defines window partitioning, ordering, and optional frame.
- **Partition**: Logical grouping (PARTITION BY) – resets calculation boundaries.
- **Ordering**: ORDER BY inside window defines deterministic row sequence per partition.
- **Frame**: Subset of ordered partition for cumulative/moving operations (`ROWS`, `RANGE`, `GROUPS`).
- **Types of Window Functions**:
  - Ranking (ROW_NUMBER, RANK, DENSE_RANK, NTILE)
  - Value (LAG, LEAD, FIRST_VALUE, LAST_VALUE, NTH_VALUE)
  - Aggregate (SUM, AVG, MIN, MAX, COUNT) as windowed forms
  - Distribution (PERCENT_RANK, CUME_DIST)
  - Percentile (percentile_disc / percentile_cont – PostgreSQL, Oracle; APPROX_* variants in some engines)
- **Logical Execution Order**: FROM → WHERE → GROUP BY → HAVING → WINDOW (OVER) → SELECT alias projection → ORDER BY (final). Window functions cannot reference aliases defined in same SELECT (except vendor extensions) and cannot appear in WHERE directly (use subquery/CTE).

### Example
```sql
SELECT order_id,
       customer_id,
       total_amount,
       SUM(total_amount) OVER (PARTITION BY customer_id) AS customer_lifetime_spend,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at) AS order_seq
FROM orders;
```

## Interview-Focused Notes
1. Difference vs GROUP BY? Window keeps row granularity; GROUP BY collapses.
2. Why partition? Segments independent running calculations.
3. Frame necessity? Needed for moving/running variants; ranking may only need ordering.
4. Can use in WHERE? Not directly—wrap in subquery/CTE.
5. Performance consideration? Sorting each partition; large sorts dominate runtime.

## Quick Recall ✅
- OVER defines analytic scope.
- Frame default for ordered aggregate = RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW (engine-specific) vs ROWS variant (Postgres uses RANGE; MySQL often defaults similarly) – verify.
- Ranking vs aggregation vs navigation categories.
- Partition resets; frame limits inside partition.
- Needs sort; index alignment reduces cost.

## Interview Traps & Confusions ⚠️
- Believing frame = partition (frame is subset of partition order).
- Forgetting window functions evaluated after WHERE (causing attempt to filter on alias prematurely).
- Using LAST_VALUE without frame specification (returns current row value unexpectedly; need frame end extended).
- Assuming deterministic order without ORDER BY in window.
- Overlooking NULL ordering differences (NULLS FIRST/LAST) affecting rank expectations.

## Bonus
### Combine Window + Filter
Top-1 per group:
```sql
SELECT * FROM (
  SELECT o.*, ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY created_at DESC) AS rn
  FROM orders o
) x WHERE rn = 1;
```

### Dense Ranking for Ties
`DENSE_RANK()` avoids gaps after ties unlike `RANK()`.

### Window Reuse (Some Engines)
Named windows (Postgres):
```sql
WINDOW w AS (PARTITION BY customer_id ORDER BY created_at);
```

### Avoid Self-Joins
Replace correlated subquery summations with window SUM OVER.
