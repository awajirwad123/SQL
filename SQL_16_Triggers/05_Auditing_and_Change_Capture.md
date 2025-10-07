# Auditing & Change Capture

## Overview
Triggers are commonly used for auditing row changes or capturing deltas for downstream processing. While effective, alternative native features (temporal tables, CDC, logical decoding) may offer more scalable and less intrusive solutions.

## Core Concepts
- **Audit Table Pattern**: Insert row change details (old/new values, user, timestamp) into append-only log.
- **Minimal vs Full Audit**: Store only changed columns vs entire row snapshot.
- **CDC Alternatives**:
  - SQL Server CDC / Temporal Tables
  - Postgres logical decoding / WAL stream / pg_audit extension
  - MySQL binlog parsing / Debezium connectors
- **Data Volume Management**: Partition audit tables; archive/purge policy.
- **Write Amplification**: Each change writes extra rows; ensure indexing of audit table for retrieval patterns.
- **Security & Integrity**: Restrict direct modifications of audit tables.

### Example (Selective Audit – SQL Server)
```sql
CREATE OR ALTER TRIGGER trgOrdersStatusAudit ON dbo.Orders
AFTER UPDATE
AS
BEGIN
  SET NOCOUNT ON;
  INSERT INTO dbo.OrderStatusAudit(order_id, old_status, new_status, changed_by, changed_at)
  SELECT d.order_id, d.status, i.status, SESSION_USER, SYSDATETIME()
  FROM deleted d JOIN inserted i ON d.order_id = i.order_id
  WHERE d.status <> i.status;
END;
```

### Postgres Audit Function
```sql
CREATE TABLE orders_audit(
  order_id BIGINT,
  old_status TEXT,
  new_status TEXT,
  changed_by TEXT,
  changed_at TIMESTAMPTZ DEFAULT now()
);
```

## Interview-Focused Notes
1. Why use triggers for auditing? Immediate, synchronous capturing of changes.
2. Drawbacks vs CDC? Higher write latency, code maintenance, possible missed DDL events.
3. How reduce audit size? Log only changed columns or hash deltas.
4. Purge strategy? Partition by month & detach/archival.
5. Integrity protection? REVOKE DML; only INSERT via trigger.

## Quick Recall ✅
- Append-only audit.
- Only log necessary fields.
- Consider native CDC for large scale.
- Partition + purge regularly.
- Index retrieval columns (entity id, timestamp).

## Interview Traps & Confusions ⚠️
- Auditing every column blindly (bloat & overhead).
- Mixing audit & business logic operations in same trigger.
- Failing to secure audit table from tampering.
- Not monitoring growth leading to storage pressure.
- Relying on triggers for cross-database replication (use CDC/streaming instead).

## Bonus
### Differential Hash
Store hash(old_row) & hash(new_row) for rapid diff detection.

### Compression
Use columnstore or partition compression for older audit partitions.

### Streaming Bridge
Trigger inserts into lightweight queue table; separate process streams out.

### Minimal JSON Audit
Store compact JSON of changed fields to reduce schema churn.
