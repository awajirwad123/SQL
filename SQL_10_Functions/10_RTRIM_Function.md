# RTRIM Function

## Overview
`RTRIM` removes trailing (right-side) spaces (and, in some dialects, specified characters) from a string. It complements `LTRIM` and is part of full normalization steps for textual data.

## Core Concepts
- Removes only trailing whitespace by default.
- Extended variant accepts second argument for character set removal (Oracle/MySQL).
- For both sides, prefer `TRIM()`; for specific side + chars use `TRIM(TRAILING ...)` (ANSI form).

**Examples:**
```sql
SELECT RTRIM('test   ') = 'test';
SELECT RTRIM('abcxx','x') = 'abc';      -- Oracle / MySQL char list

-- ANSI trailing removal
SELECT TRIM(TRAILING '0' FROM '1234000'); -- '1234'
```

## Interview-Focused Notes
1. "Difference RTRIM vs TRIM?" – RTRIM right-only; TRIM configurable (both/specific side, chars).
2. "When necessary?" – Cleaning imported fixed-width files / export artifacts with padded spaces.
3. "Risk in comparisons?" – Trailing spaces can cause false negatives unless using collations that ignore them (varies).
4. "Indexing strategy?" – Functional index on trimmed value if queries consistently apply RTRIM.
5. "Alternative for complex whitespace?" – Regex replace for multi-type trailing characters.

## Quick Recall ✅
- Right-side whitespace removal.
- Use ANSI TRIM for portability.
- Combine with LTRIM for full trim or just use TRIM().
- Hidden non-breaking spaces may persist.
- Normalize before storing or at query edge—pick one.

## Interview Traps & Confusions ⚠️
- Assuming removal of tabs/newlines (depends—often only space char 0x20).
- Over-trimming leading to key mismatch (e.g., significant trailing characters in codes).
- Treating fixed-length CHAR semantics same as VARCHAR (CHAR padded comparison sometimes ignores trailing spaces automatically).

## Bonus
### Full Normalization Pipeline
```sql
LOWER(TRIM(BOTH FROM col))
```

### Functional Index
```sql
CREATE INDEX idx_rtrim_code ON items (RTRIM(code));
```

### Regex Remove Trailing Dots
```sql
REGEXP_REPLACE(filename, '\\.+$','')
```

### Combine With Replacement
```sql
REPLACE(RTRIM(REPLACE(col, '\t', ' ')), '  ', ' ')
```
