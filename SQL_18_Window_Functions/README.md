# SQL Window Functions Module (SQL_18_Window_Functions)

## Contents
1. `01_Window_Functions_Overview.md` – Purpose & categories.
2. `02_Window_Syntax_Anatomy.md` – OVER clause structure & frames.
3. `03_Ranking_Functions.md` – ROW_NUMBER, RANK, DENSE_RANK, NTILE.
4. `04_Aggregate_Window_Functions.md` – Running totals, moving averages, percent-of.
5. `05_Frames_and_Frame_Types.md` – ROWS vs RANGE vs GROUPS & boundaries.
6. `06_Running_and_Moving_Calculations.md` – Sliding & cumulative patterns.
7. `07_Lag_Lead_and_Value_Comparisons.md` – Navigation & deltas.
8. `08_Percentiles_and_Distribution.md` – Percentiles, cumulative distribution.
9. `09_Rewrite_Patterns_with_Windows.md` – Eliminating self-joins & correlated subqueries.
10. `10_Performance_and_Optimization.md` – Sorting, indexing, frame tuning.
11. `11_AntiPatterns_and_Best_Practices.md` – Pitfalls & guidance.

## Core Heuristics
| Goal | Pattern |
|------|---------|
| Top-N per group | ROW_NUMBER + filter |
| Running total | SUM OVER (ORDER BY ...) ROWS UNBOUNDED PRECEDING |
| Moving average | AVG OVER ROWS n PRECEDING |
| Delta vs previous | LAG() difference |
| Percent-of total | val / SUM(val) OVER (PARTITION) |
| Median / percentiles | percentile_cont / percentile_disc |
| Gap detection | LEAD(ts) - ts |

## Red Flags
- LAST_VALUE returning current row (missing frame end).
- RANGE used unintentionally for row-based sliding logic.
- Window column used in WHERE (logical order misunderstanding).
- Duplicate window clauses inflating plan cost.
- Huge unbounded partitions w/ memory spills.

## Performance Checklist
1. Appropriate index on (PARTITION BY + ORDER BY)
2. Explicit ROWS frame where needed
3. Slim projection pre-window
4. Named window reuse
5. Monitor sort memory/spills

## Rewrite Playbook
| Original Pattern | Rewrite |
|------------------|---------|
| Correlated sum per group | SUM() OVER PARTITION |
| Self-join for previous row | LAG() |
| GROUP BY + join back for percent-of | Window SUM + division |
| Max timestamp dedupe | ROW_NUMBER filter |
| Ranking join for top record | ROW_NUMBER / RANK filter |

## Practice Prompts
1. Convert correlated running total query into window form.
2. Fix LAST_VALUE misuse returning current row.
3. Implement 30-day rolling sum and daily percent-of rolling total.
4. Produce percentiles (p50/p90/p95) in one query.
5. Optimize multi-metric window query suffering sort spill.

## Integration With Other Modules
- Subqueries: Many rewrites to window forms.
- Performance Tuning: Sort cost & indexing synergy.
- Transactions: Avoid long-running analytic windows in OLTP critical paths.
- Indexes: Composite index alignment reduces sorting.

## Next Suggested Modules
- Partitioning & Data Lifecycle Management
- Change Data Capture & Streaming Analytics
- Advanced Security (RLS, Masking)
- Data Quality & Validation Patterns

Leverage this module to refine analytic SQL expressions and replace procedural or multi-pass patterns with efficient declarative windows.
