# 15. Aggregation Basics

Q: How to compute grouped count/sum/avg?
A: Use GROUP BY with aggregate functions.

Example:
```sql
SELECT department_id,
       COUNT(*)        AS emp_count,
       SUM(salary)     AS total_salary,
       AVG(salary)     AS avg_salary,
       MIN(salary)     AS min_salary,
       MAX(salary)     AS max_salary
FROM employees
GROUP BY department_id;
```

Q: Filter rows before vs after grouping?
A: WHERE filters input rows; HAVING filters aggregated groups.

Q: Count non-null vs all rows?
A: COUNT(col) skips NULL; COUNT(*) counts every row.

Q: Add derived metric (percent of total)?
A: Use window over all rows or subquery:
```sql
SELECT department_id,
       COUNT(*) AS emp_count,
       100.0 * COUNT(*) / SUM(COUNT(*)) OVER() AS pct
FROM employees
GROUP BY department_id;
```

Q: Pitfall?
A: Selecting non-aggregated column not in GROUP BY (invalid / undefined depending on SQL mode).