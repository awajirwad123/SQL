# Transactions in Procedures

## Overview
Transactions group operations into atomic units. Procedures often manage transaction boundaries for multi-step business logic. Proper design avoids long-running locks and ensures consistent rollback semantics, especially when nested calls or external transaction scopes exist.

## Core Concepts
- **Autocommit Mode**: Each statement commits unless explicit transaction started.
- **Explicit Transaction Control**: `BEGIN / COMMIT / ROLLBACK` (SQL Server `BEGIN TRAN`), `START TRANSACTION` (MySQL), `BEGIN` (Postgres), `SAVEPOINT` for partial rollback.
- **Nested Transactions**: True nesting not universally supported; often simulated via savepoints (SQL Server counts nesting levels, only outermost COMMIT persists).
- **Error Handling & Transaction State**: Must check viability before commit (e.g., `XACT_STATE()` in SQL Server).
- **Procedure-Initiated vs Caller-Initiated**: Respect existing transaction if already active (avoid autonomous commit unexpectedly).
- **Idempotent Compensation**: For partial external side effects not covered by DB transaction (messaging, external calls).

### Transaction-Aware Pattern (SQL Server)
```sql
CREATE OR ALTER PROCEDURE dbo.Transfer
  @From INT, @To INT, @Amt DECIMAL(12,2)
AS
BEGIN
  SET NOCOUNT ON;
  DECLARE @local BIT = 0;
  IF @@TRANCOUNT = 0 BEGIN TRAN; SET @local = 1;
  BEGIN TRY
    UPDATE accounts SET balance = balance - @Amt WHERE id=@From;
    UPDATE accounts SET balance = balance + @Amt WHERE id=@To;
    IF @local = 1 COMMIT;
  END TRY
  BEGIN CATCH
    IF @local = 1 AND XACT_STATE() <> 0 ROLLBACK;
    THROW;
  END CATCH;
END;
```

### Savepoint (Postgres)
```sql
SAVEPOINT step1;
-- operations
ROLLBACK TO SAVEPOINT step1; -- partial undo
```

## Interview-Focused Notes
1. Why detect existing transaction? Avoid prematurely committing caller-controlled work.
2. How to partially rollback? Savepoints.
3. Long transaction risks? Lock contention, replication lag, version bloat (MVCC).
4. External side-effects? Not protected—need compensating logic.
5. Isolation level overrides? Only when necessary; resetting after change.

## Quick Recall ✅
- Keep transactions short & focused.
- Respect caller transaction if present.
- Use savepoints for granular rollback.
- Check state before commit on error paths.
- Avoid mixing external non-transactional operations mid-transaction.

## Interview Traps & Confusions ⚠️
- Always opening new transaction ignoring existing scope.
- Holding transaction open while waiting for user input / remote calls.
- Assuming nested commits create independent atomic units.
- Not handling partial failure leaving inconsistent state.
- Setting high isolation globally instead of local override.

## Bonus
### Retry Logic Placement
Wrap entire transaction for deadlock victims—ensure idempotent.

### Timeouts
Implement client-side and server lock timeouts to prevent indefinite waits.

### Audit Table on Commit
Insert into audit after successful DML; ensure part of same transaction for integrity.

### Declarative Constraints Precedence
Rely on constraints rather than procedural pre-check queries.
