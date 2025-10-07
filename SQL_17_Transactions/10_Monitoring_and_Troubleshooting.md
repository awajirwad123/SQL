# Monitoring & Troubleshooting Transactions

## Overview
Effective transaction performance and correctness depends on observability: duration, contention, abort reasons, and anomaly rates. Systematic monitoring enables proactive tuning and rapid incident resolution.

## Core Concepts
- **Metrics**: Average / p95 duration, abort rate, deadlock count, lock wait time, rows modified per transaction.
- **Instrumentation Sources**:
  - Postgres: `pg_stat_activity`, `pg_locks`, `pg_stat_database`, `pg_stat_statements`.
  - SQL Server: DMVs (`sys.dm_tran_locks`, `sys.dm_exec_requests`, Query Store).
  - MySQL: Performance Schema, INFORMATION_SCHEMA.INNODB_LOCK_WAITS.
- **Wait Event Analysis**: Distinguish lock vs I/O vs CPU saturation.
- **Long-Running Detection**: Identify sessions in transaction idle in transaction state.
- **Deadlock Logs**: Collect and parse for recurring patterns.
- **Anomaly Signals**: Serialization failure spikes, lock timeout counts.
- **Plan Regression**: Transaction slowdown often tied to plan change (stats update, index drop).
- **Auditing Tools**: Temporal tables / CDC reveal unexpected churn patterns.

### Checklist for Slow Transaction
1. Duration & start time.
2. Current blocking relationship (wait graph).
3. Query plan & row estimates.
4. Recent schema/index changes.
5. Isolation level & lock footprint.

## Interview-Focused Notes
1. Why measure p95 not just avg? Averages hide tail latency spikes affecting UX.
2. Idle-in-transaction risk? Holds locks/versions causing bloat & blocking.
3. Distinguish CPU vs lock issue? Analyze wait stats: high runnable queue vs lock waits.
4. Deadlock recurrence fix? Reorder access, add covering index, reduce transaction scope.
5. Serialization failure spike meaning? Contention on predicate conflicts—consider redesign or reduced overlapping work.

## Quick Recall ✅
- Monitor tail, not only mean.
- Idle transactions dangerous.
- Wait classification drives fix path.
- Deadlock logs actionable.
- Track plan hash for regression.

## Interview Traps & Confusions ⚠️
- Killing sessions without root cause analysis.
- Assuming high CPU always query inefficiency (may be spin waiting on lock).
- Ignoring vacuum/maintenance backlog effects.
- Overlooking isolation change side-effects.
- Treating each error individually (look for pattern aggregates).

## Bonus
### Alert Thresholds
- p95 > baseline * 2
- Deadlocks > X per hour
- Idle in transaction > 60s

### Visualization
Dashboard: transaction rate, abort %, top blocking chains.

### Blocker Script
Recursively trace blocking session tree for remediation.

### Regression Detection
Plan hash change + latency spike triggers automated rollback / investigation.
