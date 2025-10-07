# 42. Data Validation / Integrity Checks

Q: Declarative checks?
A: Constraints: NOT NULL, CHECK, FOREIGN KEY, UNIQUE, PRIMARY KEY.

Q: Pattern-based checks?
A: Use CHECK with LIKE/regex (where supported) or computed columns validated by constraint.

Q: Batch integrity audit example?
```sql
SELECT 'neg_salary' AS issue, COUNT(*) AS cnt
FROM employees WHERE salary < 0
UNION ALL
SELECT 'future_birthdate', COUNT(*) FROM employees WHERE birthdate > CURRENT_DATE
UNION ALL
SELECT 'missing_dept', COUNT(*) FROM employees e
  WHERE NOT EXISTS (SELECT 1 FROM departments d WHERE d.id = e.dept_id);
```

Q: Duplicate detection?
A: GROUP BY with HAVING COUNT(*) > 1 or UNIQUE constraint to prevent.

Q: Referential validation?
A: NOT EXISTS anti-join pattern for missing parents.

Q: Automation?
A: Schedule integrity scripts; log deltas; alert when thresholds breached.

Pitfall: Relying solely on application-side checks—race conditions still allow bad rows without DB-level constraints.