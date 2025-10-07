# 48. Stored Procedures vs Functions

Q: Stored procedure?
A: Executable program (may have multiple statements, side effects, no mandatory return value).

Q: Function?
A: Returns a value/table; intended for use in expressions; should be deterministic & side-effect free (ideally).

Key Differences:
- Invocation: CALL/EXEC vs used in SELECT/WHERE.
- Return: Procedure optional; function must return value.
- Side effects: Procedures can manage transactions, modify state; functions often restricted.
- Composability: Functions embed in queries; procedures cannot (except table-valued functions).

Example (Postgres):
```sql
CREATE FUNCTION tax(amount NUMERIC) RETURNS NUMERIC AS $$
  SELECT amount * 0.07;
$$ LANGUAGE sql IMMUTABLE;
```

Pitfall: Overusing scalar functions in SELECT for large result sets → row-by-row overhead; prefer set-based expressions.
