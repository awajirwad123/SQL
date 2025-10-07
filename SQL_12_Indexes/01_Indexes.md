# Indexes

## Overview
An index is a data structure (typically a B-Tree, hash, or other specialized form) that accelerates data retrieval at the cost of additional storage and write overhead. Indexes enable the database optimizer to locate rows without scanning the entire table. Well-chosen indexes can transform performance; poorly chosen ones can degrade it.

## Core Concepts
- **Primary Data Structures**:
  - B-Tree (default general-purpose ordered structure; supports range, equality, prefix lookups)
  - Hash (equality-only; MySQL InnoDB adaptive hash, PostgreSQL hash indexes limited use historically)
  - Bitmap (data warehousing, low-cardinality columns; Oracle, some columnar engines)
  - GIN (Generalized Inverted Index) for full-text / JSONB / array membership (PostgreSQL)
  - GiST / R-Tree variants for spatial / range queries
  - Full-text specialized (inverted index) for search
- **Logical vs Physical**: Logical definition (DDL) vs physical on-disk structure (pages, fan-out, fill factor).
- **Cardinality & Selectivity**: Higher selectivity (more unique distribution) → more benefit for filtering.
- **Covering Index**: Index contains all needed columns (key + included) so table (heap) access avoided.
- **Write Amplification**: INSERT/UPDATE/DELETE must maintain every index → too many indexes slow writes.
- **Index Only Scan**: Query satisfied entirely by index (visibility map factors in Postgres; included columns in SQL Server).
- **Statistics**: Optimizer relies on histograms/NDV to pick index; stale stats lead to bad plan choices.

**Common Use Cases**:
- Filtering predicates: `WHERE user_id = ?`
- Joins: Foreign key columns (e.g., `orders.customer_id`)
- Sorting / Grouping: `ORDER BY created_at`, `GROUP BY (status)`
- Partial/Filtered: Only subset of rows (e.g., active records) to reduce size.

## Interview-Focused Notes
1. **"What is an index?"** – Auxiliary structure mapping key values to row locations to speed lookups.
2. **"Downsides?"** – Extra storage, slower writes, maintenance overhead, potential for fragmentation.
3. **"When NOT to add?"** – Low-selectivity columns in OLTP (e.g., boolean), volatile columns with frequent updates, rarely filtered fields.
4. **"Why index foreign keys?"** – Supports efficient joins and avoids table locks on parent modifications.
5. **"Covering index use case?"** – High-read query pattern hitting same column set; eliminates heap fetch.

## Quick Recall ✅
- Index = speed reads, tax writes.
- B-Tree covers equality + range + ORDER BY alignment.
- High selectivity preferred.
- Functional / expression indexes support computed predicates.
- Keep indexes lean: only what queries need.

## Interview Traps & Confusions ⚠️
- Believing more indexes always better.
- Adding index to every column a query touches—even aggregates can be faster with fewer targeted indexes.
- Ignoring multi-column order significance.
- Forgetting to REINDEX / VACUUM (PG) or reorganize in systems needing maintenance.
- Assuming index always used (optimizer may still choose full scan if cheaper).

## Bonus
### Multi-Column Order Matters
`INDEX(a,b)` supports queries on `a` and on `(a,b)` but not efficiently on `b` alone.

### Functional / Expression Index (PostgreSQL)
```sql
CREATE INDEX idx_users_lower_email ON users (LOWER(email));
```

### Partial Index (PostgreSQL)
```sql
CREATE INDEX idx_active_sessions ON sessions (user_id) WHERE active = TRUE;
```

### Covering Index (SQL Server)
```sql
CREATE INDEX ix_orders_customer_date ON orders(customer_id, created_at) INCLUDE(total_amount, status);
```

### Detect Unused Indexes
Monitor DMV (SQL Server) or pg_stat_user_indexes (Postgres) to prune.
