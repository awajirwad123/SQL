# 35. Update Subset & Atomicity

Q: Update only specific columns?
A: Provide only changed columns in UPDATE SET; untouched columns remain.
```sql
UPDATE customers
SET last_login = CURRENT_TIMESTAMP,
    login_count = login_count + 1
WHERE id = 42;
```

Q: Atomicity meaning?
A: Entire statement commits or rolls back as a single indivisible unit—partial column changes won't persist alone.

Q: Multi-table consistency?
A: Wrap related statements in a transaction (BEGIN ... COMMIT) to ensure all-or-nothing.

Q: Conditional update?
A: Use CASE inside SET for selective modifications.
```sql
SET status = CASE WHEN balance < 0 THEN 'DELINQUENT' ELSE status END
```

Q: Pitfall?
A: Forgetting WHERE clause—updates every row (protect with safe mode / dry run).
