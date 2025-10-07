# Concurrency Phenomena

## Overview
Concurrency anomalies arise when multiple transactions interact without full isolation. Recognizing them helps choose appropriate isolation levels or mitigation strategies.

## Core Concepts
- **Dirty Read**: Transaction reads uncommitted changes of another; if rolled back, read was invalid.
- **Non-Repeatable Read**: Same row read twice returns different values due to another committed update.
- **Lost Update**: Concurrent modifications overwrite each other without detection.
- **Phantom Read**: New rows appear (or disappear) in a repeated range query.
- **Write Skew**: Interdependent rows updated concurrently causing invariant violation (classic scheduling example).
- **Read Skew (Snapshot)**: Transaction sees mix of committed states from different times (not in snapshot isolation which uses consistent snapshot, but present in lower isolation variants).
- **Dirty Write**: Overwriting uncommitted changes—generally prevented by mainstream engines with write locks.

### Examples
- **Lost Update** (if not using row locks):
  1. T1 reads balance=100.
  2. T2 reads balance=100.
  3. T1 sets balance=90 (writes -10); commits.
  4. T2 sets balance=80 (writes -20) based on stale read; lost update of -10.
  *Mitigation:* SELECT ... FOR UPDATE or optimistic version check.

- **Write Skew**:
  Two doctors ensure at least one on-call. Both read schedule (no one on call), both insert themselves off-call (or remove themselves), invariant broken. Only Serializable or explicit locking prevents.

## Interview-Focused Notes
1. Lost update prevention? Pessimistic locking or optimistic checks with version column.
2. Phantom vs non-repeatable? Phantom = new rows matching predicate; non-repeatable = same row changed.
3. Write skew occurs under? Snapshot isolation when constraints span multiple rows.
4. Dirty read risk? Decisions made on uncommitted data that might rollback.
5. Which anomaly Serializable addresses? All standard phenomena including write skew & phantoms.

## Quick Recall ✅
- Dirty read → uncommitted data.
- Non-repeatable → row changed.
- Phantom → new qualifying row.
- Lost update → concurrent overwrite.
- Write skew → multi-row invariant break.

## Interview Traps & Confusions ⚠️
- Mislabeling lost update as non-repeatable read (distinct). 
- Assuming MVCC eliminates write skew (needs Serializable/locking).
- Overusing serializable for simple phantoms; maybe explicit locking cheaper.
- Ignoring application-level invariants not enforced by constraints.
- Believing optimistic retry cost-free (heavy conflicts degrade throughput).

## Bonus
### Optimistic Concurrency Pattern
Include version column; update with `WHERE id=? AND version=?`, check rows affected.

### Predicate Lock Simulation
Select & lock index gap / range to block phantom inserts.

### Monitoring Anomalies (Postgres)
Serialization failures count; high rate indicates contention or need redesign.

### Deadlock vs Write Skew
Deadlock = circular waits; write skew may commit but breaks invariant.
