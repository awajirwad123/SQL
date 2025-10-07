# LeetCode SQL Master Sheet (Patterns & Archetypes)

IMPORTANT: This sheet summarizes common problem archetypes and reusable solution templates found across popular coding interview SQL challenges. It does NOT reproduce proprietary statements verbatim. Use these distilled patterns to rapidly map any given prompt to an execution strategy.

---
## 0. Quick Navigation
| Section | Focus | Core Tools |
|---------|-------|------------|
| 1 | Basic Retrieval & Filtering | SELECT, WHERE, DISTINCT |
| 2 | Joins & Relationships | INNER / LEFT / SELF / ANTI |
| 3 | Aggregation & Group Logic | GROUP BY, HAVING, COUNT DISTINCT |
| 4 | Ranking & Top-N | ROW_NUMBER, RANK, DENSE_RANK |
| 5 | Window Metrics | SUM OVER, LAG/LEAD, moving frames |
| 6 | Conditional Aggregation | CASE + SUM/COUNT |
| 7 | Subqueries & EXISTS | Correlated filters, IN vs EXISTS |
| 8 | Set Operations | UNION / INTERSECT / EXCEPT |
| 9 | Date & Time Patterns | Truncation, gaps, intervals |
| 10 | Self Joins / Hierarchies | Adjacency, path, manager chains |
| 11 | Pivot / Transpose | Conditional aggregation pivoting |
| 12 | String & Pattern Ops | LIKE, SUBSTRING, REGEXP (if supported) |
| 13 | Advanced Filtering | Anti-join, division, mandatory coverage |
| 14 | Cumulative / Running Analysis | Running totals, growth rates |
| 15 | Percentiles / Distribution | NTILE, percent_rank |
| 16 | Multi-Step CTE Pipelines | Layering transformations |
| 17 | Data Quality / Dedup | ROW_NUMBER + filter |
| 18 | Business Metrics Patterns | Retention, churn, conversion |
| 19 | Edge Case Handling | NULL control, 0-division, sparse data |
| 20 | Performance Heuristics | Index hints, sargability |

---
## 1. Basic Retrieval & Filtering
Pattern: Column selection + predicate narrowing.
Template:
```sql
SELECT DISTINCT col_list
FROM table
WHERE condition AND condition2;
```
Pitfalls: Unnecessary DISTINCT; misuse of OR reducing index use (rewrite with UNION ALL when selective).

## 2. Join Archetypes
| Goal | Pattern |
|------|---------|
| Combine attributes | INNER JOIN |
| Keep left unmatched | LEFT JOIN + IS NULL filter (anti) |
| Find non-overlap | LEFT JOIN ... WHERE right.id IS NULL |
| Self comparison | SELF JOIN with alias & inequality |
| Many-to-one rollup | JOIN then GROUP BY parent |

Example (anti-join):
```sql
SELECT a.id
FROM A a
LEFT JOIN B b ON b.a_id = a.id
WHERE b.a_id IS NULL;
```

## 3. Aggregation & Group Logic
Patterns: Count per category, min/max per entity, distinct counts.
```sql
SELECT category, COUNT(*) AS cnt, COUNT(DISTINCT user_id) AS users
FROM events
GROUP BY category
HAVING COUNT(*) > 5;
```
Avoid: GROUP BY columns missing in SELECT (ANSI errors); using WHERE for aggregate filters (use HAVING).

## 4. Ranking & Top-N Per Group
Top 1 per group:
```sql
SELECT * FROM (
  SELECT r.*, ROW_NUMBER() OVER (PARTITION BY grp ORDER BY metric DESC, id) AS rn
  FROM raw r
) x WHERE rn = 1;
```
Multi-top (N>1) adapt rn <= N.
Alternate: DENSE_RANK when ties must all appear.

## 5. Window Metrics
Running total, moving average, partition share.
```sql
SUM(val) OVER (PARTITION BY acct ORDER BY ts ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_val,
val * 1.0 / SUM(val) OVER (PARTITION BY acct) AS pct_of_acct
```

## 6. Conditional Aggregation
Pivot-like without native PIVOT.
```sql
SELECT user_id,
  SUM(CASE WHEN status='OPEN' THEN 1 END) AS open_cnt,
  SUM(CASE WHEN status='CLOSED' THEN 1 END) AS closed_cnt
FROM tickets
GROUP BY user_id;
```
Boolean to numeric: SUM(CASE WHEN cond THEN 1 ELSE 0 END).

## 7. Subqueries & EXISTS
Correlated filtering faster than IN for large inner sets sometimes.
```sql
SELECT u.*
FROM users u
WHERE EXISTS (
  SELECT 1 FROM orders o
  WHERE o.user_id = u.id AND o.amount > 100
);
```
NOT EXISTS for universal quantification complement.

## 8. Set Operations
De-duplicate union sets vs keep duplicates.
```sql
SELECT col FROM A
UNION
SELECT col FROM B; -- removes duplicates

SELECT col FROM A
UNION ALL
SELECT col FROM B; -- retains duplicates
```
EXCEPT / INTERSECT availability varies.

## 9. Dates & Time Bucketing
```sql
SELECT DATE(order_ts) AS d, COUNT(*)
FROM orders
WHERE order_ts >= CURRENT_DATE - INTERVAL '30 day'
GROUP BY DATE(order_ts);
```
Gap fill: calendar table LEFT JOIN.

## 10. Self Joins / Hierarchy
Manager chain (adjacency list):
```sql
SELECT e.id, m.id AS manager_id
FROM employees e
LEFT JOIN employees m ON m.id = e.manager_id;
```
Recursive (if supported) for depth/paths.

## 11. Pivot / Transpose
```sql
SELECT user_id,
  SUM(CASE WHEN action='click' THEN 1 END) AS clicks,
  SUM(CASE WHEN action='view' THEN 1 END) AS views
FROM activity
GROUP BY user_id;
```
Avoid multiple passes with separate subqueries.

## 12. String & Pattern
Normalize & filter.
```sql
SELECT id
FROM names
WHERE LOWER(name) LIKE 'a%' AND name REGEXP '^[A-Za-z]+$';
```
Beware non-sargable leading wildcards.

## 13. Relational Division / Coverage
“Users who performed ALL required actions”:
```sql
SELECT u.user_id
FROM user_actions u
WHERE action IN ('A','B','C')
GROUP BY u.user_id
HAVING COUNT(DISTINCT action) = 3;
```

## 14. Cumulative / Growth
```sql
SELECT d, daily_amt,
  SUM(daily_amt) OVER (ORDER BY d ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_amt,
  daily_amt - LAG(daily_amt) OVER (ORDER BY d) AS day_delta
FROM daily;
```

## 15. Percentile & Distribution
Approx quantiles when needed; or rank to compute bucket label.
```sql
PERCENT_RANK() OVER (ORDER BY metric) AS pr
```

## 16. Multi-Step CTE Pipeline
Layer complexity.
```sql
WITH base AS (...),
agg AS (
  SELECT key, SUM(val) AS s FROM base GROUP BY key
), ranked AS (
  SELECT *, ROW_NUMBER() OVER (ORDER BY s DESC) AS rn FROM agg
)
SELECT * FROM ranked WHERE rn <= 10;
```

## 17. Deduplication & Latest Record
```sql
SELECT * FROM (
  SELECT t.*, ROW_NUMBER() OVER (PARTITION BY natural_key ORDER BY updated_at DESC, id DESC) AS rn
  FROM staging t
) x WHERE rn = 1;
```

## 18. Business Metrics Patterns
| Metric | Pattern |
|--------|---------|
| Retention (D1/D7) | Self-join user activity by day offsets |
| Churn | Users active in prior period not in current (anti-join) |
| Conversion Funnel | Conditional aggregation across ordered steps |
| Cohort Spend | Partition by cohort_date then cumulative sum |

Retention Example:
```sql
WITH d1 AS (... users active day 0 ...),
d7 AS (... users active day 7 ...)
SELECT COUNT(*) * 1.0 / (SELECT COUNT(*) FROM d1)
FROM d1 JOIN d7 USING (user_id);
```

## 19. Edge Cases
- NULL grouping (treat separately or coalesce?)
- Division by zero (NULLIF(denom,0))
- Missing dates (calendar expansion)
- Duplicate records (dedupe first)
- Case sensitivity differing by engine

## 20. Performance Heuristics (Interview Speed Notes)
| Symptom | Fix |
|---------|-----|
| Slow correlated subquery | Convert to window or join + aggregate |
| Large DISTINCT sort | Pre-filter, check necessity, consider window dedupe |
| Non-sargable LIKE '%x' | Full scan unavoidable; maybe add trigram index (engine) |
| Many OR predicates | Break into UNION ALL of selective predicates |
| Over-wide SELECT | Project needed columns only before joins |

---
## Pattern-to-Strategy Cheat Table
| Prompt Hint | Recognize | Apply |
|-------------|-----------|-------|
| "Top N per group" | Ranking pattern | ROW_NUMBER filter |
| "Users with all actions" | Division | COUNT DISTINCT vs required count |
| "Previous vs current" | Delta | LAG difference |
| "Running total" | Cumulative | SUM OVER UNBOUNDED PRECEDING |
| "Median / percentile" | Distribution | percentile_* or window rank |
| "Active then churned" | Anti-join | LEFT JOIN IS NULL / NOT EXISTS |
| "Latest status" | Dedupe | ROW_NUMBER ORDER BY timestamp DESC |
| "Bucket into quartiles" | Distribution | NTILE(4) |
| "Rank ties no gaps" | Dense ranking | DENSE_RANK |
| "All-time and daily" | Mixed agg | SUM OVER partition + daily row |

---
## Decision Flow (Mental Model)
1. Does the result collapse rows? If yes → GROUP BY or aggregate CTE.
2. Need original granularity? Consider window.
3. Need top / latest per entity? Ranking + filter.
4. Need difference vs prior? LAG / LEAD.
5. Need conditional counts side-by-side? CASE in aggregation.
6. Need full coverage of required set? Division pattern.
7. Need distribution standing? Rank/percentile functions.
8. Complex multi-stage logic? CTE pipeline (ensure names readable).

---
## Common Mistakes (Interview Watchlist)
| Mistake | Consequence |
|---------|-------------|
| Filtering on window alias in WHERE | Logical order error |
| Using DISTINCT instead of proper GROUP BY | Hidden logic loss |
| Missing tie-breaker in ROW_NUMBER | Non-deterministic picks |
| COUNT(col) expecting NULL rows counted | Under-count |
| Using IN where EXISTS better | Performance degrade large subquery |
| Not handling division by zero | Runtime error / NULL confusion |
| Over-CTE chaining w/o necessity | Readability & perf overhead |

---
## Speed Templates (Fill-In Blanks)
Top-N per group:
```sql
WITH ranked AS (
  SELECT t.*, ROW_NUMBER() OVER (PARTITION BY grp ORDER BY metric DESC, id) AS rn
  FROM t
)
SELECT * FROM ranked WHERE rn <= __N__;
```

User coverage (all required types):
```sql
SELECT user_id
FROM actions
WHERE type IN (__LIST__)
GROUP BY user_id
HAVING COUNT(DISTINCT type) = __COUNT__;
```

Delta & running:
```sql
SELECT d, val,
  val - LAG(val) OVER (ORDER BY d) AS delta,
  SUM(val) OVER (ORDER BY d ROWS UNBOUNDED PRECEDING) AS running
FROM series;
```

Percent-of total within group:
```sql
val * 1.0 / SUM(val) OVER (PARTITION BY grp) AS pct_grp
```

Anti-join (no related rows):
```sql
SELECT a.id
FROM A a
LEFT JOIN B b ON b.a_id = a.id
WHERE b.a_id IS NULL;
```

Churn detection (previous active, now inactive):
```sql
WITH prev AS (...), cur AS (...)
SELECT p.user_id
FROM prev p
LEFT JOIN cur c ON c.user_id = p.user_id
WHERE c.user_id IS NULL;
```

---
## Practice Drill Ideas
1. Write 3 different correct solutions for top seller per category (window, correlated subquery, join+aggregate) and compare complexity.
2. Transform a correlated running total into a window-based rewrite and measure logical simplification.
3. Construct retention (D1, D7, D30) table using CTE layering and conditional aggregation.
4. Implement universal qualification (users who purchased all SKUs in a given set) and extend to at-least-K coverage.
5. Create a distribution label (low/medium/high) via NTILE then compute average metric per label.

---
## Final Blitz (One-Line Reminders)
- Ranking needs ORDER BY + tie-breaker.
- Division problems = COUNT(DISTINCT) vs required cardinality.
- Delta = current - LAG(current).
- Latest record = ROW_NUMBER DESC + rn=1.
- Percent-of = value / SUM(value) OVER partition.
- CTE layers clarify; avoid gratuitous nesting.
- Anti-join = LEFT JOIN IS NULL or NOT EXISTS.
- Moving average = AVG() OVER ROWS n PRECEDING.
- Coverage gaps require calendar or generated series.
- Always guard denominators with NULLIF.

Use this master sheet as a quick pattern matcher. Map description → archetype → template → adapt column names → validate edge cases.
