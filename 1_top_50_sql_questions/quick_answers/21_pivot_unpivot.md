# 21. Pivot & Unpivot

Q: Pivot (rows→columns)?
A: Aggregate with conditional logic or vendor PIVOT syntax.
```sql
SELECT dept,
       SUM(CASE WHEN quarter='Q1' THEN revenue END) AS q1,
       SUM(CASE WHEN quarter='Q2' THEN revenue END) AS q2
FROM sales
GROUP BY dept;
```

Q: Unpivot (columns→rows)?
A: UNION ALL or vendor UNPIVOT.
```sql
SELECT dept, 'Q1' AS quarter, q1 AS revenue FROM rev
UNION ALL
SELECT dept, 'Q2', q2 FROM rev;
```

Q: Dynamic pivot?
A: Build SQL string of target columns at runtime (requires procedural layer).

Q: Pitfall?
A: Hard-coding columns—must adjust when new categories appear.
