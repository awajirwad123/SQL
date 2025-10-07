# Nested Queries

## Overview
Nested queries (multi-level subqueries) layer multiple inner SELECT statements. Deep nesting can express complex filtering, ranking, or staged transformations, but excessive depth often signals refactor potential into CTEs, window functions, or set-based joins for readability and performance.

## Core Concepts
- **Multi-Level Nesting**: Subquery inside subquery; each layer may filter, aggregate, or reshape.
- **Progressive Reduction**: Inner query narrows candidate set; outer queries refine or enrich.
- **Refactor Tools**:
  - Common Table Expressions (CTEs)
  - Window functions (avoid nested aggregates)
  - Derived tables
- **Optimizer Flattening**: Many engines flatten/unnest where semantics allow, reducing redundant passes.
- **Readability vs Performance**: Sometimes a deeply nested query is logically clear but less maintainable; other times unnesting reveals better index usage.

### Example (Three Levels)
```sql
SELECT customer_id, total_last_30
FROM (
  SELECT customer_id, SUM(total_amount) AS total_last_30
  FROM (
    SELECT *
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 day'
  ) recent
  GROUP BY customer_id
) agg
WHERE total_last_30 > 500;
```

### Refactored with CTE
```sql
WITH recent AS (
  SELECT *
  FROM orders
  WHERE order_date >= CURRENT_DATE - INTERVAL '30 day'
), agg AS (
  SELECT customer_id, SUM(total_amount) AS total_last_30
  FROM recent
  GROUP BY customer_id
)
SELECT customer_id, total_last_30
FROM agg
WHERE total_last_30 > 500;
```

### Replace Nested Aggregation with Window Function
```sql
SELECT customer_id, total_last_30
FROM (
  SELECT customer_id,
         SUM(CASE WHEN order_date >= CURRENT_DATE - INTERVAL '30 day' THEN total_amount END) OVER (PARTITION BY customer_id) AS total_last_30
  FROM orders
) x
WHERE total_last_30 > 500;
```

## Interview-Focused Notes
1. **"Why nested queries?"** – Encapsulate stages: filter → aggregate → filter again.
2. **"Performance impact?"** – Potentially redundant materialization; optimizer may flatten but not always.
3. **"When refactor to CTE?"** – For clarity, reuse, or to isolate logic for conditional modifications.
4. **"Deep nesting smell?"** – Likely window function or join-based rewrite opportunity.
5. **"How optimizer handles?"** – Unnests merges where safe; correlated dependencies can block flattening.

## Quick Recall ✅
- Nesting expresses pipeline of transformations.
- Replace multi-pass aggregates with window functions.
- CTEs improve readability; may or may not materialize (engine/version dependent).
- Examine execution plan to verify unnesting.
- Too many layers → refactor.

## Interview Traps & Confusions ⚠️
- Assuming CTE always slower (modern engines optimize inline; depends on version e.g. PG12+ inlining non-recursive CTEs, older versions materialize).
- Overusing nesting for tasks solvable by analytic functions.
- Misinterpreting identical column names causing ambiguity.
- Believing nesting automatically isolates heavy computation (optimizer might merge it back).
- Ignoring indexing on inner-most filtering stages.

## Bonus
### Inline View vs CTE Behavior
- Some vendors historically treat CTE as optimization fence (legacy PG <12); derived tables often optimized more aggressively.

### De-correlation Opportunity
Nested `WHERE col IN (SELECT col FROM ( ... ))` can sometimes flatten to a single semi-join.

### Debugging Strategy
Unwrap outward: run inner-most subquery alone to validate row set size & cardinality assumptions.

### Minimizing Data Early
Push selective predicates to innermost layer to reduce intermediate volume.

### Hybrid Approach
Use CTE for readability + add final selective predicate outermost for plan pruning.
