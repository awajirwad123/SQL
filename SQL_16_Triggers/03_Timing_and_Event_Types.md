# Timing & Event Types

## Overview
Trigger timing (BEFORE, AFTER, INSTEAD OF) and event type (INSERT, UPDATE, DELETE, sometimes TRUNCATE or DDL) determine when and why code executes. Proper selection controls data validation, transformation, and side-effect sequencing.

## Core Concepts
- **BEFORE Triggers**: Modify or validate row before persistence. Reject by raising error. Common for derived column values, normalization, default fill (when defaults insufficient).
- **AFTER Triggers**: React post-commit of row image (logically persisted). Ideal for auditing, summary maintenance, notifications.
- **INSTEAD OF Triggers**: Replace operation (commonly on views) translating virtual operation to base table actions.
- **Multi-Event Handling**: Some DBs allow combining events (e.g., `AFTER INSERT OR UPDATE`). Keep logic branch-separated.
- **Column-Specific UPDATE**: Narrow firing scope using `UPDATE OF column_list` (Postgres, Oracle) or conditional comparison.
- **DDL Triggers** (SQL Server / Oracle): Fire on CREATE/ALTER/DROP for auditing schema changes.
- **TRUNCATE Trigger Support**: Limited; emulate via DDL trigger or auditing pattern if not supported.

### Example: BEFORE Validation (Postgres)
```sql
IF NEW.quantity <= 0 THEN
  RAISE EXCEPTION 'Quantity must be positive';
END IF;
```

### AFTER Summary Update
```sql
INSERT INTO product_sales(product_id, daily_total, sales_date)
VALUES (NEW.product_id, NEW.amount, CURRENT_DATE)
ON CONFLICT (product_id, sales_date)
DO UPDATE SET daily_total = product_sales.daily_total + EXCLUDED.daily_total;
```

### INSTEAD OF View Trigger (SQL Server)
```sql
CREATE VIEW vActiveOrders AS
SELECT * FROM Orders WHERE status='ACTIVE';
GO
CREATE TRIGGER trg_vActiveOrders_Upd ON vActiveOrders
INSTEAD OF DELETE
AS
UPDATE Orders SET status='CANCELLED'
FROM Orders o JOIN deleted d ON o.order_id = d.order_id;
```

## Interview-Focused Notes
1. When use BEFORE vs AFTER? BEFORE for validation/adjustment; AFTER for logging dependent on final values.
2. INSTEAD OF trigger use case? Enable DML on complex or filtered views.
3. Column-limited UPDATE importance? Reduces unnecessary trigger invocations.
4. Why not derive audit in BEFORE? Might log values that ultimately fail or are changed again in chain.
5. DDL trigger typical purpose? Schema change auditing / enforcing naming conventions.

## Quick Recall ✅
- BEFORE = mutate/validate.
- AFTER = react/log.
- INSTEAD OF = translate view ops.
- UPDATE OF narrows firing.
- Choose minimal necessary timing.

## Interview Traps & Confusions ⚠️
- Using AFTER when needing to alter NEW row (too late).
- Overloading a single trigger with unrelated event branches.
- Failing to anticipate cascading triggers (update inside update triggers nested effects).
- Assuming TRUNCATE fires row triggers (it doesn’t in many engines).
- Logging in BEFORE leading to orphan audit on subsequent failure.

## Bonus
### Dual Trigger Pattern
Use separate BEFORE (validation) & AFTER (audit) for clarity and reduced intermixing of concerns.

### Performance Guard
Short-circuit early: check if key columns changed before heavy recalculation.

### Controlled INSTEAD OF Write Path
Centralize complex view updates vs scattering logic in application.

### DDL Policy Enforcement
DDL trigger rejecting objects lacking naming prefix standard.
