# Frames and Frame Types

## Overview
A frame defines the subset of an ordered partition considered for a window aggregate relative to the current row. Frame type and boundaries critically affect semantics of running and moving calculations.

## Frame Types
- **ROWS**: Physical row offset boundaries.
- **RANGE**: Logical value-based boundaries relative to ORDER BY value (treats peers with same ordering value as a group).
- **GROUPS**: Peer group count boundaries (ANSI SQL:2012) – each distinct ORDER BY value is a group.

## Boundary Syntax
- `UNBOUNDED PRECEDING / FOLLOWING`
- `<n> PRECEDING / FOLLOWING`
- `CURRENT ROW`
- Value-based for RANGE: `RANGE BETWEEN INTERVAL '7 days' PRECEDING AND CURRENT ROW`

### Examples
```sql
-- Running total (deterministic)
SUM(val) OVER (ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- 7-row moving sum
SUM(val) OVER (ORDER BY ts ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)

-- 7-day moving sum (value-based time continuity)
SUM(val) OVER (ORDER BY ts RANGE BETWEEN INTERVAL '6 day' PRECEDING AND CURRENT ROW)

-- Previous + current + next peer group
COUNT(*) OVER (ORDER BY priority GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING)
```

## Choosing Frame Type
| Use Case | Frame |
|----------|-------|
| Deterministic running total | ROWS UNBOUNDED PRECEDING ... |
| Sliding fixed N rows | ROWS n PRECEDING |
| Time-based window | RANGE INTERVAL PRECEDING |
| Peer-aware expansions | GROUPS |
| Avoid peer collapse | ROWS with unique tie-breaker |

## Interview-Focused Notes
1. ROWS vs RANGE difference? ROWS counts physical rows; RANGE groups peers by order value.
2. Why GROUPS? Control peer grouping explicitly (ties = one group) w/ group count boundaries.
3. Default frame pitfalls? Might be RANGE UNBOUNDED PRECEDING CURRENT ROW leading to peer lumps.
4. Moving vs running? Bounded vs unbounded preceding boundary.
5. Time-based frames require? ORDER BY date/time and RANGE with INTERVAL (engine support varies; MySQL lacks true RANGE INTERVAL for some versions).

## Quick Recall ✅
- Frame only applies with ORDER BY.
- Use ROWS for deterministic incremental calculations.
- RANGE sensitive to duplicates.
- GROUPS expresses peer counting.
- Define frame explicitly to avoid ambiguity.

## Interview Traps & Confusions ⚠️
- Using RANGE expecting row-limited sliding window.
- Omitting ORDER BY expecting frame usage (invalid).
- Forgetting tie-breaker causing non-deterministic ROWS peers ordering.
- Performance: wide frames = large memory for partial sorts.
- Misinterpreting CURRENT ROW in RANGE (includes all peers).

## Bonus
### Unique Tie-Breaker Strategy
Append primary key to ORDER BY to disambiguate ROWS frame boundaries.

### Frame Efficiency
Index on ORDER BY columns reduces sort cost for large windows.

### Mixed Frame Use
Multiple window expressions each with distinct frames for snapshot vs moving metrics.

### Derive Rolling Distinct (Advanced)
Requires advanced features (window functions + conditional count of first occurrences) or specialized extensions.
