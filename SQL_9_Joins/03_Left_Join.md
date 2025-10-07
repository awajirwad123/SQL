# LEFT JOIN

## Overview
A `LEFT JOIN` (LEFT OUTER JOIN) returns all rows from the left table plus matched rows from the right table; right-side non-matches become `NULL` padded.

## Core Concepts
- Preservation side: Left table.
- Join condition executed; unmatched right rows replaced with `NULL` values.
- Filtering right table columns must be handled carefully.

**Syntax:**
```sql
SELECT c.customer_id, c.name, o.order_id, o.order_total
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id;
```

**Conditional Join Filtering:**
```sql
SELECT c.customer_id, c.name,
       COALESCE(o.order_total, 0) AS last_order_total
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id
 AND o.status = 'COMPLETED';
```

## Interview-Focused Notes
1. "When do you use LEFT JOIN?" – When the driving (left) entity list must remain intact regardless of matches.
2. "How to filter only unmatched rows?" – Use `WHERE o.order_id IS NULL` after the join.
3. "How to keep unmatched while filtering matches?" – Put match-specific predicates in `ON` clause.
4. "Does LEFT JOIN order matter?" – Yes for semantics: the preserved table must be physically written on the left.
5. "Performance considerations?" – Index on right join key drastically reduces probe cost.

## Quick Recall ✅
- Keeps all left rows.
- Right unmatched => NULLs.
- Filtering location matters.
- Typical for reporting & completeness.
- Use `IS NULL` pattern for anti-joins.

## Interview Traps & Confusions ⚠️
- Misplaced filter in WHERE causing unintended inner join behavior.
- Assuming `COUNT(o.order_id)` counts all customers (it skips NULL); use `COUNT(*)` or conditional count.
- Forgetting that expressions on right columns will be NULL for unmatched rows (wrap with `COALESCE`).

## Bonus
### Find Customers With No Orders (Anti-Join)
```sql
SELECT c.customer_id, c.name
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_id IS NULL;
```

### Conditional Aggregation
```sql
SELECT c.customer_id,
       COUNT(o.order_id) AS total_orders,
       SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_orders
FROM customers c
LEFT JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.customer_id;
```

### Equivalent Using NOT EXISTS (Often Preferred)
```sql
SELECT c.customer_id, c.name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

### Choosing Between LEFT JOIN IS NULL vs NOT EXISTS
- `NOT EXISTS` often clearer for anti-join intent.
- Query planner may produce identical plan; prefer readability.
