# OUTER JOIN (Concept)

## Overview
An OUTER JOIN preserves unmatched rows from one or both participating tables while filling non-matching side columns with `NULL`. There are three directional variants:
- LEFT OUTER JOIN
- RIGHT OUTER JOIN
- FULL (or FULL OUTER) JOIN

## Core Concepts
- Adds non-matching rows compared to INNER JOIN.
- Columns from the non-existing side are `NULL` padded.
- Logical model:
  - LEFT: All rows from left + matched from right.
  - RIGHT: All rows from right + matched from left.
  - FULL: All rows from both, matching where possible.
- Filtering placement (ON vs WHERE) critically affects retained rows.

**General Pattern:**
```sql
SELECT ...
FROM A
LEFT JOIN B ON A.key = B.key;  -- LEFT preserves A
```

**NULL Padding Example:**
If `B` has no matching key, columns from `B` appear as `NULL`.

## Interview-Focused Notes
1. "What is an OUTER JOIN?" – A join that returns unmatched rows with `NULL` placeholders.
2. "Difference between LEFT and FULL OUTER?" – LEFT preserves only left; FULL preserves both sides.
3. "Where should additional filters go?" – Use the `ON` clause to keep unmatched rows; a `WHERE` filter on the right-side column can convert it to an INNER join effect.
4. "When to use RIGHT over LEFT?" – Rare; LEFT is preferred for readability. RIGHT can occur when legacy or readability from perspective of second table.
5. "Why is FULL OUTER JOIN sometimes avoided?" – Not supported in MySQL (must simulate with UNION). Also can be computationally heavier and harder to reason about.

## Quick Recall ✅
- Adds unmatched rows with NULLs.
- LEFT keeps all left table rows.
- RIGHT keeps all right table rows.
- FULL keeps all rows from both.
- Filter placement matters (ON vs WHERE).

## Interview Traps & Confusions ⚠️
- Placing right-table filter in WHERE after a LEFT JOIN (accidentally negates outer behavior).
- Using RIGHT when a simple table reordering + LEFT is clearer.
- Expecting FULL OUTER JOIN in MySQL (unsupported – simulate via UNION of LEFT + RIGHT anti-semi).

## Bonus
### Anti-Pattern Example
```sql
-- Incorrect: This nullifies the outer effect
SELECT *
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
WHERE o.status = 'OPEN';  -- Filters out NULL (unmatched customers)
```

### Correct Preservation
```sql
SELECT *
FROM customers c
LEFT JOIN orders o
  ON c.customer_id = o.customer_id
 AND o.status = 'OPEN';   -- Filter belongs in ON to still keep customers
```

### FULL OUTER Simulation (MySQL Example)
```sql
SELECT c.customer_id, c.name, o.order_id
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
UNION
SELECT c.customer_id, c.name, o.order_id
FROM orders o
LEFT JOIN customers c ON c.customer_id = o.customer_id;
```

### Deciding Join Type
- Use INNER if missing relationships must be excluded.
- Use LEFT when primary entity list must remain intact.
- Use FULL for reconciliation / audit comparisons.
