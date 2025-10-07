# Numeric Functions

## Overview
Numeric functions handle arithmetic precision, rounding, statistical primitives, and transformations. Correct use matters for financial accuracy, analytics, and performance (especially with large aggregates).

## Core Concepts
- Types matter: INT vs BIGINT vs DECIMAL/NUMERIC vs FLOAT/DOUBLE.
- Rounding family: `ROUND()`, `TRUNC()`, `CEIL()/CEILING()`, `FLOOR()`. Bankers rounding differences across systems.
- Modulus & division: `MOD()`, `%` (SQL Server), integer division truncation.
- Absolute & sign: `ABS()`, `SIGN()`, `POWER()`, `SQRT()`, `EXP()`, `LN()/LOG()`.

**Examples:**
```sql
-- Force decimal division
SELECT 5 / 2;          -- May yield 2 (integer)
SELECT 5.0 / 2;        -- 2.5
SELECT CAST(5 AS DECIMAL(10,2)) / 2; -- 2.50

-- Rounding variants
SELECT ROUND(123.456, 2);    -- 123.46
SELECT TRUNC(123.456, 2);    -- 123.45 (Oracle / PG extension)
SELECT CEIL(123.01);         -- 124
SELECT FLOOR(123.99);        -- 123
```

**Precision Consideration:**
Use DECIMAL / NUMERIC for currency to avoid binary floating drift.

## Interview-Focused Notes
1. "Why not store currency in FLOAT?" – Binary floating imprecision leads to cumulative rounding errors.
2. "How to control scaling during aggregation?" – Cast early to DECIMAL with desired precision before SUM/AVG.
3. "Difference between ROUND and TRUNC?" – ROUND adjusts based on next digit; TRUNC simply chops.
4. "Handling division by zero?" – Use `NULLIF(denominator,0)` pattern.
5. "Detect negative zero?" – Avoid formatting anomalies by explicit rounding / sign correction.

## Quick Recall ✅
- Use DECIMAL for money; scale enforced.
- `NULLIF(x,0)` prevents divide errors: `metric / NULLIF(total,0)`.
- Avoid integer overflow: promote to BIGINT early.
- Pre-aggregate numeric transformations in subquery for readability.
- Beware implicit type precedence differences across dialects.

## Interview Traps & Confusions ⚠️
- Integer division returning truncated result unexpectedly.
- Relying on implicit casting of string numbers (slower / error-prone).
- Floating compare using equality (use tolerance window).
- Rounding twice (double rounding risk in financial calcs).

## Bonus
### Safe Percentage
```sql
SELECT 100.0 * num / NULLIF(den,0) AS pct
FROM metrics;
```

### Financial Rounding (Two Decimal Places)
```sql
ROUND(amount * rate, 2)
```

### Cumulative Sum (Window)
```sql
SUM(amount) OVER (PARTITION BY account ORDER BY txn_time ROWS UNBOUNDED PRECEDING)
```

### Z-Score (with window stats)
```sql
(value - AVG(value) OVER w) /
NULLIF(STDDEV_POP(value) OVER w, 0) AS zscore
WINDOW w AS (PARTITION BY category);
```

### Power / Log Transform
```sql
SELECT POWER(value, 3), LN(value+1) FROM measurements;
```
