# Security & Permissions

## Overview
Stored procedures can act as controlled gateways to data, enforcing principle of least privilege. Proper permission design limits exposure of underlying tables while enabling necessary operations.

## Core Concepts
- **Execution Rights**: Grant EXECUTE on procedure, but not direct table access.
- **Ownership Chaining (SQL Server)**: If same owner, execution can access underlying objects without explicit grants.
- **Definer vs Invoker (MySQL / Oracle)**: DEFINER executes with stored definer's privileges; INVOKER uses caller's rights.
- **EXECUTE AS (SQL Server)**: Elevates context to specified user or owner for procedure duration.
- **SQL Injection Mitigation**: Avoid string concatenation; use parameters; whitelist object names if dynamic SQL required.
- **Auditing**: Centralize sensitive operations via procedure for logging & monitoring.
- **Row-Level Security (RLS)**: Procedure may bypass or rely on policy depending on execution context.

### Grant Example (Postgres)
```sql
REVOKE ALL ON TABLE orders FROM public;
GRANT EXECUTE ON FUNCTION create_order(int, numeric) TO app_role;
```

### SQL Server Execute As Owner
```sql
CREATE OR ALTER PROCEDURE dbo.SecureOp
WITH EXECUTE AS OWNER
AS
BEGIN
  -- privileged operation
END;
```

## Interview-Focused Notes
1. Why use procedures for security? Encapsulate logic; callers need only EXECUTE permission.
2. Definer vs invoker trade-off? Definer centralizes privilege; risk if procedure vulnerable to injection.
3. Mitigate injection in dynamic SQL? Parameterization + QUOTENAME whitelisting identifiers.
4. Ownership chaining benefit? Simplifies permissions; risk if misapplied with cross-schema.
5. RLS interaction? Execution context influences row visibility; test under target role.

## Quick Recall ✅
- Grant EXECUTE; restrict table access.
- Use secure context elevation carefully.
- Parameterize dynamic SQL.
- Log sensitive procedure usage.
- Review privileges periodically.

## Interview Traps & Confusions ⚠️
- Assuming EXECUTE automatically grants table rights (it doesn't).
- Using definer rights with unchecked dynamic SQL.
- Not revoking default PUBLIC privileges.
- Elevating to sysadmin/sa unnecessarily.
- Ignoring RLS policy bypass via elevated context.

## Bonus
### Whitelisting Pattern
```sql
IF @SortColumn NOT IN ('created_at','order_id') THROW 50000,'Invalid column',1;
DECLARE @sql NVARCHAR(MAX) = N'SELECT * FROM orders ORDER BY ' + QUOTENAME(@SortColumn);
EXEC sp_executesql @sql;
```

### Audit Table
Record caller, timestamp, parameters (excluding sensitive) for regulatory ops.

### Least Privilege Baseline
Role with only EXECUTE on curated procedures; no direct DML on tables.

### Secret Handling
Fetch secrets outside DB; avoid embedding credentials within procedure code.
