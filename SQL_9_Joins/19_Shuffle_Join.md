# SHUFFLE JOIN (Distributed Repartition Join)

## Overview
A Shuffle Join (a.k.a. Repartition Join, Distributed Hash Join) redistributes both input tables across the cluster based on a hash of the join key so that matching keys land on the same node. Then local joins are performed per partition.

## Core Concepts
- Required when neither side is small enough to broadcast.
- Both sides incur network I/O (shuffle phase).
- Partitioning function: typically hash(key).
- After redistribution, engine applies hash / merge / nested loop locally.

**Execution Phases:**
1. Map: Read partitions, compute hash(key), send rows to target partitions.
2. Shuffle Exchange: Network transfer.
3. Reduce: Local join per partition.

**Example (Spark Plan):**
```
== Physical Plan ==
*(5) HashAggregate ...
+- *(4) Project ...
   +- *(4) SortMergeJoin [customer_id], [customer_id]
      :- *(2) Sort ... Exchange hashpartitioning(customer_id, 200)
      +- *(3) Sort ... Exchange hashpartitioning(customer_id, 200)
```

## Interview-Focused Notes
1. "When is shuffle join used?" – Large-large joins where broadcast infeasible.
2. "Why expensive?" – Full data repartition across network + possible sorts.
3. "How to optimize?" – Increase partition parallelism, prune columns early, filter before join.
4. "What causes skew issues?" – Hot key sends disproportionate rows to single partition → straggler.
5. "Mitigations?" – Salting keys, skew join hints, pre-aggregation.

## Quick Recall ✅
- Repartitions both inputs.
- High network cost.
- Scales to large datasets.
- Sensitive to skew & partition sizing.
- Foundation pattern in distributed engines.

## Interview Traps & Confusions ⚠️
- Forgetting to push filters below shuffle (unnecessary data movement).
- Using too few partitions => poor parallelism.
- Over-partitioning => overhead & small tasks overhead.

## Bonus
### Skew Handling (Salting Example)
```sql
-- Add random salt to skewed key range
SELECT f.*, d.segment
FROM (
  SELECT *, FLOOR(RAND()*8) AS salt FROM fact_sales WHERE customer_id = 42
  UNION ALL
  SELECT *, 0 AS salt FROM fact_sales WHERE customer_id <> 42
) f
JOIN (
  SELECT customer_id, segment, generate_series(0,7) AS salt FROM dim_customer
) d USING (customer_id, salt);
```

### Column Pruning
Select only needed columns before shuffle to reduce payload size.

### Adaptive Execution
Some engines (Spark AQE) dynamically switch broadcast vs shuffle at runtime if size smaller than estimated.

### Comparison Summary
| Strategy    | Small Side? | Network Movement |
|-------------|-------------|------------------|
| Broadcast   | Yes         | Small → All Nodes|
| Shuffle     | No          | Both Repartition |

