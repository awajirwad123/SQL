# 22. Find Duplicate Rows

Q: Identify duplicates on specific columns?
A: GROUP BY columns + HAVING COUNT(*) > 1.
```sql
SELECT email, COUNT(*) AS cnt
FROM users
GROUP BY email
HAVING COUNT(*) > 1;
```

Q: Show all duplicate rows with frequency next to each copy?
A: Use window COUNT over partition.
```sql
SELECT u.*, COUNT(*) OVER (PARTITION BY email) AS freq
FROM users u
WHERE email IN (
  SELECT email FROM users GROUP BY email HAVING COUNT(*) > 1
);
```

Q: Fast existence check only?
A: `SELECT 1 FROM users GROUP BY email HAVING COUNT(*) > 1 LIMIT 1;`

Q: Pitfall?
A: Trailing spaces / case differences—normalize before grouping if semantics require.
