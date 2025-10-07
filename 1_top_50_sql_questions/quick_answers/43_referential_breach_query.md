# 43. Detect Referential Integrity Breaches

Q: Goal?
A: Identify child rows pointing to non-existent parent keys.

Core Query Pattern:
```sql
SELECT c.*
FROM child c
LEFT JOIN parent p ON p.id = c.parent_id
WHERE p.id IS NULL; -- Orphaned child rows
```

Alternative (NOT EXISTS):
```sql
SELECT c.*
FROM child c
WHERE NOT EXISTS (
  SELECT 1 FROM parent p WHERE p.id = c.parent_id
);
```

Counting breaches:
```sql
SELECT COUNT(*) AS orphan_count
FROM child c
WHERE NOT EXISTS (
  SELECT 1 FROM parent p WHERE p.id = c.parent_id
);
```

Pitfall: False positives if parent_id legitimately nullable—add `AND c.parent_id IS NOT NULL` in that case.