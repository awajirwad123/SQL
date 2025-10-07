# 24. Highest/Lowest per Group

Q: Get employee(s) with highest salary per dept (handle ties)?
A: Rank and pick rank = 1.
```sql
SELECT dept, employee_id, salary
FROM (
  SELECT dept, employee_id, salary,
         DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rk
  FROM employees
) x
WHERE rk = 1;
```

Q: Lowest instead?
A: ORDER BY salary ASC.

Q: Single-row per group (ignore ties)?
A: Use ROW_NUMBER instead of DENSE_RANK.

Q: Alternative approach?
A: Join to subquery of max salary per dept.
```sql
SELECT e.*
FROM employees e
JOIN (
  SELECT dept, MAX(salary) AS max_sal FROM employees GROUP BY dept
) m ON e.dept = m.dept AND e.salary = m.max_sal;
```

Q: Pitfall?
A: Using correlated subquery per row without index on salary/dept causing scans.
