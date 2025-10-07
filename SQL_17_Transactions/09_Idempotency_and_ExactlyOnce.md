# Idempotency & Exactly-Once Semantics

## Overview
Idempotency ensures repeated execution yields same final state as a single execution. True exactly-once processing across distributed components is difficult; systems approximate via idempotent operations + at-least-once delivery + deduplication.

## Core Concepts
- **Idempotent Operation**: Applying multiple times does not change result after first success.
- **Exactly-Once Myth**: Network + distributed failures make strict guarantee impractical without heavy coordination.
- **Deduplication Store**: Track processed operation IDs (idempotency keys).
- **Insert-If-Not-Exists**: Use unique constraint to enforce single application.
- **Upsert**: Merge semantics to collapse create/update into deterministic final state.
- **Status State Machine**: Enforce allowed transitions; ignore duplicates at same/terminal state.
- **Outbox + Consumer Idempotency**: Consumer tracks last processed sequence/id.

### Idempotent Transfer Example
Use unique business transaction id; if repeat, verify already applied amounts.

### Upsert Pattern
```sql
INSERT INTO inventory(product_id, qty)
VALUES (1, 10)
ON CONFLICT (product_id)
DO UPDATE SET qty = inventory.qty + EXCLUDED.qty;
```

## Interview-Focused Notes
1. Why idempotency key? Detect and discard duplicate requests after retry.
2. Exactly-once alternative? At-least-once + idempotent processing + dedupe table.
3. Schema aid? Unique constraints & natural business keys.
4. State machine advantage? Reject invalid repeats gracefully.
5. Upsert benefit? Atomic increment/merge eliminates race between select & insert.

## Quick Recall ✅
- Exactly-once: build via idempotent layers.
- Unique constraint = dedupe enforcement.
- Upsert consolidates race conditions.
- Log processed keys.
- Terminal states prevent reprocessing.

## Interview Traps & Confusions ⚠️
- Believing messaging system alone ensures exactly-once.
- Using timestamps as idempotency keys (collisions/insufficient uniqueness).
- Not expiring dedupe entries leading to unbounded growth.
- Handling compensation without idempotent compensation id.
- Ignoring concurrent duplicate arrival window.

## Bonus
### Dedupe Table Design
(idempotency_key PK, status, applied_at, metadata JSONB)

### TTL Strategy
Archive / purge dedupe records after safe horizon (e.g., 30 days).

### Hash Normalization
Hash large payload for quick comparison (detect same semantics different formatting).

### Consumer Offset Check
Store last processed id per partition & verify monotonic progression.
