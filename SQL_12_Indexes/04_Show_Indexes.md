# Show Indexes

## Overview
Inspecting existing indexes helps diagnose performance, identify redundancy, and plan tuning. Each RDBMS provides catalog views or commands to list index definitions, usage metrics, size, and health.

## Core Concepts
- **PostgreSQL**:
  - List indexes on table:
    ```sql
    SELECT indexname, indexdef
    FROM pg_indexes
    WHERE tablename = 'orders';
    ```
  - Usage stats:
    ```sql
    SELECT indexrelid::regclass AS index_name, idx_scan, idx_tup_read, idx_tup_fetch
    FROM pg_stat_user_indexes
    WHERE relname = 'orders';
    ```
  - Size insights:
    ```sql
    SELECT relname AS index_name, pg_size_pretty(pg_relation_size(indexrelid)) AS size
    FROM pg_stat_user_indexes JOIN pg_class ON indexrelid = pg_class.oid
    WHERE relname = 'orders';
    ```
- **MySQL**:
  ```sql
  SHOW INDEX FROM orders;  -- Columns: Key_name, Seq_in_index, Column_name, Cardinality
  ```
- **SQL Server**:
  - Metadata:
    ```sql
    SELECT i.name, i.index_id, i.type_desc, i.is_unique, c.name AS column_name
    FROM sys.indexes i
    JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
    JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
    WHERE i.object_id = OBJECT_ID('dbo.orders');
    ```
  - Usage stats:
    ```sql
    SELECT * FROM sys.dm_db_index_usage_stats WHERE object_id = OBJECT_ID('dbo.orders');
    ```
- **Oracle**:
  ```sql
  SELECT index_name, table_name, uniqueness, status FROM user_indexes WHERE table_name = 'ORDERS';
  SELECT index_name, column_name, column_position FROM user_ind_columns WHERE table_name = 'ORDERS';
  ```

## Interview-Focused Notes
1. **"How find unused indexes?"** – Use usage stats views (e.g., pg_stat_user_indexes.idx_scan = 0 over time).
2. **"Why review index size?"** – Large indexes consume I/O/cache; pruning can improve memory utilization.
3. **"How identify redundant composite indexes?"** – Check column prefixes for overlap; superset composite may obsolete single-column index.
4. **"What is index fragmentation?"** – Page space inefficiency (SQL Server: avg_fragmentation_percent) → reorganize / rebuild.
5. **"How detect covering index usage?"** – Execution plan shows index seek + no key lookup / heap fetch.

## Quick Recall ✅
- Use catalog/system views (portable approach over client GUI).
- Monitor both scans and tuples fetched.
- Composite leftmost rule clarifies overlap.
- Track changes after major query rewrites.
- Keep baseline inventory in version control docs.

## Interview Traps & Confusions ⚠️
- Declaring unused too early (need sustained observation interval).
- Misreading cardinality in MySQL as exact (it’s approximate).
- Overreacting to occasional scans on large seldom-run queries.
- Ignoring partial/filtered index predicates when analyzing redundancy.
- Confusing physical fragmentation with logical unsortedness (not always harmful).

## Bonus
### Postgres: All Indexes By Size Desc
```sql
SELECT c.relname AS index_name, pg_size_pretty(pg_relation_size(c.oid)) AS size
FROM pg_class c
JOIN pg_am am ON c.relam = am.oid
WHERE c.relkind = 'i'
ORDER BY pg_relation_size(c.oid) DESC
LIMIT 20;
```

### SQL Server Fragmentation
```sql
SELECT index_id, avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.orders'), NULL, NULL, 'SAMPLED');
```

### Covered Query Check (SQL Server Plan)
Look for "Index Seek" + absence of "Key Lookup".

### Redundancy Review Query (MySQL)
Use `information_schema.statistics` grouping by table + index_prefix.

### Automate Metrics Snapshot
Schedule periodic capture to detect growth / usage drift.
