# Subqueries Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased; emphasize subquery / correlated subquery / ranking patterns):
1. Employees Whose Manager Left the Company (Easy)
2. Exchange Seats (Medium)
3. Movie Rating (Medium)
4. Restaurant Growth (Medium)
5. Friend Requests II: Who Has the Most Friends (Medium)
6. Investments in 2016 (Medium)
7. Department Top Three Salaries (Hard)

Format per problem: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. Employees Whose Manager Left the Company
**Goal (paraphrased)**: List employees whose `manager_id` is not present anymore in Employees table.

### Schema
`Employees(employee_id INT PK, name VARCHAR, manager_id INT NULL)`

### Sample Data
| employee_id | name  | manager_id |
|-------------|-------|-----------:|
| 1           | CEO   | NULL       |
| 2           | Ann   | 1          |
| 3           | Bob   | 9          |  -- orphan manager
| 4           | Cara  | 2          |

### Expected Output
| employee_id |
|-------------|
| 3           |

### Thinking
- We need employees whose manager_id NOT NULL and not in existing employee_id set.
- Use NOT IN subquery or LEFT JOIN + IS NULL. Subquery requested theme → use NOT IN with NULL-safe care.

### Query (Anti-Subquery)
```sql
SELECT e.employee_id
FROM Employees e
WHERE e.manager_id IS NOT NULL
  AND e.manager_id NOT IN (SELECT employee_id FROM Employees);
```

### Safer (NULL-proof) Alternative
```sql
SELECT e.employee_id
FROM Employees e
LEFT JOIN Employees m ON m.employee_id = e.manager_id
WHERE e.manager_id IS NOT NULL AND m.employee_id IS NULL;
```

### Pitfalls
- `NOT IN` + subquery containing NULL returns no rows; ensure subquery has no NULL (manager_id reference is PK so safe here).

### Variations
- Return names too.
- Count orphaned employees.

---
## 2. Exchange Seats
**Goal (paraphrased)**: Swap seat assignments pairwise (odd <-> next even). Last odd without partner remains.

### Schema
`Seat(id INT PK, student VARCHAR)`  -- id is 1-based contiguous

### Sample Data
| id | student |
|----|---------|
| 1  | A       |
| 2  | B       |
| 3  | C       |
| 4  | D       |
| 5  | E       |

### Expected Output
| id | student |
|----|---------|
| 1  | B       |
| 2  | A       |
| 3  | D       |
| 4  | C       |
| 5  | E       |

### Thinking
- For odd id with a following even: map to id+1.
- For even id: map to id-1.
- For last odd without partner: keep same id.
- Subquery/CTE helps clarity.

### Query (CASE + Correlated Subquery to Confirm Partner)
```sql
WITH s AS (
  SELECT id, student,
         CASE
           WHEN id % 2 = 1 AND EXISTS (SELECT 1 FROM Seat s2 WHERE s2.id = id + 1) THEN id + 1
           WHEN id % 2 = 0 THEN id - 1
           ELSE id
         END AS new_id
  FROM Seat
)
SELECT new_id AS id, student
FROM s
ORDER BY id;
```

### Pitfalls
- Forget checking existence → last odd shifts incorrectly.
- Using WINDOW LAG/LEAD is alternative but subquery matches theme.

### Variations
- Add grouping by row pairs to apply other transformations.

---
## 3. Movie Rating
**Goal (paraphrased)**: (Common variant) Return: (1) user with max number of ratings / tie by lexicographic name. (2) Highest rated movie (average rating) / tie by lexicographic title.

### Schemas
`Movies(movie_id INT PK, title VARCHAR)`
`Users(user_id INT PK, name VARCHAR)`
`MovieRating(movie_id INT, user_id INT, rating INT, created_at DATE)`

### Sample Data (simplified)
Movies:
| movie_id | title    |
|----------|----------|
| 1        | Alpha    |
| 2        | Beta     |

Users:
| user_id | name  |
|---------|-------|
| 10      | Ann   |
| 11      | Bob   |

MovieRating:
| movie_id | user_id | rating | created_at  |
|----------|---------|-------:|-------------|
| 1        | 10      | 5      | 2025-01-01  |
| 2        | 10      | 4      | 2025-01-02  |
| 1        | 11      | 5      | 2025-01-03  |

### Expected Output
| results |
|---------|
| Ann     |  -- most ratings (2)
| Alpha   |  -- highest average rating (5) tie between movie 1 only if beta avg < 5

### Thinking
1. Subquery count ratings per user; pick max count then lexical min.
2. Subquery average rating per movie; pick max avg then lexical min.
3. UNION ALL results in required order.

### Query
```sql
WITH user_counts AS (
  SELECT u.name, COUNT(*) AS cnt
  FROM MovieRating mr JOIN Users u ON u.user_id = mr.user_id
  GROUP BY u.name
), top_user AS (
  SELECT name
  FROM user_counts
  WHERE cnt = (SELECT MAX(cnt) FROM user_counts)
  ORDER BY name
  FETCH FIRST 1 ROW ONLY
), movie_avgs AS (
  SELECT m.title, AVG(mr.rating) AS avg_rating
  FROM MovieRating mr JOIN Movies m ON m.movie_id = mr.movie_id
  GROUP BY m.title
), top_movie AS (
  SELECT title
  FROM movie_avgs
  WHERE avg_rating = (SELECT MAX(avg_rating) FROM movie_avgs)
  ORDER BY title
  FETCH FIRST 1 ROW ONLY
)
SELECT name AS results FROM top_user
UNION ALL
SELECT title FROM top_movie;
```
(MySQL: LIMIT 1; Postgres: FETCH FIRST 1 ROW ONLY.)

### Pitfalls
- Using ORDER BY before UNION incorrectly; must apply inside subqueries.
- Averaging int without casting still fine but ensure not integer division (most DBs produce numeric).

### Variations
- Return both counts & averages separately.
- Filter by date range in MovieRating.

---
## 4. Restaurant Growth
**Goal (paraphrased)**: Compute 7-day moving average of daily customer count starting from day 7 (include only dates with at least 7 full preceding days). Show date + average rounded to 2 decimals.

### Schema
`Customer (visit_date DATE PK, customer_count INT)`  -- daily record.

### Sample Data
| visit_date  | customer_count |
|-------------|----------------|
| 2025-01-01  | 100            |
| 2025-01-02  | 120            |
| 2025-01-03  | 130            |
| 2025-01-04  | 140            |
| 2025-01-05  | 160            |
| 2025-01-06  | 170            |
| 2025-01-07  | 180            |
| 2025-01-08  | 150            |

### Expected Output (first output date = 2025-01-07)
| visit_date  | moving_avg |
|-------------|-----------:|
| 2025-01-07  | 142.86     | (sum first 7 / 7)
| 2025-01-08  | 150.00     | (drop day1 add day8)

### Thinking (Subquery Approach)
- Without window functions (or to emphasize subqueries): for each date d (>= 7th date) compute average of last 7 days using correlated subquery filtering date range.
- Use correlated subquery summing counts over interval `[d - 6, d]`.

### Query (Correlated Subqueries)
```sql
SELECT c.visit_date,
       ROUND((SELECT SUM(c2.customer_count)
              FROM Customer c2
              WHERE c2.visit_date BETWEEN c.visit_date - INTERVAL '6 day' AND c.visit_date)
             * 1.0 / 7, 2) AS moving_avg
FROM Customer c
WHERE c.visit_date >= (
  SELECT MIN(visit_date) + INTERVAL '6 day' FROM Customer
)
ORDER BY c.visit_date;
```
(If gaps in dates, spec may assume continuous calendar; otherwise need date series.)

### Pitfalls
- Including fewer than 7 rows at start (should begin only when 7 available).
- Performance on large data (better with window SUM OVER ROWS 6 PRECEDING).

### Variations
- Use window function version for efficiency.
- Parameterize window length.

---
## 5. Friend Requests II: Who Has the Most Friends
**Goal (paraphrased)**: Count friendships per user (friend requests accepted) regardless of request direction and return user with largest friend count (lexicographically smallest on tie).

### Schema
`RequestAccepted(requester_id INT, accepter_id INT, accept_date DATE)`
Each row implies mutual friendship between requester_id and accepter_id.

### Sample Data
| requester_id | accepter_id |
|--------------|-------------|
| 1            | 2           |
| 2            | 3           |
| 2            | 4           |

### Expected Output
| id | num |
|----|-----|
| 2  | 3   |  -- friends: 1,3,4

### Thinking
1. Normalize friendships into two-direction rows using UNION ALL.
2. Aggregate count per user.
3. Filter for max count (subquery for max).

### Query
```sql
WITH all_edges AS (
  SELECT requester_id AS user_id FROM RequestAccepted
  UNION ALL
  SELECT accepter_id AS user_id FROM RequestAccepted
), counts AS (
  SELECT user_id, COUNT(*) AS num
  FROM all_edges
  GROUP BY user_id
)
SELECT user_id AS id, num
FROM counts
WHERE num = (SELECT MAX(num) FROM counts)
ORDER BY id
FETCH FIRST 1 ROW ONLY;  -- tie-break by smallest id
```

### Pitfalls
- Using UNION (deduplicates) reduces counts incorrectly if same pair appears once (should count each undirected pair only once; our approach duplicates each relation intentionally into two endpoints — correct).
- Not tie-breaking deterministically.

### Variations
- Return full ranking order with DENSE_RANK() (window alternative).

---
## 6. Investments in 2016
**Goal (paraphrased)**: Sum total invested amount in 2016 and number of unique investors who invested in 2016 and return a single summary row.

### Schema
`Insurance(investor_id INT, insurance_type VARCHAR, invest_date DATE, invest_amount DECIMAL(12,2))`

### Sample Data
| investor_id | invest_date  | invest_amount |
|-------------|--------------|--------------:|
| 10          | 2016-01-02   | 5000.00       |
| 11          | 2016-05-10   | 7000.00       |
| 10          | 2017-03-01   | 4000.00       |

### Expected Output
| total_invested | unique_investors |
|----------------|------------------|
| 12000.00       | 2                |

### Thinking
- Filter year=2016.
- Aggregate SUM and COUNT(DISTINCT investor_id).
- Single row; no grouping key needed.
- Subquery not strictly needed; we can show variant with inline subquery for clarity.

### Query
```sql
SELECT COALESCE(SUM(invest_amount),0) AS total_invested,
       COUNT(DISTINCT investor_id) AS unique_investors
FROM Insurance
WHERE invest_date >= DATE '2016-01-01' AND invest_date < DATE '2017-01-01';
```

### Pitfalls
- Using YEAR(extract) function may prevent index range scan (prefer range predicate).
- Counting distinct after filtering vs before (must after).

### Variations
- Add average investment per investor (SUM / COUNT DISTINCT).
- Parameterize year using variable.

---
## 7. Department Top Three Salaries (Hard)
**Goal (paraphrased)**: For each department return employees whose salary is in the top 3 distinct salaries for that department (include ties within each distinct salary level yes/no depending on spec; usually distinct means rank by salary, skip duplicates in counting distinct ranking but include all employees at those salaries).

### Schemas
`Employee(id INT PK, name VARCHAR, salary INT, departmentId INT)`
`Department(id INT PK, name VARCHAR)`

### Sample Data
Employee:
| name  | salary | departmentId |
|-------|-------:|-------------:|
| Ann   | 90000  | 1            |
| Bob   | 80000  | 1            |
| Cara  | 80000  | 1            |
| Dan   | 70000  | 1            |
| Erin  | 95000  | 2            |
| Fred  | 60000  | 2            |

Department:
| id | name       |
|----|------------|
| 1  | Engineering|
| 2  | Sales      |

### Expected Output (Top 3 distinct salaries each dept)
| Department  | Employee | Salary |
|-------------|----------|-------:|
| Engineering | Ann      | 90000  |
| Engineering | Bob      | 80000  |
| Engineering | Cara     | 80000  |
| Engineering | Dan      | 70000  |
| Sales       | Erin     | 95000  |
| Sales       | Fred     | 60000  |

(Dept 2 only has 2 distinct salaries; both included.)

### Thinking
- Need to rank distinct salaries per department.
- Approach: subquery selecting distinct departmentId, salary with DENSE_RANK over salary desc, then join back to employees.

### Query (Distinct Salary Subquery)
```sql
WITH salary_levels AS (
  SELECT departmentId, salary,
         DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS dr
  FROM (
    SELECT DISTINCT departmentId, salary
    FROM Employee
  ) ds
)
SELECT d.name AS Department, e.name AS Employee, e.salary AS Salary
FROM Employee e
JOIN salary_levels sl ON sl.departmentId = e.departmentId AND sl.salary = e.salary
JOIN Department d ON d.id = e.departmentId
WHERE sl.dr <= 3
ORDER BY d.name, e.salary DESC, e.name;
```

### Pitfalls
- Using ROW_NUMBER (excludes ties at same salary incorrectly).
- Ranking directly on employees (OK) but then counting duplicates as separate rank if using ROW_NUMBER.
- Forgetting to join Department for department name.

### Variations
- Return only top 1 (modify dr <=1).
- Add salary rank column.

---
## Consolidated Pattern Summary
| Problem | Subquery Pattern | Key Concept | Why Subquery Helpful |
|---------|------------------|-------------|----------------------|
| Manager Left | Anti-subquery (NOT IN) | Referential orphan detection | Simple exclusion set |
| Exchange Seats | Existence check | Pairwise swap | Guard last odd row |
| Movie Rating | Aggregation subqueries + max filter | Two independent maxima | Isolate tie-breaking |
| Restaurant Growth | Correlated range aggregate | Moving window (no window funcs) | Date-based slice per row |
| Most Friends | Subquery on MAX(count) | Graph degree max | Late filter after aggregate |
| Investments 2016 | Scalar aggregate | Year filter + summary | Single-row metrics |
| Dept Top 3 | Distinct-level subquery + DENSE_RANK | Distinct top-k per group | Separate distinct ranking layer |

---
## Thinking Framework (Subqueries)
1. Identify if subquery is: IN/NOT IN set, scalar aggregate, correlated per-row, or derived table for ranking.
2. Use anti/semi subqueries for existence logic; prefer NOT EXISTS / EXISTS if NULL hazards.
3. For top-k with distinct criteria: derive distinct dimension first, rank, then join back.
4. For moving window without analytic support: correlated subquery filtering a date interval.
5. For tie-breaking: compute aggregate set then filter on equality to global max in subquery.

---
## Common Pitfalls & Fixes
| Pitfall | Fix |
|---------|-----|
| NOT IN with NULLs returns nothing | Use NOT EXISTS or ensure subquery column NOT NULL |
| Window easier but forced subquery slow | Consider indexing + range predicate or fallback to window if allowed |
| Losing ties in rank | Use DENSE_RANK or distinct-level ranking |
| Over-counting in friend requests | Normalize edges then aggregate |
| Correlated subquery scanned per row (perf) | Replace with window SUM if allowed |

---
## Practice Extensions
- Rewrite Restaurant Growth using window functions and compare explain plan.
- Implement Department Top 3 with a correlated subquery (salary IN (SELECT ... ORDER BY ... LIMIT 3)).
- Replace NOT IN with NOT EXISTS and measure performance difference on large dataset.
- Generalize Exchange Seats to swap triplets (cyclic rotation) via modular arithmetic.
- Create a view returning current price for any arbitrary date using parameter substitution (point-in-time template).

Use these patterns to master when and how subqueries clarify logic, enable set-based filtering, and implement group-wise top-k selection with distinct criteria.
