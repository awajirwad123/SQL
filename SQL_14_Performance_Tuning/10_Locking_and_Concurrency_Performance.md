# Locking & Concurrency Performance

## Overview
Concurrency control ensures consistency but can impede throughput when locking or blocking escalates. Tuning involves reducing lock contention, choosing appropriate isolation, and designing queries for short critical sections.

## Core Concepts
- **Lock Types**: Shared (read), Exclusive (write), Intent locks (hierarchical), Row vs Page vs Table.
- **Isolation Levels**: Read Committed, Repeatable Read, Serializable, Snapshot/MVCC variants; trade anomalies vs blocking.
- **MVCC (Postgres)**: Readers don’t block writers; vacuum needed to reclaim dead tuples.
- **Blocking vs Deadlock**: Blocking = waiting; deadlock = cyclic wait requiring victim.
- **Hotspot**: Single frequently updated row / counter / sequence causing serialization.
- **Lock Escalation**: SQL Server may escalate to table lock when many row locks.
- **Long Transactions**: Retain old row versions; increase bloat and conflict risk.
- **Write Amplification**: Excess indexes amplify locking work.

### Detect Blocking
- Postgres: pg_locks + pg_stat_activity join.
- SQL Server: sys.dm_tran_locks + wait stats.

### Reduce Contention
- Keep transactions short.
- Access objects in consistent order.
- Batch updates in chunks.
- Use optimistic features (Snapshot Isolation) where acceptable.

## Interview-Focused Notes
1. Why long transactions bad? Hold locks/version references → block cleanup & others.
2. Difference blocking vs deadlock? Blocking is linear wait; deadlock cyclic needing kill.
3. MVCC advantage? Readers proceed without blocking writers.
4. Hot row mitigation? Shard counters, use sequence/generator, accumulate in memory then batch apply.
5. Lower isolation effect? Fewer blocking scenarios but risk anomalies.

## Quick Recall ✅
- Short transactions; minimal round trips.
- Proper indexing reduces lock duration per row.
- Monitor waits (lock, latch, IO) to classify bottleneck.
- Avoid hotspot contention with partitioning or queue design.
- Choose isolation level to balance correctness & throughput.

## Interview Traps & Confusions ⚠️
- Believing MVCC means no contention (writers still conflict on same row).
- Using SERIALIZABLE unnecessarily (higher abort rate).
- Ignoring deadlock retry logic in application.
- Leaving idle in transaction sessions open (open transaction marker).
- Misattributing CPU-bound wait to locks without checking wait stats.

## Bonus
### Deadlock Detection Strategy
Log deadlock graphs (SQL Server) / enable `deadlock_timeout` diagnostics (Postgres) to capture patterns.

### Hot Sequence Alternative
Use `BIGINT` identity per shard + combine with shard id to ensure uniqueness.

### Chunked Update Pattern
```sql
UPDATE items
SET processed = TRUE
WHERE processed = FALSE
ORDER BY id
LIMIT 1000; -- repeat in loop
```

### Vacuum / Autovacuum Tuning (Postgres)
Ensure aggressive enough to prevent table & index bloat affecting scan times.

### Wait Profiling
Classify waits (lock vs IO vs CPU) before hypothesis on tuning direction.
