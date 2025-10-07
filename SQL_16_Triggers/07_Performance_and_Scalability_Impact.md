# Performance & Scalability Impact

## Overview
Triggers add implicit work to every qualifying DML operation. Poor design causes latency, lock contention, and cascading performance issues. Optimizing trigger cost and ensuring predictability are essential for scale.

## Core Concepts
- **Latency Amplification**: Extra writes / reads per row amplify duration of core transactions.
- **Row-Level Fan-out**: Row trigger with multiple secondary queries multiplies workload.
- **Lock Contention**: Summary table updates in trigger can serialize otherwise parallel operations (hotspot risk).
- **Recursive/Chain Invocations**: Trigger updates same table or other tables with triggers—exponential effect.
- **Plan Stability**: Trigger body queries must be tuned (index coverage) to avoid regressions.
- **Bulk Operations**: Triggers may transform linear bulk load into O(n * body cost). Consider disabling or batching logic.
- **Caching / Memoization**: Preload reference data into temp or memory table at session start rather than lookup per row (use carefully).

### Hotspot Example
Incrementing a single counter row in an AFTER INSERT trigger for each new event row.

### Mitigation
Aggregate counts asynchronously (queue or periodic batch) instead of synchronous increment.

## Interview-Focused Notes
1. Main performance risk? Hidden per-row overhead & contention.
2. How reduce summary contention? Batch outside trigger or use partitioned counters.
3. Bulk load strategy? Disable non-critical triggers; apply compensating batch logic after.
4. Detect recursion? Track nesting level; guard with session variable / context setting.
5. Plan tuning inside triggers? Same principles—SARGable predicates, proper indexing.

## Quick Recall ✅
- Keep body O(1) lightweight.
- Avoid cross-table cascade chain.
- Replace synchronous aggregation with async/batch.
- Monitor execution time distribution.
- Guard against recursion loops.

## Interview Traps & Confusions ⚠️
- Assuming minor logic safe at scale (multiplies by row count).
- Updating global counter row per insert (serialization bottleneck).
- Hidden trigger causing unexpected deadlocks.
- Believing disabling trigger has no effect on data assumptions.
- Ignoring that trigger errors abort parent DML.

## Bonus
### Profiling Approach
Add lightweight timing into log table (with rate limiting) to capture slow trigger executions.

### Adaptive Switch
Session-level flag to skip non-critical logic during maintenance window.

### Partitioned Counters
Store counts per shard key; aggregate lazily when reported.

### Change Data Capture Alternative
Use native CDC to offload heavy downstream processing from triggers.
