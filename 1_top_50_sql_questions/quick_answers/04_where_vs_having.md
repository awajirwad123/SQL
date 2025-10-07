# 4. WHERE vs HAVING

Q: Core difference?
A: WHERE filters rows before grouping; HAVING filters groups after aggregation.

Q: Example?
A: SELECT dept_id, COUNT(*) FROM employees WHERE active=1 GROUP BY dept_id HAVING COUNT(*)>5;

Q: Performance angle?
A: Push non-aggregate filters into WHERE to reduce grouped set size.

Q: Can HAVING use non-aggregated columns?
A: Only if they appear in GROUP BY (standard compliance).

Q: Common mistake?
A: Putting simple row filters in HAVING—forces unnecessary grouping work.
