# FULL JOIN (FULL OUTER JOIN)

## Overview
A `FULL OUTER JOIN` returns all rows from both tables, matching rows where the join predicate succeeds and padding with `NULL` where no match exists on either side.

## Core Concepts
- Combines effects of LEFT and RIGHT joins.
- Unmatched rows from left: right columns NULL.
- Unmatched rows from right: left columns NULL.
- Supported in PostgreSQL, SQL Server, Oracle. Not supported natively in MySQL (must emulate).

**Syntax:**
```sql
SELECT a.id AS a_id, b.id AS b_id, a.value, b.value
FROM table_a a
FULL OUTER JOIN table_b b ON a.id = b.id;
```

**MySQL Emulation (Using UNION):**
```sql
SELECT a.id AS a_id, b.id AS b_id, a.value, b.value
FROM table_a a
LEFT JOIN table_b b ON a.id = b.id
UNION
SELECT a.id, b.id, a.value, b.value
FROM table_b b
LEFT JOIN table_a a ON a.id = b.id;
```
(Use `UNION ALL` + anti-join filter if duplicates possible and performance required.)

## Interview-Focused Notes
1. "When is FULL OUTER JOIN used?" – Reconciliation, data warehouse comparison, change audits, dimension alignment.
2. "Why uncommon in OLTP?" – Usually you focus on a driving entity; FULL introduces complexity and larger result sets.
3. "How to filter only unmatched?" – Use `WHERE a.id IS NULL OR b.id IS NULL` after the join.
4. "How to simulate in MySQL?" – Two opposing LEFT JOINs with UNION / UNION ALL and exclusions.
5. "Performance considerations?" – Potentially large intermediate sets; ensure join predicate is selective & indexed.

## Quick Recall ✅
- Returns all rows from both tables.
- NULL padding on both sides.
- Ideal for diff reporting.
- Not supported in MySQL natively.
- Larger output cardinality risk.

## Interview Traps & Confusions ⚠️
- Forgetting duplicates with UNION simulation (`UNION` removes duplicates, `UNION ALL` keeps; pick deliberately).
- Mislabeling unmatched reasons: Need to distinguish left-only vs right-only via NULL checks.
- Overusing FULL when left or right suffices (adds unnecessary complexity).

## Bonus
### Classifying Match Status
```sql
SELECT COALESCE(a.id, b.id) AS id,
       CASE
         WHEN a.id IS NOT NULL AND b.id IS NOT NULL THEN 'MATCH'
         WHEN a.id IS NOT NULL AND b.id IS NULL THEN 'LEFT_ONLY'
         WHEN a.id IS NULL AND b.id IS NOT NULL THEN 'RIGHT_ONLY'
       END AS match_type
FROM table_a a
FULL OUTER JOIN table_b b ON a.id = b.id;
```

### Anti-Join Extraction
Left-only:
```sql
SELECT a.*
FROM table_a a
LEFT JOIN table_b b ON a.id = b.id
WHERE b.id IS NULL;
```
Right-only similar with roles reversed.

### Use Case Examples
- Data migration validation.
- Slowly changing dimension merge preview.
- Identifying orphaned keys across subsystems.
