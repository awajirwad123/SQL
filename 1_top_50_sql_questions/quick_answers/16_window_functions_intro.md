# 16. Window Functions (ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD)

Q: What is a window function?
A: An analytic function that computes a value across a set of related rows (partition) without collapsing them.

Core examples:
```sql
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)
RANK()       OVER (PARTITION BY dept ORDER BY salary DESC)
DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary DESC)
LAG(salary, 1) OVER (PARTITION BY dept ORDER BY salary)
LEAD(salary, 1) OVER (PARTITION BY dept ORDER BY salary)
```

Q: Scenario uses?
A:
- ROW_NUMBER: Deduplicate keep best row.
- RANK: Show ties with gaps (1,1,3).
- DENSE_RANK: Leaderboards without gaps (1,1,2).
- LAG/LEAD: Period-over-period deltas, trend analysis.
- Any: Top-N per group with filter on row_number.

Q: Advantage vs subqueries?
A: Single pass over data; keeps all rows while adding analytic columns.

Q: Pitfall?
A: Omitting ORDER BY in window—results become nondeterministic for ranking/offset functions.
