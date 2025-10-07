# GRACE HASH JOIN

## Overview
A Grace Hash Join is a multi-phase hash join variant designed to handle large tables that do not fit in memory. It partitions both inputs into smaller chunks that do fit, then performs standard in-memory hash joins per partition.

## Core Concepts
- Phases: Partition → (optional re-partition) → Build & Probe per partition.
- Uses same hash function during partitioning for both inputs so matching keys land in same partition files.
- Minimizes memory requirements at cost of extra I/O.

**High-Level Steps:**
1. Partition both relations R and S into N buckets on disk using hash(key).
2. For each i in 1..N:
   - Load partition Ri into memory; build hash table.
   - Stream partition Si and probe.

## Interview-Focused Notes
1. "When needed?" – Hash join spilling due to insufficient memory; very large datasets.
2. "Why effective?" – Divides problem into independent, memory-sized subproblems.
3. "Cost trade-off?" – Extra read/write I/O vs avoiding full nested loop explosion.
4. "Difference from simple hash join?" – Adds partition phase; simple variant assumes memory fits.
5. "Similar concept in big data?" – MapReduce / Spark shuffle partitioning prior to join.

## Quick Recall ✅
- External (disk-assisted) hash join.
- Handles large inputs exceeding RAM.
- Partitions both sides with same hash function.
- Processes partitions independently.
- I/O heavy but scalable.

## Interview Traps & Confusions ⚠️
- Confusing with broadcast/shuffle joins (those are distributed strategies).
- Assuming unlimited partitions solve skew—skewed keys still form imbalanced partitions.
- Ignoring cost of multiple passes over data.

## Bonus
### Skew Handling
Secondary hash within oversized partitions or fallback to nested loop if small.

### Partition Count Tuning
Balance between partition size (fit memory) and overhead (too many partitions increases metadata + open file cost).

### Visual Mental Model
Think: Split → Match buckets only within same hash partition → Join each subset.

### Example EXPLAIN Indicators
Look for: `Hash Join` + notes about `Batch` or spilling / disk usage in advanced execution stats.
