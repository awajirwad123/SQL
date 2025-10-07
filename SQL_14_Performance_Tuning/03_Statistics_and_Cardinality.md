# Statistics & Cardinality Estimation

## Overview
Statistics summarize data distribution (row counts, distinct values, histograms, correlation). The optimizer relies on them to estimate cardinalities. Accurate statistics lead to efficient plans; stale or missing stats cause suboptimal scans, joins, or memory grants.

## Core Concepts
- **Components**: Table row count, column NDV (number distinct values), histogram buckets, most common values (MCV), null fraction, correlation.
- **Collection Triggers**: Auto analyze / auto update thresholds; manual `ANALYZE` (Postgres), `UPDATE STATISTICS` (SQL Server legacy) / auto update, `ANALYZE TABLE` (MySQL).
- **Sampling**: Full vs sampled; trade-off accuracy vs overhead.
- **Correlation (Data Skew)**: Multi-column correlation not captured by single-column stats; leads to under/overestimates.
- **Extended / Multi-Column Stats**: Postgres extended statistics (MCV lists / functional / dependency). SQL Server statistics on composite indexes implicitly cover column order.
- **Out-of-Range Values**: Newly inserted high values not in histogram (ascending key problem) → poor estimates.
- **Parameter Sniffing Interaction**: Stats capture typical distribution; sniffed parameter may represent edge case.

### Manual Refresh Examples
- PostgreSQL: `ANALYZE orders;`
- SQL Server: `UPDATE STATISTICS dbo.orders WITH FULLSCAN;`
- MySQL: `ANALYZE TABLE orders;`

## Interview-Focused Notes
1. Why statistics matter? Drive cardinality → plan quality.
2. Skew impact? Heavy skew invalidates uniform selectivity assumption.
3. Fix multi-column correlation? Create composite index or extended statistics.
4. Ascending key problem? Frequent inserts beyond histogram upper bound; use incremental stats / trace flags / more frequent updates.
5. Detect stale stats? Estimate vs actual row divergence; last updated timestamp.

## Quick Recall ✅
- Stats drift → row estimate errors → bad plan.
- Composite predicates need composite stats or index.
- Refresh stats after bulk load.
- Out-of-range detection: actual >> estimated.
- Extended stats reduce correlation errors.

## Interview Traps & Confusions ⚠️
- Believing more frequent stats always better (overhead vs benefit).
- Ignoring multi-column predicate correlation.
- Rebuilding index when only stats update needed.
- Misusing FULLSCAN on huge tables without necessity.
- Assuming CTE or rewrite fixes when root cause is stats.

## Bonus
### Postgres Extended Statistics
```sql
CREATE STATISTICS orders_user_status (ndistinct, dependencies) ON user_id, status FROM orders;
ANALYZE orders;
```

### Detect Mismatch (Postgres)
Use `EXPLAIN (ANALYZE, BUFFERS)`; compare `rows` vs `actual rows` at earliest divergence.

### SQL Server Auto Update Threshold
Approx: 20% * rows + 500; large tables may need manual updates.

### Histograms & Equality
Equality selectivity ≈ 1 / NDV (unless MCV entry exists).

### Out-of-Range Mitigation
- More frequent stats
- Incremental partition stats
- Filtered stats (SQL Server) for volatile subset.
