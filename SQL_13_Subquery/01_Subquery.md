# Subquery (SQL Subquery)

## Overview
A subquery (inner query) is a SELECT embedded inside another SQL statement (outer query). It can return a single scalar value, a row, a column set, or an entire relation. Subqueries enable composability, conditional filtering, dynamic thresholds, set membership tests, and incremental aggregation.

## Core Concepts
- **Positions Subqueries Appear**:
  - SELECT list (scalar subquery)
  - FROM clause (derived table / inline view)
  - WHERE / HAVING (predicate filtering)
  - EXISTS / IN / ANY / ALL constructs
  - INSERT, UPDATE, DELETE (data modification using computed sets)
- **Result Cardinalities**:
  - Scalar (exactly one value)
  - Single-column multi-row (list for IN)
  - Multi-column (tuple comparison e.g., `(a,b) IN (SELECT ...)` )
  - Table (used in FROM)
- **Types**:
  - Non-correlated: independent; executed once then reused.
  - Correlated: references outer query columns (executed per row unless optimized/unnested).
- **Set Membership**:
  - `IN` / `NOT IN` vs `EXISTS` / `NOT EXISTS` (NULL sensitivity differences).
- **ANY / SOME / ALL**:
  - Compare one value to a set: `value > ALL(subquery)`, `value = ANY(subquery)`.
- **Derived Table vs CTE**: Derived table is local & inline; CTE named and can be referenced multiple times (but may materialize depending on engine/version).

### Example (Filtering by Aggregated Threshold)
```sql
SELECT o.customer_id, o.order_id, o.total_amount
FROM orders o
WHERE o.total_amount > (
  SELECT AVG(total_amount)
  FROM orders
  WHERE order_date >= CURRENT_DATE - INTERVAL '30 day'
);
```

### Scalar Subquery
```sql
SELECT order_id,
       (SELECT MAX(shipped_at) FROM shipments s WHERE s.order_id = o.order_id) AS last_ship_time
FROM orders o;
```

## Interview-Focused Notes
1. **"Define subquery"** – Query nested inside another statement to provide intermediate result sets.
2. **"Difference derived table vs subquery in WHERE"** – Derived table supplies rows to FROM; WHERE subquery supplies predicate values / membership tests.
3. **"When prefer EXISTS over IN?"** – When correlated logic or potential NULL membership issues with NOT IN.
4. **"Scalar subquery expectation?"** – Must return at most one row; else runtime error (except engines that implicitly limit with aggregate).
5. **"CTE vs subquery performance?"** – Often equivalent after optimization; historically some engines materialize CTE (e.g., old PG versions) affecting performance.

## Quick Recall ✅
- Subquery location changes role (predicate, source, expression).
- IN list vs EXISTS semantics & NULL pitfalls.
- Scalar must return single row.
- ANY/ALL enable comparative logic.
- Consider rewriting to JOIN for performance clarity.

## Interview Traps & Confusions ⚠️
- Using `NOT IN` with possible NULLs → empty result (because comparisons become UNKNOWN).
- Confusing `IN (SELECT ...)` with needing uniqueness (engine can de-duplicate implicitly but duplicates add overhead).
- Assuming subquery executes always row-by-row (optimizer may decorrelate).
- Relying on scalar subquery that returns multiple rows (error risk).
- Over-nesting instead of flattening / using window functions.

## Bonus
### ANY / ALL Example
```sql
SELECT product_id
FROM products
WHERE price < ALL (SELECT price FROM products WHERE category = 'Premium');
```

### Tuple Comparison
```sql
SELECT *
FROM orders o
WHERE (o.customer_id, o.status) IN (
  SELECT customer_id, status FROM flagged_customers
);
```

### Using a Derived Table for Pre-Aggregation
```sql
SELECT c.customer_id, c.order_count
FROM (
  SELECT customer_id, COUNT(*) AS order_count
  FROM orders
  GROUP BY customer_id
) c
WHERE c.order_count > 10;
```

### Replace Subquery with Window Function
Instead of:
```sql
SELECT o.*
FROM orders o
WHERE o.total_amount > (SELECT AVG(total_amount) FROM orders);
```
Use:
```sql
SELECT *
FROM (
  SELECT o.*, AVG(total_amount) OVER() AS global_avg
  FROM orders o
) x
WHERE total_amount > global_avg;
```

### Performance Hint
Ensure filtering subqueries have indexes on join/filter columns (e.g. index `shipments(order_id)` for scalar shipment lookup).
