# HASH JOIN

## Overview
A Hash Join builds an in-memory hash table on the smaller (build) input using the join key, then probes it with the other (probe) input. Efficient for large equi-joins without supporting indexes.

## Core Concepts
- Equality predicates only (`=`).
- Phases: Build → Probe → (Optionally) Output unmatched (outer join variants).
- Performance near linear: O(N + M) assuming good hash distribution.
- Memory dependent; may spill to disk (grace hash variant) if too large.

**Pseudo-Algorithm:**
```
# Build phase
hash_table = {}
for row in small_input:
  bucket = hash(row.key)
  hash_table[bucket].append(row)

# Probe phase
for row in large_input:
  bucket = hash(row.key)
  for candidate in hash_table[bucket]:
    if candidate.key = row.key: output join
```

**Example:**
```sql
SELECT f.fact_id, d.date_value
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key;
```
Optimizer picks hash join if no usable index and row counts large.

## Interview-Focused Notes
1. "When chosen?" – Large tables, equality join, lack of indexes, sufficient memory.
2. "Why faster than nested loop here?" – Avoids repeated scans / index probes; linear scanning both inputs.
3. "Spill behavior?" – If hash table exceeds memory, partitions & spills (grace hash join) → performance drop.
4. "Why not for range joins?" – Hashing loses ordering semantics; only equi-join semantics supported.
5. "Outer join handling?" – Probe phase tracks matched flags to emit unmatched rows if LEFT/FULL join.

## Quick Recall ✅
- Build smaller side.
- Probe larger side.
- Equality only.
- Memory sensitive.
- Spills reduce speed.

## Interview Traps & Confusions ⚠️
- Building on wrong (larger) side increases memory/time.
- Ignoring skew: heavily skewed keys create large buckets → degraded performance.
- Believing hash join always superior to merge join (order + sorted sources may favor merge).

## Bonus
### Skew Mitigation
- Analyze key distribution; create stats.
- Use salting technique (big data systems) for extreme skew.

### Memory Tuning
PostgreSQL: increase `work_mem` to prevent spilling.

### EXPLAIN Signature
```
Hash Join
  Hash Cond: (f.date_key = d.date_key)
  ->  Seq Scan on fact_sales f
  ->  Hash  -> Seq Scan on dim_date d
```

### Pre-Hash Filtering
Push predicates before build to shrink hash table.
