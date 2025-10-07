# LAG / LEAD and Value Comparisons

## Overview
Navigation (value) window functions access other rows in the partition relative to current row. They enable change detection, difference computations, forward/backward comparisons, and gap analysis.

## Functions
- `LAG(expr, offset, default)` – Value from prior row.
- `LEAD(expr, offset, default)` – Value from subsequent row.
- `FIRST_VALUE(expr)` / `LAST_VALUE(expr)` – First/last value within frame (careful with default frame!).
- `NTH_VALUE(expr, n)` – nth value within frame.

### Examples
```sql
-- Previous row delta
amount - LAG(amount) OVER (PARTITION BY acct ORDER BY ts) AS amount_delta

-- Percentage change
(amount - LAG(amount) OVER w) / NULLIF(LAG(amount) OVER w,0) AS pct_change
WINDOW w AS (PARTITION BY symbol ORDER BY trade_ts)

-- Forward look (LEAD)
LEAD(event_time) OVER (PARTITION BY session_id ORDER BY event_time) - event_time AS dwell_ms

-- First purchase date per customer on each row
FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING) AS first_order_date
```

## Frame Considerations
- Navigation functions ignore frame except FIRST/LAST/NTH which depend on frame boundaries.
- For LAST_VALUE to return true partition last value, extend frame to UNBOUNDED FOLLOWING.

## Interview-Focused Notes
1. LAG vs self-join? Simpler, single pass; avoids join complexity.
2. LAST_VALUE pitfall? Default frame ends at current row -> returns current row unless frame extended.
3. Provide default argument why? Avoid NULL for missing preceding row; supply neutral baseline.
4. Gap analysis? LEAD(ts) - ts reveals interval; can flag large gaps.
5. NTH_VALUE use case? Retrieve specific percentile position after ordering (but percentile_cont may be better for real interpolation).

## Quick Recall ✅
- LAG = previous; LEAD = next.
- FIRST/LAST need explicit frame for full partition view.
- Use NULLIF to avoid divide-by-zero.
- Navigation pairs with ranking/aggregates.
- Differences & pct change common interview tasks.

## Interview Traps & Confusions ⚠️
- Using LAST_VALUE without frame extension.
- Expecting navigation across partitions (restarts per partition).
- Neglecting order uniqueness (non-deterministic previous row).
- Overusing multiple LAG calls instead of specifying offset parameter.
- Performance issues from repeated identical LAG expressions (alias with subquery).

## Bonus
### Multi-Offset LAG
```sql
LAG(val,1) OVER w AS prev1,
LAG(val,2) OVER w AS prev2
```

### Change Classification
```sql
CASE
  WHEN val > LAG(val) OVER w THEN 'UP'
  WHEN val < LAG(val) OVER w THEN 'DOWN'
  ELSE 'SAME'
END AS direction
```

### Forward Fill (Last Non-Null)
```sql
LAST_VALUE(col) IGNORE NULLS OVER (
  PARTITION BY grp ORDER BY ts
  ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
) AS col_ff
```
(IGNORE NULLS support engine-dependent.)

### Session Gap Flag
```sql
CASE WHEN LEAD(ts) OVER w - ts > INTERVAL '5 min' THEN 1 END AS gap_break
```