# Aggregate Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased; aggregate / ratio / conditional metrics focus):
1. Not Boring Movies
2. Average Selling Price
3. Project Employees I
4. Percentage of Users Attended a Contest
5. Queries Quality and Percentage
6. Monthly Transactions I (conditional monthly rollup)
7. Immediate Food Delivery II
8. Game Play Analysis IV

Format per problem: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. Not Boring Movies
**Goal (paraphrased)**: Retrieve non-boring movies with odd primary key id, ordered by rating descending.

### Schema
`Cinema(id INT PK, movie VARCHAR, description VARCHAR, rating DECIMAL(3,1))`

### Sample Data
| id | movie        | description | rating |
|----|--------------|-------------|-------:|
| 1  | Alpha        | drama       | 7.5    |
| 2  | Beta         | boring      | 8.0    |
| 3  | Gamma        | action      | 9.1    |
| 4  | Delta        | thriller    | 6.2    |

### Expected Output
| id | movie  | description | rating |
|----|--------|-------------|-------:|
| 3  | Gamma  | action      | 9.1    |
| 1  | Alpha  | drama       | 7.5    |

### Thinking
1. Odd id → `id % 2 = 1`.
2. Exclude rows where description = 'boring'.
3. Order by rating DESC.

### Query
```sql
SELECT id, movie, description, rating
FROM Cinema
WHERE description <> 'boring'
  AND id % 2 = 1
ORDER BY rating DESC;
```

### Pitfalls
- Using `!=` vs `<>` (both fine) but forgetting case sensitivity (use LOWER if needed).
- Ordering ascending accidentally.

### Variations
- Top N non-boring odd movies: add `LIMIT N`.
- Include even IDs by parameterizing condition.

---
## 2. Average Selling Price
**Goal (paraphrased)**: Compute weighted average price per product = sum(price * units) / sum(units) for products with sales.

### Schemas
`Prices(product_id INT, start_date DATE, end_date DATE, price DECIMAL(10,2))`
`UnitsSold(product_id INT, purchase_date DATE, units INT)`
(Each units row falls within exactly one price range for that product.)

### Sample Data
Prices:
| product_id | start_date  | end_date    | price |
|------------|-------------|-------------|------:|
| 1          | 2025-01-01  | 2025-01-31  | 10.00 |
| 1          | 2025-02-01  | 2025-02-28  | 12.00 |
| 2          | 2025-01-01  | 2025-02-28  | 5.00  |

UnitsSold:
| product_id | purchase_date | units |
|------------|---------------|------:|
| 1          | 2025-01-10    | 5     |
| 1          | 2025-02-10    | 3     |
| 2          | 2025-02-05    | 10    |

### Expected Output
| product_id | average_price |
|------------|---------------|
| 1          | 10.75         | -- (5*10 + 3*12)/8 = 86/8 = 10.75
| 2          | 5.00          |

### Thinking
1. Join UnitsSold to appropriate price row using date range.
2. Weighted average formula.
3. Group by product_id; format to 2 decimals.

### Query (generic ANSI)
```sql
SELECT u.product_id,
       ROUND(SUM(u.units * p.price) * 1.0 / SUM(u.units), 2) AS average_price
FROM UnitsSold u
JOIN Prices p ON p.product_id = u.product_id
             AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY u.product_id
ORDER BY u.product_id;
```

### Pitfalls
- Missing date boundary inclusive checks.
- Integer division (ensure decimal multiplication).
- Including products with no sales (spec usually excludes).

### Variations
- Include zero-sales products with NULL average (LEFT JOIN around UnitsSold aggregated separately).
- Use window function to show average alongside each sale event.

---
## 3. Project Employees I
**Goal (paraphrased)**: For each project, compute average years of experience among assigned employees.

### Schemas
`Project(project_id INT, employee_id INT)`
`Employee(employee_id INT PK, name VARCHAR, experience_years INT)`

### Sample Data
Project:
| project_id | employee_id |
|------------|-------------|
| 10         | 1           |
| 10         | 2           |
| 11         | 2           |
| 11         | 3           |

Employee:
| employee_id | name  | experience_years |
|-------------|-------|------------------|
| 1           | Ann   | 5                |
| 2           | Bob   | 7                |
| 3           | Cara  | 4                |

### Expected Output
| project_id | average_years |
|------------|---------------|
| 10         | 6.00          | -- (5+7)/2
| 11         | 5.50          | -- (7+4)/2

### Thinking
1. Join Project with Employee.
2. Average experience per project.
3. Round to 2 decimals (spec sometimes requests integer rounding; adjust if needed).

### Query
```sql
SELECT p.project_id,
       ROUND(AVG(e.experience_years * 1.0), 2) AS average_years
FROM Project p
JOIN Employee e ON e.employee_id = p.employee_id
GROUP BY p.project_id
ORDER BY p.project_id;
```

### Pitfalls
- Double counting if duplicates in Project.
- Using integer AVG without casting (some DBs auto-cast; explicit safer).

### Variations
- Include project with no employees (LEFT JOIN from Projects master list).
- Provide count of employees also.

---
## 4. Percentage of Users Attended a Contest
**Goal (paraphrased)**: For each contest, show number of unique registered users and percentage of total users (rounded to 2 decimals), ordered by participation descending then contest id.

### Schemas
`Users(user_id INT PK, user_name VARCHAR)`
`Register(contest_id INT, user_id INT)` (a row per registration)

### Sample Data
Users:
| user_id |
|---------|
| 1       |
| 2       |
| 3       |
| 4       |

Register:
| contest_id | user_id |
|------------|---------|
| 100        | 1       |
| 100        | 2       |
| 101        | 2       |
| 101        | 3       |
| 101        | 4       |

### Expected Output
| contest_id | participants | percentage |
|------------|--------------|------------|
| 101        | 3            | 75.00      |
| 100        | 2            | 50.00      |

### Thinking
1. Total users constant denominator = (SELECT COUNT(*) FROM Users).
2. Count distinct users per contest.
3. Percentage = participants / total * 100.
4. Order by percentage desc then contest_id asc.

### Query
```sql
WITH total AS (SELECT COUNT(*) AS total_users FROM Users)
SELECT r.contest_id,
       COUNT(DISTINCT r.user_id) AS participants,
       ROUND(COUNT(DISTINCT r.user_id) * 100.0 / t.total_users, 2) AS percentage
FROM Register r
CROSS JOIN total t
GROUP BY r.contest_id, t.total_users
ORDER BY percentage DESC, r.contest_id ASC;
```

### Pitfalls
- Forget DISTINCT if duplicate registrations possible (depends on constraints).
- Integer math causing 0 percentages.

### Variations
- Show cumulative participation share over contests.
- Add rank by percentage using window function.

---
## 5. Queries Quality and Percentage
**Goal (paraphrased)**: For each query_name compute quality = average(rating/position). Also compute overall poor query percentage (rating < 3) among all rows.

### Schema
`Queries(query_name VARCHAR, result VARCHAR, position INT, rating INT)`
- rating 1–5, lower position = better.

### Sample Data
| query_name | result  | position | rating |
|------------|---------|---------:|-------:|
| alpha      | A1      | 1        | 5      |
| alpha      | A2      | 2        | 4      |
| beta       | B1      | 1        | 2      |
| beta       | B2      | 2        | 2      |

### Expected Output
Quality table example:
| query_name | quality |
|------------|---------|
| alpha      | 3.00    | -- ((5/1)+(4/2))/2 = (5 + 2)/2 = 3.5 → depending on spec rounding; using AVG(rating/position)
| beta       | 1.50    | -- ((2/1)+(2/2))/2 = (2 + 1)/2 = 1.5

Poor percentage (rating < 3): 3 of 4 = 75.00

(Exact formatting may vary by platform; adapt rounding.)

### Thinking
1. `quality`: compute per-row (rating * 1.0 / position) then average per query_name.
2. Poor percentage: count rows rating < 3 divided by total.
3. Separate outputs often required; here we show approach for both.

### Query (per-query quality)
```sql
SELECT query_name,
       ROUND(AVG(rating * 1.0 / position), 2) AS quality
FROM Queries
GROUP BY query_name
ORDER BY query_name;
```

### Query (poor percentage)
```sql
SELECT ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS poor_query_percentage
FROM Queries;
```

### Pitfalls
- Averaging rating then dividing by avg position (wrong) vs averaging (rating/position).
- Integer division. Cast numerator to decimal.

### Variations
- Weighted quality by inverse position: SUM(rating/position)/COUNT.
- Flag queries with quality below threshold.

---
## 6. Monthly Transactions I
**Goal (paraphrased)**: For each month and country, compute counts and amounts of total vs approved transactions.

### Schema
`Transactions(id INT PK, country VARCHAR, state VARCHAR, amount DECIMAL(10,2), trans_date DATE)`
- state examples: 'approved','declined'.

### Sample Data
| id | country | state     | amount | trans_date  |
|----|---------|-----------|-------:|------------|
| 1  | US      | approved  | 100.00 | 2025-01-05 |
| 2  | US      | declined  |  50.00 | 2025-01-07 |
| 3  | CA      | approved  |  70.00 | 2025-01-20 |
| 4  | US      | approved  | 120.00 | 2025-02-02 |

### Expected Output
| month    | country | trans_count | approved_count | trans_total_amount | approved_total_amount |
|----------|---------|-------------|----------------|--------------------|-----------------------|
| 2025-01  | CA      | 1           | 1              | 70.00              | 70.00                 |
| 2025-01  | US      | 2           | 1              | 150.00             | 100.00                |
| 2025-02  | US      | 1           | 1              | 120.00             | 120.00                |

### Thinking
1. Extract month string (e.g., `DATE_FORMAT(trans_date, '%Y-%m')`).
2. Group by month & country.
3. Conditional counts and sums.
4. Order by month, country.

### Query (MySQL style; adjust date formatting per engine)
```sql
SELECT DATE_FORMAT(trans_date, '%Y-%m') AS month,
       country,
       COUNT(*) AS trans_count,
       SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END) AS approved_count,
       ROUND(SUM(amount), 2) AS trans_total_amount,
       ROUND(SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END), 2) AS approved_total_amount
FROM Transactions
GROUP BY month, country
ORDER BY month, country;
```
(Postgres: `TO_CHAR(trans_date, 'YYYY-MM')`.)

### Pitfalls
- Grouping on raw date not month.
- Double rounding (round at presentation only).
- Case-sensitivity of state values.

### Variations
- Include approval rate: approved_count / trans_count.
- Filter on specific countries.

---
## 7. Immediate Food Delivery II
**Goal (paraphrased)**: Among first orders of all customers, compute percentage that were delivered on the order date (immediate), rounded to 2 decimals.

### Schema
`Delivery(delivery_id INT PK, customer_id INT, order_date DATE, customer_pref_delivery_date DATE)`

### Sample Data
| delivery_id | customer_id | order_date  | customer_pref_delivery_date |
|-------------|-------------|-------------|-----------------------------|
| 1           | 10          | 2025-01-01  | 2025-01-01                  |
| 2           | 10          | 2025-01-10  | 2025-01-12                  |
| 3           | 11          | 2025-01-02  | 2025-01-05                  |
| 4           | 12          | 2025-01-03  | 2025-01-03                  |

### Expected Output
| immediate_percentage |
|---------------------|
| 66.67               | -- first orders: cust 10 (immediate), 11 (not), 12 (immediate) → 2/3

### Thinking
1. Identify first order per customer: MIN(order_date).
2. Restrict to those rows, test if order_date = customer_pref_delivery_date.
3. Compute count immediate / total * 100.

### Query
```sql
WITH first_orders AS (
  SELECT d.*,
         ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date, delivery_id) AS rn
  FROM Delivery d
)
SELECT ROUND(SUM(CASE WHEN order_date = customer_pref_delivery_date THEN 1 ELSE 0 END)
            * 100.0 / COUNT(*), 2) AS immediate_percentage
FROM first_orders
WHERE rn = 1;
```
(If ties: earliest by order_date then smallest delivery_id.)

### Pitfalls
- Not limiting to first order only (inflates denominator).
- Using MIN join but retaining multiple first-day duplicates → double counting.

### Variations
- Show numerator & denominator separately.
- Immediate percentage per month (GROUP BY month(order_date)).

---
## 8. Game Play Analysis IV
**Goal (paraphrased)**: Fraction of players who logged in again the day immediately after their first login (rounded 2 decimals).

### Schema
`Activity(player_id INT, device_id INT, event_date DATE, games_played INT)`

### Sample Data
| player_id | event_date  | games_played |
|-----------|-------------|-------------:|
| 1         | 2025-01-01  | 5            |
| 1         | 2025-01-02  | 3            |
| 2         | 2025-01-05  | 2            |
| 2         | 2025-01-07  | 1            |
| 3         | 2025-01-10  | 4            |
| 3         | 2025-01-11  | 1            |

### Expected Output
| fraction |
|----------|
| 66.67    | -- players 1 & 3 returned next day; player 2 did not → 2/3

### Thinking
1. First login date per player via MIN(event_date).
2. Check existence of row at first_date + 1.
3. Count players meeting condition / total distinct players.
4. Round to 2 decimals.

### Query
```sql
WITH first_login AS (
  SELECT player_id, MIN(event_date) AS first_date
  FROM Activity
  GROUP BY player_id
), returns AS (
  SELECT f.player_id,
         CASE WHEN EXISTS (
           SELECT 1 FROM Activity a
           WHERE a.player_id = f.player_id
             AND a.event_date = DATEADD(day, 1, f.first_date) -- SQL Server; Postgres/MySQL: f.first_date + INTERVAL '1 day'
         ) THEN 1 ELSE 0 END AS returned
  FROM first_login f
)
SELECT ROUND(SUM(returned) * 100.0 / COUNT(*), 2) AS fraction
FROM returns;
```

### Pitfalls
- Joining on next day using player_id only (must filter by date).
- Counting multiple days beyond first+1 (irrelevant).
- Using self join but duplicating rows if multiple matches (should be at most one anyway because unique date per player/day, but guard with EXISTS).

### Variations
- Also compute retention at +2 days (replace interval with 2 days & alias metrics).
- Output both raw counts and fraction.

---
## Consolidated Pattern Summary
| Problem | Core Aggregate / Concept | Key Pattern | Risk Area |
|---------|--------------------------|-------------|-----------|
| Not Boring Movies | Filtering + ordering | WHERE + modulo + sort | Wrong operator / order |
| Average Selling Price | Weighted average | SUM(units*price)/SUM(units) | Date range join correctness |
| Project Employees I | Mean per group | AVG() with cast | Duplicate project rows |
| Percentage Contest | Ratio of counts | COUNT(DISTINCT)/total | Integer division |
| Queries Quality | Per-row expression average | AVG(rating/position) | Averaging separately |
| Monthly Transactions I | Conditional rollup | SUM(CASE WHEN ...) | Month extraction inconsistencies |
| Immediate Food Delivery II | First-row filtering + ratio | ROW_NUMBER + filter | Not isolating first order |
| Game Play Analysis IV | Retention ratio | MIN date + EXISTS next-day | Off-by-one interval |

---
## Thinking Framework (Aggregates)
1. Identify grouping keys (time bucket? entity? composite?).
2. Determine if weighted vs simple average (weighted needs numerator & denominator).
3. Ratios: ensure decimal math (cast or *1.0) and guard zero denominator with NULLIF.
4. Conditional aggregation: embed boolean inside SUM/COUNT.
5. First-event logic: use ROW_NUMBER or MIN + join/EXISTS.
6. Date arithmetic: use correct interval syntax per dialect.

---
## Common Pitfalls & Fixes
| Pitfall | Fix |
|---------|-----|
| Integer division yields 0 | Cast numerator or denominator to decimal |
| Duplicate base rows inflate sums | Deduplicate (DISTINCT) or aggregate earlier |
| Wrong date bucket | Standardize via TO_CHAR / DATE_FORMAT |
| Weighted avg mis-specified | Always divide sum(weighted) by sum(weights) |
| Multiple first rows per entity | Use ROW_NUMBER deterministic ordering |

---
## Practice Extensions
- Compute approval rate in Monthly Transactions (approved_count / trans_count).
- Add median selling price using percentile functions (if supported).
- Multi-retention table: Day1, Day7, Day30 rates in one query using conditional EXISTS.
- Contest participation cumulative share ranking.
- Weighted quality using exponential decay for position (e.g., rating/POWER(2, position-1)).

Use these patterns to quickly map aggregate-centric interview prompts into robust, performant SQL solutions.
