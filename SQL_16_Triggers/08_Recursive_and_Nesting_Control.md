# Recursive & Nesting Control

## Overview
Trigger recursion and nested trigger chains can lead to unexpected loops, exponential work, or stack-like effects on system resources. Proper guards and architectural discipline prevent runaway behavior.

## Core Concepts
- **Direct Recursion**: Trigger on table modifies same table, firing itself again.
- **Indirect Recursion**: Table A trigger writes Table B; Table B trigger writes Table A.
- **Nesting Depth Limits**: SQL Server has nesting limit (32 by default). Others rely on stack depth or execution guard.
- **Session / Context Flags**: Use session variables / context info to prevent re-entry.
- **Idempotent Updates**: Only apply changes when state truly differs to avoid unnecessary updates that re-fire triggers.
- **Bulk vs Incremental**: Batch modifications outside triggers to reduce nested invocation risk.

### Guard Pattern (SQL Server)
```sql
IF TRIGGER_NESTLEVEL() > 1 RETURN; -- simple guard
```

### Postgres GUC Flag Pattern
```sql
PERFORM set_config('app.skip_triggers','on',true); -- inside controlled session
```

### Conditional Update
```sql
UPDATE accounts SET status='ACTIVE' WHERE id=@id AND status <> 'ACTIVE';
```
Avoids pointless update firing triggers.

## Interview-Focused Notes
1. What causes trigger recursion? Modifying same table or circular dependencies among triggers.
2. Mitigation strategies? Nest level checks, context flags, idempotent DML.
3. Why harmful? Performance blow-up, deadlocks, logical anomalies.
4. Detecting recursion? Monitor repeated stack of same trigger; metadata table log.
5. Alternative design? Move multi-step cascading logic to stored procedure with explicit sequencing.

## Quick Recall ✅
- Prevent re-entry with guard.
- Idempotent state change to avoid redundant fire.
- Limit cross-table cascades.
- Centralize complex workflow outside triggers.
- Monitor nesting metrics (DMVs / logs).

## Interview Traps & Confusions ⚠️
- Disabling recursion globally harming legitimate cases.
- Blindly updating row with same values.
- Hidden recursion via view triggers referencing base table triggers.
- Forgetting to re-enable recursion flag after maintenance.
- Assuming engine automatically blocks infinite loops (some only error after depth limit).

## Bonus
### Diagnostic Logging
Write first occurrence per session of high nesting depth to troubleshooting table.

### Safe Batch Script
Disable non-critical triggers; run batch updates; re-enable; run validation query to detect anomalies.

### Hash of Row State
Store checksum to detect actual change vs redundant update.

### Static Analysis
Script to inspect trigger bodies for cross-table modification cycles.
