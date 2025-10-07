# ACID Properties

## Overview
ACID defines the foundational guarantees of transactional systems: Atomicity, Consistency, Isolation, Durability. Understanding each property and its implementation aids in evaluating edge cases and behavior under concurrency or failure.

## Core Concepts
- **Atomicity**: All-or-nothing semantics. Achieved via write-ahead logs and undo/rollback segments.
- **Consistency**: Transaction moves database from one valid state to another preserving invariants (constraints, triggers, referential integrity). Responsibility is shared: engine enforces declarative rules; application ensures domain invariants.
- **Isolation**: Effects of concurrent transactions appear as if executed serially within defined anomaly allowances (depends on isolation level).
- **Durability**: Once committed, changes persist after crash—log flush, fsync, replication guarantees.

### Implementation Mechanisms
| Property | Mechanism |
|----------|-----------|
| Atomicity | Undo/rollback log, WAL records |
| Consistency | Constraints, triggers, application validation |
| Isolation | Locking, MVCC, predicate locking, snapshot versions |
| Durability | WAL flush, replication commit quorum, storage reliability |

## Interview-Focused Notes
1. Which ACID part does application influence most? Consistency (domain invariants).
2. Isolation vs Consistency difference? Isolation manages concurrency effects; consistency ensures rule adherence.
3. Can atomicity fail after crash? Only if durability incomplete—rare with proper WAL flush.
4. MVCC relates to which property? Isolation (version snapshots) and aids consistency.
5. Does ACID guarantee performance? No—only correctness semantics.

## Quick Recall ✅
- ACID ensures correctness, not speed.
- Isolation levels may intentionally allow anomalies for performance.
- Consistency broader than constraints; includes invariants.
- Durability implies log flush before acknowledgment.
- Atomicity pairs with rollback ability.

## Interview Traps & Confusions ⚠️
- Believing high isolation always required (overhead).
- Assuming constraints alone guarantee full domain consistency.
- Thinking durability equals replication (replication lag can exist post-commit).
- Ignoring hardware/fsync configuration effect on durability.
- Confusing snapshot isolation with serializable correctness.

## Bonus
### Relaxed Isolation + Compensating Logic
Sometimes allow lower isolation with detection/reconciliation (optimistic concurrency pattern).

### Durability Tuning
Synchronous commit vs async commit trades latency vs risk window (Postgres `synchronous_commit`).

### Consistency Layers
1. Physical (no torn pages) 2. Structural (indexes, fk) 3. Logical/business (invariants) 4. Application-level workflows.

### Atomic Multi-Row Insert
Engine ensures either all row modifications inside transaction visible or none.
