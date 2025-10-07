# Query Rewriting Patterns

## Overview
Query rewriting refactors SQL to produce equivalent results with lower cost, improved index usage, or simplified plan shape. Optimizers perform many rewrites automatically, but deliberate manual rewrites can unlock better performance when the optimizer misses opportunities.

## Core Concepts
- **Predicate Pushdown**: Move filters as close to base tables as possible.
- **Projection Minimization**: Select only needed columns; reduce I/O & memory footprint.
- **Aggregation Reordering**: Pre-aggregate before join to shrink intermediate sets.
- **Set vs Procedural**: Replace row-by-row logic (cursors/UDF loops) with set operations.
- **IN / EXISTS / JOIN Equivalence**: Select form that best matches required semantics & plan potential.
- **De-correlation**: Convert correlated subqueries into joins or window functions.
- **Window Functions**: Replace self-joins or nested subqueries for ranking or partition aggregates.
- **UNION vs UNION ALL**: Use ALL if duplicates not a problem (avoids sort/distinct).
- **OR to UNION ALL**: Split disjoint predicates enabling index seeks (if selective) then combine.

### Example: Aggregation Pushdown
Instead of:
```sql
SELECT c.region, SUM(o.total_amount)
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
GROUP BY c.region;
```
If many orders per customer, pre-aggregate:
```sql
WITH order_totals AS (
  SELECT customer_id, SUM(total_amount) AS cust_total
  FROM orders GROUP BY customer_id
)
SELECT c.region, SUM(o.cust_total)
FROM customers c
JOIN order_totals o ON o.customer_id = c.customer_id
GROUP BY c.region;
```

### OR to UNION ALL
```sql
SELECT * FROM users WHERE status='ACTIVE' OR created_at >= CURRENT_DATE - INTERVAL '7 day';
```
Rewrite (if predicates mostly disjoint):
```sql
SELECT * FROM users WHERE status='ACTIVE'
UNION ALL
SELECT * FROM users WHERE created_at >= CURRENT_DATE - INTERVAL '7 day' AND status <> 'ACTIVE';
```

## Interview-Focused Notes
1. Why push predicates early? Shrinks data volume for downstream operators.
2. Difference UNION vs UNION ALL? ALL skips duplicate elimination sort.
3. When de-correlate subquery? When correlated execution repeated & row counts high.
4. When split OR predicates? When each predicate individually selective & indexes exist.
5. Benefit of window function rewrite? Single pass vs multiple self-joins or nested subqueries.

## Quick Recall ✅
- Filter early, project narrow.
- Aggregate earlier when it reduces rows significantly.
- Prefer UNION ALL unless dedupe required.
- Use window functions for ranking / partition stats.
- Eliminate function wrapping index columns.

## Interview Traps & Confusions ⚠️
- Over-splitting queries adding overhead.
- Rewriting for micro optimization with no measurement.
- Using DISTINCT to hide duplication issues caused by wrong join.
- Overusing CTEs creating artificial fences (old engine versions).
- Unnecessary ORDER BY in subqueries (unless needed for window logic).

## Bonus
### DISTINCT vs GROUP BY
If only removing duplicates, GROUP BY columns alone may produce clearer plan choices (engine dependent).

### Top-N Per Group
Window function with ROW_NUMBER filter vs correlated subquery selecting MIN/MAX.

### Materialized View Candidate
If rewrite repeatedly used & expensive: consider indexed/materialized view.

### Expression Normalization
Replace `WHERE date(created_at)=...` with range predicate using sargable `created_at >= ... AND < ...`.

### Detect Over-Fetch
Check columns referenced by application; remove unused SELECT list fields.
