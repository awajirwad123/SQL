# BLOCK NESTED LOOP JOIN

## Overview
A Block Nested Loop Join improves on naive nested loops by reading chunks (blocks) of the outer input into memory and probing the inner input more efficiently (sometimes with scanned caching). It reduces repeated I/O passes over the inner table.

## Core Concepts
- Reads outer in batches (buffered pages) instead of row-by-row.
- Inner may still be scanned sequentially per batch unless indexed optimization available.
- Benefit: Fewer rescans; increased I/O efficiency.
- Used when hash/merge join not applicable (e.g., non-equality predicates) or memory limited.

**Conceptual Pseudo:**
```
while (block = read_next_outer_block()):
  for each row S in inner_scan:
    for each row R in block:
      if join_match(R,S): output
```

## Interview-Focused Notes
1. "How differs from classic nested loop?" – Processes outer in blocks to reduce overhead.
2. "When chosen?" – Moderate-sized join where hash join not suitable (non-equi) or memory constrained.
3. "What improves performance?" – Larger buffer size, inner table sequential scan speed.
4. "Why not always use hash join?" – Hash requires equality predicates and memory for hash table.
5. "Can indexes help?" – Yes; if inner index exists, engine may hybridize approach.

## Quick Recall ✅
- Batch-oriented nested loop.
- Less random I/O.
- Good for range/non-equi joins.
- Falls between naive NLJ and hash join in performance.
- I/O optimization focus.

## Interview Traps & Confusions ⚠️
- Thinking it changes algorithmic complexity (still potentially large).
- Assuming it appears explicitly in all EXPLAIN outputs (abstracted in some systems).
- Misusing for large equi-joins better served by hash/merge.

## Bonus
### Non-Equi Join Example
```sql
SELECT p.product_id, s.season_id
FROM products p
JOIN seasons s ON p.launch_date BETWEEN s.start_date AND s.end_date;
```
Hash join cannot apply (range condition) → block nested loop candidate.

### Tuning Levers
- Increase work memory (Postgres `work_mem`) to allow hash join alternative.
- Add derived key (e.g., precomputed period_id) to convert range join into equi-join.

### EXPLAIN Recognition
May appear as `Nested Loop` with buffer usage notes; block strategy internal.
