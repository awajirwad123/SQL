# 46. Transactions, Commits & Rollbacks

Q: What is a transaction?
A: Atomic unit of work ensuring ACID (Atomicity, Consistency, Isolation, Durability).

Q: Basic pattern?
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT; -- or ROLLBACK on error
```

Q: Partial failure handling?
A: Use SAVEPOINT for granular rollback.
```sql
SAVEPOINT before_bonus;
UPDATE employees SET bonus = bonus * 1.1 WHERE dept = 'ENG';
ROLLBACK TO SAVEPOINT before_bonus; -- revert just that change
```

Q: Autocommit mode?
A: Each statement commits automatically unless explicit transaction started.

Q: Why rollback?
A: Undo uncommitted changes after errors or validation failure.

Q: Pitfall?
A: Long-running open transactions → bloat (MVCC) and lock contention.
