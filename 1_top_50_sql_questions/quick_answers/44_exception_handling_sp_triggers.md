# 44. Exception Handling in Stored Procedures / Triggers

Q: How handle exceptions?
A: Use engine-specific blocks (e.g., TRY...CATCH in SQL Server, EXCEPTION in PL/pgSQL, DECLARE HANDLER in MySQL) to capture errors, log, and rollback.

Examples:
-- PostgreSQL
```
BEGIN
  -- logic
EXCEPTION WHEN others THEN
  RAISE NOTICE 'Error encountered';
  ROLLBACK; -- if in explicit transaction
END;
```
-- SQL Server
```
BEGIN TRY
  -- logic
END TRY
BEGIN CATCH
  ROLLBACK TRAN;
  INSERT INTO error_log(error_number, message) VALUES (ERROR_NUMBER(), ERROR_MESSAGE());
END CATCH;
```

Best Practices:
- Keep triggers minimal; avoid complex logic.
- Log context (keys, timestamps) for diagnosis.
- Fail fast on integrity errors; do not swallow silently.

Pitfall: Swallowing exceptions causes silent data corruption or mismatched transactional state.