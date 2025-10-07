# Window Syntax Anatomy

## Overview
The `OVER` clause structure: `OVER ( [PARTITION BY ...] [ORDER BY ...] [frame_clause] )`. Understanding each component ensures correct calculation semantics and performance.

## Core Components
1. **PARTITION BY**: Splits rows into independent groups.
2. **ORDER BY (Window)**: Defines logical ordering for ranking, navigation, running computations.
3. **Frame Clause** (optional): Limits subset around current row for aggregates / navigation adjustments.
4. **Named Window**: Reusable base definition extended per function.
5. **Multiple Windows**: Each function may define own OVER clause.

### Syntax Variants
```sql
-- Basic
SUM(amount) OVER ()

-- Partition Only
SUM(amount) OVER (PARTITION BY region)

-- Partition + Order (implicit default frame)
SUM(amount) OVER (PARTITION BY region ORDER BY order_date)

-- Explicit Frame
SUM(amount) OVER (
  PARTITION BY region ORDER BY order_date
  ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
)

-- Named Window (Postgres / SQL Server 2022+ limited support via WINDOW clause in some dialects)
SELECT
  SUM(amount) OVER w,
  AVG(amount) OVER w
FROM sales
WINDOW w AS (PARTITION BY region ORDER BY order_date);
```

## Frame Clauses
- `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`
- `ROWS BETWEEN 5 PRECEDING AND 5 FOLLOWING`
- `RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW`
- `GROUPS BETWEEN 1 PRECEDING AND CURRENT ROW`

## ORDER BY vs Frame Defaults
- If ORDER BY present & no frame, default often is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (NOT the entire partition for non-unique ordering with duplicates; duplicates grouped for RANGE semantics). Using ROWS gives strictly row-based boundary.

## Interview-Focused Notes
1. When omit ORDER BY? For partition-level aggregates independent of order (like per-customer total). Ranking/navigation requires ordering.
2. ROWS vs RANGE? ROWS counts physical row offsets; RANGE groups peers with same ORDER BY value. RANGE with numeric/date supports value-based frames.
3. Why explicit frame sometimes? To ensure running total vs entire partition; or define sliding window.
4. Named window benefit? DRY when multiple functions share same partition/order.
5. GROUPS usage? Frame based on peer groups (ties) not row counts.

## Quick Recall ✅
- PARTITION segments; ORDER orders; FRAME slices ordered partition.
- Default frame may hide unintended peer grouping.
- Explicit ROWS frame for deterministic running totals.
- RANGE can be non-deterministic with duplicates if expecting row-by-row progression.
- GROUPS frames tie-aware.

## Interview Traps & Confusions ⚠️
- Expecting ROW_NUMBER semantics with RANGE frame (ties collapse size).
- Forgetting to alias computed windows for reuse (readability cost).
- Overusing named windows for single usage (noise).
- Assuming window ORDER BY equals final result ORDER BY (they are separate phases).
- Misplacing frame: navigation functions like LAG/LEAD ignore frame definitions.

## Bonus
### Explicit Running Total Safer Form
```sql
SUM(amount) OVER (PARTITION BY acct ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

### Moving 7-Day Range
```sql
SUM(amount) OVER (
  PARTITION BY acct ORDER BY ts
  RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW
)
```

### Peer-Aware GROUPS Example
```sql
COUNT(*) OVER (PARTITION BY status ORDER BY priority GROUPS BETWEEN 1 PRECEDING AND CURRENT GROUP)
```

### Mix & Extend Named Window
```sql
WINDOW base AS (PARTITION BY region ORDER BY order_date),
       last3 AS (base ROWS BETWEEN 2 PRECEDING AND CURRENT ROW);
```