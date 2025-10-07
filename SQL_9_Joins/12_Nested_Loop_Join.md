# NESTED LOOP JOIN

## Overview
A Nested Loop Join (NLJ) is the simplest physical join algorithm: for each row in the outer (driving) input, probe the inner input for matching rows. Conceptually like a double for-loop.

## Core Concepts
- Works well when outer input is small and inner input indexed on join key.
- Always correct; optimizer picks it for selective lookups.
- Time complexity worst case: O(N × M).
- Variants: naive (no index), index nested loop, block nested loop.

**Pseudo-Algorithm:**
```
for each row R in outer:
  for each row S in inner:
    if R.key = S.key then output R ⨝ S
```

**Example Plan Trigger:**
```sql
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.customer_id
WHERE o.order_id IN (123, 456, 789);
```
Small outer (filtered orders) probing indexed customers.

## Interview-Focused Notes
1. "When is nested loop efficient?" – Small outer + indexed inner.
2. "Why still used vs hash join?" – Lower startup cost; great for highly selective probes.
3. "How to force (in some systems)?" – Hints: e.g. `/*+ USE_NL(table) */` (Oracle) or join hints in SQL Server.
4. "What degrades performance?" – Large outer + non-indexed inner (full scan per outer row).
5. "How to optimize?" – Create index on inner table's join column; reduce outer cardinality early via filters.

## Quick Recall ✅
- Basic double loop.
- Startup cost minimal.
- Best with selective probes.
- Worst with large unindexed inputs.
- Foundation for index & block variants.

## Interview Traps & Confusions ⚠️
- Confusing logical join type (INNER/LEFT) with physical algorithm (NLJ).
- Believing NLJ always bad at scale (with good indexes it's very fast).
- Ignoring join order: optimizer may reorder to enable efficient NLJ.

## Bonus
### Cardinality Reduction First
Push filters to shrink outer input before join.

### Monitoring (PostgreSQL EXPLAIN Example)
```
Nested Loop  (cost=... rows=...)
  ->  Index Scan using order_pkey on orders ...
  ->  Index Scan using customers_pkey on customers ...
```

### Anti-Pattern Example
Joining two large tables without indexes: triggers massive repeated scans—prefer hash or merge join.
