# 26. CTEs & Recursive CTEs

Q: What is a CTE?
A: A named temporary result set defined with WITH, improving readability and reuse within a single statement.

Q: Basic example?
```sql
WITH high_value AS (
  SELECT * FROM orders WHERE amount > 1000
)
SELECT COUNT(*) FROM high_value;
```

Q: Recursive CTE purpose?
A: Traverse hierarchies or generate sequences.

Q: Recursive pattern?
```sql
WITH RECURSIVE org AS (
  SELECT employee_id, manager_id, 0 AS lvl
  FROM employees
  WHERE manager_id IS NULL  -- anchor
  UNION ALL
  SELECT e.employee_id, e.manager_id, o.lvl + 1
  FROM employees e
  JOIN org o ON e.manager_id = o.employee_id
)
SELECT * FROM org;
```

Q: Termination?
A: Stops when recursive step returns no new rows.

Q: Pitfall?
A: Missing cycle prevention; use level limit or DISTINCT if risk of loops.
