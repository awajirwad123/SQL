# Performance & Caching

## Overview
Stored procedures can improve performance through plan reuse, reduced network latency, and set-based batching. However, misuse (row-by-row loops, excessive dynamic SQL) can degrade performance. Monitoring execution metrics ensures continued efficiency.

## Core Concepts
- **Plan Caching**: Procedure statements parameterized—plan often reused; parameter sniffing risk.
- **Set-Based Batch**: Consolidate multiple similar statements into one multi-row DML.
- **Avoid RBAR (Row-By-Agonizing-Row)**: Loops / cursors degrade throughput vs set operations.
- **Instrumentation**: Capture execution duration, logical reads, row counts.
- **Dynamic SQL Overhead**: Reduces plan reuse unless parameterized; consider sp_executesql with parameters (SQL Server).
- **Result Shape Stability**: Stable schemas aid plan caching & client binding.
- **Temp Objects**: Excess temp table creation can cause contention (tempdb) or catalog bloat.

### Parameterized Dynamic SQL (SQL Server)
```sql
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM orders WHERE customer_id=@c AND created_at>=@d';
EXEC sp_executesql @sql, N'@c INT,@d DATETIME2', @c=@CustomerId, @d=@Since;
```

### Batch Insert (Postgres)
```sql
INSERT INTO logs(level, message, created_at)
SELECT level, message, created_at FROM unlogged_staging_batch;
```

## Interview-Focused Notes
1. Why procedures faster sometimes? Lower round trips + plan reuse.
2. Performance pitfall? Cursor loops updating one row at a time.
3. Parameter sniffing mitigation? OPTIMIZE FOR, recompile selectively, or design neutral plan.
4. Dynamic SQL risk? Cache fragmentation, injection if unsafe.
5. Monitoring approach? Query store / pg_stat_statements / performance schema across runs.

## Quick Recall ✅
- Favor set operations.
- Parameterize dynamic SQL.
- Monitor high-frequency procs for regression.
- Avoid unnecessary temp objects.
- Revisit plan after schema/stat changes.

## Interview Traps & Confusions ⚠️
- Assuming procedure always superior to prepared parameterized statement.
- Blanket RECOMPILE causing CPU overhead.
- Wide SELECT * harming I/O.
- Returning unused columns increases network cost.
- Ignoring plan changes after index drop/add.

## Bonus
### Lightweight Telemetry Table
Log proc name, duration ms, rows processed, timestamp for trend analysis.

### Hash Match vs Nested Loop Shift
Plan change may correlate with stats update; baseline plan hash for alerting.

### Unlogged / Minimal Logging (Engine Dependent)
Use when redo durability not needed for transient staging.

### Caching Strategy
Combine app-layer caching with stable procedure outputs (idempotent reads).
