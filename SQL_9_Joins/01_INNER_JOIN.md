# INNER JOIN

## Overview
An `INNER JOIN` returns only the rows where the join condition finds a match in **both** tables. It is the most common and default type of join when people simply write `JOIN`.

## Core Concepts
- Returns intersection of two (or more) tables based on the join predicate.
- Rows with no match on either side are excluded.
- Equivalent logical set operation: intersection (on the join key domain), though operationally performed via join algorithms.
- Typical use: Combine related attributes from normalized tables.

**Syntax (ANSI Standard):**
```sql
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
INNER JOIN departments d
    ON e.department_id = d.department_id;
```
**Abbreviation:** `INNER` keyword is optional:
```sql
SELECT ... FROM employees e JOIN departments d ON e.department_id = d.department_id;
```

**Multiple Joins:**
```sql
SELECT o.order_id, c.customer_name, s.shipper_name
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
JOIN shippers s ON o.shipper_id = s.shipper_id;
```

## Interview-Focused Notes
1. "What does an INNER JOIN do?" – Returns rows with matching keys in both tables.
2. "Difference between WHERE filter and ON predicate?" – `ON` defines matching logic between tables; `WHERE` filters the result *after* rows are formed (except in OUTER JOIN semantics where placement matters for preserving rows).
3. "Is `INNER JOIN` commutative?" – Logically yes: A ⋈ B = B ⋈ A (ignoring column order). Physically, optimizer may reorder.
4. "Can INNER JOIN reduce row counts?" – Yes (filters unmatched). Can also increase (if 1:N expands rows).
5. "Join condition best practice?" – Always join on indexed, type-compatible keys; avoid implicit cross joins with filters in WHERE (old style) for clarity.

## Quick Recall ✅
- Only matching rows.
- "JOIN" defaults to INNER.
- Use ON for relationships, WHERE for post-join filters.
- Can amplify rows with 1-to-many.
- Requires compatible data types for best performance.

## Interview Traps & Confusions ⚠️
- Writing Cartesian product accidentally: forgetting ON clause in some dialects (MySQL pre-ANSI style with comma join).
- Applying filter on the wrong side in OUTER vs INNER logic (less an issue here but foundational).
- Assuming order of tables affects result set (only affects column order, not logical rows).

## Bonus
- Performance relies on indexes on join columns (e.g., foreign key columns + primary keys).
- Optimizer may switch to hash, merge, or nested loop join algorithms.
- Use table aliases to avoid name collisions and improve readability.

### Example With Additional Filter
```sql
SELECT e.employee_id, e.first_name, d.department_name
FROM employees e
JOIN departments d ON e.department_id = d.department_id
WHERE d.is_active = TRUE;
```
The department filter happens after matches are made.
