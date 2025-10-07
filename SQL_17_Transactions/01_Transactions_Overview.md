# Transactions Overview

## Overview
A transaction is an atomic, consistent, isolated, and durable (ACID) unit of work that groups one or more SQL statements so they either all succeed or none persist. Transactions protect data integrity in the face of concurrency, failures, and partial operations.

## Core Concepts
- **Unit of Work**: Logical boundary (e.g., transfer funds, place order + items + inventory update).
- **Autocommit**: Many engines run each statement as its own transaction unless explicit `BEGIN`/`START TRANSACTION`.
- **Commit vs Rollback**: `COMMIT` makes changes visible globally; `ROLLBACK` reverts to the transaction start (or savepoint).
- **Durability Mechanism**: Write-ahead log (WAL / redo log); ensures persistence after crash once commit flush acknowledged.
- **Isolation vs Concurrency**: Higher isolation → fewer anomalies but more blocking/overhead.
- **Logical vs Physical Consistency**: Constraints + application invariants enforced by transaction boundaries.
- **Short-Lived Principle**: Keep transactions minimal to reduce contention & lock duration.

### Basic Example
```sql
BEGIN;  -- or START TRANSACTION
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- or ROLLBACK on error
```

## Interview-Focused Notes
1. Definition? Atomic sequence of operations committed or rolled back together.
2. Why group statements? Maintain consistency across related changes.
3. Autocommit impact? Each statement isolated; multi-step invariants not protected.
4. Durability guarantee? After COMMIT acknowledged and log flushed, survives crash.
5. Why minimize transaction time? Reduce lock contention and version retention.

## Quick Recall ✅
- BEGIN → work → COMMIT (or ROLLBACK).
- ACID: Atomicity, Consistency, Isolation, Durability.
- Autocommit can break multi-step invariants.
- Keep short & deterministic.
- Logging ensures durability.

## Interview Traps & Confusions ⚠️
- Assuming partial commits mid-transaction without explicit savepoints.
- Leaving idle transaction open (holding locks / versions).
- Confusing transaction with session (session may have multiple transactions).
- Believing durability before COMMIT (uncommitted changes can be lost).
- Using transaction for long-running reporting (use snapshot features instead).

## Bonus
### Retry On Transient Errors
Identify retriable errors (deadlock, serialization failure) and re-run whole transaction.

### Idempotent Design
Use unique business keys to avoid double application on retry.

### Monitoring Metric
Track average & p95 transaction duration; investigate prolonged ones.

### Statement vs Transaction Atomicity
Single statement DDL often implicit transaction; multi-statement requires explicit boundary.
