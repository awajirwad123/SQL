# 27. EXPLAIN / Execution Plans

Q: Purpose of EXPLAIN?
A: Shows the optimizer's chosen access paths, join methods, ordering, and estimated costs/cardinality to diagnose performance.

Q: Key things to look for?
A: Full table scans on large tables, wrong join order, missing / unused indexes, high estimated vs actual row mismatch (when actuals available), expensive sorts/hash aggregates.

Q: Common join methods?
A: Nested Loop (good for selective lookups), Hash Join (large sets, equality), Merge Join (pre-sorted inputs).

Q: Example (generic):
```sql
EXPLAIN SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON o.customer_id = c.id
WHERE o.created_at >= CURRENT_DATE - INTERVAL '30 day'
GROUP BY c.name;
```

Q: Pitfall?
A: Treating estimated plan as absolute truth—statistics staleness can mislead; analyze/refresh stats first.
