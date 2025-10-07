# Isolation Levels

## Overview
Isolation level controls which concurrency anomalies are allowed. Stricter isolation reduces anomalies but can increase contention, blocking, or abort rates. Different engines map ANSI levels to specific behaviors with nuances.

## Core Concepts
- **ANSI Levels**:
  - Read Uncommitted (rarely exposed; may allow dirty reads) – often maps to Read Committed with NOLOCK hints.
  - Read Committed – Prevents dirty reads; may allow non-repeatable reads & phantom reads.
  - Repeatable Read – Prevents dirty & non-repeatable reads; phantoms may be allowed (MySQL InnoDB actually prevents many via gap locks; Postgres treats Repeatable Read as Snapshot Isolation preventing phantoms logically).
  - Serializable – Emulates serial execution (via predicate locks / SSI / strict 2PL / range locks).
- **Snapshot / MVCC Variants**:
  - Snapshot Isolation (SI): Consistent snapshot for duration; write-write conflicts cause abort (Postgres Repeatable Read; SQL Server SNAPSHOT).
- **Phenomena** (ANSI): Dirty read, non-repeatable read, phantom read.
- **Implementation**:
  - Lock-based 2PL (SQL Server locking under Repeatable Read / Serializable).
  - MVCC with predicate/next-key locking (InnoDB).
  - Serializable Snapshot Isolation (Postgres) with predicate conflict detection.

### Level Matrix (Simplified)
| Phenomenon | Read Committed | Repeatable Read | Serializable |
|------------|----------------|-----------------|--------------|
| Dirty Read | ❌ | ❌ | ❌ |
| Non-Repeatable Read | ✅ | ❌ | ❌ |
| Phantom | ✅ | ✅/❌* | ❌ |
(*Engine-dependent)

## Interview-Focused Notes
1. Why not always Serializable? Performance cost (more blocking/aborts) & complexity.
2. Snapshot vs Serializable? Snapshot prevents dirty/non-repeatable; may allow write skew; Serializable prevents it.
3. Write skew example? Two doctors on call scheduling conflict missing mutual exclusion without predicate lock.
4. Repeatable Read differences? Postgres = SI semantics; MySQL adds gap locks preventing phantom inserts.
5. Choosing level strategy? Default Read Committed + escalate narrower transactions as needed.

## Quick Recall ✅
- Higher isolation = fewer anomalies + more cost.
- Snapshot avoids read locks; conflicts resolved at commit (optimistic).
- Serializable ensures full serial order (may abort). 
- Read Committed widely default for OLTP.
- Write skew only prevented at Serializable.

## Interview Traps & Confusions ⚠️
- Assuming Repeatable Read identical across engines.
- Believing NOLOCK is harmless (dirty reads, torn data possible).
- Confusing SI with Serializable safety.
- Over-escalating isolation globally without targeted need.
- Ignoring phantom issue in range-based constraints.

## Bonus
### Promote-on-Conflict Pattern
Start at Read Committed; retry at higher isolation if conflict detection reveals anomalies.

### Range Constraint Enforcement
Use explicit locking `SELECT ... FOR UPDATE` on key range to emulate predicate lock.

### Detect Serialization Failures
Postgres error SQLSTATE '40001' (serialization_failure) → safe to retry.

### Mixed Workloads
Analytics read at Snapshot; OLTP writes at Read Committed; critical invariants at Serializable only when needed.
