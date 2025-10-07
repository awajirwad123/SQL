# 11. UNION vs UNION ALL

Q: UNION?
A: Combines result sets and removes duplicates (implicit DISTINCT) with sort/dedup cost.

Q: UNION ALL?
A: Appends result sets preserving duplicates (no dedup cost)—faster.

Q: When choose UNION?
A: When semantic requirement: each logical row should appear once regardless of source.

Q: When choose UNION ALL?
A: When duplicates are meaningful or guaranteed absent (e.g., mutually exclusive filters).

Q: Example:
```sql
SELECT id FROM new_customers
UNION ALL
SELECT id FROM legacy_customers; -- keep duplicates if both sources contain same id
```

Q: Pitfall?
A: Using UNION by default—needless performance hit if duplicates impossible.
