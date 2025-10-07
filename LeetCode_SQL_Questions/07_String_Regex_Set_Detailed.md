# String / Regex / Clause Set 01 – Detailed Walkthroughs (Schemas • Sample Data • Reasoning • Queries)

Problems (paraphrased; focus on string normalization, pattern matching, conditional filtering, deduplication, positional ranking):
1. Fix Names in a Table (Formatting / Case Normalization)
2. Patients With a Condition (Token Pattern Search)
3. Delete Duplicate Emails (De-Dup Keep Lowest Id)
4. Second Highest Salary (Ranking / Distinct Ordering)
5. Group Sold Products By The Date (Aggregation + Concatenation)
6. List the Products Ordered in a Period (Date Range Filter + Aggregate)
7. Find Users With Valid E-Mails (Regex Validation)

Format: Schema • Sample Input • Expected Output • Thinking • Query • Pitfalls • Variations.

---
## 1. Fix Names in a Table
**Goal (paraphrased)**: Normalize full names so first letter uppercase, remaining letters lowercase.

### Schema
`Users(user_id INT PK, name VARCHAR)`  -- names may be inconsistently cased.

### Sample Data
| user_id | name        |
|--------:|-------------|
| 1       | alICe       |
| 2       | BOB         |
| 3       | charlie     |

### Expected Output (SELECT form)
| user_id | name   |
|--------:|--------|
| 1       | Alice  |
| 2       | Bob    |
| 3       | Charlie|

### Thinking
- Need: UPPER(first char) + LOWER(rest).
- MySQL: `CONCAT(UPPER(LEFT(name,1)), LOWER(SUBSTRING(name,2)))`
- Postgres: `INITCAP(name)` (but can alter behavior for apostrophes). Provide universal expression.

### Query (Non-destructive SELECT)
```sql
SELECT user_id,
       CONCAT(UPPER(LEFT(name,1)), LOWER(SUBSTRING(name,2))) AS name
FROM Users
ORDER BY user_id;
```

### Update Variant
```sql
UPDATE Users
SET name = CONCAT(UPPER(LEFT(name,1)), LOWER(SUBSTRING(name,2)));
```

### Pitfalls
- Names with leading spaces (TRIM first).
- Multi-word names need per-word capitalization (split logic or use INITCAP/REGEXP_REPLACE if available).

### Variations
- Title-case each word: use regex or split (Postgres: `INITCAP(TRIM(name))`).
- Preserve certain lowercase particles ("van", "de") via post-adjust CASE mapping.

---
## 2. Patients With a Condition
**Goal (paraphrased)**: Return patient_ids whose `conditions` column (space-separated codes) contains code starting with 'DIAB1'. Must match whole token (avoid partial inside another code without delimiter) and may appear at beginning or after space.

### Schema
`Patients(patient_id INT PK, conditions VARCHAR)`  -- example: 'A12 DIAB100 X5', 'DIAB100', 'A1 DIAB1X'

### Sample Data
| patient_id | conditions           |
|-----------:|----------------------|
| 10         | DIAB100 X5           |
| 11         | A12 DIAB1A           |
| 12         | DIAB199 A12          |
| 13         | A12 B50              |

Assume we want tokens starting with 'DIAB1' (prefix) → patient 10 (DIAB100) & 12 (DIAB199); patient 11 has DIAB1A (still prefix) depending on spec; adapt as needed.

### Expected Output (prefix interpretation)
| patient_id |
|-----------:|
| 10         |
| 11         |
| 12         |

### Thinking
- Need boundary: start of string OR space before token.
- Pattern forms:
  - LIKE: `conditions LIKE 'DIAB1%' OR conditions LIKE '% DIAB1%'`
  - Regex (MySQL 8+): `REGEXP '(?:^| )DIAB1'` (if only presence; extend with `[^ ]*` for variable suffix).
- If require entire code exactly equals 'DIAB1': add boundary `( |$)`.

### Query (Flexible prefix)
```sql
SELECT patient_id
FROM Patients
WHERE conditions LIKE 'DIAB1%'
   OR conditions LIKE '% DIAB1%'
ORDER BY patient_id;
```

### Regex Variant (exact code or with trailing digits/letters) – MySQL
```sql
SELECT patient_id
FROM Patients
WHERE conditions REGEXP '(^| )DIAB1';
```

### Pitfalls
- `NOT LIKE` misuse excluding start-of-string.
- Over-matching (e.g., DIAB12 vs DIAB1 wanted). Add tighter regex: `(^| )DIAB1([A-Za-z0-9]*)`.

### Variations
- Exact token match only: regex `(^| )DIAB1( |$)`.
- Case-insensitive: apply `UPPER(conditions)` or use case-insensitive regex flag.

---
## 3. Delete Duplicate Emails
**Goal (paraphrased)**: Remove duplicate email rows, keeping the one with smallest id.

### Schema
`Person(id INT PK, email VARCHAR)`

### Sample Data
| id | email             |
|----|-------------------|
| 1  | a@x.com           |
| 2  | b@x.com           |
| 3  | a@x.com           |

### Expected Post-Delete
| id | email   |
|----|---------|
| 1  | a@x.com |
| 2  | b@x.com |

### Thinking
- Identify duplicates where id > MIN(id) for same email.
- Use self-join or subquery.
- MySQL supports DELETE alias join pattern.

### Delete Query (MySQL style)
```sql
DELETE p1
FROM Person p1
JOIN Person p2 ON p2.email = p1.email AND p2.id < p1.id;
```

### ANSI-Compatible Approach (Two-Step)
1. Find IDs to remove.
```sql
SELECT id FROM Person p
WHERE id NOT IN (
  SELECT MIN(id) FROM Person GROUP BY email
);
```
2. Then delete using those IDs.

### Pitfalls
- NOT IN with NULL emails (ensure email NOT NULL or use NOT EXISTS).
- Accidentally keeping highest id by wrong join direction.

### Variations
- Replace duplicates by updating to canonical id mapping table.
- Count duplicates before deletion for audit.

---
## 4. Second Highest Salary
**Goal (paraphrased)**: Return second distinct highest salary (NULL if not present).

### Schema
`Employee(id INT PK, salary INT)`

### Sample Data
| id | salary |
|----|-------:|
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
| 4  | 300    |

### Expected Output
| SecondHighestSalary |
|---------------------|
| 200                 |

### Thinking
- Distinct ordering descending skip first.
- Solutions:
  1. Subquery with OFFSET.
  2. HAVING count of greater salaries.
  3. Window function: DENSE_RANK.

### Query (Distinct + OFFSET) MySQL/Postgres
```sql
SELECT (
  SELECT DISTINCT salary
  FROM Employee
  ORDER BY salary DESC
  LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

### Window Variant
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS r
  FROM Employee
) x
WHERE r = 2;
```

### Pitfalls
- Using ROW_NUMBER leads to skipping ties incorrectly.
- Using `MAX(salary)` with `salary < (SELECT MAX(salary))` returns highest below even if duplicates—OK but returns NULL if only one distinct.

### Variations
- Third highest: adjust OFFSET or r=3.
- Return top N distinct with ranks.

---
## 5. Group Sold Products By The Date
**Goal (paraphrased)**: For each sell_date, return count of distinct products sold and a comma-separated list of product names ordered lexicographically.

### Schemas
`Activities(sell_date DATE, product VARCHAR)` (one row per sale event).

### Sample Data
| sell_date  | product |
|------------|---------|
| 2025-01-01 | Apple   |
| 2025-01-01 | Apple   |
| 2025-01-01 | Banana  |
| 2025-01-02 | Carrot  |
| 2025-01-02 | Banana  |

### Expected Output
| sell_date  | num_sold | products          |
|------------|----------|-------------------|
| 2025-01-01 | 2        | Apple,Banana      |
| 2025-01-02 | 2        | Banana,Carrot     |

### Thinking
- Distinct product count per date.
- Sorted concatenated distinct product names.
- Use GROUP_CONCAT / STRING_AGG with ORDER BY.

### Query (MySQL)
```sql
SELECT sell_date,
       COUNT(DISTINCT product) AS num_sold,
       GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

### Postgres Variant
```sql
SELECT sell_date,
       COUNT(DISTINCT product) AS num_sold,
       STRING_AGG(DISTINCT product, ',' ORDER BY product) AS products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

### Pitfalls
- Omitting DISTINCT in concatenation causing duplicates listed.
- Missing ORDER BY inside aggregator leads to arbitrary order.

### Variations
- Add total event count separate from distinct product count.
- Output JSON array (Postgres: `json_agg(DISTINCT product ORDER BY product)`).

---
## 6. List the Products Ordered in a Period
**Goal (paraphrased)**: List product names and total quantity sold for orders within a date range; exclude products not sold in that range.

### Schemas
`Orders(order_id INT, order_date DATE, product_id INT, quantity INT)`
`Products(product_id INT PK, product_name VARCHAR)`

### Sample Data
Orders:
| order_id | order_date  | product_id | quantity |
|----------|-------------|-----------:|--------:|
| 1        | 2025-03-01  | 10         | 3       |
| 2        | 2025-03-04  | 11         | 5       |
| 3        | 2025-04-02  | 10         | 2       |

Products:
| product_id | product_name |
|------------|--------------|
| 10         | Keyboard     |
| 11         | Mouse        |
| 12         | Monitor      |

Date window: 2025-03-01 to 2025-03-31 inclusive.

### Expected Output
| product_name | total_quantity |
|--------------|----------------|
| Keyboard     | 3              |
| Mouse        | 5              |

### Thinking
1. Filter orders by date range using range predicate (avoid function on column).
2. Join to Products for names.
3. Aggregate quantity per product.
4. Sort output (by product_name or total desc as required).

### Query
```sql
SELECT p.product_name, SUM(o.quantity) AS total_quantity
FROM Orders o
JOIN Products p ON p.product_id = o.product_id
WHERE o.order_date BETWEEN DATE '2025-03-01' AND DATE '2025-03-31'
GROUP BY p.product_name
ORDER BY p.product_name;
```

### Pitfalls
- Using `MONTH(order_date)=3` prevents index usage.
- Inclusive end-date off-by-one if using `<` vs `<=` pattern.

### Variations
- Sort by total_quantity DESC then name.
- Add cumulative share (window function) across products in range.

---
## 7. Find Users With Valid E-Mails
**Goal (paraphrased)**: Return users whose email matches allowed pattern: local part may contain letters, digits, underscores, dots or hyphens; domain only letters; extension 2–3 letters.

### Schema
`Users(user_id INT PK, email VARCHAR)`

### Sample Data
| user_id | email                |
|--------:|----------------------|
| 1       | good_user@test.com   |
| 2       | BadUser@domain.toolong |
| 3       | bad..dots@test.co    |
| 4       | ok-1@Example.ORG     | (case-insensitive) |

### Expected Output (user_ids 1 & 4) – depending on strictness of consecutive dots rule.
| user_id | email              |
|--------:|--------------------|
| 1       | good_user@test.com |
| 4       | ok-1@Example.ORG   |

### Thinking
- Regex components: `^` anchor, local `[A-Za-z0-9._-]+`, `@`, domain `[A-Za-z]+`, dot, extension `[A-Za-z]{2,3}` `$`.
- Case-insensitive comparison; many DB regex engines already case-insensitive or need `LOWER(email)`.

### Query (MySQL)
```sql
SELECT user_id, email
FROM Users
WHERE email REGEXP '^[A-Za-z0-9._-]+@[A-Za-z]+\.[A-Za-z]{2,3}$';
```

### Case-Insensitive Normalization
```sql
SELECT user_id, email
FROM Users
WHERE LOWER(email) REGEXP '^[a-z0-9._-]+@[a-z]+\.[a-z]{2,3}$';
```

### Pitfalls
- Allowing multiple consecutive dots (if restricted, refine: `(?!.*\.\.)` negative lookahead—Note: MySQL 8 does not support lookahead; need alternative logic or accept).
- Allowing numeric-only TLD (excluded by letter pattern).

### Variations
- Extended TLD length: change `{2,3}` to `{2,10}`.
- Enforce at least one letter in local part: add `[A-Za-z0-9._-]*[A-Za-z][A-Za-z0-9._-]*`.

---
## Consolidated Pattern Summary
| Problem | Core Pattern | Key Function / Clause | Risk |
|---------|--------------|-----------------------|------|
| Fix Names | String slicing & case | CONCAT, UPPER, LOWER | Multi-word edge cases |
| Patients Condition | Token boundary match | LIKE with space or REGEXP | Partial over-match |
| Delete Duplicate Emails | De-dup keep min id | Self-join DELETE | NULL / join direction |
| Second Highest Salary | Distinct top-k | ORDER + LIMIT/OFFSET or DENSE_RANK | Ties mishandled |
| Group Sold Products By Date | Distinct aggregation + concat | GROUP_CONCAT / STRING_AGG | Missing DISTINCT/ORDER |
| Products in Period | Range filter + aggregate | BETWEEN date range | Non-sargable functions |
| Valid E-Mails | Regex validation | REGEXP anchor pattern | Overly permissive local part |

---
## Thinking Framework (Strings / Regex / Clauses)
1. Normalize first (TRIM / case) before pattern checks.
2. Decide if pattern requires token boundary vs substring; add anchors or spaces accordingly.
3. Distinct vs total: ensure correct use of DISTINCT inside both count & concatenation.
4. Ranking with ties: DENSE_RANK vs OFFSET; prefer window for portability.
5. Validation queries: anchor regex with ^ and $ to avoid partial matches.
6. Range filtering: use BETWEEN or inclusive/exclusive boundaries without wrapping column in functions.

---
## Common Pitfalls & Fixes
| Pitfall | Fix |
|---------|-----|
| `NOT IN` with NULL underlying values | Use NOT EXISTS or ensure NOT NULL PK set |
| Incorrect capitalization for multi-word names | Use word-wise INITCAP or regex replace |
| Over-matching condition codes | Include boundary pattern `( |^)` & `( |$)` |
| Missing tie handling | Use DENSE_RANK for distinct rank logic |
| Unordered product list | ORDER BY inside GROUP_CONCAT/STRING_AGG |
| Regex partial match | Anchor both ends | 

---
## Practice Extensions
- Extend Fix Names to proper case each word excluding particles (e.g., 'of','and').
- Modify Patients query to return only *exact* code matches (no prefix) with regex.
- Add rank for each salary and list employees with top 3 distinct salaries (merge with earlier ranking pattern).
- Build monthly concatenated product lists limited to top 5 alphabetically.
- Add email invalid reason classification (CASE on various regex tests).

Use these patterns to strengthen string manipulation, pattern matching, and ranking logic energy in SQL interview settings.
