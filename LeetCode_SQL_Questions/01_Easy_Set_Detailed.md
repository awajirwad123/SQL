# Easy Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems Covered (paraphrased):
1. Recyclable and Low Fat Products
2. Find Customer Referee
3. Big Countries
4. Article Views I
5. Invalid Tweets

Each section includes: (a) Schema, (b) Sample Input, (c) Target Output, (d) Thought Process, (e) Canonical Query, (f) Edge Cases & Pitfalls, (g) Variations.

---
## 1. Recyclable and Low Fat Products
**Goal (paraphrased)**: Return IDs of products that are BOTH low-fat and recyclable.

### Schema
`Products(product_id INT PK, product_name VARCHAR, low_fats CHAR(1), recyclable CHAR(1), ... )`
- Flags typically 'Y' or 'N'.

### Sample Data
| product_id | product_name | low_fats | recyclable |
|------------|--------------|----------|------------|
| 1          | Chips        | N        | Y          |
| 2          | Oat Bar      | Y        | Y          |
| 3          | Soda         | Y        | N          |
| 4          | Cereal       | Y        | Y          |

### Expected Output
| product_id |
|------------|
| 2          |
| 4          |

### Thinking Process
1. Filter rows where both conditions hold simultaneously → logical AND.
2. Select just the key (no aggregation required; each product unique already).
3. No need DISTINCT because primary key ensures uniqueness.
4. Order not mandated unless explicitly required; stable order often by product_id.

### Query
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y'
  AND recyclable = 'Y'
ORDER BY product_id;  -- optional
```

### Edge Cases / Pitfalls
- Lowercase flags ('y') – consider UPPER(low_fats) = 'Y'.
- Unexpected NULL values: add `AND low_fats IS NOT NULL AND recyclable IS NOT NULL` if data dirty.
- Index use: composite index (low_fats, recyclable) useful if table very large and selectivity moderate.

### Variations
- Either low-fat OR recyclable: change AND → OR.
- Return count instead of list: wrap with COUNT(*).

---
## 2. Find Customer Referee
**Goal (paraphrased)**: List names of customers whose referee is NOT a particular ID (e.g., 2) OR who have no referee.

### Schema
`Customer(id INT PK, name VARCHAR, referee_id INT NULL REFERENCES Customer(id))`

### Sample Data
| id | name  | referee_id |
|----|-------|------------|
| 1  | Alice | NULL       |
| 2  | Bob   | NULL       |
| 3  | Carl  | 2          |
| 4  | Dana  | 1          |
| 5  | Evan  | 2          |
| 6  | Finn  | 3          |

Target exclude those whose referee_id = 2 but include NULL.

### Expected Output
| name  |
|-------|
| Alice |
| Bob   |
| Dana  |
| Finn  |

### Thinking Process
1. We need rows where referee_id ≠ 2 OR referee_id IS NULL.
2. Using only `referee_id <> 2` will exclude NULL because comparison yields UNKNOWN.
3. Combine inequality with explicit NULL acceptance.
4. Select the name column only.

### Query
```sql
SELECT name
FROM Customer
WHERE referee_id <> 2
   OR referee_id IS NULL
ORDER BY name;  -- optional lexical ordering
```

### Edge Cases / Pitfalls
- Using `!=` without handling NULLs removes desired rows.
- If referee target changes (parameterized), substitute variable.
- If negative logic expands (exclude many IDs), consider `NOT IN (...) OR referee_id IS NULL` (watch for NULL in list).

### Variations
- Only those explicitly referred by id 2: `WHERE referee_id = 2`.
- Those with ANY referee (non-null): `WHERE referee_id IS NOT NULL`.

---
## 3. Big Countries
**Goal (paraphrased)**: Return name, population, area for countries with very large area OR very large population thresholds.

### Schema
`World(name VARCHAR PK, continent VARCHAR, area INT, population INT, gdp BIGINT, ...)`

### Sample Data
| name     | continent | area   | population | gdp    |
|----------|-----------|--------|-----------:|--------|
| Alpha    | Asia      | 900000 |   8000000  | 100000 |
| Beta     | Europe    | 500000 |  60000000  | 300000 |
| Gamma    | Africa    | 200000 |   3000000  |  20000 |
| Delta    | N. America|3500000 |  10000000  | 900000 |

Thresholds (commonly used patterns): area ≥ 3000000 OR population ≥ 25000000.

### Expected Output
| name  | population | area    |
|-------|-----------:|--------:|
| Beta  | 60000000   | 500000  |
| Delta | 10000000   | 3500000 |

### Thinking Process
1. Two independent size dimensions; meeting either qualifies.
2. Disjunctive logic (OR) – cannot collapse into AND; separate selectivity.
3. Select only required columns.
4. ORDER BY optional (e.g., name or area desc) if presentation needed.

### Query
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000
   OR population >= 25000000
ORDER BY name;  -- optional
```

### Edge Cases / Pitfalls
- Accidentally using AND → overly restrictive.
- Very sparse huge countries: index on area/pop helpful; OR may defeat single index usage (engine may do union of index scans or full scan).
- Negative or zero values (unlikely) – rely on domain constraints.

### Variations
- Both thresholds: replace OR with AND.
- Include GDP threshold: append `OR gdp >= <value>`.

---
## 4. Article Views I
**Goal (paraphrased)**: Identify authors who have viewed their own articles; return unique author IDs.

### Schema
`Views(article_id INT, author_id INT, viewer_id INT, view_date DATE, ... )`
- An author can appear multiple times viewing their own articles.

### Sample Data
| article_id | author_id | viewer_id | view_date   |
|------------|-----------|-----------|-------------|
| 10         | 5         | 7         | 2025-01-01  |
| 11         | 5         | 5         | 2025-01-02  |
| 12         | 6         | 6         | 2025-01-03  |
| 13         | 7         | 5         | 2025-01-05  |
| 11         | 5         | 5         | 2025-01-06  |

### Expected Output
| id |
|----|
| 5  |
| 6  |

### Thinking Process
1. Self-view condition: author_id = viewer_id.
2. Need distinct authors only; multiple events collapse.
3. Two equivalent approaches: DISTINCT or GROUP BY.
4. Order ascending for deterministic output.

### Query
```sql
SELECT DISTINCT author_id AS id
FROM Views
WHERE author_id = viewer_id
ORDER BY id;
```

(Alternate)
```sql
SELECT author_id AS id
FROM Views
WHERE author_id = viewer_id
GROUP BY author_id
ORDER BY author_id;
```

### Edge Cases / Pitfalls
- Forget DISTINCT → duplicates.
- Different article_id not relevant; don't over-group.
- If IDs large and many duplicates, DISTINCT vs GROUP BY similar cost; rely on execution plan.

### Variations
- Count how many self-views per author: add `COUNT(*)` with GROUP BY.
- Filter only authors with ≥ 2 self-views: `HAVING COUNT(*) >= 2`.

---
## 5. Invalid Tweets
**Goal (paraphrased)**: Find tweet IDs where content length exceeds an allowed maximum (e.g., > 15 chars).

### Schema
`Tweets(tweet_id INT PK, content VARCHAR)`

### Sample Data
| tweet_id | content          |
|----------|------------------|
| 1        | Hello            |
| 2        | DataRocks        |
| 3        | Short msg        |
| 4        | ThisIsWayTooLongForLimit |
| 5        | Medium_Length    |

Assuming limit = 15 characters.

### Expected Output
| tweet_id |
|----------|
| 4        |

### Thinking Process
1. Need character length, not byte length (multi-byte safety).
2. Use appropriate length function per engine: `CHAR_LENGTH` (MySQL/Postgres), `LEN` (SQL Server), `LENGTH` (Oracle).
3. Filter strictly greater than limit.
4. Select only ID.

### Query (MySQL/Postgres style)
```sql
SELECT tweet_id
FROM Tweets
WHERE CHAR_LENGTH(content) > 15
ORDER BY tweet_id;  -- optional
```

### Engine Notes
- MySQL: `CHAR_LENGTH` vs `LENGTH` difference with multibyte.
- SQL Server: `LEN(content) > 15` (ignores trailing spaces).
- Oracle: `LENGTH(content) > 15`.

### Edge Cases / Pitfalls
- Trailing spaces: SQL Server LEN trims them; potential mismatch vs spec.
- Multi-byte characters: ensure character limit not byte limit.
- NULL content: condition yields NULL (excluded) → acceptable or explicitly handle.

### Variations
- Return also actual length: add `CHAR_LENGTH(content) AS content_len`.
- Classify: use CASE to label valid/invalid.

---
## Consolidated Reasoning Patterns
| Problem | Core Predicate Type | Key Insight | Typical Pitfall |
|---------|---------------------|-------------|-----------------|
| Recyclable & Low Fat | Dual AND flags | Simple conjunctive filter | Using OR | 
| Customer Referee | Inequality + NULL inclusion | Two-value logic vs three-valued SQL | Dropping NULLs | 
| Big Countries | Threshold OR | Disjunctive qualification | AND misuse | 
| Article Views I | Self-equality + dedupe | Need DISTINCT to collapse | Forget DISTINCT | 
| Invalid Tweets | Length constraint | Character vs byte length | Wrong length fn |

---
## General Thinking Checklist (Easy Level)
1. Clarify if conditions combine with AND or OR.
2. Identify whether duplicates must be eliminated.
3. Check for NULL semantics—do NULLs pass or fail the condition?
4. Decide if ordering required (spec vs default nondeterministic set). 
5. Confirm if length/aggregation must be per-row or grouped.
6. Use minimal columns (avoid SELECT * unless needed).

---
## Performance Micro-Notes
- All five queries are typically index-light; full scans acceptable for small teaching tables.
- Adding composite boolean flag index only beneficial at scale with high selectivity.
- DISTINCT vs GROUP BY for a single column yield similar execution plans.

---
## Practice Extensions
| Extension | Idea |
|-----------|------|
| Flag Composition | Add a third flag and require any two of three true (use CASE to sum booleans) |
| Referee Network | Count indirect referees (self-join chain) |
| Multi-Metric Country Filter | Combine area pop and GDP scoring expression |
| Article Engagement | Add window function to rank authors by self-view frequency |
| Tweet Validation | Enforce length + prohibited substring filter |

Use these detailed breakdowns to build a repeatable intuition: identify predicate type, duplication handling, NULL treatment, projection minimalism, then finalize query.
