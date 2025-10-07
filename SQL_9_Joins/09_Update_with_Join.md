# UPDATE with JOIN

## Overview
An `UPDATE` with JOIN modifies rows in one table based on data matched from another table. Supported syntactically with vendor variations (MySQL, SQL Server) or via correlated subqueries / FROM clause (PostgreSQL, SQL Server).

## Core Concepts
- Target table = the one being updated.
- Join table(s) supply new values or filter logic.
- Must ensure 1:1 match to avoid unintended multiplicative updates (some dialects error, others repeat updates nondeterministically).

**MySQL Syntax:**
```sql
UPDATE products p
JOIN categories c ON p.category_id = c.category_id
SET p.category_name_cache = c.category_name
WHERE c.is_active = 1;
```

**PostgreSQL / SQL Server (FROM style):**
```sql
UPDATE products p
SET category_name_cache = c.category_name
FROM categories c
WHERE p.category_id = c.category_id
  AND c.is_active = TRUE;
```

**Correlated Subquery Alternative:**
```sql
UPDATE products p
SET category_name_cache = (
    SELECT c.category_name
    FROM categories c
    WHERE c.category_id = p.category_id
)
WHERE EXISTS (
    SELECT 1 FROM categories c
    WHERE c.category_id = p.category_id AND c.is_active = TRUE
);
```

## Interview-Focused Notes
1. "How do you update a table based on another?" – Use UPDATE with JOIN (dialect specific) or correlated subquery.
2. "Why cautious with many-to-many?" – Multiple matches produce ambiguity; ensure uniqueness via constraints or aggregation.
3. "How to prevent accidental mass updates?" – Add a restrictive WHERE clause + first run as SELECT preview.
4. "Should you index?" – Absolutely index the join predicate column(s) for performance.
5. "What if joined value missing?" – Decide whether to skip (INNER semantics) or set to NULL (LEFT semantics—with dialect-specific syntax).

## Quick Recall ✅
- Join drives new values.
- Ensure 1:1 mapping.
- FROM vs JOIN syntax differs by RDBMS.
- Always preview with SELECT.
- Consider transaction + backup before large batch.

## Interview Traps & Confusions ⚠️
- Forgetting WHERE: updates all rows (catastrophic).
- Multi-row subquery returning >1 row (error in most RDBMS for scalar assignment).
- Non-deterministic updates if no unique constraint on joined column.

## Bonus
### Safe Preview Pattern
```sql
SELECT p.product_id, p.category_name_cache AS old_val, c.category_name AS new_val
FROM products p
JOIN categories c ON p.category_id = c.category_id
WHERE c.is_active = TRUE
LIMIT 20;
```

### Conditional Update
```sql
UPDATE customers c
SET loyalty_tier = CASE
    WHEN o.total_spent >= 10000 THEN 'PLATINUM'
    WHEN o.total_spent >= 5000 THEN 'GOLD'
    ELSE 'STANDARD'
END
FROM customer_totals o
WHERE o.customer_id = c.customer_id;
```

### Handling Null Source
```sql
UPDATE products p
SET supplier_name_cache = COALESCE(s.supplier_name, 'UNKNOWN')
FROM suppliers s
WHERE p.supplier_id = s.supplier_id;
```

### Anti-Join Based Clearing
```sql
UPDATE products p
SET discontinued_flag = TRUE
WHERE NOT EXISTS (
  SELECT 1 FROM inventory i WHERE i.product_id = p.product_id
);
```
