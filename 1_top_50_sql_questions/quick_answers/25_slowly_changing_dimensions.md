# 25. Slowly Changing Dimensions (SCD)

Q: What is SCD?
A: Technique to manage historical attribute changes in dimension tables for analytics.

Q: Type 1?
A: Overwrite old value (no history). Simple, loses audit.

Q: Type 2?
A: New row per change with start/end timestamps (or current flag) to preserve full history.
```sql
-- Columns: surrogate_key, natural_key, attr, valid_from, valid_to, is_current
```

Q: Type 3?
A: Limited history using additional columns (e.g., previous_value). Rare.

Q: Implement Type 2 upsert sketch?
A:
1. Close current row: set valid_to = change_time, is_current = false.
2. Insert new row with new attributes, valid_from = change_time, valid_to = '9999-12-31', is_current = true.

Q: Query current version?
A: `WHERE is_current = true` or `valid_to = '9999-12-31'`.

Q: Point-in-time lookup?
A: `WHERE key=? AND ts BETWEEN valid_from AND valid_to`.

Q: Pitfall?
A: Overlapping validity windows causing ambiguous history—enforce constraint or trigger.
