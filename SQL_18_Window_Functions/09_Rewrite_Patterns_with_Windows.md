# Rewrite Patterns with Window Functions

## Overview
Window functions often replace correlated subqueries, self-joins, or procedural loops with a single-pass declarative form. Recognizing rewrite opportunities improves performance and clarity.

## Patterns
1. **Top-N Per Group**: ROW_NUMBER filter replaces self-join max selection.
2. **Running Totals**: SUM OVER replaces correlated SUM subquery.
3. **Deduplication**: ROW_NUMBER partition/ORDER + filter = 1 replaces self-join on max timestamp.
4. **Percent-of-Total**: Value / SUM OVER partition replaces two-phase aggregation.
5. **Gap Detection**: LAG difference replaces self-join ordering by time.
6. **Delta / Change Classification**: LAG() with CASE vs procedural cursor.
7. **First/Last Non-Null**: Forward/backfill using LAST_VALUE IGNORE NULLS vs nested subquery.
8. **Ranked Filtering**: RANK vs DISTINCT ON (Postgres) vs MIN/MAX join.

### Examples
- Correlated Sum Rewrite:
```sql
-- Before
SELECT o.*, (
  SELECT SUM(amount) FROM orders o2
  WHERE o2.customer_id = o.customer_id
) AS cust_total
FROM orders o;
-- After
SELECT o.*, SUM(amount) OVER (PARTITION BY customer_id) AS cust_total
FROM orders o;
```

- Dedupe Latest Row:
```sql
SELECT * FROM (
  SELECT r.*, ROW_NUMBER() OVER (PARTITION BY natural_key ORDER BY updated_at DESC) AS rn
  FROM raw_feed r
) x WHERE rn=1;
```

- Gap Detection:
```sql
SELECT ts, prev_ts, ts - prev_ts AS gap
FROM (
  SELECT ts, LAG(ts) OVER (ORDER BY ts) AS prev_ts
  FROM events
) t WHERE ts - prev_ts > INTERVAL '5 min';
```

## Interview-Focused Notes
1. Why replace correlated subqueries? Reduce repeated scans; single window pass.
2. When not use window rewrite? If filtering can leverage index-only subquery more efficiently (rare) or memory constraints for large sorts.
3. Dedupe ties? ORDER BY deterministic tie-breaker required.
4. Percent-of-total sensitive to? Division by zero; cast for precision.
5. Gap detection reliance? Proper ordering; time zone normalization if needed.

## Quick Recall ✅
- Window = single pass multi-metric.
- ROW_NUMBER + filter for top-N/dedupe.
- LAG for deltas/gaps.
- SUM OVER for running/partition totals.
- Avoid over-nesting; chain windows succinctly.

## Interview Traps & Confusions ⚠️
- Filtering on window alias in same SELECT (needs subquery).
- Non-deterministic ORDER BY causing inconsistent duplicates removal.
- Over-application where simple aggregate + join suffices.
- Memory blow-up from unnecessary large sorts.
- Expecting window to eliminate need for indexing (still need ORDER BY support).

## Bonus
### Conditional Aggregation with Windows
Combine CASE inside window: `SUM(CASE WHEN status='OPEN' THEN 1 END) OVER (...)`.

### Multiple Metrics Shared Window
Use named WINDOW to avoid repeat clause duplication.

### Replace DISTINCT ON (Postgres)
Portable pattern: ROW_NUMBER partition + ORDER BY + rn=1.

### Semi-Join Replacement
RANK filter can emulate greatest-per-group join pattern.
