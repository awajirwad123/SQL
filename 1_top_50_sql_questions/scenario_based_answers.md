# Scenario-Based & Practical SQL Answers

This document consolidates detailed, interview-ready walkthroughs for common real-world SQL problem scenarios.

---
## 1. Troubleshooting an Abnormally Slow-Running Query

### Mindset & Goal
Identify the true bottleneck (I/O, CPU, memory spill, locking, or bad plan) and apply the smallest effective change while preserving correctness.

### 5 Phases
1. Reproduce & Baseline
   - Capture exact SQL + parameter values, runtime, rows returned.
   - Note recent deploys, data growth, workload spikes.
   - Run `EXPLAIN` / `EXPLAIN ANALYZE` (or equivalents) to see plan shape + estimates vs actuals.
2. Inspect Execution Plan
   - Red flags: large full scans, wrong join order, huge estimate vs actual mismatch, sorts/hashes spilling, non-sargable predicates.
3. Isolate Root Causes
   | Symptom | Diagnostic | Common Fix |
   |---------|-----------|------------|
   | Index not used | Predicate uses function / type mismatch | Rewrite predicate; create composite index |
   | Bad cardinality | Estimates far off | Refresh statistics / extended stats |
   | Join explosion | Row counts balloon mid-plan | Pre-aggregate; reorder joins; filter earlier |
   | Parameter sniffing | Good plan for typical, bad for skew | Recompile hint / plan guide / parameterization strategy |
   | Lock waits | Lock DMVs / blocking tree | Reduce txn scope; add index to shorten scan |
4. Remediate Iteratively
   - Make predicate sargable: `DATE(col)=...` → range bounds.
   - Add covering index only if proven hot path.
   - Replace correlated subqueries with set-based joins/windows.
   - Limit returned columns (avoid `SELECT *`).
   - Verify each change re-running with warm cache if relevant.
5. Prevent Recurrence
   - Enable slow query logging or `pg_stat_statements` / Query Store.
   - Baseline key query latencies.
   - Document index intent.

### Typical Rewrite Example
```sql
-- Before (non-sargable)
SELECT * FROM orders WHERE DATE(created_at) = '2025-10-01';
-- After
SELECT col_list FROM orders
WHERE created_at >= '2025-10-01' AND created_at < '2025-10-02';
```

### Pitfalls
- Over-indexing to chase single query performance.
- Ignoring contention (wait events) and blaming plan.
- Measuring cold vs warm cache inconsistently.

---
## 2. Safely Migrating a Large Table

### Scenario
Table `orders` (hundreds of millions of rows) requires schema evolution: add NOT NULL column + widen numeric precision.

### 6-Step Plan
1. Assess & Plan
   - Size, indexes, dependencies (FKs, views, triggers, replication).
   - Downtime tolerance → choose shadow table vs in-place phased.
2. Prepare Target Structure
```sql
CREATE TABLE orders_new (
  order_id BIGINT PRIMARY KEY,
  customer_id BIGINT NOT NULL,
  order_timestamp TIMESTAMP NOT NULL,
  total_amount NUMERIC(14,2) NOT NULL,
  channel VARCHAR(20) NOT NULL DEFAULT 'WEB'
);
CREATE INDEX idx_orders_new_customer_time ON orders_new(customer_id, order_timestamp);
```
3. Bulk Backfill (Chunked)
```sql
INSERT INTO orders_new (...columns...)
SELECT ... FROM orders
WHERE order_id > :last_id
ORDER BY order_id
LIMIT :batch_size; -- loop
```
4. Delta Sync
   - Use CDC, triggers, or `updated_at` watermark to copy changes since initial copy.
5. Switchover
```sql
BEGIN; -- short lock window
ALTER TABLE orders RENAME TO orders_old;
ALTER TABLE orders_new RENAME TO orders;
COMMIT;
```
   - Rebuild constraints, grants, dependent objects if needed.
6. Validate & Cleanup
   - Row counts, checksums, sample spot checks.
   - Retain `orders_old` for rollback window, then drop.

### Optimizations
- Partition swap if partitioned.
- Disable non-essential secondary indexes during load; build after.
- Use parallel copy (multiple disjoint id ranges) respecting I/O capacity.

### Pitfalls
- One giant transaction exhausting logs.
- No index on new FK columns causing post-migration slowdowns.
- Missing replication or ETL re-point.

---
## 3. Splitting a Delimited String Into Rows

### Goal
Convert `tags = 'red,blue,green'` into row set.

### Engine-Specific Patterns
PostgreSQL:
```sql
SELECT id, unnest(string_to_array(tags, ',')) AS tag
FROM products;
```
SQL Server:
```sql
SELECT id, value AS tag
FROM products CROSS APPLY STRING_SPLIT(tags, ',');
```
MySQL (8+ example using JSON trick):
```sql
SELECT id, TRIM(JSON_UNQUOTE(j.value)) AS tag
FROM products p
JOIN JSON_TABLE(CONCAT('["', REPLACE(p.tags, ',', '","'), '"]'), '$[*]' COLUMNS(value JSON PATH '$')) j;
```
Oracle:
```sql
SELECT id, REGEXP_SUBSTR(tags, '[^,]+', 1, LEVEL) AS tag
FROM products
CONNECT BY REGEXP_SUBSTR(tags, '[^,]+', 1, LEVEL) IS NOT NULL
  AND PRIOR id = id AND PRIOR SYS_GUID() IS NOT NULL;
```
Numbers/Recursive (portable concept): generate positional indexes then slice substrings.

### Edge Handling
- Trim spaces: `TRIM(tag)`.
- Remove empties: `WHERE tag <> ''`.

### Pitfalls
- Re-splitting on every query instead of normalizing to a child table.
- Regex overhead for very large strings.

---
## 4. Distinct Users Active Every Consecutive Day (Given Date Range)

### Simplest (Assuming One Record per User/Day & Continuous Window)
```sql
SELECT COUNT(*) AS users_full_streak
FROM (
  SELECT user_id
  FROM activity
  WHERE activity_date BETWEEN DATE '2025-09-01' AND DATE '2025-09-07'
  GROUP BY user_id
  HAVING COUNT(DISTINCT activity_date) = 7
) t;
```

### Robust (Check No Missing Day Explicitly - Postgres)
```sql
WITH days AS (
  SELECT generate_series('2025-09-01'::date,'2025-09-07'::date,'1 day') AS d
), user_days AS (
  SELECT DISTINCT user_id, activity_date
  FROM activity
  WHERE activity_date BETWEEN '2025-09-01' AND '2025-09-07'
), expected AS (
  SELECT u.user_id, d.d
  FROM (SELECT DISTINCT user_id FROM user_days) u
  CROSS JOIN days d
)
SELECT COUNT(DISTINCT e.user_id)
FROM expected e
LEFT JOIN user_days ud ON ud.user_id = e.user_id AND ud.activity_date = e.d
GROUP BY e.user_id
HAVING COUNT(ud.activity_date) = COUNT(*);
```

### Gap & Islands (Identify a Single Island Covering the Range)
```sql
WITH filtered AS (
  SELECT DISTINCT user_id, activity_date
  FROM activity
  WHERE activity_date BETWEEN '2025-09-01' AND '2025-09-07'
), grp AS (
  SELECT user_id, activity_date,
         activity_date - (ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY activity_date))*INTERVAL '1 day' AS grp_key
  FROM filtered
), islands AS (
  SELECT user_id, MIN(activity_date) AS start_d, MAX(activity_date) AS end_d, COUNT(*) AS span_len
  FROM grp
  GROUP BY user_id, grp_key
)
SELECT COUNT(*)
FROM islands
WHERE start_d='2025-09-01' AND end_d='2025-09-07' AND span_len=7;
```

### Pitfalls
- Duplicate rows inflate counts—use DISTINCT.
- Time zone drift (normalize to UTC day boundary).
- Holidays or closed days—use business calendar rather than raw date math.

---
## 5. Sampling N Random Rows

### 1. ORDER BY RANDOM() (Simple, Expensive)
```sql
SELECT * FROM events ORDER BY RANDOM() LIMIT 100; -- Full scan + sort
```
Pros: Exact, trivial. Cons: Unscalable for huge tables.

### 2. TABLESAMPLE (Approximate)
PostgreSQL/SQL Server:
```sql
SELECT * FROM events TABLESAMPLE SYSTEM (0.5); -- ~0.5% pages
```
Pros: Fast. Cons: Approximate row count; page-level bias.

### 3. Random ID Technique (Dense PK)
```sql
WITH bounds AS (SELECT MIN(id) AS lo, MAX(id) AS hi FROM events),
rand AS (
  SELECT (lo + (random()*(hi-lo)))::bigint AS candidate FROM bounds, generate_series(1,500)
)
SELECT e.* FROM events e JOIN rand r ON r.candidate = e.id LIMIT 100;
```
Pros: Avoids global sort. Cons: Gaps need oversampling.

### 4. Deterministic Hash Sample
```sql
SELECT *
FROM events
WHERE MOD(ABS(HASHTEXT(CAST(id AS text))), 1000) < 10
LIMIT 100; -- ~1% stable sample, adjust modulus
```
Pros: Repeatable subsets for A/B analysis. Cons: Not strictly uniform if hash biased or modulus poorly chosen.

### 5. Reservoir (Streaming / Procedural)
Implement outside pure SQL for very large streams; true O(n) single pass with fixed memory.

### Pitfalls
- Misusing ORDER BY RANDOM() on billions of rows.
- Expecting TABLESAMPLE to yield exact N.
- Sampling after restrictive WHERE (introduces bias if predicate already filtered distribution).

---
## Recap Cheat Lines
- Slow query: reproduce → plan → isolate (stats/index/sargability/locks) → minimal fix → baseline tracking.
- Large migration: shadow or phased; chunk copy + delta; atomic rename; checksum validate.
- Split delimited: use engine-native split or numbers table; normalize long-term.
- Consecutive active days: COUNT = span, or gap-islands, or anti-missing join vs calendar.
- Sampling: choose based on precision vs performance vs determinism.

---
## Suggested Follow-Ups
- Extend with median & percentile patterns.
- Add advanced locking diagnostics cheat sheet.
- Integrate calendar dimension for business-day streak logic.

Let me know if you want a PDF-ready consolidated handbook or flashcard extraction next.
