# 20. Top-K Records per Category

Q: Top 3 per category pattern?
A: Rank within category then filter by rank <= K.
```sql
SELECT category, item_id, score
FROM (
  SELECT category, item_id, score,
         ROW_NUMBER() OVER (PARTITION BY category ORDER BY score DESC) AS rn
  FROM items
) t
WHERE rn <= 3;
```

Q: Keep ties (e.g., all items tied for 3rd)?
A: Use RANK and filter `rank <= 3`.

Q: Why not correlated subquery with LIMIT? 
A: LIMIT inside correlated subqueries is dialect-specific and harder to reason about; window approach is standard.

Q: Performance tip?
A: Composite index (category, score DESC) helps ordering per partition.

Q: Pitfall?
A: Using ORDER BY only at end—must be inside window for per-category ordering.
