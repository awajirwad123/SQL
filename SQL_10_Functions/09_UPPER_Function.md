# UPPER Function

## Overview
`UPPER` converts alphabetic characters in a string to uppercase according to the database’s collation and locale rules. Commonly used for normalization in case-insensitive comparisons and indexing strategies.

## Core Concepts
- Case folding may be locale-sensitive (Turkish dotted/dotless I issue).
- Often paired with `LOWER` depending on normalization strategy; choose one consistently.
- For performance, create a functional index on transformed value instead of applying UPPER in predicate repeatedly.
- Non-alphabetic characters unaffected.

**Examples:**
```sql
SELECT UPPER('Hello World') = 'HELLO WORLD';
SELECT * FROM users WHERE UPPER(username) = UPPER(:input_username);
```

**Functional Index (Postgres):**
```sql
CREATE INDEX idx_users_username_upper ON users (UPPER(username));
-- Then
SELECT * FROM users WHERE UPPER(username) = UPPER('admin');
```

## Interview-Focused Notes
1. "Why normalize case?" – Ensures deterministic matching independent of user input variety.
2. "Why index the expression?" – Without it, full scan applies UPPER row-by-row.
3. "Locale issue example?" – Turkish 'i' / 'I' mapping; sometimes store canonical ASCII or use CITEXT type (Postgres) instead.
4. "UPPER vs COLLATION-based case-insensitive index?" – Native case-insensitive collations may outperform expression-based.
5. "Security angle?" – Prevents login enumeration differences from case variance.

## Quick Recall ✅
- Converts letters only.
- Expression index aids performance.
- Locale matters for edge cases.
- Use consistent normalization direction (UPPER or LOWER, not both).
- Consider CITEXT or case-insensitive collations where available.

## Interview Traps & Confusions ⚠️
- Double-normalizing (wasted compute).
- Ignoring input trimming before UPPER.
- Assuming case-insensitive collation exists with same semantics across DB engines.
- Expecting UPPER to transliterate accented chars (it only uppercases). 

## Bonus
### Combined Normalization
```sql
LOWER(TRIM(email)) AS email_key
```

### Case-Insensitive Unique (Postgres)
```sql
CREATE UNIQUE INDEX uq_users_email_ci ON users (LOWER(email));
```

### CITEXT Alternative (Postgres)
```sql
CREATE EXTENSION IF NOT EXISTS citext;
ALTER TABLE users ALTER COLUMN email TYPE citext;
```

### Conditional Comparison
```sql
WHERE UPPER(col) = 'CRITICAL'  -- Use domain check instead ideally
```
