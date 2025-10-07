# Sorting & Grouping Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased; focus on DISTINCT counts, grouping, sorting, coverage):
1. Number of Unique Subjects Taught by Each Teacher
2. User Activity for the Past 30 Days I
3. Product Sales Analysis III
4. Classes With at Least 5 Students
5. Find Followers Count
6. Biggest Single Number
7. Customers Who Bought All Products

Format: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. Number of Unique Subjects Taught by Each Teacher
**Goal (paraphrased)**: For every teacher, count distinct subjects they teach.

### Schema
`Teacher(teacher_id INT, subject_id INT, dept_id INT)` — multiple rows per (teacher_id, subject_id) possible if duplicates; we want unique subjects.

### Sample Data
| teacher_id | subject_id | dept_id |
|------------|------------|---------|
| 1          | 101        | 10      |
| 1          | 101        | 10      |
| 1          | 102        | 10      |
| 2          | 101        | 11      |
| 2          | 103        | 11      |
| 3          | 104        | 12      |

### Expected Output
| teacher_id | cnt |
|------------|-----|
| 1          | 2   |
| 2          | 2   |
| 3          | 1   |

### Thinking
- Group by teacher_id.
- Use COUNT(DISTINCT subject_id) to collapse duplicate subject assignments.
- ORDER BY teacher_id for deterministic output.

### Query
```sql
SELECT teacher_id, COUNT(DISTINCT subject_id) AS cnt
FROM Teacher
GROUP BY teacher_id
ORDER BY teacher_id;
```

### Pitfalls
- Using COUNT(*) double counts duplicates.
- Forget ORDER BY if platform expects stable ordering.

### Variations
- Also list dept diversity: COUNT(DISTINCT dept_id).
- Filter teachers with ≥ 2 distinct subjects (HAVING clause).

---
## 2. User Activity for the Past 30 Days I
**Goal (paraphrased)**: Return the number of distinct active users for each date in the last 30 days (inclusive) ending at the max date present.

### Schema
`Activity(user_id INT, session_id INT, activity_date DATE, activity_type VARCHAR)`

### Sample Data
| user_id | session_id | activity_date | activity_type |
|---------|------------|---------------|---------------|
| 1       | 10         | 2025-03-01    | open          |
| 2       | 11         | 2025-03-01    | click         |
| 1       | 12         | 2025-03-02    | close         |
| 3       | 13         | 2025-03-05    | open          |
| 2       | 14         | 2025-03-05    | click         |

Assume max date = 2025-03-05 → window start = 2025-02-05 (if < earliest, only existing dates appear).

### Expected Output (example for existing dates only)
| activity_date | active_users |
|---------------|--------------|
| 2025-03-01    | 2            |
| 2025-03-02    | 1            |
| 2025-03-05    | 2            |

(If spec requires all calendar days including zero-activity days you need a date series; base version usually lists only existing dates.)

### Thinking
1. Derive max date.
2. Filter rows where activity_date in [max_date - 29, max_date].
3. Group by activity_date counting distinct user_id.
4. Sort ascending by date.

### Query
```sql
WITH mx AS (SELECT MAX(activity_date) AS max_d FROM Activity)
SELECT a.activity_date,
       COUNT(DISTINCT a.user_id) AS active_users
FROM Activity a
JOIN mx ON a.activity_date BETWEEN mx.max_d - INTERVAL '29 day' AND mx.max_d
GROUP BY a.activity_date
ORDER BY a.activity_date;
```
(MySQL: `mx.max_d - INTERVAL 29 DAY`; SQL Server: `DATEADD(day,-29,mx.max_d)`.)

### Pitfalls
- Using 30 instead of 29 day offset (off-by-one yields 31 days).
- Counting sessions instead of users.
- Missing a date series if spec demands blank days (needs calendar table or generated series).

### Variations
- Include zero-activity days via generated series (recursive CTE or system date table).
- Add moving 7-day average active users (window function overlay).

---
## 3. Product Sales Analysis III
**Goal (paraphrased)**: For each product, find its first sales year and total quantity sold in that first year.

### Schema
`Sales(product_id INT, year INT, quantity INT, price DECIMAL(10,2))`

### Sample Data
| product_id | year | quantity | price |
|------------|------|---------:|------:|
| 1          | 2019 | 10       | 20.00 |
| 1          | 2020 | 15       | 18.00 |
| 2          | 2020 |  5       | 30.00 |
| 2          | 2021 |  7       | 32.00 |

### Expected Output
| product_id | first_year | total_quantity |
|------------|------------|----------------|
| 1          | 2019       | 10             |
| 2          | 2020       | 5              |

### Thinking
1. Identify earliest year per product (MIN(year)).
2. Restrict to rows of that year and sum quantity.
3. Two approaches: subquery join or window function ranking.

### Query (subquery join)
```sql
WITH first_year AS (
  SELECT product_id, MIN(year) AS first_year
  FROM Sales
  GROUP BY product_id
)
SELECT s.product_id, f.first_year,
       SUM(s.quantity) AS total_quantity
FROM Sales s
JOIN first_year f ON f.product_id = s.product_id AND s.year = f.first_year
GROUP BY s.product_id, f.first_year
ORDER BY s.product_id;
```

### Window Variant
```sql
SELECT product_id, year AS first_year, quantity AS total_quantity
FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY year) AS rn
  FROM Sales
) x
WHERE rn = 1
ORDER BY product_id;
```
(Only valid if exactly one row per product/year; otherwise need SUM in first-year subset.)

### Pitfalls
- Summing across all years instead of first.
- Using MIN(price) incorrectly as proxy for year.

### Variations
- Also include first-year revenue: SUM(quantity * price).
- Add second-year growth metric (JOIN year+1 rows).

---
## 4. Classes With at Least 5 Students
**Goal (paraphrased)**: Return class names (or ids) where at least 5 distinct students are enrolled.

### Schema
`Courses(student VARCHAR, class VARCHAR)`

### Sample Data
| student | class  |
|---------|--------|
| a       | math   |
| b       | math   |
| c       | math   |
| d       | math   |
| e       | math   |
| f       | eng    |
| g       | eng    |

### Expected Output
| class |
|-------|
| math  |

### Thinking
1. Group by class.
2. Count distinct students (if duplicates possible) else COUNT(*).
3. HAVING threshold ≥ 5.

### Query
```sql
SELECT class
FROM Courses
GROUP BY class
HAVING COUNT(DISTINCT student) >= 5
ORDER BY class;
```

### Pitfalls
- Using WHERE instead of HAVING.
- Forgetting DISTINCT when duplicates present.

### Variations
- Return class plus student_count.
- Parameterize threshold.

---
## 5. Find Followers Count
**Goal (paraphrased)**: For every user appearing as someone being followed, return their follower count.

### Schema
`Followers(user_id INT, follower_id INT)`

### Sample Data
| user_id | follower_id |
|---------|-------------|
| 1       | 10          |
| 1       | 11          |
| 2       | 10          |
| 2       | 12          |
| 2       | 10          |  -- duplicate follow example (should decide uniqueness)

### Expected Output (assuming raw count including duplicates):
| user_id | followers_count |
|---------|-----------------|
| 1       | 2               |
| 2       | 3               |

### Thinking
- If duplicates should not count twice, use COUNT(DISTINCT follower_id).
- Group by user_id.

### Query (distinct recommended)
```sql
SELECT user_id, COUNT(DISTINCT follower_id) AS followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

### Pitfalls
- Counting duplicates artificially inflates counts.
- Reversing columns misunderstand direction.

### Variations
- Include users with zero followers by joining with Users master table.
- Add ranking by follower count (DENSE_RANK OVER ordering).

---
## 6. Biggest Single Number
**Goal (paraphrased)**: Find largest number that occurs exactly once. Return NULL (or empty result) if no unique number exists.

### Schema
`MyNumbers(num INT)`

### Sample Data
| num |
|-----|
| 8   |
| 8   |
| 3   |
| 10  |
| 10  |
| 7   |

### Expected Output
| num |
|-----|
| 7   |

(7 is the largest value with frequency = 1.)

### Thinking
1. Aggregate frequency per num.
2. Filter WHERE cnt = 1.
3. Order descending and take top 1.

### Query (portable)
```sql
SELECT num
FROM (
  SELECT num, COUNT(*) AS cnt
  FROM MyNumbers
  GROUP BY num
) x
WHERE cnt = 1
ORDER BY num DESC
FETCH FIRST 1 ROW ONLY;  -- MySQL: LIMIT 1
```

### Pitfalls
- Forget descending order (returns smallest unique if asc).
- Using MAX(num) without filtering uniqueness.

### Variations
- Return also frequency distribution table.
- If tie (multiple unique) list them all (omit LIMIT).

---
## 7. Customers Who Bought All Products
**Goal (paraphrased)**: Find customer IDs who purchased every product at least once.

### Schemas
`Customer(customer_id INT, product_key INT)`  -- purchase events (duplicates possible)
`Product(product_key INT PK)`  -- list of all products

### Sample Data
Product:
| product_key |
|-------------|
| 1           |
| 2           |
| 3           |

Customer:
| customer_id | product_key |
|-------------|-------------|
| 10          | 1           |
| 10          | 2           |
| 10          | 3           |
| 11          | 1           |
| 11          | 2           |
| 12          | 1           |
| 12          | 2           |
| 12          | 3           |

### Expected Output
| customer_id |
|-------------|
| 10          |
| 12          |

### Thinking
1. Total distinct product count from Product table.
2. For each customer, count distinct product_key purchased.
3. Keep those where count equals total.
4. Equivalent to relational division pattern.

### Query
```sql
WITH total AS (SELECT COUNT(*) AS prod_cnt FROM Product)
SELECT c.customer_id
FROM Customer c
CROSS JOIN total t
GROUP BY c.customer_id, t.prod_cnt
HAVING COUNT(DISTINCT c.product_key) = t.prod_cnt
ORDER BY c.customer_id;
```

### Pitfalls
- Using COUNT(*) instead of COUNT(DISTINCT) when duplicates exist.
- Computing total via Customer table (wrong if some product never purchased).

### Variations
- Find customers who bought at least K products (HAVING count >= K parameter).
- Return coverage ratio (count distinct / total) for all customers.

---
## Consolidated Pattern Summary
| Problem | Core Technique | Key Clause(s) | Concept Name |
|---------|----------------|---------------|--------------|
| Unique Subjects | DISTINCT count per group | COUNT(DISTINCT) | Grouping + Dedup |
| 30-Day Activity | Sliding date window + grouping | BETWEEN / date arithmetic | Time-window aggregation |
| Product Sales III | First-year filter + aggregate | MIN(year)+JOIN / ROW_NUMBER | First-occurrence rollup |
| Classes ≥ 5 | Threshold group filter | HAVING COUNT(DISTINCT) >= N | Group filter |
| Followers Count | Distinct count | COUNT(DISTINCT follower_id) | Degree centrality |
| Biggest Single | Frequency =1 + max | HAVING cnt=1; ORDER DESC LIMIT | Relational division variant (filter + max) |
| Bought All Products | Division (coverage) | HAVING count(distinct)=total | Set containment |

---
## Thinking Framework (Sorting & Grouping)
1. Identify grouping keys and distinctness requirements.
2. Decide if global total needed (cross join small scalar CTE).
3. For coverage problems: compare per-entity distinct count to global distinct count.
4. For time windows: anchor date (max or given) then compute interval.
5. For extremum-with-condition (largest unique): filter first, then order/limit.

---
## Common Pitfalls & Fixes
| Pitfall | Fix |
|---------|-----|
| Duplicate rows inflating counts | Use COUNT(DISTINCT) or pre-deduplicate |
| Off-by-one in 30-day window | Use 29-day offset inclusive |
| Missing products never purchased | Derive total from Product table |
| Returning smallest unique | ORDER BY num DESC |
| Counting sessions instead of users | COUNT(DISTINCT user_id) |

---
## Practice Extensions
- Add a query listing top 3 teachers by distinct subject count.
- Extend 30-day activity to also show rolling 7-day avg (window function).
- For Product Sales III compute YoY growth between first and second year (if present).
- Add follower rank column using DENSE_RANK.
- Coverage ratio for all customers (list customer_id + distinct_count / total_products).

Use these grouping patterns to quickly map counting, coverage, and extremum tasks into efficient SQL queries.
