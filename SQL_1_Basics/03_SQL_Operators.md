# SQL Operators

## Overview
SQL operators are symbols and keywords used to perform operations on data in queries. They're essential for filtering, comparing, and manipulating data. Interviewers test your ability to use operators effectively in WHERE clauses, JOIN conditions, and calculated fields.

## Core Concepts

### 1. **Arithmetic Operators**

Used for mathematical calculations.

```sql
+   -- Addition
-   -- Subtraction
*   -- Multiplication
/   -- Division
%   -- Modulus (remainder)

-- Examples:
SELECT 
    price,
    price * 1.10 AS price_with_tax,      -- 10% tax
    quantity % 2 AS is_odd,               -- Odd/even check
    total_amount / quantity AS unit_price
FROM orders;
```

### 2. **Comparison Operators**

Used to compare values in WHERE, HAVING, and JOIN conditions.

```sql
=       -- Equal to
!=, <>  -- Not equal to (both work)
>       -- Greater than
<       -- Less than
>=      -- Greater than or equal to
<=      -- Less than or equal to

-- Examples:
SELECT * FROM products WHERE price >= 100;
SELECT * FROM users WHERE status != 'inactive';
SELECT * FROM orders WHERE order_date > '2025-01-01';
```

### 3. **Logical Operators**

Combine multiple conditions.

```sql
AND     -- All conditions must be TRUE
OR      -- At least one condition must be TRUE
NOT     -- Negates a condition

-- Examples:
SELECT * FROM products 
WHERE category = 'Electronics' 
  AND price < 1000 
  AND in_stock = TRUE;

SELECT * FROM users 
WHERE country = 'USA' 
   OR country = 'Canada';

SELECT * FROM orders 
WHERE NOT status = 'cancelled';
```

**Operator Precedence**: `NOT` > `AND` > `OR` (use parentheses for clarity!)

```sql
-- Without parentheses (confusing):
WHERE a = 1 OR b = 2 AND c = 3  -- Evaluates as: a=1 OR (b=2 AND c=3)

-- With parentheses (clear):
WHERE (a = 1 OR b = 2) AND c = 3
```

### 4. **Special Operators**

#### **BETWEEN** (Range Check)
```sql
-- Inclusive range
SELECT * FROM products 
WHERE price BETWEEN 100 AND 500;

-- Equivalent to:
WHERE price >= 100 AND price <= 500;

-- With dates:
SELECT * FROM orders 
WHERE order_date BETWEEN '2025-01-01' AND '2025-12-31';
```

#### **IN** (Multiple Values)
```sql
-- Check if value exists in a list
SELECT * FROM users 
WHERE country IN ('USA', 'UK', 'Canada');

-- Equivalent to:
WHERE country = 'USA' OR country = 'UK' OR country = 'Canada';

-- With subquery:
SELECT * FROM orders 
WHERE customer_id IN (SELECT id FROM vip_customers);
```

#### **LIKE** (Pattern Matching)
```sql
-- Wildcards:
%   -- Zero or more characters
_   -- Exactly one character

SELECT * FROM users WHERE email LIKE '%@gmail.com';      -- Ends with
SELECT * FROM products WHERE name LIKE 'Apple%';          -- Starts with
SELECT * FROM products WHERE sku LIKE 'A_C%';             -- A, any char, C, then anything
SELECT * FROM users WHERE name LIKE '%john%';             -- Contains 'john'

-- Case sensitivity varies by database!
-- MySQL: LIKE is case-insensitive by default
-- PostgreSQL: LIKE is case-sensitive, use ILIKE for insensitive
```

#### **IS NULL / IS NOT NULL**
```sql
-- Check for NULL values (NULL is "unknown", not zero or empty string)
SELECT * FROM users WHERE phone IS NULL;
SELECT * FROM products WHERE description IS NOT NULL;

-- WRONG WAY (doesn't work with NULL):
-- WHERE phone = NULL    ❌
-- WHERE phone != NULL   ❌
```

#### **EXISTS** (Subquery Check)
```sql
-- Returns TRUE if subquery returns any rows
SELECT * FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.id
);

-- More efficient than IN for large datasets
```

### 5. **Bitwise Operators** (Advanced)

```sql
&   -- Bitwise AND
|   -- Bitwise OR
^   -- Bitwise XOR
~   -- Bitwise NOT

-- Example: Permission flags
SELECT * FROM users 
WHERE permissions & 4 = 4;  -- Check if read permission (bit 3) is set
```

### 6. **String Operators**

```sql
-- Concatenation (varies by database)
||              -- SQL standard (PostgreSQL, SQLite)
CONCAT()        -- MySQL, SQL Server
+               -- SQL Server only

-- Examples:
SELECT first_name || ' ' || last_name AS full_name FROM users;  -- PostgreSQL
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;  -- MySQL
```

### 7. **Set Operators**

Combine results of multiple queries.

```sql
UNION       -- Combines results, removes duplicates
UNION ALL   -- Combines results, keeps duplicates (faster)
INTERSECT   -- Returns common rows
EXCEPT      -- Returns rows in first query but not in second (MINUS in Oracle)

-- Example:
SELECT name FROM customers_usa
UNION
SELECT name FROM customers_canada;

-- Performance tip: Use UNION ALL if duplicates don't matter (faster)
```

## Interview-Focused Notes

### Common Interview Questions

1. **"What's the difference between AND and OR?"**
   - AND: All conditions must be true
   - OR: At least one condition must be true

2. **"How do you check for NULL values?"**
   - Use `IS NULL` or `IS NOT NULL` (never use = or !=)

3. **"BETWEEN vs >= and <="**
   - BETWEEN is inclusive (includes both endpoints)
   - More readable, but same performance

4. **"IN vs OR: Which is better?"**
   - IN is cleaner and more readable
   - Performance is similar for small lists
   - IN with subquery can be optimized differently

5. **"What's the difference between UNION and UNION ALL?"**
   - UNION removes duplicates (slower)
   - UNION ALL keeps duplicates (faster)

### How to Explain in Interviews
"SQL operators allow us to filter and manipulate data efficiently. I always use the most readable operator for the task—for example, BETWEEN for ranges and IN for multiple values. Understanding operator precedence is crucial, especially when combining AND/OR, which is why I use parentheses to make conditions explicit."

## Quick Recall ✅

### Must-Remember Operators
```
Comparison:  =, !=, >, <, >=, <=
Logical:     AND, OR, NOT
Range:       BETWEEN ... AND ...
List:        IN (value1, value2, ...)
Pattern:     LIKE with % and _
NULL check:  IS NULL, IS NOT NULL
Existence:   EXISTS (subquery)
Set:         UNION, UNION ALL, INTERSECT, EXCEPT
```

### Operator Precedence (High to Low)
```
1. Arithmetic (*, /, %)
2. Arithmetic (+, -)
3. Comparison (=, !=, >, <, etc.)
4. NOT
5. AND
6. OR
```

### Wildcards for LIKE
```
%     → Zero or more characters
_     → Exactly one character

Examples:
'A%'     → Starts with A
'%Z'     → Ends with Z
'%SQL%'  → Contains SQL
'A_C'    → A, any single char, then C
```

## Interview Traps & Confusions ⚠️

### Common Mistakes

1. **NULL Comparison**
   ```sql
   ❌ WHERE column = NULL
   ❌ WHERE column != NULL
   ✅ WHERE column IS NULL
   ✅ WHERE column IS NOT NULL
   ```

2. **LIKE Case Sensitivity**
   - MySQL: `LIKE` is case-insensitive (default)
   - PostgreSQL: `LIKE` is case-sensitive, use `ILIKE`
   - Always verify database behavior!

3. **AND/OR Precedence**
   ```sql
   -- This query is ambiguous:
   WHERE a = 1 OR b = 2 AND c = 3
   
   -- Evaluates as: a = 1 OR (b = 2 AND c = 3)
   
   ✅ Use parentheses:
   WHERE (a = 1 OR b = 2) AND c = 3
   ```

4. **BETWEEN Inclusivity**
   ```sql
   WHERE age BETWEEN 18 AND 25  -- Includes 18 AND 25
   
   -- If you need exclusive:
   WHERE age > 18 AND age < 25  -- Excludes 18 and 25
   ```

5. **Empty String vs NULL**
   ```sql
   '' IS NOT NULL   -- TRUE (empty string is a value)
   NULL IS NOT NULL -- FALSE
   
   SELECT * WHERE name = '';    -- Finds empty strings
   SELECT * WHERE name IS NULL; -- Finds NULL values
   ```

6. **IN with NULL**
   ```sql
   WHERE column IN (1, 2, NULL)
   -- If column is NULL, this does NOT match!
   -- NULL in IN list is ignored in comparisons
   ```

### Tricky Interview Scenarios

**Q: "What's the output?"**
```sql
SELECT * FROM users 
WHERE age = 25 OR age = 30 AND country = 'USA';

❌ Answer: "Users aged 25 or 30 from USA"
✅ Answer: "Users aged 25 (any country) OR (aged 30 AND from USA)"
Reason: AND has higher precedence than OR
```

**Q: "Why doesn't this work?"**
```sql
SELECT * FROM products WHERE price = NULL;

❌ Returns no results (NULL can't be compared with =)
✅ Use: WHERE price IS NULL
```

**Q: "Difference between != and <>"**
```sql
Answer: They're identical! Both mean "not equal"
!= is more common, <> is SQL standard
```

**Q: "What's faster: IN or OR?"**
```sql
-- Generally similar performance for small lists
-- IN is more readable
-- Database optimizer often converts them to the same execution plan

WHERE id IN (1, 2, 3)       -- Preferred
WHERE id = 1 OR id = 2 OR id = 3  -- Works but verbose
```

## Bonus

### Related Concepts
- **WHERE vs HAVING** → Operators work in both, but HAVING is for aggregates
- **JOIN Conditions** → Use comparison operators to define relationships
- **Indexes** → Operators affect index usage (LIKE 'a%' uses index, LIKE '%a' doesn't)
- **Query Optimization** → Operator choice impacts execution plans

### Real-World Query Examples

#### E-commerce Product Search
```sql
SELECT * FROM products
WHERE category IN ('Electronics', 'Computers')
  AND price BETWEEN 500 AND 2000
  AND (brand = 'Apple' OR brand = 'Samsung')
  AND in_stock = TRUE
  AND name LIKE '%laptop%';
```

#### Customer Segmentation
```sql
SELECT * FROM customers
WHERE (total_purchases > 1000 OR vip_status = TRUE)
  AND country IN ('USA', 'UK', 'Canada')
  AND email IS NOT NULL
  AND last_purchase_date >= DATE_SUB(CURRENT_DATE, INTERVAL 90 DAY);
```

#### Data Cleanup
```sql
-- Find incomplete records
SELECT * FROM users
WHERE phone IS NULL 
   OR email NOT LIKE '%@%.%'
   OR address = '';
```

### Performance Tips

✅ **DO:**
- Use `IN` instead of multiple `OR` conditions
- Use `EXISTS` instead of `IN` for subqueries with large result sets
- Use `BETWEEN` for readability
- Use indexes on columns in WHERE clauses

❌ **AVOID:**
- `LIKE '%text%'` (can't use index)
- `OR` on different columns (hard to optimize)
- Functions on indexed columns: `WHERE UPPER(name) = 'JOHN'` (index not used)

### Quick Reference Table

| Operator | Use Case | Example |
|----------|----------|---------|
| `=` | Exact match | `WHERE id = 5` |
| `IN` | Multiple values | `WHERE id IN (1,2,3)` |
| `BETWEEN` | Range | `WHERE age BETWEEN 18 AND 65` |
| `LIKE` | Pattern | `WHERE email LIKE '%@gmail.com'` |
| `IS NULL` | NULL check | `WHERE phone IS NULL` |
| `EXISTS` | Subquery | `WHERE EXISTS (SELECT ...)` |
| `AND` | All conditions | `WHERE a = 1 AND b = 2` |
| `OR` | Any condition | `WHERE a = 1 OR b = 2` |

---

**Pro Tip**: Always use parentheses in complex conditions for clarity and to avoid precedence bugs!
