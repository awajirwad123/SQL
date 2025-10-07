# Clustered Index vs Non-Clustered Index

## Overview
A clustered index defines the physical row order (or logical row ordering structure) of a table’s data pages. A non-clustered index is a secondary structure mapping key values to row locators (either the clustered key or a heap RID). Understanding the distinction is essential for performance tuning in SQL Server, MySQL (InnoDB), and to contrast with PostgreSQL’s heap + separate indexes model.

## Core Concepts
### Clustered Index
- **Physical Ordering**: Data rows stored in index key order (SQL Server, InnoDB). Only one per table because there is only one physical ordering.
- **Primary Key Tie-In**: Default target for primary key in SQL Server and InnoDB (unless specified otherwise with a different clustered index).
- **Lookup Cost**: Non-clustered index entries store clustered key as pointer; wide clustered key increases all secondary index sizes.
- **Fragmentation**: Random inserts into middle cause page splits; mitigate with fill factor / sequential keys (e.g., identity, monotonic surrogate, UUID v7).

### Non-Clustered Index
- **Structure**: Separate B-Tree; leaves store either (a) row identifier (heap) or (b) clustered key.
- **Covering**: INCLUDE columns (SQL Server)/index-only scans when all needed data in index.
- **Multiplicity**: Many allowed.

### PostgreSQL Contrast
- No formal clustered index concept; heap unsorted. `CLUSTER` command can reorder physically once, but not maintained automatically. Index scans navigate via pointers.

### MySQL InnoDB Specifics
- Clustered index = primary key; if none defined, hidden 6-byte row id created.
- Secondary indexes store primary key values as references, not physical addresses.

### Choice of Clustered Key
- Stable, narrow, ever-increasing preferred.
- Avoid GUID v4 (random) for hot OLTP tables (fragmentation, page splits, cache churn).

## Interview-Focused Notes
1. **"What is a clustered index?"** – Primary data storage ordered by key, single per table.
2. **"Why choose narrow clustered key?"** – Propagates into all non-clustered entries; wide key bloats indexes.
3. **"Heap vs clustered table?"** – Heap has no defined physical ordering; lookups may require bookmark lookups.
4. **"Why sequential keys reduce fragmentation?"** – Appending at end avoids page splits.
5. **"PostgreSQL equivalent?"** – None persistent; relies on heap + B-Tree; can manually CLUSTER but not auto-maintained.

## Quick Recall ✅
- One clustered index (ordering) vs many non-clustered (pointers).
- Secondary index entries include clustered key.
- Narrow, monotonically increasing clustered key ideal.
- Random keys → fragmentation.
- Covering reduces extra lookups.

## Interview Traps & Confusions ⚠️
- Thinking PostgreSQL has always-on clustered index (it doesn’t).
- Choosing natural, mutable wide text as clustered key (expensive cascades).
- Using random GUID v4 in high insert load without mitigation.
- Believing CLUSTER command persists ordering automatically in PG.
- Ignoring fill factor / page split cost.

## Bonus
### Clustered Key Design Example (SQL Server)
```sql
CREATE TABLE orders (
  order_id BIGINT IDENTITY PRIMARY KEY CLUSTERED,
  customer_id BIGINT NOT NULL,
  created_at DATETIME2 NOT NULL,
  total_amount DECIMAL(12,2) NOT NULL
);
```

### Non-Clustered Covering Index
```sql
CREATE NONCLUSTERED INDEX ix_orders_customer_created
  ON orders(customer_id, created_at)
  INCLUDE(total_amount);
```

### Mitigate GUID Fragmentation
- Use Sequential GUID / COMB / UUID v7.
- Batch insert sorted by key.

### Checking Fragmentation (SQL Server)
```sql
SELECT index_id, avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), OBJECT_ID('dbo.orders'), NULL, NULL, 'SAMPLED');
```

### Postgres Occasional Recluster
```sql
CLUSTER orders USING orders_created_at_idx;  -- One-time reorder
VACUUM (FULL) rarely if necessary.
```
