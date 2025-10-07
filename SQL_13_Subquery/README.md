# SQL Subqueries Module (SQL_13_Subquery)

## Contents
1. `01_Subquery.md` – Fundamentals (positions, cardinalities, IN/EXISTS, scalar, ANY/ALL, tuple tests).
2. `02_Correlated_Subqueries.md` – Correlation semantics, EXISTS/NOT EXISTS, decorrelation, performance.
3. `03_Nested_Queries.md` – Multi-level nesting, refactoring, window & CTE alternatives.

## Learning Goals
- Master placement of subqueries (SELECT, FROM, WHERE, HAVING) & their implications.
- Understand when to use EXISTS vs IN vs JOIN.
- Recognize and refactor inefficient correlated and nested patterns.
- Apply window functions / CTEs to reduce depth and improve clarity.

## Decision Heuristics
| Task | Preferred Pattern |
|------|-------------------|
| Existence check | `EXISTS (SELECT 1 ...)` |
| Membership with possible NULLs | `EXISTS` instead of `NOT IN` |
| Fetch extra columns | JOIN / derived table |
| One scalar value per row group | Window function or correlated aggregate (if selective) |
| Multi-stage transform | CTE chain |

## Red Flags / Smells
- `NOT IN (SELECT ...)` where subquery column nullable.
- Deep (3+ layer) nesting doing simple aggregations.
- Repeated correlated aggregates on same table (consider pre-aggregation or window).
- Scalar subquery performing full scan each outer row (index missing).
- Duplicate logic across nested layers (mergeable).

## Performance Tips
- Index correlation columns (`inner_table.foreign_key`).
- Use selective predicates earliest (push down).
- Convert correlated EXISTS to semi-join and verify plan.
- Replace repeated scalar lookups with join + aggregation.
- Check execution plan for “Nested Loop + Subquery” patterns to spot potential unnesting.

## Practice Prompts
1. Rewrite a correlated subquery using a JOIN; preserve semantics.
2. Diagnose why `NOT IN` returned zero rows when nulls present.
3. Flatten a 4-level nested aggregation using window functions.
4. Add indexing strategy for an EXISTS filter on recent activity.
5. Compare plan for IN vs EXISTS on same dataset.

## Next Suggested Modules
- Execution Plans & Costing (tie in subquery unnesting)
- Window Functions Deep Dive
- Set Operations (UNION / INTERSECT / EXCEPT) interplay with subqueries

Use this module with Joins & Indexes to evaluate rewrite alternatives quickly during interviews.
