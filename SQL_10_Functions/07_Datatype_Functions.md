# Datatype Functions

## Overview
Datatype functions interrogate, validate, or transform data types at runtime: checking types, coercing, inspecting metadata, and handling specialized domain types (arrays, UUID, geometry) depending on the RDBMS.

## Core Concepts
- Type inspection: `PG_TYPEOF(expr)` (Postgres), `SQL_VARIANT_PROPERTY()` (SQL Server variant), `DUMP()` (Oracle), `DATA_TYPE` from INFORMATION_SCHEMA.
- Null & type-safe comparisons: `ISNUMERIC()` (SQL Server, imperfect), regex alternative for robust validation.
- Specialized types: UUID/GUID generation, array length, type casting across domains.

**Examples:**
```sql
-- Postgres
SELECT PG_TYPEOF(payload), PG_TYPEOF(ARRAY[1,2,3]);
SELECT gen_random_uuid();  -- extension module

-- SQL Server
SELECT SQL_VARIANT_PROPERTY(CAST(123 AS sql_variant), 'BaseType');
SELECT NEWID();

-- Oracle
SELECT DUMP(123.45) FROM dual; -- Internal representation info

-- MySQL
SELECT JSON_TYPE(JSON_EXTRACT(payload,'$.x'));
```

## Interview-Focused Notes
1. "Why check type at runtime?" – Debugging ETL anomalies, verifying dynamic SQL outputs, validating JSON content.
2. "How to validate numeric string robustly?" – Use regex: `value ~ '^[0-9]+$'` (Postgres) or `value NOT LIKE '%[^0-9]%'` (SQL Server pattern inversion).
3. "UUID vs INT PK?" – UUID avoids coordination, global uniqueness; trade-offs: larger index, random btree fragmentation (mitigate with UUID v7 / sequential). INT smaller & faster.
4. "How to detect array length?" – `CARDINALITY(array_col)` (standard), `array_length()` (PG specific).
5. "Why prefer explicit domain types (e.g., `INET`) ?" – Built-in validation + specialized operators + indexing strategies.

## Quick Recall ✅
- `PG_TYPEOF` for introspection (dev time).
- Use modern UUID versions for index locality (v7 timestamp-ordered emerging).
- Regex beats loose numeric testers (`ISNUMERIC` accepts scientific notation & symbols).
- Domain-specific types add integrity (IP, geometry) vs storing as generic text.
- Validate before casting to reduce runtime errors.

## Interview Traps & Confusions ⚠️
- Assuming GUID randomness = secure (predictability of some versions).
- Using high-cardinality random UUID as clustered key (fragmentation).
- Misusing INFORMATION_SCHEMA performance-wise (system catalogs faster sometimes).
- Believing `ISNUMERIC` strict (it’s permissive: '12e3' true).

## Bonus
### Sequential-Friendly UUID (Postgres extension)
```sql
SELECT gen_random_uuid(); -- (PGCrypto) traditional v4
-- For better index locality consider UUID v7 (extension / future native support)
```

### Array Unnest + Count
```sql
SELECT user_id, COUNT(*) AS tag_count
FROM users CROSS JOIN LATERAL unnest(tags) AS tag
GROUP BY user_id;
```

### Domain Type Example
```sql
CREATE DOMAIN positive_int AS INT CHECK (VALUE > 0);
```

### INFORMATION_SCHEMA Type Lookup
```sql
SELECT column_name, data_type
FROM information_schema.columns
WHERE table_name = 'orders';
```

### Validate IPv4 (Simplistic)
```sql
value ~ '^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)(\\.(?!$)){4}$'
```
