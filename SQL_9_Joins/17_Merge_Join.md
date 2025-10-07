# MERGE JOIN (Sort-Merge Join)

## Overview
A Merge Join combines two sorted inputs by advancing pointers in each stream, outputting matches as it proceeds. Efficient for large equi-joins when both sides are pre-sorted or have usable ordered indexes.

## Core Concepts
- Requires both inputs sorted on join key(s).
- Linear pass once sorted: O(N + M) after O(N log N + M log M) sort if not pre-sorted.
- Supports range-based matching extensions in some systems (e.g., inequality band joins with modifications).

**Simplified Algorithm:**
```
while i < A.len and j < B.len:
  if A[i].key = B[j].key: output all combinations; advance both
  elif A[i].key < B[j].key: i++
  else: j++
```

**Example (Pre-Sorted via Indexes):**
```sql
SELECT f.fact_id, d.date_value
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key;
```
If both tables clustered/indexed on `date_key`, merge join avoids hashing.

## Interview-Focused Notes
1. "When chosen?" – Large joins where both sides already sorted (clustered index, order by pipeline).
2. "Advantages vs hash join?" – No hash table build; preserves ordering; can pipeline results early.
3. "Disadvantages?" – Requires sorting if not already; not ideal for highly selective small lookups.
4. "Equality only?" – Primarily equality; some RDBMS adapt to band joins with extra logic.
5. "Output order?" – Matches sorted by join key.

## Quick Recall ✅
- Stream-based join.
- Needs sorted inputs.
- Great for very large, already-ordered tables.
- Order-preserving.
- Avoids memory hash overhead.

## Interview Traps & Confusions ⚠️
- Overlooking cost of prerequisite sorts.
- Assuming always faster than hash join (depends on sort cost & memory).
- Expecting it for random unsorted inputs.

## Bonus
### Composite Key Merge
Sort by (k1, k2); merge compares lexicographically.

### EXPLAIN Signature
```
Merge Join
  Merge Cond: (f.date_key = d.date_key)
  -> Index Scan using fact_date_idx on fact_sales f
  -> Index Scan using dim_date_pkey on dim_date d
```

### Forcing Merge Join
Optimizer hints (Oracle: `/*+ USE_MERGE */`, SQL Server: `OPTION (MERGE JOIN)`), mainly for testing.

### Comparison Summary
| Aspect        | Merge Join | Hash Join        | Nested Loop        |
|---------------|------------|------------------|--------------------|
| Pre-Req       | Ordered    | None (Equality)  | None               |
| Best For      | Large sorted | Large unsorted   | Small outer + index|
| Memory        | Low/Moderate| Moderate to High | Low                |
| Ordering Kept | Yes        | No               | Depends            |
