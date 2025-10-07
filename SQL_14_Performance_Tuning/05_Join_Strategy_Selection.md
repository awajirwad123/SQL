# Join Strategy Selection

## Overview
The optimizer selects a physical join algorithm based on estimated cardinalities, available indexes, join predicates, and memory. Understanding trade-offs helps craft queries & indexes that favor efficient strategies.

## Core Concepts
- **Nested Loop Join**: Best for small outer + indexed inner; O(outer * log(inner)). Great for selective probes.
- **Hash Join**: Build hash table on smaller input; good for large unsorted sets; requires memory; spills degrade performance.
- **Merge Join**: Requires both inputs sorted on join key; efficient linear pass; may need sort operations.
- **Batch / Vectorized Execution**: Columnar or batch mode can reduce CPU per row (SQL Server / analytical engines).
- **Join Order**: Reordered to minimize intermediate row explosion; driven by estimated selectivity.
- **Predicate Type**: Equi-join vs non-equi (range) influences algorithm (hash needs hashable equality keys).
- **Skew Handling**: Heavy skew can bloat one side of hash or cause imbalance.
- **Distributed Systems**: Broadcast vs shuffle joins (see distributed join file earlier).

### Algorithm Selection Heuristics
| Situation | Likely Choice |
|-----------|---------------|
| Small outer, index on inner | Nested Loop |
| Large, no useful indexes | Hash |
| Both pre-sorted on key | Merge |
| Need early row output (LIMIT) | Nested Loop + Index |

## Interview-Focused Notes
1. Hash vs nested loop? Hash for large unindexed sets; nested for repeated selective lookups.
2. Why merge join needs ordering? Without sorted inputs, must sort—costly if large.
3. Impact of wrong cardinality? Choose nested loops on huge sets (catastrophic) or hash on tiny sets (overhead).
4. How influence join order? Provide selective predicates early; accurate stats.
5. When add composite index for join? If join plus filter on leading columns repeatedly.

## Quick Recall ✅
- Plan = algorithm + order synergy.
- Nested loop sensitive to inner access path quality.
- Hash join memory spills hurt.
- Merge ideal for pre-sorted streams.
- Distributed: minimize data movement.

## Interview Traps & Confusions ⚠️
- Forcing hints ignoring future data growth.
- Misinterpreting cost difference due to stale stats.
- Ignoring effect of LIMIT on chosen plan.
- Assuming hash join always slower (can outperform loops on volume).
- Forgetting inequality joins block merge without extra sorting.

## Bonus
### Skew Mitigation
- Add stats on skewed column
- Use salting / splitting for extreme skew in distributed systems.

### Memory Grant Observations
Monitor hash join build size vs actual; adjust stats or query shape.

### Early Abort Optimization
Semi-joins (EXISTS) allow nested loop early termination.

### Composite Join Support
Index on `(foreign_key, selective_filter)` improves nested loops.

### Partial Hash Table Build (Adaptive Engines)
Some systems adapt join strategy mid-execution when row counts diverge.
