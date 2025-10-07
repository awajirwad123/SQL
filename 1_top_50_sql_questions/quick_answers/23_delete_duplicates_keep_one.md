# 23. Delete Duplicates Keep One

Q: Pattern?
A: Use ROW_NUMBER partitioned by duplicate-defining columns; delete rows with rn > 1.
```sql
WITH ranked AS (
  SELECT id, email,
         ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) AS rn
  FROM users
)
DELETE FROM users
WHERE id IN (SELECT id FROM ranked WHERE rn > 1);
```

Q: MySQL variant (no CTE pre-8.0)?
A: Self-join on minimal id: `DELETE u1 FROM users u1 JOIN users u2 ON u1.email=u2.email AND u1.id>u2.id;`

Q: Keep newest instead?
A: ORDER BY timestamp DESC in ROW_NUMBER ordering.

Q: Pitfall?
A: Not creating backup / log before destructive dedupe; always verify row counts first.
