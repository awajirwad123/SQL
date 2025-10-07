# Locking & Deadlocks

## Overview
Locking coordinates concurrent access; deadlocks occur when transactions cyclically wait on each other. Understanding lock types and escalation helps prevent contention and resolve conflicts.

## Core Concepts
- **Lock Types**: Shared (S), Exclusive (X), Intent (IS/IX), Update (SQL Server), Row vs Page vs Table.
- **MVCC**: Readers avoid blocking writers (versioned), but writers still contend on row modification.
- **Lock Escalation**: Engine may escalate many row locks to table lock to reduce overhead.
- **Deadlock**: Circular wait; engine detects and aborts a victim transaction.
- **Detection & Resolution**: Deadlock monitor (SQL Server), Wait-for graph (general concept).
- **Preventive Ordering**: Access objects in consistent global order.
- **Hotspot Rows**: Frequently updated counters cause contention.
- **Lock Wait Timeout**: Fails fast to avoid indefinite waits (`lock_timeout` Postgres).

### Deadlock Example
T1 locks Row A then needs Row B; T2 locks Row B then needs Row A → cycle.

### Mitigations
- Order resource acquisition.
- Keep transactions short.
- Reduce locking granularity (row vs table) with proper filtering indexes.
- Retry aborted victim transaction.

## Interview-Focused Notes
1. Define deadlock? Cyclic waiting—requires victim resolution.
2. Why consistent ordering? Prevent cycles.
3. Difference blocking vs deadlock? Blocking is linear wait; deadlock cyclic.
4. Avoid hotspot? Shard counters or batch increments asynchronously.
5. Detect lock issues? Monitor waits / system views (e.g., `pg_locks`, `sys.dm_tran_locks`).

## Quick Recall ✅
- Writers still conflict under MVCC.
- Deadlock = cycle; blocking ≠ cycle.
- Order + short duration reduce risk.
- Indexes help by narrowing locked rows.
- Retry logic for deadlock victims.

## Interview Traps & Confusions ⚠️
- Treating all waits as deadlocks.
- Long user think-time inside transaction.
- Escalation surprises due to large batch updates.
- Ignoring lock timeouts configuration.
- Assuming read-only snapshot prevents write conflicts.

## Bonus
### Deadlock Graph Analysis
Extract conflicting statements; add consistent ordering or finer predicate filters.

### Partial Update Approach
Chunk updates to reduce simultaneous lock footprint.

### Lock Timeout Setting
Set moderate timeout to fail fast and trigger retries.

### Hot Partition Strategy
Partition table to distribute write load; localize locks.
