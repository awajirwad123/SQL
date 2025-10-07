# Retry & Error Patterns

## Overview
Transient failures (deadlocks, serialization anomalies, network glitches) require safe retry strategies. Proper patterns prevent duplicate side-effects and livelocks.

## Core Concepts
- **Retriable Errors**: Deadlock victim, serialization failure, lock timeout (sometimes), connection reset.
- **Non-Retriable**: Constraint violation (logic error), syntax error, permission denied.
- **Exponential Backoff**: Space retries to reduce contention spikes.
- **Idempotency**: Retry safe if operation can be applied once or multiple times without divergence.
- **Detecting Partial Effects**: Use unique business keys & status flags.
- **Max Attempts**: Prevent infinite loop; escalate after threshold.
- **Circuit Breaker**: Temporarily halt retries if systemic issue detected.

### Retry Pseudocode
```
for attempt in 1..MAX:
  begin transaction
    perform work
  commit
  if success break
  if retriable_error then sleep(backoff(attempt)) else raise
```

### Postgres Serialization Failure
SQLSTATE '40001' → safe to retry entire transaction.

## Interview-Focused Notes
1. Why whole-transaction retry? Ensure atomicity; partial statement retry may break invariants.
2. Distinguish retriable vs logic errors? Use error codes classification layer.
3. Idempotency mechanism? Unique constraint or version check ensures safe reapply.
4. Backoff importance? Reduces thundering herd after conflict spike.
5. Max attempts rationale? Avoid unbounded loops, escalate for manual intervention.

## Quick Recall ✅
- Retry only safe classified errors.
- Full transaction scope repeat.
- Idempotent keys / versions protect duplication.
- Backoff & jitter reduce collision.
- Log attempts & final failure.

## Interview Traps & Confusions ⚠️
- Retrying constraint violations (wasteful).
- Retrying mid-transaction after partial commit (data corruption risk).
- Over-aggressive immediate retries amplifying contention.
- Ignoring dead letter handling after exhaustion.
- Failing to propagate original error context after final failure.

## Bonus
### Jitter Strategy
Randomized additional delay to avoid synchronized retry storms.

### Observability Fields
Record attempt_count, last_error_code, total_elapsed_ms.

### Adaptive Backoff
Increase delay when queue length / lock wait metrics high.

### Retry Budget
Cap total time spent per user request to maintain SLA.
