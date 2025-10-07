# Joins Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased; join & aggregation focus):
1. Replace Employee ID With The Unique Identifier (Employee ↔ Unique mapping)
2. Product Sales Analysis I
3. Customer Who Visited but Did Not Make Any Transactions
4. Rising Temperature (date adjacency + comparison)
5. Average Time of Process per Machine
6. Employee Bonus
7. Students and Examinations
8. Managers with at Least 5 Direct Reports
9. Confirmation Rate

Format per problem: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. Replace Employee ID With The Unique Identifier
**Goal (paraphrased)**: Show each employee's unique identifier (if exists) alongside name.

### Schemas
`Employees(id INT PK, name VARCHAR)`
`EmployeeUNI(id INT PK, unique_id INT UNIQUE)`  -- same id refers to Employees.id

### Sample Data
Employees:
| id | name   |
|----|--------|
| 1  | Alice  |
| 2  | Bob    |
| 3  | Carol  |

EmployeeUNI:
| id | unique_id |
|----|-----------|
| 1  | 101       |
| 3  | 103       |

### Expected Output
| unique_id | name  |
|-----------|-------|
| 101       | Alice |
| NULL      | Bob   |
| 103       | Carol |

### Thinking
- Need all employees even if no mapping ⇒ LEFT JOIN.
- Select mapped unique_id (nullable) plus name.
- Order by name or id if deterministic order requested.

### Query
```sql
SELECT u.unique_id, e.name
FROM Employees e
LEFT JOIN EmployeeUNI u ON u.id = e.id
ORDER BY e.id;  -- optional
```

### Pitfalls
- Using INNER JOIN would drop employees without mapping.
- Ambiguous column name if selecting `id` from both tables without alias.

### Variations
- Filter only employees with a mapping: change to INNER JOIN.
- Count how many have unique ids vs not (add aggregation).

---
## 2. Product Sales Analysis I
**Goal (paraphrased)**: For each product, show total quantity sold.

### Schemas
`Sales(sale_id INT, product_id INT, quantity INT, sale_date DATE)`
`Product(product_id INT PK, product_name VARCHAR)`

### Sample Data
Sales:
| sale_id | product_id | quantity |
|---------|------------|---------:|
| 1       | 10         | 5        |
| 2       | 10         | 3        |
| 3       | 11         | 7        |

Product:
| product_id | product_name |
|------------|--------------|
| 10         | Keyboard     |
| 11         | Mouse        |
| 12         | Monitor      |

### Expected Output
| product_name | total_quantity |
|--------------|----------------|
| Keyboard     | 8              |
| Mouse        | 7              |

### Thinking
- Only products appearing in Sales required (INNER JOIN) for typical phrasing.
- Aggregate quantity per product_id then join to product name (either join then GROUP BY or subquery aggregate then join).

### Query
```sql
SELECT p.product_name, SUM(s.quantity) AS total_quantity
FROM Sales s
JOIN Product p ON p.product_id = s.product_id
GROUP BY p.product_name
ORDER BY p.product_name;
```

### Pitfalls
- Including products with zero sales inadvertently (LEFT JOIN + COALESCE) changes requirement.
- Summing duplicates due to accidental extra joins.

### Variations
- Include zero-sale products: LEFT JOIN Product p LEFT JOIN Sales s ... GROUP BY p fields.
- Add revenue if price column exists.

---
## 3. Customer Who Visited but Did Not Make Any Transactions
**Goal (paraphrased)**: List customer ids who appear in Visits but have no Transactions record for that visit.

### Schemas
`Visits(visit_id INT PK, customer_id INT)`
`Transactions(transaction_id INT PK, visit_id INT, amount DECIMAL)`

### Sample Data
Visits:
| visit_id | customer_id |
|----------|-------------|
| 1        | 5           |
| 2        | 5           |
| 3        | 6           |
| 4        | 7           |

Transactions:
| transaction_id | visit_id | amount |
|----------------|----------|-------:|
| 10             | 2        | 40.00  |
| 11             | 4        | 15.00  |

### Expected Output
| customer_id |
|-------------|
| 5           |  -- from visit 1 only (visit 2 had a transaction but existence of at least one no-transaction visit qualifies depending on interpretation; common solution lists both unique customers or counts)  
| 6           |

(If requirement is distinct customers with at least one visit lacking a transaction.)

### Thinking
- Need visits without matching Transactions → LEFT JOIN and filter NULL.
- Then deduplicate customer ids (DISTINCT or GROUP BY).
- Interpret spec carefully: some variants expect each `customer_id` once, others may count occurrences.

### Query (distinct customers having at least one no-transaction visit)
```sql
SELECT DISTINCT v.customer_id
FROM Visits v
LEFT JOIN Transactions t ON t.visit_id = v.visit_id
WHERE t.visit_id IS NULL
ORDER BY v.customer_id;
```

### Pitfalls
- Using INNER JOIN excludes non-transaction visits.
- Filtering on t.transaction_id IS NULL but joining incorrectly on wrong key.

### Variations
- Count number of no-transaction visits per customer.
- Exclude customers who ever transacted (different logic: customers only with zero transactions across all visits) → HAVING approach.

---
## 4. Rising Temperature
**Goal (paraphrased)**: Find days where temperature is higher than the previous day.

### Schema
`Weather(id INT PK, recordDate DATE, temperature INT)` (id typically sequential; recordDate unique)

### Sample Data
| id | recordDate  | temperature |
|----|-------------|------------:|
| 1  | 2025-01-01  | 30          |
| 2  | 2025-01-02  | 32          |
| 3  | 2025-01-03  | 31          |
| 4  | 2025-01-04  | 35          |

### Expected Output
| id |
|----|
| 2  |
| 4  |

### Thinking (Join Approach)
- Need current row joined to previous date row where recordDate = currentDate - 1 day.
- Compare temperatures; return current id when greater.

### Query (Self Join)
```sql
SELECT w1.id
FROM Weather w1
JOIN Weather w2 ON w2.recordDate = DATEADD(day, -1, w1.recordDate)  -- SQL Server syntax
WHERE w1.temperature > w2.temperature
ORDER BY w1.id;
```
(Adjust date subtraction: `w1.recordDate = w2.recordDate + INTERVAL '1 day'` in Postgres/MySQL.)

### Window Function Variant
```sql
SELECT id
FROM (
  SELECT id, recordDate, temperature,
         LAG(temperature) OVER (ORDER BY recordDate) AS prev_temp
  FROM Weather
) s
WHERE temperature > prev_temp;
```

### Pitfalls
- Joining on id-1 assumption (fails if gaps) instead of date difference.
- Off-by-one using >= instead of >.

### Variations
- Include difference: select (temperature - prev_temp) AS delta.
- Rolling 3-day increase streak detection with windows.

---
## 5. Average Time of Process per Machine
**Goal (paraphrased)**: Compute average processing time per machine from start/end activity logs.

### Schema
`Activity(machine_id INT, process_id INT, activity_type ENUM('start','end'), timestamp INT)`
- For each (machine_id, process_id) we have one start and one end.

### Sample Data
| machine_id | process_id | activity_type | timestamp |
|------------|------------|---------------|----------:|
| 1          | 1          | start         | 100       |
| 1          | 1          | end           | 150       |
| 1          | 2          | start         | 200       |
| 1          | 2          | end           | 260       |
| 2          | 1          | start         | 120       |
| 2          | 1          | end           | 170       |

### Expected Output
| machine_id | processing_time |
|------------|-----------------|
| 1          | 55.0            |  -- ((150-100)+(260-200))/2
| 2          | 50.0            |

### Thinking
1. Pair start with end per (machine_id, process_id).
2. Compute duration end_ts - start_ts.
3. Average per machine.
4. Approaches: self join start/end rows OR conditional aggregation.

### Query (Self Join)
```sql
WITH paired AS (
  SELECT a.machine_id, a.process_id,
         e.timestamp - s.timestamp AS duration
  FROM Activity s
  JOIN Activity e ON e.machine_id = s.machine_id
                  AND e.process_id = s.process_id
                  AND e.activity_type = 'end'
  WHERE s.activity_type = 'start'
)
SELECT machine_id, AVG(duration) AS processing_time
FROM paired
GROUP BY machine_id
ORDER BY machine_id;
```

### Conditional Aggregation Variant
```sql
SELECT machine_id,
       AVG(CASE WHEN activity_type='end' THEN timestamp END -
           CASE WHEN activity_type='start' THEN timestamp END) OVER (PARTITION BY machine_id) -- less portable
FROM Activity;  -- not recommended; clearer to pair explicitly
```

### Pitfalls
- Joining start to end without restricting to same process_id.
- Double counting if duplicate start/end anomalies.
- Integer division vs decimal (cast if needed).

### Variations
- Median instead of average (percentile_cont).
- Exclude outliers > threshold.

---
## 6. Employee Bonus
**Goal (paraphrased)**: List employee name and bonus for those with bonus < threshold (e.g., 1000) or bonus is NULL.

### Schemas
`Employee(empId INT PK, name VARCHAR, managerId INT NULL)`
`Bonus(empId INT PK, bonus INT NULL)`

### Sample Data
Employee:
| empId | name   |
|-------|--------|
| 1     | Alice  |
| 2     | Bob    |
| 3     | Carol  |

Bonus:
| empId | bonus |
|-------|-------|
| 1     | 800   |
| 2     | 1500  |

### Expected Output
| name  | bonus |
|-------|-------|
| Alice | 800   |
| Carol | NULL  |

### Thinking
- Need all qualifying employees – those with bonus < 1000 OR no bonus row.
- LEFT JOIN on Bonus then filter condition.

### Query
```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON b.empId = e.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL
ORDER BY e.name;
```

### Pitfalls
- Using INNER JOIN loses NULL bonus employees.
- Condition `b.bonus < 1000` alone excludes NULL rows.

### Variations
- Add threshold parameterization.
- Show also a status column (e.g., 'Needs Review').

---
## 7. Students and Examinations
**Goal (paraphrased)**: For every (student, subject) pair, show how many times the student attended that subject’s exam; include zero counts.

### Schemas
`Students(student_id INT PK, student_name VARCHAR)`
`Subjects(subject_name VARCHAR PK)`
`Examinations(student_id INT, subject_name VARCHAR)`  -- each row = one attendance

### Sample Data
Students:
| student_id | student_name |
|------------|--------------|
| 1          | Alice        |
| 2          | Bob          |

Subjects:
| subject_name |
|--------------|
| Math         |
| Physics      |

Examinations:
| student_id | subject_name |
|------------|--------------|
| 1          | Math         |
| 1          | Math         |
| 2          | Physics      |

### Expected Output
| student_id | student_name | subject_name | attended_exams |
|------------|--------------|--------------|----------------|
| 1          | Alice        | Math         | 2              |
| 1          | Alice        | Physics      | 0              |
| 2          | Bob          | Math         | 0              |
| 2          | Bob          | Physics      | 1              |

### Thinking
1. Cartesian product (Students × Subjects) to guarantee every pair.
2. LEFT JOIN Examinations to count matches.
3. GROUP BY all key columns.

### Query
```sql
SELECT s.student_id, s.student_name, subj.subject_name,
       COUNT(e.subject_name) AS attended_exams
FROM Students s
CROSS JOIN Subjects subj
LEFT JOIN Examinations e
  ON e.student_id = s.student_id
 AND e.subject_name = subj.subject_name
GROUP BY s.student_id, s.student_name, subj.subject_name
ORDER BY s.student_id, subj.subject_name;
```

### Pitfalls
- Using INNER JOIN eliminates zero attendance pairs.
- Counting * might count unintended columns if duplicated join.

### Variations
- Include percentage of total exams per student.
- Show only subjects with zero attendance (add HAVING COUNT(*)=0).

---
## 8. Managers with at Least 5 Direct Reports
**Goal (paraphrased)**: List names of employees who are managers of 5+ distinct direct reports.

### Schema
`Employee(id INT PK, name VARCHAR, managerId INT NULL)`
- managerId references Employee.id.

### Sample Data
| id | name   | managerId |
|----|--------|-----------|
| 1  | Alice  | NULL      |
| 2  | Bob    | 1         |
| 3  | Carol  | 1         |
| 4  | Dan    | 1         |
| 5  | Erin   | 1         |
| 6  | Frank  | 1         |
| 7  | Grace  | 2         |

### Expected Output
| name  |
|-------|
| Alice |

### Thinking
1. Self-join employee table: subordinates referencing managerId.
2. Group by manager id; HAVING count(*) >= 5.
3. Join back (or aggregate directly using self alias) to get manager name.

### Query
```sql
SELECT m.name
FROM Employee m
JOIN Employee e ON e.managerId = m.id
GROUP BY m.id, m.name
HAVING COUNT(*) >= 5
ORDER BY m.name;
```

### Pitfalls
- Counting DISTINCT unnecessary if each subordinate row unique; add if duplicates possible.
- Missing manager name if forgetting self alias.

### Variations
- List manager id plus direct_report_count.
- Threshold parameterization.

---
## 9. Confirmation Rate (Medium)
**Goal (paraphrased)**: For each user, compute confirmation rate = confirmed actions / total actions (0 if none). Round or format depending on platform rounding rules.

### Schemas
`Signups(user_id INT PK, time_joined DATETIME)`
`Confirmations(user_id INT, action VARCHAR, time DATETIME)` where action in ('confirmed','timeout', ...)

### Sample Data
Signups:
| user_id | time_joined        |
|---------|--------------------|
| 1       | 2025-01-01 09:00:00 |
| 2       | 2025-01-01 09:05:00 |
| 3       | 2025-01-01 10:00:00 |

Confirmations:
| user_id | action     | time                |
|---------|------------|---------------------|
| 1       | confirmed  | 2025-01-01 09:01:00 |
| 1       | timeout    | 2025-01-01 09:02:30 |
| 2       | timeout    | 2025-01-01 09:06:10 |
| 2       | timeout    | 2025-01-01 09:07:00 |

### Expected Output (example rounding 2 decimals)
| user_id | confirmation_rate |
|---------|-------------------|
| 1       | 0.50              |  -- 1 confirmed / 2 total
| 2       | 0.00              |  -- 0 / 2
| 3       | 0.00              |  -- no actions → 0

### Thinking
1. All users from Signups must appear (even with zero confirmations) ⇒ LEFT JOIN.
2. Need numerator: count of rows with action='confirmed'.
3. Denominator: total confirmation attempts per user.
4. Handle zero denominator returning 0 not NULL (use COALESCE or CASE).
5. Apply rounding formatting.

### Query (ANSI style)
```sql
SELECT s.user_id,
       ROUND(COALESCE(SUM(CASE WHEN c.action = 'confirmed' THEN 1 ELSE 0 END) * 1.0 /
                     NULLIF(COUNT(c.action), 0), 0), 2) AS confirmation_rate
FROM Signups s
LEFT JOIN Confirmations c ON c.user_id = s.user_id
GROUP BY s.user_id
ORDER BY s.user_id;
```

### Pitfalls
- Using INNER JOIN drops users with no confirmation attempts.
- Division by zero without NULLIF.
- Integer division; multiply numerator by 1.0 or cast to decimal.
- Rounding differences: some platforms require FORMAT or CAST with decimal precision.

### Variations
- Show raw counts alongside rate.
- Compute weekly aggregated rates (GROUP BY user_id, week(time)).

---
## Consolidated Pattern Summary
| Problem | Join Type | Core Extra Logic | Key Function / Clause |
|---------|-----------|------------------|-----------------------|
| Replace Employee ID | LEFT | Preserve all employees | LEFT JOIN + select nullable mapping |
| Product Sales Analysis I | INNER | Aggregate quantity | SUM + GROUP BY |
| Customer No Transaction | LEFT | Anti-join (NULL filter) | WHERE right.key IS NULL |
| Rising Temperature | SELF | Date adjacency compare | Self join or LAG() |
| Avg Time per Machine | SELF | Pair start/end & average | WITH + duration calc |
| Employee Bonus | LEFT | Include missing bonus rows | OR bonus IS NULL |
| Students & Exams | CROSS + LEFT | Complete grid & count | CROSS JOIN + COUNT |
| Managers ≥ 5 Reports | SELF | Count subordinates | HAVING COUNT(*) >= N |
| Confirmation Rate | LEFT | Conditional numerator & safe division | CASE + NULLIF + ROUND |

---
## Thinking Framework for Join Problems
1. Baseline set: Which table defines required output row set? Use it on LEFT side.
2. Cardinality: One-to-one? One-to-many? Prevent duplicates before aggregation.
3. Inclusion: Need zero-count rows? Use outer join + COUNT(col) pattern.
4. Anti / Semi logic: Use LEFT JOIN ... IS NULL (anti) or EXISTS (semi) depending on direction.
5. Metrics: Use conditional aggregation for labeled counts in single pass.
6. Stability: Enforce ordering only if required by spec.

---
## Index / Performance Micro Notes
| Scenario | Helpful Index |
|----------|---------------|
| Foreign key joins | Child table FK column |
| Self join on managerId | (managerId) |
| Weather date adjacency | (recordDate) (support date lookup) |
| Activity pairing | (machine_id, process_id, activity_type) |
| Confirmation Rate | (user_id, action) covering counts |

For small practice datasets, full scans are fine; indexes matter at scale.

---
## Practice Extensions
- Convert Rising Temperature self join to window form & compare readability.
- Extend Confirmation Rate to rolling 7-day window (windowed SUM/COUNT filter by time).
- Add a query to list managers and depth of management chain (recursive CTE).
- Modify Students & Exams to show percentage of subjects attended.
- Add anomaly detection for Activity durations > 95th percentile.

Use this set to cement join selection, anti-join patterns, conditional aggregation, and safe division techniques.
