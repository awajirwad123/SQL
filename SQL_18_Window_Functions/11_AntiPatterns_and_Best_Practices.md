# Anti-Patterns & Best Practices (Window Functions)

## Overview
Proper use of window functions improves clarity and performance. Misuse leads to non-determinism, excessive resource consumption, and logical errors.

## Anti-Patterns
| Anti-Pattern | Issue | Better Approach |
|--------------|-------|-----------------|
| Omitting ORDER BY in ranking | Non-deterministic ordering | Specify ORDER BY with tie-breaker |
| Using RANGE for row-limited logic | Unexpected peer grouping | Use ROWS frame |
| LAST_VALUE without extended frame | Returns current row | Add `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` |
| Repeated identical OVER clauses | Redundant sorts | Named window / reuse |
| Window in WHERE via alias | Invalid phase order | Wrap in subquery/CTE |
| Large unbounded frames | Memory & spill risk | Bounded/filtered frames or pre-agg |
| Over-windowing (window for simple total) | Complexity & overhead | Plain aggregate + join if collapsing fine |
| Calculating same LAG multiple times | Extra work | Alias in subquery |
| Ignoring null semantics in ranking | Unstable tie behavior | Explicit NULLS FIRST/LAST |
| Expecting row count-based frame with duplicates using RANGE | Unexpected size | Use ROWS |

## Best Practices
1. Always declare ORDER BY for ranking & navigation.
2. Use ROWS frames for deterministic running / moving calculations.
3. Define explicit frames (avoid relying on ambiguous defaults).
4. Reuse window specs where possible.
5. Filter & project early to shrink window workload.
6. Provide deterministic tie-breakers (PK) in ORDER BY.
7. Document complex frame logic with comments.
8. Combine multiple metrics in single pass.
9. Validate performance with plan + timing (sort cost, spills).
10. Use appropriate approximation for large percentile workloads.

## Interview-Focused Notes
1. Why explicit frame? Clarity & correct semantics (e.g., LAST_VALUE pitfalls).
2. ROWS vs RANGE for moving average? ROWS ensures fixed number; RANGE groups peers.
3. When revert to GROUP BY? When aggregation collapses rows and no per-row metric needed.
4. Why tie-breaker needed? Stability across executions.
5. How handle large partitions? Pre-aggregate or partition further logically.

## Quick Recall ✅
- Frame matters; defaults can mislead.
- Determinism via ORDER BY + tiebreaker.
- Reuse windows to reduce sort overhead.
- Use subquery for post-window filtering.
- Pick ROWS for strict sliding windows.

## Interview Traps & Confusions ⚠️
- Believing windows run before WHERE.
- Using windowed column in GROUP BY incorrectly.
- Expecting window to prune rows (it doesn’t filter).
- Misinterpreting percent_rank as exact percentile value threshold.
- Overcomplicating simple aggregate tasks with windows.

## Bonus
### Comment Template
`/* Running total: ROWS frame ensures no peer grouping anomalies */`

### Regression Detection
Baseline execution time & row counts; alert if sort spill occurs (temp usage spike).

### Window Spec Normalization Script
Detect duplicate window expressions for refactor opportunities.

### Metrics Dashboard
Track window query frequency, average sort memory, spill events.
