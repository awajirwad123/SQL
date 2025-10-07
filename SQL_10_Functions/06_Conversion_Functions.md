# Conversion Functions

## Overview
Conversion functions change data types: string to numeric, date parsing, casting between numerics, binary ↔ text, and explicit shaping for safe comparison or storage. Proper usage prevents silent truncation and runtime errors.

## Core Concepts
- Explicit vs implicit conversion: prefer explicit for reliability & index usage.
- Core constructs: `CAST(expr AS TYPE)` (ANSI), vendor-specific shortcuts (e.g., `::type` in Postgres, `CONVERT(type, expr)` in SQL Server, `TO_NUMBER`, `TO_CHAR`, `TO_DATE` in Oracle).
- Formatting model functions (Oracle) vs pattern specifiers (Postgres `TO_CHAR`).
- Safe parsing: handle invalid input via `TRY_CONVERT` (SQL Server), `TRY_CAST`, `SAFE_CAST` (BigQuery), or regex pre-validation.

**Examples:**
```sql
-- Generic cast
SELECT CAST('123' AS INT);              -- All dialects (Oracle INTEGER alias)
SELECT '2025-10-01'::date;              -- PostgreSQL shorthand

-- SQL Server
SELECT CONVERT(DATE, '2025-10-01');
SELECT TRY_CONVERT(INT, '12x');         -- NULL (safe)

-- Oracle
SELECT TO_DATE('2025-10-01','YYYY-MM-DD') FROM dual;
SELECT TO_CHAR(SYSDATE,'YYYY-MM');

-- Postgres formatting
SELECT TO_CHAR(NOW(),'YYYY-Q');
```

## Interview-Focused Notes
1. "When can implicit casts hurt performance?" – Wrapping indexed column in cast inside predicate prevents index seek.
2. "Difference CAST vs CONVERT?" – CAST = ANSI standard; CONVERT adds style/format codes (SQL Server).
3. "How to handle invalid numeric strings?" – Use TRY_* variants or filter via regex before casting.
4. "Why convert early in ETL?" – Enforces schema consistency, prevents downstream type ambiguity.
5. "Date parsing pitfalls?" – Locale/format mismatches; always specify format mask where needed (Oracle, `TO_DATE`).

## Quick Recall ✅
- Use explicit CAST for clarity.
- Use safe conversion functions in ingestion pipelines.
- Avoid function on column side in WHERE for index usage.
- Convert to DECIMAL before aggregation for money.
- Use consistent timezone normalization before casting to date (truncate after UTC conversion).

## Interview Traps & Confusions ⚠️
- Silent truncation of longer VARCHAR into smaller target.
- Relying on ambiguous date format `MM/DD/YYYY` vs `DD/MM/YYYY`.
- Big int to int overflow not throwing error in some dialects (wrap-around or error depending on settings).
- Hex/binary interpretation differences.

## Bonus
### Safe Division
```sql
CAST(numerator AS DECIMAL(18,4)) / NULLIF(denominator,0)
```

### Currency Normalization
```sql
CAST(REPLACE(amount_text, ',', '') AS DECIMAL(12,2)) AS amount
```

### ISO Week-Year Extract (Then Cast)
```sql
CAST(EXTRACT(ISOWEEK FROM order_date) AS INT) AS iso_week
```

### JSON Scalar to Native
```sql
CAST(payload->>'count' AS INT) AS count_value  -- Postgres
```

### Try Casting Table (SQL Server)
```sql
SELECT value, TRY_CAST(value AS INT) AS as_int
FROM raw_values;
```
