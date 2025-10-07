# Drop Index

## Overview
`DROP INDEX` removes an index, freeing storage and eliminating its maintenance cost. Pruning unused or redundant indexes is essential for sustaining write performance. Always verify dependencies (indexed views, constraints) before drop.

## Core Concepts
- **Basic Syntax**:
  - PostgreSQL / MySQL / Oracle:
    ```sql
    DROP INDEX idx_orders_customer_date;
    ```
  - SQL Server (need table qualification):
    ```sql
    DROP INDEX idx_orders_customer_date ON orders;
    ```
- **IF EXISTS** (PG / MySQL 5.7+ / SQL Server 2016+):
  ```sql
  DROP INDEX IF EXISTS idx_old;
  ```
- **Concurrent (Postgres)**: No; concurrency option for *create* only; drop acquires lock (PG14 added `CONCURRENTLY` for drop: `DROP INDEX CONCURRENTLY idx_old;`).
- **Constraints**: Primary key / unique constraints create implicit indexes—drop constraint, not its backing index directly.
- **Dependencies**: Indexed views or partial indexes supporting queries—validate usage stats.

**Identify Candidates (Examples)**
- PostgreSQL:
  ```sql
  SELECT indexrelid::regclass AS index_name, idx_scan, pg_size_pretty(pg_relation_size(indexrelid))
  FROM pg_stat_user_indexes
  WHERE idx_scan = 0 AND pg_relation_size(indexrelid) > 0;
  ```
- SQL Server (DMVs): `sys.dm_db_index_usage_stats` joined with `sys.indexes`.

## Interview-Focused Notes
1. **"Why drop an index?"** – Unused, redundant, overlapping, or harming write throughput.
2. **"How to detect unused?"** – Usage stats DMV / catalog metrics over representative workload window.
3. **"Can you drop an index backing a constraint?"** – Not directly; must drop constraint (PK/UNIQUE) which removes its index.
4. **"Risks?"** – Slower queries that were relying on it; plan regressions.
5. **"Mitigation?"** – Capture execution plans or run in staging; keep DDL script to recreate quickly.

## Quick Recall ✅
- Drop redundant and unused indexes.
- Keep one composite instead of many single-column (when patterns align).
- Monitor after removal for regression.
- Don't drop PK / unique supporting indexes inadvertently.
- Use size + scan count to prioritize.

## Interview Traps & Confusions ⚠️
- Misidentifying unique constraint index as safe to drop.
- Dropping index needed for foreign key lock avoidance.
- Snapshotting usage over too short a window (false unused signal).
- Forgetting to re-evaluate maintenance tasks after index removal.
- Over-reliance on size only (small but critical index removed).

## Bonus
### Safe Rollback Script
Before drop, script recreate:
```sql
CREATE INDEX idx_orders_customer_date ON orders(customer_id, created_at);
```

### Postgres Concurrent Drop (PG14+)
```sql
DROP INDEX CONCURRENTLY idx_old_large;
```

### Find Redundant Pairs (Postgres Example)
```sql
SELECT a.indexrelid::regclass AS superset, b.indexrelid::regclass AS subset
FROM pg_index a
JOIN pg_index b ON a.indrelid = b.indrelid
WHERE a.indexrelid <> b.indexrelid
  AND a.indkey @> b.indkey  -- a covers b columns
  AND NOT b.indisunique;
```

### Archive DDL
Store DDL in version control to allow quick restoration.
