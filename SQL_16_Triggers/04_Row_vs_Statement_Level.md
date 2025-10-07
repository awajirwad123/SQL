# Row-Level vs Statement-Level

## Overview
Granularity affects how often a trigger fires and what data it can access. Row-level triggers execute per affected row, enabling per-row mutation and validation. Statement-level triggers execute once per SQL statement regardless of rows affected, better for aggregated logic.

## Core Concepts
- **Row-Level**:
  - Access NEW/OLD row images.
  - Useful for column derivation, row validation, row-specific auditing.
  - Potentially high overhead for bulk operations.
- **Statement-Level**:
  - Access aggregate pseudo tables (SQL Server inserted/deleted, Oracle via cursors). No direct NEW/OLD single-row variables.
  - Efficient for summarizing effects or batch auditing.
- **Hybrid Needs**:
  - Use statement-level to gather set summary; row-level for per-row modifications.
  - Postgres transition tables approximate statement-level sets.
- **Performance Consideration**: N row-level invocations vs 1 statement invocation—choose minimal necessary granularity.

### Row-Level Example (MySQL)
```sql
CREATE TRIGGER trg_users_norm_email
BEFORE INSERT ON users
FOR EACH ROW
SET NEW.email = LOWER(NEW.email);
```

### Statement-Level Example (SQL Server)
```sql
CREATE OR ALTER TRIGGER trgOrdersBatchAudit ON dbo.Orders
AFTER INSERT
AS
INSERT INTO OrdersBatchAudit(batch_rows, created_at)
SELECT COUNT(*) , SYSDATETIME() FROM inserted;
```

### Postgres Transition Table
```sql
CREATE TRIGGER trg_orders_batch
AFTER INSERT ON orders
REFERENCING NEW TABLE AS new_rows
FOR EACH STATEMENT
EXECUTE FUNCTION process_batch();
```

## Interview-Focused Notes
1. Difference row vs statement trigger? Per-row vs per statement invocation.
2. When choose statement-level? Batch metrics or aggregate side-effects.
3. When row-level essential? Data normalization or per-row constraints adjustment.
4. Performance impact? Row-level multiplies procedure overhead by row count.
5. Transition tables purpose? Provide set context to statement-level logic.

## Quick Recall ✅
- Row-level: precise; expensive at scale.
- Statement-level: efficient aggregation.
- Transition tables: set visibility in Postgres.
- Choose least granular meeting requirement.
- Combine cautiously.

## Interview Traps & Confusions ⚠️
- Implementing expensive logic row-level for large bulk loads.
- Assuming statement-level can mutate individual rows (cannot directly).
- Forgetting difference in pseudo table semantics across engines.
- Overusing both levels for same event causing duplication.
- Ignoring batching alternatives (ETL staging) for mass transformations.

## Bonus
### Bulk Load Strategy
Disable row triggers temporarily; run batch; re-enable & reconcile if safe (auditing exceptions apply).

### Hybrid Audit
Row-level for critical fields + statement-level summary metrics.

### Transition Table Filtering
Process only subset of inserted rows matching criteria (e.g., high-value orders).

### Performance Profiling
Log elapsed time inside triggers for large DML to detect hotspots.
