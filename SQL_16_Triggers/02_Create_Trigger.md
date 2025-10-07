# Create Trigger

## Overview
Creating a trigger wires an event (e.g., INSERT) on a table or view to procedural logic. Implementation differs by RDBMS but shares structural elements: event specification, timing, granularity, and body/function.

## Core Concepts
- **PostgreSQL Syntax** (function-based):
```sql
CREATE OR REPLACE FUNCTION audit_orders()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO orders_audit(order_id, old_status, new_status, changed_at)
  VALUES (NEW.id, OLD.status, NEW.status, now());
  RETURN NEW;
END;$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_orders_audit
AFTER UPDATE OF status ON orders
FOR EACH ROW
WHEN (OLD.status IS DISTINCT FROM NEW.status)
EXECUTE FUNCTION audit_orders();
```
- **MySQL Syntax** (body inline):
```sql
CREATE TRIGGER trg_orders_set_created
BEFORE INSERT ON orders
FOR EACH ROW
SET NEW.created_at = IFNULL(NEW.created_at, NOW());
```
- **SQL Server Syntax** (statement-level, inserted/deleted tables):
```sql
CREATE OR ALTER TRIGGER trgOrdersAudit ON dbo.Orders
AFTER UPDATE
AS
BEGIN
  SET NOCOUNT ON;
  INSERT INTO dbo.OrdersAudit(order_id, old_status, new_status, changed_at)
  SELECT d.order_id, d.status, i.status, SYSDATETIME()
  FROM deleted d JOIN inserted i ON d.order_id = i.order_id
  WHERE d.status <> i.status;
END;
```
- **Oracle**:
```sql
CREATE OR REPLACE TRIGGER trg_orders_audit
AFTER UPDATE OF status ON orders
FOR EACH ROW
WHEN (OLD.status != NEW.status)
BEGIN
  INSERT INTO orders_audit(order_id, old_status, new_status, changed_at)
  VALUES (:OLD.id, :OLD.status, :NEW.status, SYSTIMESTAMP);
END;
```

### Key Elements
- Event list (INSERT, UPDATE [OF columns], DELETE)
- Timing (BEFORE, AFTER, INSTEAD OF)
- Granularity (FOR EACH ROW vs statement once)
- Conditional firing (WHEN clause or IF logic)
- Body or function reference

## Interview-Focused Notes
1. Why specify UPDATE OF column list? Restrict firing to relevant changes—reduces overhead.
2. Difference function-based vs inline body? Reuse & testability (function) vs simplicity (inline).
3. WHEN clause purpose? Short-circuit trigger logic when condition not met.
4. How to access old/new rows? Engine pseudo tables/records (`NEW`, `OLD`, `inserted`, `deleted`).
5. Why keep trigger minimal? Avoid latency amplification & contention.

## Quick Recall ✅
- Use column-limited UPDATE triggers.
- Prefer function for complex logic.
- Filter early (WHEN / IF guard).
- Avoid external network calls.
- Document reason for existence.

## Interview Traps & Confusions ⚠️
- Creating multiple overlapping triggers instead of consolidating.
- Forgetting RETURN NEW in BEFORE trigger (Postgres) causing errors.
- Not handling NULL-safe comparisons (IS DISTINCT FROM / <> plus null logic).
- Side effects relying on non-deterministic ordering of triggers.
- Overuse for default value population (use DEFAULT constraints instead).

## Bonus
### Idempotent Deployment
`CREATE OR REPLACE FUNCTION` + drop & recreate trigger ensures consistent body.

### Testing Strategy
Wrap DML in transaction and ROLLBACK to verify effect without persistent changes.

### Dry Run Flag
Use session variable / GUC (Postgres) to optionally skip heavy logic for bulk loads.

### Partitioned Tables
Attach trigger to parent vs partitions (engine-specific behavior—Postgres needs per-partition in older versions).
