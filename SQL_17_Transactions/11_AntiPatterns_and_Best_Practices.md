# Anti-Patterns & Best Practices (Transactions)

## Overview
Proper transactional design balances consistency and performance. Recognizing anti-patterns prevents contention, anomalies, and operational pain.

## Anti-Patterns
| Anti-Pattern | Issue | Better Approach |
|--------------|-------|-----------------|
| Long user think-time inside transaction | Locks/version retention | Do work first, commit, then present result |
| Catch & ignore errors (no rollback) | Silent corruption risk | Always rollback on failure |
| Over-wide transaction scope | Increased contention & deadlock risk | Narrow to minimal operations |
| Unnecessary Serializable isolation | Abort overhead, reduced throughput | Use lower level + targeted locks |
| Manual select-then-insert race | Lost uniqueness | Use INSERT ... ON CONFLICT / MERGE |
| Holding transaction for external API call | Unpredictable latency | Commit before external call; saga pattern |
| Retrying non-retriable constraint errors | Waste & possible spam | Surface to user; no loop |
| No idempotency on retry | Double application | Idempotency keys / version columns |
| Lack of monitoring on abort reasons | Hidden hotspots | Track error codes & wait profiles |
| Disabling autocommit globally | Hidden multi-statement transactions | Explicit demarcation only |

## Best Practices
1. Keep transactions short & set-based.
2. Use proper isolation level; escalate only when needed.
3. Employ optimistic concurrency (version column) where conflicts rare.
4. Enforce uniqueness with constraints not manual checks.
5. Implement full transaction retry wrapper for retriable errors.
6. Log abort reasons and classify.
7. Idempotency for external-facing operations.
8. Use savepoints sparingly for recoverable sub-steps.
9. Avoid mixing unrelated operations.
10. Monitor tail latency & deadlocks continuously.

## Interview-Focused Notes
1. Why minimize scope? Reduce locking & chance of contention.
2. Optimistic vs pessimistic? Optimistic retries on conflict; pessimistic locks early.
3. Unique constraint advantage? Atomic enforcement vs race-prone manual logic.
4. Isolation escalation trade-off? Consistency vs throughput; choose case-by-case.
5. Idempotency importance? Safe retries in distributed & failure scenarios.

## Quick Recall ✅
- Short, focused, monitored.
- Constraints > manual logic.
- Retry only safe errors.
- Idempotent external operations.
- Pick isolation pragmatically.

## Interview Traps & Confusions ⚠️
- Confusing snapshot with serializable.
- Using serializable globally.
- Ignoring partial failure semantics.
- Overusing savepoints for trivial steps.
- Treating autocommit off as safe default (hidden open txns).

## Bonus
### Transaction Budget
Define max expected duration (e.g., <200ms OLTP) & alert on breaches.

### Abort Classification Table
(error_code, count, last_seen_at, mitigation) for proactive tuning.

### Conflict Hotspot Heatmap
Track row/table frequently in deadlocks/serialization errors.

### Pre-Commit Validation Hook
Check invariants cheaply just before commit for early fail detection.
