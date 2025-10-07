# Create Index

## Overview
`CREATE INDEX` defines an index on one or more columns (or expressions) to accelerate data access. Proper design depends on workload patterns: filters, joins, sorts, and updates. Index creation strategy balances performance with storage and write amplification.

## Core Concepts
- **Syntax (Generic)**:
  ```sql
  CREATE INDEX idx_orders_customer_date
  ON orders (customer_id, created_at);
  ```
- **Unique Index**: Enforces uniqueness: `CREATE UNIQUE INDEX ...` (see separate file).
- **Composite Index**: Multi-column; leftmost prefix rule applies.
- **Include / Covering (SQL Server / PG 11+ via INCLUDE)**: Non-key columns appended for covering without affecting tree ordering.
  ```sql
  CREATE INDEX ix_orders_customer ON orders(customer_id) INCLUDE (status, total_amount);
  ```
- **Expression / Functional Index**:
  ```sql
  CREATE INDEX idx_lower_email ON users (LOWER(email)); -- Postgres
  ```
- **Filtered / Partial**:
  ```sql
  CREATE INDEX idx_open_tickets ON tickets (priority) WHERE status = 'OPEN'; -- PG
  ```
- **Concurrency & Locking**:
  - PostgreSQL: `CREATE INDEX CONCURRENTLY` reduces write blocking.
  - SQL Server: `WITH (ONLINE = ON)` (Enterprise edition) for online build.
  - MySQL: Algorithm `INPLACE` vs `COPY` depending on storage engine.
- **Fill Factor**: Reserve space on pages for future growth (e.g., `WITH (FILLFACTOR = 90)` SQL Server; Postgres storage parameter).

## Interview-Focused Notes
1. **"How decide index columns?"** – Follow most selective, equality predicates first; then range; match ORDER BY if needed.
2. **"Difference single vs composite index?"** – Composite supports multi-column predicates; order critical; single covers only one dimension.
3. **"Why create functional index?"** – Avoid function-wrapped column preventing index usage.
4. **"What is fill factor?"** – Page fullness threshold controlling fragmentation vs space usage trade-off.
5. **"Online index build benefit?"** – Reduces blocking downtime for production systems.

## Quick Recall ✅
- Create minimal necessary index; avoid overlap duplicates.
- Order columns: equality → range → ordering.
- Use partial/filtered to shrink index size for hot subset.
- Functional indexes for computed predicate reuse.
- Online / concurrently for availability.

## Interview Traps & Confusions ⚠️
- Placing low-selectivity boolean as leading column.
- Duplicate overlapping indexes (waste storage): `(a)` and `(a,b)` if `(a)` alone unused.
- Ignoring maintenance cost on high-churn tables.
- Building index during peak load causing I/O spikes.
- Believing index automatically used if exists.

## Bonus
### Cover Ordering + Filter
```sql
CREATE INDEX ix_events_user_time ON events(user_id, event_time);
-- Supports WHERE user_id=? AND event_time BETWEEN ... ORDER BY event_time
```

### Multi-Column Selectivity Strategy
Prefer `(customer_id, created_at)` over `(created_at, customer_id)` if queries always filter by customer.

### Recreate with Lower Fill Factor (Postgres)
```sql
REINDEX INDEX idx_orders_customer_date;
ALTER INDEX idx_orders_customer_date SET (fillfactor = 85);
```

### Avoid Function in Predicate
Instead of:
```sql
WHERE LOWER(email) = 'x'
```
Use functional index + same expression.

### Concurrent Build (Postgres)
```sql
CREATE INDEX CONCURRENTLY idx_sessions_last_seen ON sessions(last_seen_at);
```