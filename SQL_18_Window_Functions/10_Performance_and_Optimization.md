# Performance & Optimization (Window Functions)

## Overview
Window queries often require sorting per partition/order specification. Sorting and large frame materialization dominate cost. Optimization focuses on reducing sort work, leveraging indexes, limiting partition size, and avoiding redundant computations.

## Cost Drivers
- Sort operations (global or per-partition)
- Large partitions with wide rows (memory spill risk)
- Wide SELECT lists causing extra I/O before window pass
- Repeated identical OVER clauses (redundant sort if engine not smart)
- Complex expressions inside window ORDER BY (prevent index usage)

## Optimization Techniques
1. **Index Alignment**: Create composite index matching PARTITION BY + ORDER BY prefix (where feasible) to support index-order scan and reduce sort.
2. **Pre-Aggregation**: Aggregate raw detail to daily/hour buckets before window for time-series moving windows.
3. **Slim Projection Early**: Use subquery/CTE to select only needed columns before window step.
4. **Frame Narrowing**: Use bounded ROWS when full partition not needed.
5. **Window Reuse**: Named windows or repeat detection to avoid extra sorts.
6. **Decompose Query**: Break extremely large multi-window queries into staged temp table (with index) if memory spills.
7. **Approximation**: Use approximate percentiles rather than heavy sort for large distributions when acceptable.
8. **Predicate Pushdown**: Filter early (WHERE) before window since windows can't help with row elimination.

### Example: Index to Support Running Total
`CREATE INDEX ix_orders_customer_created ON orders(customer_id, created_at);`

### Memory Spill Indicators
- Execution plan shows SORT/HASH spill warnings.
- Increased temp db / work_mem usage.

## Interview-Focused Notes
1. Biggest cost? Sorting for ORDER BY inside window.
2. How reduce sort? Align index order or narrow dataset first.
3. Row vs range frame performance? RANGE may group peers unpredictably; ROWS deterministic and sometimes lighter.
4. Repeated window spec? Reuse or rely on optimizer deduplication (not always guaranteed across engines).
5. Spills mitigation? Increase work memory / reduce row width / partition size.

## Quick Recall ✅
- Sort dominates cost.
- Index can avoid explicit sort.
- Filter & slim columns early.
- Bounded frames cheaper than unbounded.
- Pre-aggregate high-volume detail first.

## Interview Traps & Confusions ⚠️
- Assuming index always avoids sort (function expressions break match).
- Selecting unnecessary columns bloating memory.
- Using unbounded frames when only N rows needed.
- Expecting engine to automatically merge identical windows.
- Over-indexing just for one rare analytic query.

## Bonus
### Combine Metrics
Compute multiple metrics in one pass to amortize cost: `SUM()`, `AVG()`, `COUNT()` over same window.

### Partial Materialization
Stage intermediate set for extremely large computations, create covering index, then apply windows.

### Execution Plan Review
Check if plan shows WINDOW AGG after SORT; validate row counts & memory grant.

### Adaptive Chunking
Process large partitioned dataset by date ranges and union results if window logic allows segmentation.
