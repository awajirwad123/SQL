# Correlated Subqueries

## Overview
A correlated subquery references columns from its outer query. It is logically evaluated once per outer row (though optimizers often transform it into a join or apply an index lookup). Common use cases: per-row existence checks, conditional aggregates, top-N within partition when window functions unavailable.

## Core Concepts
- **Correlation**: Inner query has predicate like `WHERE inner.customer_id = outer.customer_id` referencing the outer alias.
- **Logical vs Physical Execution**: While semantics imply row-by-row invocation, the optimizer may decorrelate (convert to semi-join, apply index seeks, or use hash/merge strategy).
- **Forms**:
  - EXISTS / NOT EXISTS
  - Scalar correlated subquery
  - Correlated IN (less typical vs EXISTS)
- **Semi-Join / Anti-Join**: EXISTS often becomes semi-join; NOT EXISTS becomes anti-join.
- **Index Leverage**: Performance hinges on selective index access inside subquery.

### Basic EXISTS Example
```sql
SELECT c.customer_id, c.name
FROM customers c
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.customer_id = c.customer_id
    AND o.order_date >= CURRENT_DATE - INTERVAL '30 day'
);
```

### Correlated Aggregate (Scalar Subquery)
```sql
SELECT o.order_id,
       (SELECT COUNT(*) FROM order_items i WHERE i.order_id = o.order_id) AS item_count
FROM orders o;
```

### NOT EXISTS Anti-Join
```sql
SELECT c.customer_id
FROM customers c
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.customer_id = c.customer_id
);
```

## Interview-Focused Notes
1. **"Define correlated subquery"** – Subquery referencing outer row values; logically per-row evaluation.
2. **"Performance risks?"** – Potential N * inner cost if not decorrelated; mitigated by indexes and optimizer rewriting.
3. **"EXISTS vs IN (correlated)"** – EXISTS stops at first match; IN often materializes set (engine dependent); EXISTS safer with NULL.
4. **"When prefer JOIN over correlated?"** – When needing multiple columns from inner table or for clarity/performance after ensuring no duplication issues.
5. **"Decorrelating?"** – Rewriting correlated subquery into a derived table / join or using window functions to reduce repeated computation.

## Quick Recall ✅
- Correlated: inner depends on outer.
- EXISTS = semi-join; NOT EXISTS = anti-join.
- Index inner predicate columns.
- Rewrite to JOIN/aggregation when retrieving multiple attributes.
- Watch scalar correlated aggregates for hot paths.

## Interview Traps & Confusions ⚠️
- Assuming guaranteed slow performance (modern optimizers often unnest).
- Forgetting duplicates when rewriting to JOIN (need DISTINCT or GROUP BY).
- Using `COUNT(*)` inside correlated subquery when only existence needed (use EXISTS for early exit).
- Misusing NOT IN vs NOT EXISTS with NULL semantics.
- Leaving unindexed correlation columns causing full scans.

## Bonus
### Rewrite Correlated Aggregate to JOIN
Instead of:
```sql
SELECT o.order_id,
       (SELECT SUM(quantity) FROM order_items i WHERE i.order_id = o.order_id) AS qty
FROM orders o;
```
Rewrite:
```sql
SELECT o.order_id, COALESCE(SUM(i.quantity),0) AS qty
FROM orders o
LEFT JOIN order_items i ON i.order_id = o.order_id
GROUP BY o.order_id;
```

### EXISTS Short-Circuit Benefit
Stops after first match; efficient for presence tests with proper index.

### Anti-Join Performance
`NOT EXISTS` is usually safer than `NOT IN (SELECT ...)` if NULLs might appear.

### Partial Index Assist (Postgres)
```sql
CREATE INDEX idx_orders_recent ON orders (customer_id)
WHERE order_date >= CURRENT_DATE - INTERVAL '30 day';
```
Optimizes recent activity EXISTS check.

### Lateral / APPLY Alternative
Some correlated forms better expressed via LATERAL (Postgres) / CROSS APPLY (SQL Server) to fetch top-N related rows.
