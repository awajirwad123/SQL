# 17. PARTITION BY vs GROUP BY

Q: Core difference?
A: GROUP BY collapses rows into one per group; PARTITION BY defines row windows but preserves every original row.

Q: Example difference?
```sql
-- GROUP BY (one row per dept)
SELECT dept, AVG(salary) FROM employees GROUP BY dept;

-- PARTITION BY (each row keeps its identity + dept avg)
SELECT emp_id, dept, salary,
       AVG(salary) OVER (PARTITION BY dept) AS dept_avg
FROM employees;
```

Q: When choose PARTITION BY?
A: When you need both row-level detail and group-level metrics simultaneously.

Q: Can they mix?
A: Yes (aggregate first, then window over result set, or vice versa in subqueries/CTEs).

Q: Pitfall?
A: Thinking PARTITION BY reduces row count—it does not.
