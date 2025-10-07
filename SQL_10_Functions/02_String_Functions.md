# String Functions

## Overview
String functions transform, search, slice, pad, compare, and format textual data. Inefficient string handling is a common performance and correctness pitfall in SQL interviews.

## Core Concepts
- Categories: Case conversion, trimming, substringing, pattern search, replacement, concatenation, splitting, length, hashing.
- Collation influences comparison & ordering; case sensitivity differs by DB (e.g., MySQL default collation case-insensitive).
- Deterministic vs locale-aware (e.g., Turkish `İ` case folding problems).

**Common Functions (Cross-Dialect):**
| Purpose | PostgreSQL | MySQL | SQL Server | Oracle |
|---------|-----------|-------|------------|--------|
| Length | `LENGTH()` / `CHAR_LENGTH()` | `CHAR_LENGTH()` | `LEN()` | `LENGTH()` |
| Upper  | `UPPER()` | `UPPER()` | `UPPER()` | `UPPER()` |
| Lower  | `LOWER()` | `LOWER()` | `LOWER()` | `LOWER()` |
| Substring | `SUBSTRING(str FROM start FOR len)` | `SUBSTRING(str, start, len)` | `SUBSTRING(str, start, len)` | `SUBSTR(str, start, len)` |
| Trim | `TRIM([LEADING|TRAILING] chars FROM str)` | `TRIM()` | `LTRIM()` / `RTRIM()` / `TRIM()` | `TRIM()` |
| Replace | `REPLACE(str, from, to)` | same | same | same |
| Position | `POSITION(substr IN str)` | `LOCATE(substr,str)` | `CHARINDEX(substr,str)` | `INSTR(str,substr)` |
| Concat | `str1 || str2` | `CONCAT()` | `+` or `CONCAT()` | `||` |

**Examples:**
```sql
-- Case-insensitive search (safe)
WHERE LOWER(email) = LOWER('Test@Example.com');

-- Avoid leading wildcard for index use
WHERE last_name LIKE 'Sm%';

-- Extract domain
SELECT SUBSTRING(email FROM POSITION('@' IN email)+1) AS domain FROM users; -- PG
```

## Interview-Focused Notes
1. "Why avoid `SELECT *` with string transformations?" – Increases network + compute; push projection.
2. "How to do case-insensitive match efficiently?" – Use functional index (e.g., `CREATE INDEX ON users (LOWER(email));`).
3. "Difference between `CHAR_LENGTH` and `OCTET_LENGTH`?" – Character count vs bytes (multi-byte UTF). Useful for storage vs display.
4. "How handle trimming variations?" – Use explicit variant: `LTRIM`, `RTRIM`, or `TRIM(BOTH ...)`.
5. "Prevent SQL injection via concatenation?" – Use bind parameters; string functions do not sanitize.

## Quick Recall ✅
- Normalize (upper/lower) for case-insensitive equality.
- Use functional/partial indexes for performance.
- LIKE leading `%` defeats B-Tree index (unless extension/trigram index).
- Concatenate with `||` (ANSI) not plus (avoid accidental numeric cast).
- Avoid repeated heavy functions inside large scans (precompute).

## Interview Traps & Confusions ⚠️
- Mistaking `LEN()` vs `LENGTH()` between dialects.
- Collation causing unexpected ordering (accent-insensitive sorts).
- Using `RTRIM()` to remove arbitrary characters (it only removes trailing spaces unless dialect-specific char list specified).
- Unbounded `REPLACE()` causing performance issues on huge text fields.

## Bonus
### Split Part (Postgres)
```sql
SELECT split_part(url, '/', 3) AS host FROM web_logs;
```

### Safe Email Normalization
```sql
LOWER(TRIM(email))
```

### Hash for Deduplication
```sql
SELECT md5(LOWER(TRIM(email))) AS email_fingerprint FROM users; -- PG
```

### Remove Non-Digits (Regex Postgres)
```sql
REGEXP_REPLACE(phone, '\\D', '', 'g')
```

### Reverse (Vendor Specific)
- SQL Server: `REVERSE(str)`
- PostgreSQL: extension or manual (`array_to_string(reverse(string_to_array(str,'')),'')`).
