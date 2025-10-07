# Execution Plans & Cost Estimation

## Overview
An execution plan is the optimizer's chosen operator tree for a query. Cost estimation uses statistics to approximate CPU + I/O work. Reading plans effectively lets you predict and address inefficiencies early.

## Core Concepts
- **Logical vs Physical**: Logical rewrite (predicate pushdown, join reordering) precedes physical operator selection.
- **Cardinality Nodes**: Each operator has estimated row count; large error cascades.
- **Operator Categories**: Access (Seq Scan, Index Seek), Join (Nested Loop / Hash / Merge), Aggregation (Hash Agg, Sort + Group), Sort, Filter.
- **Cost Model Inputs**: Table size, distinct values, histogram density, CPU calibration, I/O cost constants.
- **Row Source Order**: Reading from right-to-left (common depiction) or top-down depending on tool.
- **Actual vs Estimated**: Discrepancies highlight stats or skew issues.
- **Plan Shape Influencers**: Hints, parameter sniffing, outdated stats, row goal (LIMIT), parallelism thresholds.

### Example (Simplified)
```
Hash Join (cost=... rows=1000)
  -> Seq Scan customers (rows=1000)
  -> Hash
       -> Index Scan orders (rows=1000)
```

### Basic Plan Acquisition
- PostgreSQL: `EXPLAIN (ANALYZE, BUFFERS, VERBOSE)`
- SQL Server: Actual Execution Plan / SET STATISTICS IO / TIME
- MySQL: `EXPLAIN FORMAT=JSON` for richer detail.

## Interview-Focused Notes
1. Why row estimates critical? They determine join strategy & memory grants.
2. Hash vs Nested Loop choice? Row counts & availability of appropriate indexes.
3. What is a "row goal"? LIMIT encourages plan favoring early tuple production (e.g., index + Top-N heap fetch).
4. Sort avoidance? Use index order or preordered access path.
5. Plan regression causes? Stats drift, parameter changes, index create/drop, engine version upgrade.

## Quick Recall ✅
- Always pair EXPLAIN with ANALYZE to see reality.
- Large estimate errors → fix stats / rewrite predicates.
- Access path + join algorithm synergy matters.
- Avoid forced hints unless last resort.
- Save plan hash & baseline for regression detection.

## Interview Traps & Confusions ⚠️
- Interpreting estimated plan only (misleading).
- Confusing cost units with time (they are relative).
- Assuming join order matches query text (optimizer reorders).
- Ignoring parallel plan overhead on small queries.
- Blindly trusting hints from legacy code.

## Bonus
### Detect Cardinality Error (Postgres)
Look for `rows=1000 actual rows=500000` disparity.

### Memory Grant Issues (SQL Server)
Excessive grant leads to concurrency reduction; insufficient leads to spills (watch for Sort / Hash spills).

### Plan Caching Hash Key
Capture plan_id or hash for monitoring changes in Query Store / pg_stat_statements (normalized query id).

### Fast-First-Row Optimization
LIMIT encourages index-only + partial evaluation; verify if full scan still chosen.

### Visual Plan Reading Order
Traverse leaves to root; confirm each operator’s row estimate vs actual to locate divergence origin.
