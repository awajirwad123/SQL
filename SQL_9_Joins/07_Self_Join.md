# SELF JOIN

## Overview
A self join is a regular join where a table is joined to itself. It's used to model hierarchical, sequential, or relational comparisons within the same entity set.

## Core Concepts
- Requires table aliasing to distinguish logical roles.
- Common patterns: parent-child trees, adjacency lists, peer comparisons (e.g., employees vs managers, product version diffs).
- Logical structure identical to joining two different tables.

**Hierarchy Example (Employee → Manager):**
```sql
SELECT e.employee_id, e.first_name AS employee_name,
       m.employee_id AS manager_id, m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

**Sequential Comparison Example (Find Prior Row):**
```sql
SELECT curr.order_id, curr.order_date,
       prev.order_id AS previous_order_id,
       prev.order_date AS previous_order_date
FROM orders curr
LEFT JOIN orders prev
  ON prev.customer_id = curr.customer_id
 AND prev.order_date < curr.order_date
ORDER BY curr.customer_id, curr.order_date;
```
(Add window functions for better performance in RDBMS that support them.)

## Interview-Focused Notes
1. "What is a self join?" – Joining a table to itself to compare rows or traverse relationships.
2. "Why alias required?" – To disambiguate the two logical roles of the same table.
3. "Hierarchy modeling?" – Adjacency list: row stores pointer (foreign key) to parent row; self join retrieves parent.
4. "Alternatives?" – Recursive CTEs for multi-level traversal.
5. "Performance considerations?" – Index the linking column (e.g., `manager_id`).

## Quick Recall ✅
- Same table, different roles.
- Use aliases (e.g., `e`, `m`).
- Good for hierarchies & comparisons.
- Often combined with OUTER JOIN to retain root nodes.
- Might be replaced with window functions.

## Interview Traps & Confusions ⚠️
- Forgetting LEFT JOIN to retain root (no manager) rows.
- Infinite recursion confusion (self join alone doesn’t recurse; recursion needs CTE).
- Using multiple self joins when a window function would be clearer.

## Bonus
### Detect Same-Manager Peers
```sql
SELECT a.employee_id AS emp_a, b.employee_id AS emp_b, a.manager_id
FROM employees a
JOIN employees b
  ON a.manager_id = b.manager_id
 AND a.employee_id < b.employee_id;
```

### Compare Salary to Manager
```sql
SELECT e.employee_id, e.salary,
       m.salary AS manager_salary,
       e.salary - m.salary AS delta
FROM employees e
LEFT JOIN employees m ON m.employee_id = e.manager_id;
```

### Root Identification
```sql
SELECT employee_id
FROM employees
WHERE manager_id IS NULL;  -- Roots of hierarchy
```
