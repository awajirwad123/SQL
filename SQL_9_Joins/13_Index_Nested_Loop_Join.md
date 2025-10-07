# INDEX NESTED LOOP JOIN

## Overview
An Index Nested Loop Join optimizes the inner-loop lookup by using an index on the inner table's join key. For each outer row, the algorithm performs an indexed point (or range) lookup instead of scanning the inner table fully.

## Core Concepts
- Still a nested loop structure, but inner scan replaced with index probe.
- Complexity: O(N × log M) (roughly) if index balanced and point lookups.
- Ideal when outer result set is small after filters.

**Example:**
```sql
SELECT o.order_id, o.order_date, c.customer_name
FROM orders o
JOIN customers c ON c.customer_id = o.customer_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '7 days';
```
If only a few thousand recent orders, each probes `customers` via PK index.

## Interview-Focused Notes
1. "What differentiates index nested loop from naive nested loop?" – Inner scans replaced by indexed lookups.
2. "When is it chosen?" – Outer filtered + good index on inner join column.
3. "What degrades performance?" – Outer becomes large (index lookup overhead accumulates).
4. "How to improve?" – Add covering index to reduce fetch (include columns).
5. "Why not use hash join instead?" – Hash has higher startup cost; index NLJ wins on small selective workloads.

## Quick Recall ✅
- Index accelerates inner lookups.
- Great for OLTP queries.
- Sensitive to outer row count growth.
- Benefits from covering (composite) indexes.
- Low memory footprint.

## Interview Traps & Confusions ⚠️
- Assuming index NLJ always best if index exists—large outer set may still favor hash join.
- Missing composite index when join + filter conditions both used.
- Forgetting to analyze selectivity; stale stats mislead optimizer.

## Bonus
### Composite Index Example
If query filters and joins:
```sql
SELECT *
FROM line_items li
JOIN products p ON p.product_id = li.product_id
WHERE li.order_id = 5001;
```
Index `line_items(order_id, product_id)` supports both filter + join.

### Covering Index Strategy
Add included columns (SQL Server) or widen index (Postgres) to avoid heap lookups.

### EXPLAIN Clue
Look for pattern: `Nested Loop` + `Index Scan` / `Index Seek` on inner.
