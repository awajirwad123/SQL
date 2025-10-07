# JSON Functions

## Overview
JSON functions enable semi-structured data ingestion, querying, transformation, and indexing inside relational engines. Widely used for flexible attributes, event payloads, and integration with external APIs.

## Core Concepts
- Storage types: Text vs native JSON (`JSON`, `JSONB` in PostgreSQL, `NVARCHAR` + JSON in SQL Server, `JSON` in MySQL, `CLOB` + functions in Oracle pre-21c then native JSON data type).
- Accessors & navigation:
  - PostgreSQL: `->`, `->>`, `#>`, `#>>`
  - MySQL: `JSON_EXTRACT(doc,'$.a.b')`, `->` shorthand, `->>` unquoted
  - SQL Server: `JSON_VALUE()`, `JSON_QUERY()`
  - Oracle: `JSON_EXISTS`, `JSON_VALUE`, `JSON_TABLE`
- Indexing: GIN (PG JSONB), computed columns (SQL Server persisted), functional indexes on extracted path, MySQL generated columns.

**Examples:**
```sql
-- PostgreSQL extract
SELECT payload->'customer'->>'id' AS customer_id
FROM events
WHERE (payload->'meta'->>'type') = 'checkout';

-- MySQL extraction
SELECT JSON_EXTRACT(payload,'$.customer.id') AS customer_id
FROM events
WHERE JSON_EXTRACT(payload,'$.meta.type') = '"checkout"';

-- SQL Server
SELECT JSON_VALUE(payload,'$.customer.id') AS customer_id
FROM events
WHERE JSON_VALUE(payload,'$.meta.type') = 'checkout';
```

## Interview-Focused Notes
1. "Why JSONB over JSON (Postgres)?" – Binary representation, faster containment queries, indexing support; order of object fields not preserved.
2. "When not to use JSON?" – When attributes are query-critical and strongly structured; use proper columns and normalization.
3. "How to index a JSON path?" – Create generated/computed column with extracted value, then index.
4. "Difference JSON_VALUE vs JSON_QUERY (SQL Server)?" – VALUE returns scalar; QUERY returns object/array.
5. "Migration path from flexible to structured?" – Identify frequently accessed keys -> materialize columns -> backfill -> enforce constraints.

## Quick Recall ✅
- Use JSON for flexibility, not to avoid schema design entirely.
- Extract scalar vs object differs by function choice.
- Indexing essential for performance (generated columns / GIN / functional index).
- Validate JSON structure using `IS JSON` (Oracle) / `JSON_VALID()` (MySQL) / constraint checks.
- Avoid over-nesting—depth affects performance.

## Interview Traps & Confusions ⚠️
- Treating JSON storage as free—scans become CPU-bound parsing overhead.
- Using text operators on JSON expecting structural awareness.
- Forgetting quoting rules (`->>` returns text vs `->` JSON in Postgres).
- Large JSON updates rewriting entire value (PG JSONB immutability aspect).

## Bonus
### Functional Index (Postgres)
```sql
CREATE INDEX idx_event_customer ON events ((payload->'customer'->>'id'));
```

### Generated Column (MySQL)
```sql
ALTER TABLE events
ADD customer_id VARCHAR(50) GENERATED ALWAYS AS (JSON_UNQUOTE(JSON_EXTRACT(payload,'$.customer.id'))) STORED,
ADD INDEX (customer_id);
```

### Validate Shape
```sql
CHECK (jsonb_typeof(payload->'customer') = 'object'); -- PG
```

### Flatten JSON (Oracle JSON_TABLE)
```sql
SELECT jt.customer_id, jt.total
FROM orders o,
JSON_TABLE(o.payload, '$'
  COLUMNS (
    customer_id VARCHAR2(30) PATH '$.customer.id',
    total NUMBER PATH '$.summary.total'
  ) jt;
```

### Partial Containment (PG)
```sql
payload @> '{"meta": {"type": "checkout"}}'
```
