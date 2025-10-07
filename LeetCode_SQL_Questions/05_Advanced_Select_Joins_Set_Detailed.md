# Advanced Select & Joins Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased):
1. The Number of Employees Which Report to Each Employee
2. Primary Department for Each Employee
3. Triangle Judgement
4. Consecutive Numbers
5. Product Price at a Given Date
6. Last Person to Fit in the Bus
7. Count Salary Categories

Format: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. The Number of Employees Which Report to Each Employee
**Goal (paraphrased)**: For each manager show number of direct reports and average age (rounded) of those reports. Only managers with ≥1 report.

### Schema
`Employees(employee_id INT PK, name VARCHAR, reports_to INT NULL, age INT)`
- `reports_to` references `employee_id` of manager.

### Sample Data
| employee_id | name  | reports_to | age |
|-------------|-------|------------|----:|
| 1           | CEO   | NULL       | 50  |
| 2           | Ann   | 1          | 30  |
| 3           | Bob   | 1          | 28  |
| 4           | Cara  | 2          | 26  |
| 5           | Dan   | 2          | 27  |

### Expected Output
| employee_id | reports_count | average_age |
|-------------|---------------|------------:|
| 1           | 2             | 29          |
| 2           | 2             | 26          |

### Thinking
1. Self-join: subordinate e joins manager m on e.reports_to = m.employee_id.
2. Count subordinates; average their ages.
3. Round average to nearest integer (or floor per spec; adapt rounding function to dialect).
4. Group by manager.

### Query
```sql
SELECT m.employee_id,
       COUNT(*) AS reports_count,
       ROUND(AVG(e.age)) AS average_age  -- use FLOOR/ROUND per requirement
FROM Employees e
JOIN Employees m ON e.reports_to = m.employee_id
GROUP BY m.employee_id
ORDER BY m.employee_id;
```

### Pitfalls
- Including managers with zero reports (filtered out automatically by inner join).
- Averaging manager age instead of subordinate ages (ensure using e.age).

### Variations
- Include manager name: add `m.name`.
- Filter managers with ≥ N reports: HAVING COUNT(*) >= N.

---
## 2. Primary Department for Each Employee
**Goal (paraphrased)**: For each employee, return primary department. A row is primary if flag = 'Y'. If employee only has one department (all flags 'N' or only row), treat it as primary.

### Schema
`EmployeeDepartment(employee_id INT, department_id INT, primary_flag CHAR(1))` -- primary_flag in ('Y','N').

### Sample Data
| employee_id | department_id | primary_flag |
|-------------|---------------|--------------|
| 10          | 1             | N            |
| 10          | 2             | Y            |
| 11          | 1             | N            |
| 12          | 3             | N            | -- only department (implicit primary)

### Expected Output
| employee_id | department_id |
|-------------|---------------|
| 10          | 2             |
| 11          | 1             | (only if spec says choose any when no primary but multi; usually not returned unless single or flagged) |
| 12          | 3             |

(Original spec: if an employee has exactly one department assign it; else choose where primary_flag='Y'. Employees with multiple departments and no 'Y' may be excluded or choose lowest dept; adapt to spec.)

### Thinking
1. Count departments per employee.
2. Select row where primary_flag='Y'.
3. UNION with single-department employees where no 'Y'.
4. Alternatively window function ranking.

### Query (Window Approach)
```sql
WITH enriched AS (
  SELECT *,
         COUNT(*) OVER (PARTITION BY employee_id) AS dept_count,
         SUM(CASE WHEN primary_flag='Y' THEN 1 ELSE 0 END) OVER (PARTITION BY employee_id) AS has_primary,
         ROW_NUMBER() OVER (
           PARTITION BY employee_id
           ORDER BY (CASE WHEN primary_flag='Y' THEN 1 ELSE 0 END) DESC, department_id
         ) AS rn
  FROM EmployeeDepartment
)
SELECT employee_id, department_id
FROM enriched
WHERE rn = 1
  AND (has_primary > 0 OR dept_count = 1)
ORDER BY employee_id;
```

### Pitfalls
- Returning multiple departments if primary flag missing.
- Forgetting to treat single department employees specially.

### Variations
- If multiple primaries (data error) keep the smallest department_id via ORDER BY tiebreaker.
- Return also a derived `is_chosen_primary` indicator.

---
## 3. Triangle Judgement
**Goal (paraphrased)**: For each row decide if three lengths form a valid triangle.

### Schema
`Triangle(x INT, y INT, z INT)`

### Sample Data
| x | y | z |
|---|---|---|
| 2 | 3 | 4 |
| 1 | 1 | 3 |

### Expected Output
| x | y | z | triangle |
|---|---|---|----------|
| 2 | 3 | 4 | Yes      |
| 1 | 1 | 3 | No       |

### Thinking
Triangle inequality: a + b > c for all permutations.

### Query
```sql
SELECT x, y, z,
       CASE WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes' ELSE 'No' END AS triangle
FROM Triangle;
```

### Pitfalls
- Using ≥ instead of > (degenerate line segments incorrectly classified).
- Overflow on large ints (rare; cast to BIGINT if needed).

### Variations
- Classify by type (equilateral, isosceles, scalene).
- Compute perimeter when valid.

---
## 4. Consecutive Numbers
**Goal (paraphrased)**: Find numbers appearing at least 3 times in a row (consecutive rows by id order).

### Schema
`Logs(id INT PK, num INT)`

### Sample Data
| id | num |
|----|-----|
| 1  | 5   |
| 2  | 5   |
| 3  | 5   |
| 4  | 7   |
| 5  | 7   |
| 6  | 7   |
| 7  | 7   |
| 8  | 8   |

### Expected Output
| ConsecutiveNums |
|-----------------|
| 5               |
| 7               |

### Thinking
1. Compare each row with previous 1 and 2 rows.
2. Window functions: LAG and pattern test; or self-join on id offsets.
3. Distinct numbers satisfying triple occurrence.

### Query (Window LAG)
```sql
WITH w AS (
  SELECT num,
         LAG(num,1) OVER (ORDER BY id) AS p1,
         LAG(num,2) OVER (ORDER BY id) AS p2
  FROM Logs
)
SELECT DISTINCT num AS ConsecutiveNums
FROM w
WHERE num = p1 AND num = p2
ORDER BY ConsecutiveNums;
```

### Pitfalls
- Missing DISTINCT → duplicates if run length > 3.
- Ordering by num instead of id inside window.

### Variations
- Parameterize run length N using window count over partitions of (id - row_number) technique.
- Return longest run per number with additional window logic.

---
## 5. Product Price at a Given Date
**Goal (paraphrased)**: For each product, report price on target date (e.g., '2019-08-16'): latest `new_price` whose `change_date` ≤ target. If no prior change, default price 10.

### Schema
`Products(product_id INT, new_price DECIMAL(10,2), change_date DATE)`

### Sample Data
| product_id | new_price | change_date  |
|------------|-----------|--------------|
| 1          | 20.00     | 2019-08-14   |
| 1          | 25.00     | 2019-08-18   |
| 2          | 30.00     | 2019-08-10   |
| 3          | 15.00     | 2019-08-01   |

Target date: 2019-08-16.

### Expected Output
| product_id | price |
|------------|-------|
| 1          | 20.00 |
| 2          | 30.00 |
| 3          | 15.00 |

### Thinking
1. Need all distinct products appearing in table.
2. Filter rows with change_date ≤ target.
3. Rank rows per product by change_date DESC and pick first.
4. If no candidate row, default 10.

### Query (Window)
```sql
WITH ranked AS (
  SELECT product_id,
         new_price,
         change_date,
         ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY change_date DESC) AS rn
  FROM Products
  WHERE change_date <= DATE '2019-08-16'
), all_products AS (
  SELECT DISTINCT product_id FROM Products
)
SELECT a.product_id,
       COALESCE(r.new_price, 10) AS price
FROM all_products a
LEFT JOIN ranked r ON r.product_id = a.product_id AND r.rn = 1
ORDER BY a.product_id;
```

### Pitfalls
- Forgetting to include products that first change after target date (should default 10).
- Using MAX on new_price incorrectly (need price associated with latest date, not highest price).

### Variations
- Parameterize target date.
- Return also effective change_date used.

---
## 6. Last Person to Fit in the Bus
**Goal (paraphrased)**: People queue with weights, bus has weight limit (e.g., 1000). Board in turn order until adding next person would exceed limit. Return last boarded person name.

### Schema
`Queue(person_id INT, person_name VARCHAR, weight INT, turn INT)`
- `turn` ascending defines boarding order.
- Constant: `@limit = 1000` (adjust as needed).

### Sample Data
| person_id | person_name | weight | turn |
|-----------|-------------|--------|-----:|
| 1         | Alice       | 300    | 1    |
| 2         | Bob         | 400    | 2    |
| 3         | Cara        | 350    | 3    |
| 4         | Dan         | 200    | 4    |

Limit 1000: Boarding weights cumulative after Cara would be 1050 (>1000) so last fitting is Bob.

### Expected Output
| person_name |
|-------------|
| Bob         |

### Thinking
1. Order by turn, compute cumulative sum.
2. Keep rows where running_total ≤ limit.
3. Return the one with max running_total.

### Query (Window)
```sql
WITH ordered AS (
  SELECT *,
         SUM(weight) OVER (ORDER BY turn ROWS UNBOUNDED PRECEDING) AS running_wt
  FROM Queue
)
SELECT person_name
FROM ordered
WHERE running_wt = (
  SELECT MAX(running_wt) FROM ordered WHERE running_wt <= 1000
);
```
(MySQL lacks QUALIFY; use subquery filter.)

### Pitfalls
- Using total sum vs running sum.
- Ties: if two people exactly at limit on different turns impossible; running sums strictly increasing.

### Variations
- Return list of all boarded people: filter running_wt <= limit.
- Add capacity as parameter table or CTE.

---
## 7. Count Salary Categories
**Goal (paraphrased)**: Count employees in salary buckets: Low (<20000), Average (20000–50000 inclusive), High (>50000). Show zero counts for empty categories.

### Schema
`Accounts(account_id INT PK, income INT)`

### Sample Data
| account_id | income |
|------------|--------|
| 1          | 15000  |
| 2          | 20000  |
| 3          | 50000  |
| 4          | 60000  |

### Expected Output
| category | accounts_count |
|----------|----------------|
| Low      | 1              |
| Average  | 2              |
| High     | 1              |

### Thinking
1. Derive classification with CASE.
2. Aggregate counts per category.
3. Ensure all categories present: UNION seed categories if needed then LEFT JOIN actual counts.

### Query (Ensuring All Categories)
```sql
WITH buckets AS (
  SELECT 'Low' AS category
  UNION ALL SELECT 'Average'
  UNION ALL SELECT 'High'
), counts AS (
  SELECT CASE
           WHEN income < 20000 THEN 'Low'
           WHEN income BETWEEN 20000 AND 50000 THEN 'Average'
           ELSE 'High'
         END AS category,
         COUNT(*) AS cnt
  FROM Accounts
  GROUP BY 1
)
SELECT b.category, COALESCE(c.cnt, 0) AS accounts_count
FROM buckets b
LEFT JOIN counts c ON c.category = b.category
ORDER BY FIELD(b.category,'Low','Average','High'); -- MySQL ordering; others use CASE
```
(Postgres ordering: `ORDER BY CASE b.category WHEN 'Low' THEN 1 WHEN 'Average' THEN 2 ELSE 3 END`.)

### Pitfalls
- Missing categories with zero employees.
- Boundary errors: inclusive 20000 and 50000 belong to Average.

### Variations
- Add percentage per category: cnt / total.
- Parameterize thresholds via settings table.

---
## Consolidated Pattern Summary
| Problem | Core Technique | Key Clause(s) | Category |
|---------|----------------|---------------|----------|
| Employees Report Count | Self-join aggregate | JOIN + GROUP BY | Hierarchy rollup |
| Primary Department | Window rank + conditional | ROW_NUMBER + filters | Conditional selection |
| Triangle Judgement | Row-level classification | CASE inequalities | Validation logic |
| Consecutive Numbers | Window lag pattern detection | LAG + DISTINCT | Sequence analysis |
| Product Price at Date | As-of latest value | ROW_NUMBER over date | Temporal point-in-time |
| Last Person Bus | Running sum threshold | SUM() OVER + max filter | Cumulative capacity |
| Salary Categories | Bucketing + category seeding | CASE + UNION seed | Distribution bucket |

---
## Thinking Framework (Advanced Select / Joins)
1. Identify row set driver (all products, all employees, etc.).
2. For point-in-time lookups: rank rows descending by effective date and pick rn=1.
3. For classification: implement CASE first, aggregate second.
4. For hierarchy metrics: self-join subordinate→manager then group.
5. For sequence detection: window LAG/LEAD or ID arithmetic grouping.
6. For cumulative constraint: compute running metric then filter by max satisfying condition.

---
## Common Pitfalls & Fixes
| Pitfall | Fix |
|---------|-----|
| Wrong average source (manager vs subordinates) | Use subordinate alias in AVG |
| Multiple primary departments returned | Window rank & filter on first only |
| Using ≥ for triangle inequalities | Strict > comparison |
| Missing DISTINCT for consecutive values | Wrap in subquery & SELECT DISTINCT |
| Choosing MAX(price) instead of latest date | Order by change_date DESC + ROW_NUMBER |
| Using total sum for capacity | Running cumulative SUM over order |
| Omitting zero-count salary buckets | Seed categories via CTE/UNION |

---
## Practice Extensions
- Compute transitive (not just direct) report counts via recursive CTE.
- Add previous day price difference for Product Price at date (need historical join).
- Generalize consecutive detection for length N parameter.
- Add cumulative average weight stopping threshold variant (when average weight exceeds X).
- Salary category percentages with ranking by proportion.

Use these patterns to strengthen handling of temporal, hierarchical, and sequence-based SQL interview challenges.
