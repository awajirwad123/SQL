# Distributed & Long-Running Transactions

## Overview
Distributed transactions span multiple resource managers (databases, message queues). Long-running business processes may exceed practical DB transaction duration, requiring alternative patterns (sagas, compensation). Two-phase commit (2PC) ensures atomic commit across participants but introduces complexity and blocking risks.

## Core Concepts
- **Two-Phase Commit (2PC)**:
  1. Prepare: Coordinator asks participants to promise commit (flush intent / locks).
  2. Commit: If all prepared, coordinator issues commit; else rollback.
- **Problems**: Blocking if coordinator fails after prepare, increased lock hold time.
- **Three-Phase Commit**: Adds pre-commit to reduce blocking (rare in practice due to complexity).
- **Saga Pattern**: Sequence of local transactions with compensating actions for rollback semantics.
- **Idempotency**: Critical for retries in distributed workflows.
- **Message Outbox**: Ensures DB state change + event publication atomicity (store event in DB then forward reliably).
- **TCC (Try-Confirm/Cancel)**: Reservation then confirmation or cancellation semantics.
- **Timeouts**: Long-held locks degrade throughput; avoid spanning user interactions.

### Saga Example (High-Level)
1. Reserve inventory (local transaction). 2. Authorize payment. 3. Create shipping order. If step fails, apply compensations (release inventory, void payment auth).

## Interview-Focused Notes
1. Why avoid long DB transactions? Lock contention, resource holding, potential bloat.
2. 2PC drawback? Blocking + single point (coordinator) risk.
3. Saga vs 2PC? Saga offers eventual consistency with compensations; not strictly atomic globally.
4. Outbox pattern solves? Dual-write problem to DB + message broker.
5. Idempotency design? Use unique operation keys to ignore duplicates on retry.

## Quick Recall ✅
- 2PC: prepare + commit phases.
- Sagas: sequence + compensation.
- Avoid human wait inside transaction.
- Outbox ensures reliable event publishing.
- Idempotent keys crucial.

## Interview Traps & Confusions ⚠️
- Using DB transaction to span external API calls.
- Expecting strict atomicity across heterogeneous systems without coordination cost.
- Ignoring compensation failure path.
- Overusing distributed transactions where eventual consistency acceptable.
- Not deduplicating messages/events on consumer side.

## Bonus
### Idempotency Key Table
Store key + status; reject duplicates early.

### Outbox Consumer
Background process reads outbox table rows (status=pending), publishes, marks sent.

### Partial Failure Observability
Correlate saga steps with trace id for debugging.

### Timeout Guard
Abort saga step if exceeds SLA, trigger compensation.
