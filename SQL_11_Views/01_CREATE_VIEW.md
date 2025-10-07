# CREATE VIEW

## Overview
`CREATE VIEW` defines a named, stored query (a virtual table) whose result set is computed on demand. Views encapsulate query complexity, enforce abstraction boundaries, simplify permissions, and sometimes enable limited logical data independence. A view stores no data (except in *materialized views* or indexed views) – it is resolved at execution time.

## Core Concepts
- **Virtual Table**: A view behaves like a table in `SELECT` queries, but its rows are derived from its defining query each time (unless materialized).
- **Syntax (ANSI Baseline)**:
  ```sql
  CREATE VIEW active_customers AS
  SELECT customer_id, name, status
  FROM customers
  WHERE status = 'ACTIVE';
  ```
- **`CREATE OR REPLACE VIEW`**: (PostgreSQL, Oracle, MySQL 5.7+) updates definition atomically without dropping privileges. In SQL Server use `ALTER VIEW`.
- **Column List**: You can override underlying column names:
  ```sql
  CREATE VIEW recent_orders(order_id, order_ts, amount) AS
  SELECT id, created_at, total
  FROM orders
  WHERE created_at >= CURRENT_DATE - INTERVAL '7 days';
  ```
- **Security / Permission Layer**: Grant `SELECT` on the view while restricting underlying tables.
- **WITH CHECK OPTION**: Prevents DML through the view that would create rows not visible via the view predicate:
  ```sql
  CREATE VIEW us_customers AS
  SELECT * FROM customers WHERE country = 'US'
  WITH CHECK OPTION;
  ```
- **SCHEMABINDING / WITH SCHEMABINDING (SQL Server)**: Locks underlying schema (needed for indexed views) and prevents changes to dependent columns without dropping the view.
- **Materialized vs Standard**: Materialized (Postgres `CREATE MATERIALIZED VIEW`, Oracle, some warehouses) physically stores result; standard recalculates.

## Interview-Focused Notes
1. **"Why use a view?"** – Abstraction, security (column/row filtering), centralizing business logic, reuse, compatibility for evolving physical schema.
2. **"Performance: does a normal view improve speed?"** – No by itself; it's syntactic sugar unless materialized / indexed.
3. **"When to use materialized view?"** – Expensive aggregations reused frequently with tolerable staleness.
4. **"Can you index a view?"** – Only special cases (SQL Server Indexed View / Materialized view refresh). Regular views inherit indexes of underlying tables.
5. **"Difference between view and table?"** – Table stores data; view stores query definition.

## Quick Recall ✅
- Virtual; no stored rows (unless materialized).
- `CREATE OR REPLACE` for safe redefinition (dialect dependent).
- Use column lists to rename / reorder.
- Add `WITH CHECK OPTION` to enforce predicate integrity.
- Permissions can be granted on view only.

## Interview Traps & Confusions ⚠️
- Assuming performance gain from a plain view.
- Forgetting to schema-qualify underlying tables causing search_path issues (Postgres).
- Changing underlying table (dropping needed column) breaks dependent view.
- Over-nesting views → complex execution plans (“view stacking” anti-pattern).
- View vs CTE confusion: CTE is per-statement; view is persistent object.

## Bonus
### Updatable View Requirements (Typical)
- Single base table
- No aggregation / DISTINCT / GROUP BY / HAVING / UNION / LIMIT
- All NOT NULL columns present
- No window functions

### Grant Through View
```sql
GRANT SELECT ON active_customers TO analyst_role;
```

### View Ownership Security Barrier (Postgres)
Use `SECURITY DEFINER` (function-based) or RLS policies; plain views evaluate with invoker rights.

### Materialized View Refresh Example (Postgres)
```sql
CREATE MATERIALIZED VIEW mv_sales_daily AS
SELECT date_trunc('day', created_at) AS day,
       SUM(total) AS revenue
FROM orders
GROUP BY 1;

REFRESH MATERIALIZED VIEW CONCURRENTLY mv_sales_daily; -- With unique index.
```

### Dependency Inspection
```sql
-- Postgres
SELECT * FROM pg_depend WHERE refobjid = 'active_customers'::regclass;
```