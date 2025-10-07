# DROP VIEW

## Overview
`DROP VIEW` removes a view definition from the catalog. It does **not** affect the underlying base tables. For materialized or indexed views, dropping also removes stored data or indexes bound to the view. Use cautiously in shared environments.

## Core Concepts
- **Basic Syntax**:
  ```sql
  DROP VIEW view_name;
  ```
- **Safety Modifiers**:
  - `IF EXISTS` (PostgreSQL, MySQL, SQL Server 2016+): prevent error if absent.
  - `CASCADE` (Postgres): automatically drops dependent views.
  - `RESTRICT` (default): refuse drop if dependencies exist.
- **Multiple Views**:
  ```sql
  DROP VIEW IF EXISTS v1, v2, v3; -- Postgres / MySQL
  ```
- **Materialized View**: Must use `DROP MATERIALIZED VIEW` (Postgres / Oracle). SQL Server uses `DROP VIEW` even for indexed views.
- **Permissions**: Requires ownership or adequate DROP privilege.

**Examples:**
```sql
-- Safe drop
DROP VIEW IF EXISTS active_customers;

-- Postgres cascade
DROP VIEW IF EXISTS sales_summary CASCADE;

-- MySQL
DROP VIEW IF EXISTS sales_summary;

-- Oracle (no IF EXISTS pre-21c):
DROP VIEW sales_summary;
```

## Interview-Focused Notes
1. **"Does dropping a view delete data?"** – No; only definition removed (materialized view data removed though).
2. **"When use CASCADE?"** – When intentionally decommissioning a chain of dependent views; always review dependency tree.
3. **"How to avoid breaking consumers?"** – Deprecation: rename or leave stub that raises warning until clients migrate.
4. **"Difference dropping materialized vs normal view?"** – Materialized stores physical snapshot; dropping deletes that snapshot + indexes.
5. **"What about privileges?"** – Need appropriate rights; dependent grants vanish with view.

## Quick Recall ✅
- `DROP VIEW` only removes metadata.
- `IF EXISTS` = safe idempotent scripts.
- `CASCADE` can remove more than expected—review.
- Materialized view uses distinct keyword.
- Indexed view (SQL Server) dropped like normal view.

## Interview Traps & Confusions ⚠️
- Thinking data lost (it isn't unless materialized specific storage).
- Accidentally dropping widely used analytics view without notice.
- Forgetting to clear caching layers referencing old definition.
- Using CASCADE inadvertently removing unrelated dependent objects.

## Bonus
### Dependency Pre-Check (Postgres)
```sql
SELECT dependent.relname AS referencing_view
FROM pg_depend d
JOIN pg_rewrite r ON d.objid = r.oid
JOIN pg_class dependent ON r.ev_class = dependent.oid
WHERE d.refobjid = 'sales_summary'::regclass;
```

### Soft Deprecation Stub
```sql
CREATE OR REPLACE VIEW old_view AS
SELECT NULL::int AS deprecated_id
WHERE raise_notice('old_view deprecated; use new_view'); -- Pseudocode (use logging function)
```

### Transactional Drop (Rollback Safety)
```sql
BEGIN;
DROP VIEW reporting_rollup;
-- Validate other objects
ROLLBACK; -- if issue discovered
```

### Recreate from Source Control
Store canonical view definitions in versioned DDL scripts for repeatability.
