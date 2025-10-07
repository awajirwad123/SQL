# 41. Orphaned Rows

Q: What are orphaned rows?
A: Child table rows referencing parent keys that no longer exist (or never did).

Impacts:
- Inaccurate reports/aggregates.
- ETL failures or referential drift.
- Wasted storage / broken navigation in apps.

Detection (with proper FK constraint this is prevented):
```sql
SELECT c.*
FROM child c
LEFT JOIN parent p ON p.id = c.parent_id
WHERE p.id IS NULL; -- potential orphans
```

Preventive Measures:
- Define FOREIGN KEY with ON DELETE CASCADE/RESTRICT as appropriate.
- Batch integrity audits: count mismatch, orphan scan scheduled.
- Use deferred constraints only when necessary; revalidate after bulk load.

Remediation:
- Reattach (fix parent_id) if correct parent known.
- Delete if obsolete.
- Backfill parent using archived source.

Pitfall: Soft-deleting parent without cascading soft-delete flag to children.