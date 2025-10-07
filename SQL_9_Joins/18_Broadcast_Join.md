# BROADCAST JOIN

## Overview
A Broadcast Join (a distributed system join strategy) replicates a small table to all worker nodes so each node can join it with its local partition of the large table without shuffling the large table's data.

## Core Concepts
- Used in distributed SQL engines (Spark, BigQuery, Snowflake, Presto, etc.).
- Small side fully copied (broadcast) to all executors.
- Eliminates shuffle of large table → big performance win when size asymmetry large.
- Memory bound by replicated table size × number of workers.

**When Chosen:**
- Broadcast side below threshold (e.g., Spark auto broadcast < 10MB by default).
- Join type supports it (e.g., equi-joins typically).

**Spark Hint Example:**
```sql
SELECT /*+ BROADCAST(dim_customer) */ f.*, d.segment
FROM fact_sales f
JOIN dim_customer d ON f.customer_id = d.customer_id;
```

## Interview-Focused Notes
1. "What is broadcast join?" – Replicate small table to all nodes to avoid shuffling large table.
2. "Why fast?" – Removes network exchange for big side; local hash join per node.
3. "When not appropriate?" – Small table too large to broadcast; memory pressure risks spills/GC.
4. "Comparison to shuffle join?" – Shuffle repartitions both sides; broadcast leaves large side unchanged.
5. "Manual override?" – Provide hint or config to force broadcast when auto cost model misestimates.

## Quick Recall ✅
- Small → broadcasted.
- Avoids large table movement.
- Threshold-based.
- Memory trade-off.
- Common in star schema joins.

## Interview Traps & Confusions ⚠️
- Forcing broadcast of huge table (performance collapse / OOM).
- Ignoring data skew—broadcast doesn’t cure hot keys in large table.
- Broadcasting multiple mid-sized tables → cumulative memory blowup.

## Bonus
### Star Schema Pattern
Fact table joined to multiple small dimensions—each dimension broadcast individually.

### Tuning Spark
Increase `spark.sql.autoBroadcastJoinThreshold` for larger broadcast if executors sized appropriately.

### Fallback Behavior
If broadcast attempt fails (memory), engine may retry with shuffle join.

### Diagnosing Plan
EXPLAIN shows `BroadcastHashJoin` or `BroadcastNestedLoopJoin` depending on predicate.
