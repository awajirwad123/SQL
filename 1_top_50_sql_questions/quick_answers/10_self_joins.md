# 10. Self JOINs

Q: What is a self join?
A: A table joined to itself using table aliases.

Q: Common use case?
A: Hierarchies (employee → manager), adjacency lists, sequencing (compare row to previous).

Q: Example (manager name):
```sql
SELECT e.employee_id, e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

Q: Sequencing example (find increasing salaries):
```sql
SELECT a.emp_id, a.salary
FROM salaries a
JOIN salaries b ON a.emp_id = b.emp_id AND a.year = b.year + 1 AND a.salary > b.salary;
```

Q: Pitfall?
A: Missing alias clarity leading to ambiguous column references.
