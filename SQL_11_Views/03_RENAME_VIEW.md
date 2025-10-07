# RENAME VIEW

## Overview
Renaming a view changes its identifier without altering its underlying definition. Syntax differs significantly by RDBMS. Proper renaming maintains dependency integrity (or fails if bound). Consider impact on dependent code, permissions, and tools.

## Core Concepts
### Dialect-Specific Syntax
| RDBMS | Syntax |
|-------|--------|
| PostgreSQL | `ALTER VIEW old_name RENAME TO new_name;` |
| MySQL | `RENAME TABLE old_view TO new_view;` or `RENAME TABLE schema.old_view TO schema.new_view;` |
| SQL Server | `EXEC sp_rename 'schema.old_name', 'new_name';` (object-level) |
| Oracle | `RENAME old_name TO new_name;` (object in current schema) |
| SQLite | `ALTER TABLE old_name RENAME TO new_name;` (views treated similarly) |

### Considerations
- **Dependencies**: Stored procedures, other views, application SQL strings may reference old name.
- **Permissions**: Grants typically persist (name changes tracked internally) except in some legacy tools.
- **Schema Qualification**: Always qualify object names to reduce collisions (`schema.view`).
- **Search Path Effects**: In Postgres, renaming might expose a different object earlier in path if naming overlaps.

**Examples:**
```sql
-- PostgreSQL
ALTER VIEW sales_summary RENAME TO sales_summary_v2;

-- MySQL
RENAME TABLE sales_summary TO sales_summary_v2;

-- SQL Server
EXEC sp_rename 'reporting.sales_summary', 'sales_summary_v2';
```

## Interview-Focused Notes
1. **"How do you rename a view in Postgres vs SQL Server?"** – Postgres: `ALTER VIEW ... RENAME TO`; SQL Server: `sp_rename`.
2. **"What breaks after renaming?"** – Hard-coded references in application code or dynamic SQL; dependent objects typically auto-update metadata.
3. **"Safer alternative to renaming?"** – Create new view, keep old as wrapper / deprecated until callers migrate; then drop.
4. **"How to find dependents before rename?"** – Query system catalogs (e.g., `pg_depend`, `sys.sql_expression_dependencies`).
5. **"Why might you version views?"** – Non-breaking evolution; provide v2 while v1 remains stable for consumers.

## Quick Recall ✅
- Syntax not portable.
- Use dependency check before rename.
- Keep compatibility wrapper if many clients.
- Permissions usually persist.
- Consider semantic version naming pattern.

## Interview Traps & Confusions ⚠️
- Assuming `ALTER VIEW ... RENAME` works everywhere.
- Overwriting a table name accidentally in MySQL using `RENAME TABLE`.
- Not invalidating cached ORM metadata.
- Renaming a materialized view (some systems restrict or require drop/create).

## Bonus
### Postgres Dependency Inspection
```sql
SELECT refobjid::regclass AS dependent
FROM pg_depend
WHERE objid = 'sales_summary'::regclass;
```

### Compatibility Wrapper Pattern
```sql
CREATE OR REPLACE VIEW sales_summary AS
SELECT * FROM sales_summary_v2;  -- Temporary bridge
```

### Naming Convention
`viewname_v1`, `viewname_v2` → deprecate older after migration window.

### Rollback Strategy
Keep transaction open:
```sql
BEGIN;
ALTER VIEW sales_summary RENAME TO sales_summary_backup;
ALTER VIEW sales_summary_v2 RENAME TO sales_summary;  -- Swap
-- If issue:
ROLLBACK;  -- Names revert
COMMIT;    -- If success
```