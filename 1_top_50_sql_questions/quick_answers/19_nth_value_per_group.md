# 19. Nth Value in Each Group

Q: Get 2nd / 3rd / Nth highest per group?
A: Use window ranking then filter.
```sql
SELECT dept, employee_id, salary
FROM (
  SELECT dept, employee_id, salary,
         DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rk
  FROM employees
) x
WHERE rk = 2; -- 2nd highest (change to 3 for 3rd, etc.)
```

Q: Difference ROW_NUMBER vs RANK vs DENSE_RANK here?
A: For distinct Nth value (skip duplicates) prefer DENSE_RANK; for physical Nth row regardless of ties use ROW_NUMBER.

Q: Single-row alternative (Postgres)?
A: `SELECT DISTINCT ON (dept) ... ORDER BY dept, salary DESC` (but only picks 1—combine with offset for Nth via subquery).

Q: Pitfall?
A: Using MAX with WHERE salary < (SELECT MAX...) chaining—breaks with duplicate/top gaps and is inefficient.
