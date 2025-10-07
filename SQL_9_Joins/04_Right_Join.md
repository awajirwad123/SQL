# RIGHT JOIN

## Overview
A `RIGHT JOIN` (RIGHT OUTER JOIN) returns all rows from the right table plus matching rows from the left table. It is functionally symmetric to a `LEFT JOIN` with table order swapped, but is less frequently used for readability reasons.

## Core Concepts
- Preservation side: Right table.
- Often replaced by rewriting as LEFT JOIN with reversed order.
- Same NULL padding logic as LEFT JOIN but on opposite side.

**Syntax:**
```sql
SELECT o.order_id, o.order_total, c.customer_id, c.name
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id;
```
Equivalent (preferred) LEFT JOIN rewrite:
```sql
SELECT o.order_id, o.order_total, c.customer_id, c.name
FROM orders o
LEFT JOIN customers c ON c.customer_id = o.customer_id;
```

## Interview-Focused Notes
1. "When to use RIGHT JOIN?" – Rarely; typically only when logical narrative flows from right table first or for legacy code.
2. "Is RIGHT JOIN necessary?" – No; any RIGHT JOIN can be converted to LEFT JOIN by swapping table positions.
3. "Why avoid RIGHT JOIN?" – Reduces cognitive switching; teams standardize on LEFT JOIN only.
4. "Does performance differ vs rewritten LEFT JOIN?" – No (optimizer produces identical plan).
5. "How to find unmatched rows on left when using RIGHT JOIN?" – `WHERE c.customer_id IS NULL` after the join.

## Quick Recall ✅
- Mirror of LEFT JOIN.
- Rarely recommended stylistically.
- Convert by swapping tables.
- Same filter placement rules apply.
- Performance neutral relative to equivalent LEFT.

## Interview Traps & Confusions ⚠️
- Mixing LEFT and RIGHT in complex chains—hard to reason about directionality.
- Misinterpreting which side is preserved.
- Forgetting to adjust null checks when refactoring to LEFT JOIN.

## Bonus
### Anti-Join Rewritten
RIGHT JOIN version:
```sql
SELECT o.order_id
FROM customers c
RIGHT JOIN orders o ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL; -- Orders with no customer (data issue)
```
Preferred using LEFT JOIN by swapping order:
```sql
SELECT o.order_id
FROM orders o
LEFT JOIN customers c ON c.customer_id = o.customer_id
WHERE c.customer_id IS NULL;
```

### Full Comparison Tip
If you must show both matched and unmatched from both sides, use FULL JOIN (or simulate if unsupported) instead of combining RIGHT + LEFT manually.
