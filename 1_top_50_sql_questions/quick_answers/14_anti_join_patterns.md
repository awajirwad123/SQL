# 14. Rows in One Table Not in Another

Q: Goal?
A: Return rows existing in table A absent in table B.

Q: NOT EXISTS pattern (preferred):
```sql
SELECT a.*
FROM table_a a
WHERE NOT EXISTS (
  SELECT 1 FROM table_b b WHERE b.key = a.key
);
```

Q: LEFT JOIN / IS NULL pattern:
```sql
SELECT a.*
FROM table_a a
LEFT JOIN table_b b ON b.key = a.key
WHERE b.key IS NULL;
```

Q: Why NOT use NOT IN blindly?
A: NULL in subquery list causes zero-row result due to unknown comparisons.

Q: Performance tip?
A: Index table_b.key; semi/anti join becomes efficient set lookup.

Q: Pitfall?
A: Using LEFT JOIN IS NULL but additional filters on b.* in WHERE (turns into inner join accidentally). Use filters in ON clause.
