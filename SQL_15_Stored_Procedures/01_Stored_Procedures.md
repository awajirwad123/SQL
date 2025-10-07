# Stored Procedures

## Overview
A stored procedure is a named, pre-compiled (or cached) executable block of SQL (and procedural extensions) stored in the database catalog. Procedures encapsulate multi-step logic, enforce consistency, reduce network round trips, and centralize business or batch operations. Some platforms distinguish between functions and procedures (returning value vs none); others blur the line.

## Core Concepts
- **Encapsulation**: Bundle DML, conditional flow, error handling.
- **Pre-Compilation / Caching**: Execution plan often cached after first run (SQL Server) reducing parse/compile overhead.
- **Parameters**: IN, OUT, INOUT, table-valued, default values.
- **Transaction Scope**: Can start/commit/rollback (dialect dependent restrictions inside nested contexts).
- **Security Boundary**: DEFINER vs INVOKER rights (MySQL/Oracle); EXECUTE AS (SQL Server) to control privilege elevation.
- **Side Effects**: Can modify data, call other procedures, produce multiple result sets.
- **Determinism & Idempotency**: Important in retries and replication logic.

### Dialect Notes
- PostgreSQL historically uses `FUNCTION` for most logic; `CREATE PROCEDURE` (v11+) adds transactional control with `CALL`.
- SQL Server: `CREATE PROCEDURE` + T-SQL features (TRY/CATCH, table variables, temp tables).
- MySQL: `DELIMITER` management in clients; explicit IN/OUT/INOUT.
- Oracle: PL/SQL `PROCEDURE` and `FUNCTION`, packages group related routines.

### Simple Example (SQL Server)
```sql
CREATE OR ALTER PROCEDURE dbo.IncrementLoginCount
  @UserId INT
AS
BEGIN
  SET NOCOUNT ON;
  UPDATE Users SET login_count = login_count + 1 WHERE user_id = @UserId;
END;
```

## Interview-Focused Notes
1. Why stored procedures? Reduce round trips, centralize logic, consistent security enforcement.
2. Procedure vs function? Functions (typically) return value and are side-effect-limited (depends on engine); procedures can perform broader DML & multiple result sets.
3. Performance benefit? Plan reuse + reduced client/server chatter.
4. Risks? Logic sprawl, harder version control, vendor lock-in, overusing procedural loops.
5. Security advantages? Granular EXECUTE permission; avoid exposing underlying tables.

## Quick Recall ✅
- Encapsulate multi-step DB logic.
- Parameterized & reusable.
- Plan caching reduces overhead.
- EXECUTE permissions manage access.
- Avoid excessive procedural looping; prefer set-based.

## Interview Traps & Confusions ⚠️
- Treating them as a substitute for proper schema design.
- Mixing presentation formatting (string building) inside database layer.
- Ignoring transaction semantics (implicit vs explicit commit behavior).
- Over-reliance causing business logic duplication across tiers.
- Assuming always faster than parameterized ad-hoc queries (not if trivial / rarely used).

## Bonus
### Set-Based Refactor
Replace cursor loops with single `UPDATE ... FROM` or window functions.

### Logging Audit Pattern
Central audit procedure for inserts/updates referencing context (user, timestamp).

### Safe Wrapper Pattern
Outer procedure handles transaction + error mapping; inner focused on pure set logic.

### Migration Strategy
Gradually externalize logic to services where agility needed; keep data integrity rules local.
