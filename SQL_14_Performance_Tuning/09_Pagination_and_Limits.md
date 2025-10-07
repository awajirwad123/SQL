# Pagination & Limits

## Overview
Pagination retrieves subsets of ordered results for user interfaces or batch processing. Naïve approaches (OFFSET large N) cause increasing latency. Efficient pagination minimizes skipped work and leverages indexes.

## Core Concepts
- **OFFSET Cost**: Engine still must scan/produce skipped rows then discard.
- **Keyset (Seek) Pagination**: Use last seen key(s) as starting point (WHERE key > last_key) to avoid OFFSET.
- **Stable Ordering**: Deterministic ORDER BY required for consistent pagination; include unique tiebreaker (e.g., primary key) if necessary.
- **Covering Index**: Supports ORDER BY + WHERE simultaneously to avoid sort + extra lookups.
- **Window-Based Paging**: ROW_NUMBER() in derived table; may still scan all rows first.
- **Scrolling vs Jump-To-Page**: Keyset great for scrolling forward; random page jumps require counting (costly) or approximate methods.

### Inefficient
```sql
SELECT * FROM orders ORDER BY created_at DESC OFFSET 500000 LIMIT 50;
```

### Keyset Pagination
```sql
-- First page
SELECT * FROM orders
ORDER BY created_at DESC, order_id DESC
LIMIT 50;
-- Next page (use last row values)
SELECT * FROM orders
WHERE (created_at, order_id) < (:last_created_at, :last_order_id)
ORDER BY created_at DESC, order_id DESC
LIMIT 50;
```

### Covering Index Example
`CREATE INDEX ix_orders_created_at_id ON orders (created_at DESC, order_id DESC) INCLUDE (customer_id, total_amount);`

## Interview-Focused Notes
1. OFFSET performance issue? Still processes skipped rows.
2. Keyset requirement? Strict, unique, monotonic ordering components.
3. When still use OFFSET? Small pages, low total row counts, admin tools.
4. Jump to page 1000? Consider approximate counts or redesigned UX (cursor-based).
5. Benefit of covering index? Avoid sort + random heap lookups; stable ordering.

## Quick Recall ✅
- Prefer keyset over deep OFFSET.
- Always include unique tiebreaker.
- Index matches ORDER BY sequence.
- Paginate deterministically.
- Large counts expensive; cache or approximate.

## Interview Traps & Confusions ⚠️
- Omitting secondary key causing duplicates across pages.
- Using inconsistent filters between pages.
- Sorting on unindexed expression (forces filesort / temp sort).
- Attempting backward pagination without reverse comparator logic.
- Over-counting total pages every request (scales poorly).

## Bonus
### Backward Pagination
Reverse comparator: `(created_at, order_id) > (:first_created_at, :first_order_id)` with same ordering.

### Approximate Count
Use reltuples (Postgres) or sys indexes row estimates for fast approximate total.

### Seek with Composite Filters
Append filter predicates to keyset WHERE ensuring index compatibility.

### Window Row_Number Paging
```sql
WITH ranked AS (
  SELECT o.*, ROW_NUMBER() OVER (ORDER BY created_at DESC, order_id DESC) AS rn
  FROM orders o
)
SELECT * FROM ranked WHERE rn BETWEEN 1001 AND 1050;
```
(Still processes all prior rows.)

### Infinite Scroll Strategy
Load more with keyset; avoid page numbers entirely.
