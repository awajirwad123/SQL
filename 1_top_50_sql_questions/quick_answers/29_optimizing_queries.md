# 29. Optimizing Queries

Q: Core goal?
A: Reduce I/O and CPU by enabling efficient access paths and minimizing processed rows early.

Key Techniques:
- Sargable predicates (no functions wrapping indexed columns).
- Proper composite index order (equality cols first, then range, then ordering).
- Avoid SELECT *; fetch only needed columns to help covering indexes.
- Replace correlated subqueries with joins or window functions where possible.
- Pre-aggregate early in subqueries/CTEs to shrink data before joins.

Example rewrite:
```sql
-- Bad
SELECT * FROM orders WHERE DATE(created_at) = CURRENT_DATE;
-- Good
SELECT col_list FROM orders
WHERE created_at >= CURRENT_DATE AND created_at < CURRENT_DATE + INTERVAL '1 day';
```

Pitfalls: Ignoring stale statistics; unnecessary DISTINCT to hide duplicates; functions on both sides of join condition disabling index usage.
