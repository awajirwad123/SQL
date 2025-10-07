# Transaction Control & Savepoints

## Overview
Transaction control statements manage lifecycle: begin, commit, rollback, and partial rollback via savepoints. Savepoints enable recovery from sub-operation failures without aborting the entire transaction.

## Core Concepts
- **BEGIN / START TRANSACTION**: Opens transaction; disables autocommit for its scope.
- **COMMIT**: Persist changes; release locks.
- **ROLLBACK**: Undo all changes since transaction start.
- **SAVEPOINT name**: Marks intermediate rollback point.
- **ROLLBACK TO SAVEPOINT name**: Revert to savepoint while keeping transaction open.
- **RELEASE SAVEPOINT name**: Optional; frees resources.
- **Nested Transactions**: Often simulated via savepoints; true nesting rare.
- **Error Handling Pattern**: Try section per logical unit; rollback to savepoint on failure.

### Example (Postgres)
```sql
BEGIN;
SAVEPOINT step1;
INSERT INTO orders(...);
-- failure? ROLLBACK TO SAVEPOINT step1; continue
SAVEPOINT step2;
INSERT INTO order_items(...);
COMMIT;
```

### SQL Server
Use `BEGIN TRAN`, `COMMIT`, `ROLLBACK TRAN`, with savepoints via `SAVE TRAN savepoint_name`.

## Interview-Focused Notes
1. Why savepoints? Partial error recovery inside larger atomic unit.
2. Performance cost? Additional log markers; use judiciously.
3. Nested transactions exist? Usually emulated; only outer commit persists.
4. When rollback entire transaction? When invariant cannot be restored locally.
5. Releasing savepoint benefit? Minor resource clarity; optional in many systems.

## Quick Recall ✅
- Savepoints = internal checkpoint.
- Only outermost commit final.
- Use for multi-stage logical units.
- Rollback to savepoint keeps later work intact.
- Excess savepoints add overhead.

## Interview Traps & Confusions ⚠️
- Assuming inner commit final (still pending outer commit).
- Overusing savepoints every statement (noise & overhead).
- Not checking error state before continuing.
- Misnaming / reusing savepoints causing confusion.
- Forgetting to rollback after unrecoverable failure.

## Bonus
### Pattern: Try / Compensate
For each stage: savepoint, attempt, on exception rollback to savepoint & log; decide continue or abort.

### Bulk Load Batch
Commit every N rows via application-level commit, not savepoints (smaller transactions reduce contention).

### Automatic Retry Blocks
Combine savepoints with retry loops for transient constraint conflicts.

### Logging with Savepoints
Record stage transitions for audit of partial failures.
