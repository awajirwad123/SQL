# 33. Transaction Isolation Levels

Q: Goal?
A: Balance consistency vs concurrency by controlling visibility of uncommitted/phantom changes.

Phenomena:
- Dirty Read (see uncommitted)
- Non-Repeatable Read (row changes between reads)
- Phantom Read (new/missing rows matching predicate)
- Serialization Anomaly (inconsistent aggregate outcome)

Standard levels (ANSI):
| Level | Prevents |
|-------|----------|
| READ UNCOMMITTED | (Allows all; no dirty read prevention) |
| READ COMMITTED | Dirty reads |
| REPEATABLE READ | + Non-repeatable reads |
| SERIALIZABLE | + Phantom reads (full serial equivalence) |

Notes:
- In PostgreSQL, READ COMMITTED & REPEATABLE READ use MVCC; SERIALIZABLE adds predicate conflict checks.
- MySQL InnoDB REPEATABLE READ also prevents phantom via next-key locks (different semantics from ANSI naming).

Trade-offs: Higher isolation = more locking / aborts, lower concurrency.

Pitfall: Assuming naming is identical across engines—always confirm vendor specifics.
