# Control Flow & Error Handling

## Overview
Procedural extensions provide control-of-flow constructs (IF, CASE, LOOP, WHILE, FOR) and exception handling (TRY/CATCH, EXCEPTION blocks). Effective error handling maps low-level database errors to meaningful application signals while preserving atomicity.

## Core Concepts
- **Conditionals**: `IF...ELSE`, `CASE` for inline expressions.
- **Loops**: WHILE (SQL Server, MySQL), FOR / LOOP in PL/pgSQL / PL/SQL. Should be minimized; set-based operations preferred.
- **Error Handling**:
  - SQL Server: `BEGIN TRY ... END TRY BEGIN CATCH ... END CATCH` + `ERROR_NUMBER()`, `XACT_STATE()`.
  - PostgreSQL: `BEGIN ... EXCEPTION WHEN unique_violation THEN ... END;`
  - MySQL: `DECLARE CONTINUE / EXIT HANDLER FOR ...`
  - Oracle: `EXCEPTION WHEN ... THEN ...`
- **Transaction Safety**: Rollback on failure; partial work avoidance.
- **Idempotency**: Re-running procedure after failure should not corrupt state (check existence / status columns).
- **Return Codes vs RAISERROR**: Modern style favors throwing exceptions with specific SQLSTATE / error numbers.

### SQL Server TRY/CATCH
```sql
BEGIN TRY
  BEGIN TRAN;
  UPDATE accounts SET balance = balance - @amt WHERE id=@from;
  UPDATE accounts SET balance = balance + @amt WHERE id=@to;
  COMMIT TRAN;
END TRY
BEGIN CATCH
  IF XACT_STATE() <> 0 ROLLBACK TRAN;
  THROW; -- rethrow original
END CATCH;
```

### PostgreSQL EXCEPTION
```sql
BEGIN
  INSERT INTO users(email) VALUES(p_email);
EXCEPTION WHEN unique_violation THEN
  RAISE EXCEPTION 'Email % already exists', p_email USING ERRCODE='unique_violation';
END;
```

## Interview-Focused Notes
1. Why minimize loops? Set-based SQL faster & simpler maintenance.
2. How ensure atomicity? Enclose dependent DML in transaction + rollback on error.
3. Benefit of rethrowing vs swallowing? Maintains observability & debugging context.
4. Why classify error codes? Application-level retry vs user feedback decisions.
5. Idempotent design? Prevent duplicate side effects (e.g., unique keys, status flags).

## Quick Recall ✅
- Use TRY/CATCH or EXCEPTION block for rollback.
- Prefer set operations to loops.
- Translate DB errors contextually.
- Keep critical section short.
- Ensure idempotency for retriable actions.

## Interview Traps & Confusions ⚠️
- Swallowing exceptions then returning success.
- Rolling back outer transaction unintentionally (nested transaction misunderstanding).
- Infinite loop due to improper loop exit condition.
- Using GOTO-like constructs harming readability.
- Overloading generic error codes losing specificity.

## Bonus
### Retry Classification
- Deadlock victim – safe to retry.
- Unique violation – logic error (no retry unless different input).

### Partial Retry Pattern
Wrap only small conflicting DML section in retry loop with backoff.

### Logging Strategy
Central log procedure capturing error number, context parameters, execution time.

### Declarative Constraints First
Let constraint raise error; handle gracefully—simpler than manual pre-check race condition.
