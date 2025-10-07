# DELETE JOIN

## Overview
A DELETE with JOIN removes rows from a target table based on matching (or non-matching) rows in another table. Syntax differs among SQL dialects. It is powerful—and dangerous—so preview logic first.

## Core Concepts
- Target (deleted-from) table must be unambiguous.
- Used for: cascading cleanup, pruning orphan rows, conditional purges.
- Anti-join variant deletes when no related child/parent exists.

**MySQL Syntax (Multi-table DELETE):**
```sql
DELETE p
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE c.is_active = 0;
```
Delete from multiple tables simultaneously:
```sql
DELETE o, oi
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
WHERE o.order_date < '2024-01-01';
```

**PostgreSQL / SQL Server (USING / FROM):**
```sql
DELETE FROM products p
USING categories c
WHERE p.category_id = c.category_id
  AND c.is_active = FALSE;
```

**Anti-Join Orphan Cleanup:**
```sql
DELETE FROM order_items oi
WHERE NOT EXISTS (
  SELECT 1 FROM orders o WHERE o.order_id = oi.order_id
);
```

## Interview-Focused Notes
1. "How do you delete based on another table?" – Use DELETE with USING/FROM (Postgres/SQL Server) or JOIN clause (MySQL) or anti-join with NOT EXISTS.
2. "How to delete orphan rows?" – `DELETE ... WHERE NOT EXISTS (SELECT 1 ...)` pattern.
3. "Why not rely entirely on ON DELETE CASCADE?" – Sometimes business requires archival, soft deletes, or selective pruning.
4. "How to test first?" – Replace DELETE with SELECT of target keys.
5. "Transaction?" – Wrap large deletes in a transaction; batch if table huge.

## Quick Recall ✅
- Dialect-specific syntax.
- Dangerous—preview first.
- Anti-joins common use case.
- Index join columns for speed.
- Consider soft delete alternative.

## Interview Traps & Confusions ⚠️
- Omitting WHERE (catastrophic full-table delete).
- Misidentifying target alias in multi-table delete (MySQL).
- Lock escalation / log growth on large deletions (need batching).

## Bonus
### Preview Pattern
```sql
SELECT p.product_id
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE c.is_active = 0
LIMIT 50;
```

### Batch Delete Loop (Pseudo-SQL)
```sql
-- Repeat until 0 rows affected
DELETE FROM events
WHERE created_at < NOW() - INTERVAL '180 days'
LIMIT 5000;  -- MySQL (Postgres emulate with CTE ordering + LIMIT)
```

### Soft Delete Strategy
```sql
UPDATE users
SET deleted_at = CURRENT_TIMESTAMP
WHERE user_id = 123;  -- Instead of physical delete
```

### Referential Integrity Tip
Prefer foreign keys with ON DELETE CASCADE for tightly bound child rows (e.g., order_items). Use DELETE JOIN only for ad hoc cleanup or conditional selective rules.
