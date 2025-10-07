# SQL Transactions Module (SQL_17_Transactions)

## Contents
1. `01_Transactions_Overview.md` – Fundamentals & lifecycle.
2. `02_ACID_Properties.md` – Atomicity, Consistency, Isolation, Durability mechanisms.
3. `03_Isolation_Levels.md` – ANSI levels, snapshot vs serializable nuances.
4. `04_Concurrency_Phenomena.md` – Dirty read, phantom, write skew, lost update.
5. `05_Transaction_Control_and_Savepoints.md` – BEGIN/COMMIT/ROLLBACK, savepoints.
6. `06_Locking_and_Deadlocks.md` – Locks, deadlocks, mitigation.
7. `07_Distributed_and_LongRunning.md` – 2PC, sagas, outbox.
8. `08_Retry_and_Error_Patterns.md` – Retriable vs non-retriable, backoff.
9. `09_Idempotency_and_ExactlyOnce.md` – Idempotent design, dedupe.
10. `10_Monitoring_and_Troubleshooting.md` – Metrics, diagnostics, regression detection.
11. `11_AntiPatterns_and_Best_Practices.md` – Pitfalls matrix & operational hygiene.

## Decision Heuristics
| Question | Guidance |
|----------|----------|
| Isolation needed? | Start Read Committed; escalate per anomaly |
| Retry on failure? | Only on known transient codes |
| Prevent lost update? | Version column + checked update or row lock |
| Large workflow? | Saga / compensation, not long DB txn |
| Distributed atomicity? | 2PC only if strict consistency > availability |

## Red Flags
- Long-lived idle transactions
- Repeated serialization / deadlock spikes
- Manual uniqueness checks
- Serializable set globally
- External API inside transaction

## Performance Checklist
1. Avg / p95 transaction duration
2. Deadlocks per hour
3. Abort rate (by code)
4. Lock wait distribution
5. Plan hash changes for top write queries

## Reliability Checklist
- Idempotency keys implemented
- Retry policy documented
- Savepoint usage justified
- Conflict hotspots profiled
- Saga compensation paths tested

## Practice Prompts
1. Diagnose lost update scenario & propose pessimistic + optimistic solutions.
2. Implement retry wrapper for serialization failures with exponential backoff.
3. Design idempotent order creation API with dedupe key.
4. Migrate a long-running transaction to saga pattern.
5. Evaluate isolation trade-offs for report vs OLTP mix.

## Integration With Other Modules
- Locking & Concurrency Performance (Performance Tuning module)
- Stored Procedures (transaction demarcation patterns)
- Triggers (avoid long logic inside transactions)
- Indexes (narrow lock footprint via selectivity)

## Next Suggested Modules
- Partitioning & Data Lifecycle Management
- Advanced Window Function Optimization
- Data Security (RLS, Encryption, Masking)
- Change Data Capture & Streaming

Use this module to reason about correctness + concurrency choices in high scale systems.
