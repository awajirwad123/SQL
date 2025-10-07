# Access Paths & Index Usage

## Overview
Access path is how the database retrieves required rows: sequential scan, index seek/scan, index-only scan, bitmap scan, or specialized access (partition pruning). Optimizing access path selection reduces I/O and latency.

## Core Concepts
- **Sequential (Full) Scan**: Reads every page; best for large proportion of table or no usable index.
- **Index Seek**: Navigate tree to qualifying key range; ideal for high selectivity.
- **Index Scan**: Traverses large range or full index; can be cheaper than table scan if narrower.
- **Index-Only Scan**: All needed columns in index, avoiding table/heap fetch (visibility map gating in PG).
- **Bitmap Heap Scan (PG)**: Combines multiple index conditions; sets bit map then visits table pages once.
- **SARGability**: Search ARGument able—predicate form enabling index use (avoid function wrapping column, mismatched types, leading wildcard).
- **Composite Index Order**: Equality columns first, then range, then ordering columns for maximum utility.
- **Covering vs Narrow**: Balance storage/write overhead vs eliminating lookups.
- **Partition Pruning**: Limit scanned partitions; needs partition key predicate.

### SARGable Rewrite
Instead of:
```sql
WHERE LOWER(email) = 'a@x.com'
```
Use functional index or normalized stored value.

### Leading Wildcard Anti-Pattern
`WHERE name LIKE '%abc'` cannot use standard B-Tree prefix; consider reverse index or full-text.

## Interview-Focused Notes
1. Define SARGable: Predicate structured so index can be applied directly to filter range.
2. Why index-only scans faster? Avoids random I/O to base table.
3. When full scan better? Low selectivity (fetch large %), or small table fits in memory.
4. Multi-condition filtering? Use composite index or multiple indexes (bitmap OR combination if engine supports).
5. Partition pruning requirement? Constraining predicate on partition key (or expression enabling derivation).

## Quick Recall ✅
- Avoid function on column side.
- Normalize types (no implicit casts on indexed column).
- Rightmost LIKE wildcard OK: `prefix%`.
- Cover frequently executed hot queries only.
- Monitor index usage & eliminate redundancy.

## Interview Traps & Confusions ⚠️
- Believing every index scan is bad (may still be efficient).
- Adding columns to cover rarely executed query.
- Ignoring collation/case-sensitivity effects on index usage.
- Overlooking implicit casting (e.g., comparing text to int) causing seq scan.
- Expecting partition elimination without explicit predicate.

## Bonus
### Composite Index Utility Matrix
`(a,b,c)` supports: a; (a,b); (a,b,c) — NOT just b or (b,c).

### Partial Index (PG) as Selectivity Booster
Index only active subset for higher density & smaller structure.

### Bitmap Combination Example (PG)
Two separate single-column indexes combined for OR conditions.

### Covering with INCLUDE (SQL Server)
```sql
CREATE INDEX ix_orders_user_date ON orders(user_id, created_at) INCLUDE (status, total_amount);
```

### Detect Implicit Cast (PG)
Use `EXPLAIN` to spot `::` operator; fix schema or cast literal.
