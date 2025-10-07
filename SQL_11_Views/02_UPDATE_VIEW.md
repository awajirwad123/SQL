# UPDATE VIEW (Altering & Updating Through Views)

## Overview
There are two distinct concepts: (1) **Modifying a view's definition** (`ALTER VIEW` or `CREATE OR REPLACE VIEW`) and (2) **Issuing DML (UPDATE/INSERT/DELETE) through an *updatable view*** to affect underlying base tables. This file covers both and the rules governing updatability.

## Core Concepts
### Altering Definition
- **PostgreSQL / Oracle / MySQL**: `CREATE OR REPLACE VIEW v AS ...`
- **SQL Server**: `ALTER VIEW v AS ...`
- **Renaming Columns**: Must re-specify the full definition.
- **SCHEMABINDING (SQL Server)**: Use in `ALTER VIEW` to enforce dependency integrity.

### Updatable Views
- A view is *potentially updatable* if the system can map modified columns directly to a single underlying base table row.
- Disqualifiers typically include: `DISTINCT`, aggregate functions, `GROUP BY`, `HAVING`, `UNION/UNION ALL`, window functions, set operations, subquery in SELECT list producing derived columns, joins (some systems allow limited join updates—MySQL can update one side).
- **WITH CHECK OPTION** ensures any inserted/updated row remains visible.

**Examples:**
```sql
-- Alter/Replace definition
CREATE OR REPLACE VIEW active_products AS
SELECT product_id, name, price
FROM products
WHERE discontinued = FALSE;

-- Update through view (updatable)
UPDATE active_products
SET price = price * 1.05
WHERE product_id = 1001;

-- Will fail if view not updatable or violates predicate when WITH CHECK OPTION used.
```

### Detecting Updatability
| Dialect | Helper |
|---------|--------|
| PostgreSQL | `information_schema.views.is_updatable` |
| MySQL | `information_schema.VIEWS.IS_UPDATABLE` |
| SQL Server | Limited; rely on rules / errors |

## Interview-Focused Notes
1. **"How do you change a view safely?"** – Use `CREATE OR REPLACE` (or `ALTER VIEW` in SQL Server) inside a transaction if dependent objects rely on it.
2. **"What makes a view updatable?"** – Single base table + no disqualifying clauses + all mandatory columns exposed.
3. **"What does WITH CHECK OPTION do?"** – Prevents DML that would move a row outside the view's predicate.
4. **"Can you update through a join view?"** – Often not; MySQL may allow if only one table columns are modified and others key-preserved.
5. **"Difference between altering and refreshing?"** – Refresh applies to materialized views; altering changes definition.

## Quick Recall ✅
- Redefine: `CREATE OR REPLACE VIEW` or `ALTER VIEW`.
- Updatable: Simple projection of single base table.
- Use `WITH CHECK OPTION` to enforce predicate integrity.
- Materialized view change requires DROP + recreate (some systems) or `REFRESH` for data.
- DML blocked if view not inherently updatable.

## Interview Traps & Confusions ⚠️
- Assuming all views support DML.
- Forgetting to include NOT NULL columns → inserts fail.
- Adding aggregation then attempting UPDATE.
- Relying on view for performance (no index layer unless special indexed/materialized view).
- Confusing `ALTER VIEW` (definition) with `UPDATE view_name` (data pass-through).

## Bonus
### Check Updatability (Postgres)
```sql
SELECT table_schema, table_name, is_updatable
FROM information_schema.views
WHERE table_name = 'active_products';
```

### WITH LOCAL vs CASCADED CHECK OPTION (Some Dialects)
- `LOCAL`: Apply only to current view
- `CASCADED`: Apply to entire chain of underlying views.

### INSTEAD OF Triggers (SQL Server / Oracle)
Enable complex updatable semantics by intercepting DML on a view and routing manually to base tables.

### Incremental Price Adjustment via View
```sql
UPDATE active_products SET price = price + 5 WHERE price < 20;
```

### Defensive Redefinition (Transactional)
```sql
BEGIN;
CREATE OR REPLACE VIEW active_products AS ... ;
COMMIT;
```