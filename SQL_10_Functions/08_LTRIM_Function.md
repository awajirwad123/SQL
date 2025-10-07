# LTRIM Function

## Overview
`LTRIM` removes leading (left-side) whitespace (and optionally specified characters in some dialects) from a string. It’s useful for cleaning imported data, normalizing user input, and preparing values for comparison.

## Core Concepts
- Default behavior: strip leading spaces; tab handling varies (generally preserved unless specified char list variant used).
- Multi-char removal: Oracle/MySQL allow second argument specifying characters to trim.
- Contrast with `TRIM()` (both sides) and `RTRIM()` (right side only).

**Syntax Examples:**
```sql
-- Basic
SELECT LTRIM('   test') = 'test';

-- Oracle / MySQL char set removal
SELECT LTRIM('xx-abc','x-') FROM dual;  -- yields 'abc'

-- ANSI TRIM equivalent for left-side
SELECT TRIM(LEADING '0' FROM '000123'); -- '123'
```

## Interview-Focused Notes
1. "Difference between LTRIM and TRIM?" – LTRIM only left; TRIM can specify side + chars.
2. "Why explicit trimming?" – Normalizes keys where leading spaces cause join mismatches.
3. "How to trim only spaces not tabs?" – Provide explicit character set or pre-replace.
4. "Does LTRIM handle Unicode spaces?" – Typically only ASCII space; advanced whitespace (NBSP) may persist—use regex replace if needed.
5. "Impact on indexes?" – Expressions on columns may prevent index usage unless functional index created.

## Quick Recall ✅
- Left-only removal.
- Use TRIM with LEADING for portability.
- Provide char list for non-space removal (dialect-dependent).
- Beware hidden non-breaking spaces.
- Combine with UPPER/LOWER for normalized comparisons.

## Interview Traps & Confusions ⚠️
- Assuming LTRIM removes all whitespace variants.
- Forgetting to standardize both insertion & querying sides.
- Double trimming in ETL (wasted CPU on large feeds).
- Using LTRIM where full TRIM required (incomplete cleanup).

## Bonus
### Functional Index (Postgres)
```sql
CREATE INDEX idx_clean_name ON customers (LTRIM(name));
```

### Regex Alternative (Postgres)
```sql
REGEXP_REPLACE(name, '^\s+', '')
```

### Chain Normalization
```sql
LOWER(TRIM(BOTH FROM name)) AS normalized_name
```

### Remove Zeros Left
```sql
SELECT LTRIM('000987','0'); -- '987'
```
