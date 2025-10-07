# RECURSIVE JOIN (Recursive CTE for Hierarchies)

## Overview
A recursive join operation is typically implemented in SQL via a Recursive Common Table Expression (CTE) to traverse hierarchical (tree or graph-like) relationships stored using adjacency lists (parent_id references). It repeatedly self-joins a table until termination.

## Core Concepts
- Structure: `WITH RECURSIVE cte AS (anchor UNION ALL recursive-step)`
- Anchor member: Base level (e.g., root nodes where parent_id IS NULL).
- Recursive member: Joins CTE back to table to fetch next level.
- Terminates when recursive step returns zero rows.
- Produces depth-enriched result useful for tree rendering, aggregation, path construction.

**Example: Employee Hierarchy**
```sql
WITH RECURSIVE org AS (
  -- Anchor: roots (e.g., CEO(s))
  SELECT e.employee_id,
         e.first_name,
         e.manager_id,
         0 AS depth,
         CAST(e.employee_id AS VARCHAR(200)) AS path
  FROM employees e
  WHERE e.manager_id IS NULL

  UNION ALL

  -- Recursive: join next level
  SELECT c.employee_id,
         c.first_name,
         c.manager_id,
         p.depth + 1 AS depth,
         CONCAT(p.path, '>', c.employee_id) AS path
  FROM employees c
  JOIN org p ON c.manager_id = p.employee_id
)
SELECT *
FROM org
ORDER BY path;
```

**Aggregating With Hierarchy:**
```sql
WITH RECURSIVE dept_tree AS (
  SELECT d.department_id, d.parent_department_id, d.name, 0 AS depth
  FROM departments d WHERE parent_department_id IS NULL
  UNION ALL
  SELECT c.department_id, c.parent_department_id, c.name, p.depth + 1
  FROM departments c
  JOIN dept_tree p ON c.parent_department_id = p.department_id
)
SELECT depth, COUNT(*) AS nodes_at_level
FROM dept_tree
GROUP BY depth
ORDER BY depth;
```

## Interview-Focused Notes
1. "When do you use a recursive CTE?" – To traverse hierarchical data (org charts, category trees, bill of materials, graph-like expansions).
2. "What is the anchor vs recursive member?" – Anchor seeds initial rows; recursive repeatedly expands.
3. "How to prevent infinite recursion?" – Ensure proper termination (e.g., no cycles) + optional cycle detection.
4. "Performance concerns?" – Deep recursion can be slow; index parent/child keys; limit depth where possible.
5. "Alternatives?" – Nested sets, materialized path, closure tables for faster reads at cost of write complexity.

## Quick Recall ✅
- Uses WITH RECURSIVE.
- UNION ALL between anchor & recursive parts.
- Depth column useful for formatting.
- Path building aids cycle detection & ordering.
- Requires adjacency list design.

## Interview Traps & Confusions ⚠️
- Forgetting `UNION ALL` (UNION adds unnecessary dedupe cost).
- Cycle causing infinite recursion (database stops with error after max iterations).
- Using recursion for shallow hierarchies where simple self join suffices.

## Bonus
### Cycle Detection (PostgreSQL Example)
```sql
WITH RECURSIVE org AS (
  SELECT employee_id, manager_id, ARRAY[employee_id] AS path
  FROM employees WHERE manager_id IS NULL
  UNION ALL
  SELECT e.employee_id, e.manager_id, path || e.employee_id
  FROM employees e
  JOIN org o ON e.manager_id = o.employee_id
  WHERE NOT e.employee_id = ANY(path)  -- Avoid cycles
)
SELECT * FROM org;
```

### Limiting Depth
```sql
WITH RECURSIVE cat AS (
  SELECT category_id, parent_id, 0 AS depth FROM categories WHERE parent_id IS NULL
  UNION ALL
  SELECT c.category_id, c.parent_id, p.depth + 1
  FROM categories c
  JOIN cat p ON c.parent_id = p.category_id
  WHERE p.depth < 5  -- Stop at depth 5
)
SELECT * FROM cat;
```

### Materialized Path Alternative
Store path string like `1/4/9` and query with `LIKE '1/4/%'` (add indexing strategies) instead of recursion for some workloads.

### Closure Table Alternative
Maintain separate table storing all ancestor-descendant pairs with depth.
