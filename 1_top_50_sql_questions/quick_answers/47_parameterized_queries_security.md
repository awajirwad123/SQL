# 47. Parameterized Queries & SQL Injection Defense

Q: Goal?
A: Separate code from data to prevent malicious input from altering query structure.

Q: Parameterized query concept?
A: Use placeholders bound to values by driver (e.g., $1, ?, :name) instead of string concatenation.

Q: Example (Postgres):
```sql
PREPARE get_user(text) AS
  SELECT * FROM users WHERE email = $1;
EXECUTE get_user('alice@example.com');
```

Q: App-layer example (pseudo):
```python
cur.execute("SELECT * FROM users WHERE id = %s", (user_id,))
```

Q: Additional defenses?
A: Least-privilege accounts, input validation, stored procedures, escaping when dynamic identifiers unavoidable.

Q: Pitfall?
A: Dynamic ORDER BY or table names built from raw input—must whitelist allowed tokens.
