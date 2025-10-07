# Date Functions

## Overview
Date and time functions manipulate temporal values for extraction, arithmetic, formatting, truncation, comparison, timezone handling, and period generation. Mastery is critical for reporting, cohort analysis, retention metrics, and scheduling logic.

## Core Concepts
- Categories:
  - Extraction: `YEAR()`, `MONTH()`, `DAY()`, `DATE_PART()`
  - Truncation/Flooring: `DATE_TRUNC()`, `TRUNC(date, 'MM')`, `DATETRUNC()`
  - Arithmetic: `DATEADD()`, `INTERVAL` arithmetic (`date + INTERVAL '7 days'`)
  - Difference: `DATEDIFF()`, `AGE()` (PostgreSQL)
  - Formatting: `TO_CHAR()`, `FORMAT()` (SQL Server), `DATE_FORMAT()` (MySQL)
  - Current temporal: `CURRENT_DATE`, `CURRENT_TIMESTAMP`, `NOW()`, `SYSDATE` (Oracle nuance)
  - Time zone: `AT TIME ZONE`, `CONVERT_TZ()` (MySQL)
  - Rounding/Bucketing: Week starts, month boundaries, quarter grouping.
- Determinism: Some functions (e.g., `NOW()`) stable per statement (Postgres), others evaluated per row (MySQL depends on context).
- Time Zone vs UTC: Store in UTC, convert at presentation.

**Cross-Dialect Examples:**
```sql
-- Extract year
SELECT EXTRACT(YEAR FROM order_date) AS order_year FROM orders;          -- ANSI / PG
SELECT YEAR(order_date) AS order_year FROM orders;                        -- MySQL / SQL Server

-- Add 7 days
SELECT order_date + INTERVAL '7 days' FROM orders;                        -- ANSI / PG
SELECT DATEADD(day, 7, order_date) FROM orders;                           -- SQL Server
SELECT DATE_ADD(order_date, INTERVAL 7 DAY) FROM orders;                  -- MySQL

-- Truncate to month
SELECT DATE_TRUNC('month', order_date) FROM orders;                       -- Postgres
SELECT TRUNC(order_date, 'MM') FROM orders;                               -- Oracle
SELECT DATEFROMPARTS(YEAR(order_date), MONTH(order_date), 1) FROM orders; -- SQL Server (construct)

-- Difference in days
SELECT DATEDIFF(day, order_date, shipped_date) FROM orders;               -- SQL Server
SELECT DATEDIFF(shipped_date, order_date) FROM orders;                    -- MySQL (arg order differs!)
SELECT shipped_date - order_date AS day_diff FROM orders;                 -- Postgres interval cast
```

## Interview-Focused Notes
1. "How do you bucket by month?" – `DATE_TRUNC('month', ts)` (PG) or `TRUNC(dt, 'MM')` (Oracle) or construct first-of-month.
2. "How to get last day of month?" – `DATE_TRUNC('month', dt) + INTERVAL '1 month - 1 day'` (PG) or `LAST_DAY(dt)` (MySQL/Oracle).
3. "Why store UTC?" – Avoid ambiguous DST transitions; convert at edge.
4. "Difference between `NOW()` and `CURRENT_TIMESTAMP`?" – Usually synonyms; `CLOCK_TIMESTAMP()` (PG) gives wall-clock per call.
5. "Week start alignment?" – Use `DATE_TRUNC('week', dt)` (PG default Monday) or adjust via arithmetic for Sunday-based systems.

## Quick Recall ✅
- Use extraction for grouping indexes (materialize cautiously).
- `DATE_TRUNC` for period rollups.
- `AGE()` expresses interval difference in calendar semantics (PG only).
- Cross-dialect `DATEDIFF` argument order differs.
- Prefer UTC storage, apply TZ late.

## Interview Traps & Confusions ⚠️
- Confusing month difference via subtracting dates (gives days not months).
- Misusing `BETWEEN` for end-exclusive ranges (include full day end boundary with `< next_day`).
- DST hour loss/gain causing 23/25 hour day metrics.
- Using implicit casting of strings to dates (locale dependent / non-deterministic).

## Bonus
### Safe Date Range Filter (Half-Open Interval)
```sql
WHERE event_time >= '2025-10-01'::timestamp
  AND event_time <  '2025-11-01'::timestamp;
```

### Cohort Month Key
```sql
SELECT to_char(signup_date, 'YYYY-MM') AS cohort_month FROM users; -- PG
```

### Generate Series of Dates (PostgreSQL)
```sql
SELECT d::date
FROM generate_series('2025-01-01', '2025-01-07', INTERVAL '1 day') d;
```

### Index Consideration
Avoid wrapping column in function in predicate (`WHERE DATE(order_date)=...`)—rewrite as range to keep index use.

### Week Start Adjustment (Sunday)
```sql
SELECT DATE_TRUNC('week', order_date + INTERVAL '1 day') - INTERVAL '1 day'
FROM orders;
```
