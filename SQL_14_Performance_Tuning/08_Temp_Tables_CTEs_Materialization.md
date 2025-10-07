# Temp Tables, CTEs & Materialization

## Overview
Intermediate result handling affects performance: temporary tables, CTEs, derived tables, and materialized views each have trade-offs for reuse, statistics accuracy, and optimization barriers.

## Core Concepts
- **CTE (Common Table Expression)**: Named subquery; may inline (modern Postgres) or materialize (older versions / recursive). Enhances readability.
- **Derived Table**: Inline anonymous subquery in FROM; often inlined/optimized.
- **Temporary Table**: Persist for session/scope; allows indexing + stats collection; overhead of write + potential logging.
- **Materialized View**: Precomputed, stored; requires refresh policy.
- **Spill / Memory Pressure**: Large intermediate sets might spill to disk during sorting or hashing; sometimes staging into temp table with targeted indexes helps.
- **Reuse Frequency**: If reused multiple times, temp table or CTE reuse can outperform re-executing subquery.
- **Optimization Fence**: Some constructs prevent pushdown; causing less efficient plan.

### When Use Temp Table
- Complex multi-phase transform reused 2+ times.
- Need indexes on intermediate set.
- Break up monstrous plan for manageability.

### Example Pattern
```sql
-- Stage heavy filter
CREATE TEMP TABLE recent_orders AS
SELECT * FROM orders WHERE order_date >= CURRENT_DATE - INTERVAL '7 day';
CREATE INDEX ON recent_orders (customer_id);

-- Use multiple times
SELECT customer_id, COUNT(*) FROM recent_orders GROUP BY customer_id;
SELECT r.* FROM recent_orders r JOIN customers c ON c.customer_id = r.customer_id;
```

## Interview-Focused Notes
1. CTE vs temp table? CTE usually inline; temp table physically materialized with stats + indexing.
2. When materialized view? Expensive aggregate reused frequently and tolerance for staleness.
3. Risk of temp tables? Extra I/O, potential contention in tempdb (SQL Server), planning overhead.
4. Inline vs materialize? Engine/version dependent; verify with plan.
5. Benefit of indexing temp table? Accelerate multiple lookups; must justify creation cost.

## Quick Recall ✅
- Use temp table for reuse + indexing.
- CTE readability; not guaranteed performance improvement.
- Materialized view for repeated heavy queries.
- Avoid unnecessary staging layering.
- Confirm actual optimization behavior via plan.

## Interview Traps & Confusions ⚠️
- Assuming every CTE materializes (not always).
- Creating temp tables for single use (wasted cost).
- Forgetting to index large temp tables.
- Over-refreshing materialized view (costly).
- Using materialized view where latency tolerance = zero.

## Bonus
### Postgres Materialized View Refresh
```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_summary;
```

### SQL Server Table Variable vs Temp Table
Table variables lack stats (pre-SQL Server 2019) → poor estimates; prefer temp table for sizable data.

### Break Large Plan
Insert into temp table after selective filter reduces join complexity & memory.

### CTE Inlining Behavior
Postgres ≥12: non-recursive CTE often inlined, improving optimization freedom.

### Staleness Strategy
Schedule off-peak refresh matching business SLA.
