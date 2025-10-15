# LeetCode Easy #584: Find Customer Referee

## 📋 Problem Statement

Write a SQL query to report the names of the customer that are **not referred by the customer with id = 2**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Customer Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| referee_id  | int     |
+-------------+---------+
```
- `id` is the primary key column for this table.
- Each row indicates the id and the name of a customer and the id of the customer who referred them.

## 📊 Sample Data

**Customer Table:**
| id | name | referee_id |
|----|------|------------|
| 1  | Will | null       |
| 2  | Jane | null       |
| 3  | Alex | 2          |
| 4  | Bill | null       |
| 5  | Zack | 1          |
| 6  | Mark | 2          |

**Expected Output:**
| name |
|------|
| Will |
| Jane |
| Bill |
| Zack |

**Explanation:** Customers Alex and Mark were referred by customer 2, so they are excluded. All others are included.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find customers NOT referred by customer with id = 2
- This involves NULL handling (customers with no referee)
- Include customers with referee_id != 2 OR referee_id IS NULL
- Exclude only those where referee_id = 2

### 2. **Key Insights**
- NULL values need special handling - they should be included
- Cannot use `!= 2` alone because NULL != 2 evaluates to UNKNOWN, not TRUE
- Need explicit NULL check with OR condition
- This is a common SQL NULL handling trap

### 3. **Interview Discussion Points**
- "I need to handle NULL values carefully since they represent customers with no referee"
- "NULL != 2 doesn't work as expected - need explicit IS NULL check"
- "This tests understanding of three-valued logic in SQL"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Exclusion Criteria
```sql
-- Exclude only customers where referee_id = 2
-- Include all others: referee_id != 2 OR referee_id IS NULL
```

### Step 2: Handle NULL Values
```sql
-- NULL values should be included (no referee means not referred by customer 2)
-- Use OR condition to handle both cases
```

### Step 3: Select Required Output
```sql
-- Return only customer names
```

## ✅ Optimized SQL Solution

```sql
SELECT name
FROM Customer
WHERE referee_id != 2 OR referee_id IS NULL;
```

### Alternative Solutions

**Using COALESCE/ISNULL:**
```sql
-- MySQL/PostgreSQL
SELECT name
FROM Customer
WHERE COALESCE(referee_id, 0) != 2;

-- SQL Server
SELECT name
FROM Customer
WHERE ISNULL(referee_id, 0) != 2;
```

**Using NOT IN with NULL handling:**
```sql
SELECT name
FROM Customer
WHERE referee_id NOT IN (2) OR referee_id IS NULL;
-- Note: NOT IN with NULLs requires explicit NULL handling
```

**Using CASE statement for clarity:**
```sql
SELECT name
FROM Customer
WHERE CASE 
    WHEN referee_id IS NULL THEN 1
    WHEN referee_id != 2 THEN 1
    ELSE 0
END = 1;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Referral Chain Analysis**
```sql
-- "Find all customers in a referral chain starting from customer 2"

WITH RECURSIVE ReferralChain AS (
    -- Base case: Direct referrals from customer 2
    SELECT id, name, referee_id, 1 as level
    FROM Customer
    WHERE referee_id = 2
    
    UNION ALL
    
    -- Recursive case: Referrals from previously found customers
    SELECT c.id, c.name, c.referee_id, rc.level + 1
    FROM Customer c
    INNER JOIN ReferralChain rc ON c.referee_id = rc.id
    WHERE rc.level < 5  -- Prevent infinite recursion
)
SELECT level, name, id
FROM ReferralChain
ORDER BY level, name;
```

#### 2. **Referral Statistics**
```sql
-- "Analyze referral patterns and statistics"

SELECT 
    CASE 
        WHEN referee_id IS NULL THEN 'Self-Acquired'
        ELSE CONCAT('Referred by Customer ', referee_id)
    END AS referral_source,
    COUNT(*) AS customer_count,
    GROUP_CONCAT(name ORDER BY name) AS customers
FROM Customer
GROUP BY referee_id
ORDER BY customer_count DESC;
```

#### 3. **Top Referrers Analysis**
```sql
-- "Find customers who have referred the most people"

SELECT 
    r.id AS referrer_id,
    r.name AS referrer_name,
    COUNT(c.id) AS referral_count,
    GROUP_CONCAT(c.name ORDER BY c.name) AS referred_customers
FROM Customer r
INNER JOIN Customer c ON r.id = c.referee_id
GROUP BY r.id, r.name
ORDER BY referral_count DESC, r.name;
```

#### 4. **Data Quality Validation**
```sql
-- "Check for data quality issues in referral data"

-- Check for self-referrals (customer referring themselves)
SELECT 'Self-Referrals' AS issue, COUNT(*) AS count
FROM Customer
WHERE id = referee_id

UNION ALL

-- Check for circular referrals (A refers B, B refers A)
SELECT 'Potential Circular Referrals' AS issue, COUNT(*) AS count
FROM Customer c1
INNER JOIN Customer c2 ON c1.referee_id = c2.id AND c2.referee_id = c1.id

UNION ALL

-- Check for invalid referee IDs (referrer doesn't exist)
SELECT 'Invalid Referee IDs' AS issue, COUNT(*) AS count
FROM Customer c1
LEFT JOIN Customer c2 ON c1.referee_id = c2.id
WHERE c1.referee_id IS NOT NULL AND c2.id IS NULL;
```

#### 5. **Business Impact Analysis**
```sql
-- "Calculate referral program effectiveness"

WITH ReferralStats AS (
    SELECT 
        COUNT(*) as total_customers,
        COUNT(referee_id) as referred_customers,
        COUNT(DISTINCT referee_id) as active_referrers
    FROM Customer
)
SELECT 
    total_customers,
    referred_customers,
    total_customers - referred_customers as self_acquired,
    active_referrers,
    ROUND(referred_customers * 100.0 / total_customers, 2) as referral_rate,
    ROUND(referred_customers * 1.0 / NULLIF(active_referrers, 0), 2) as avg_referrals_per_referrer
FROM ReferralStats;
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (NULL handling in JOINs)
2. **#183 - Customers Who Never Order** (NOT EXISTS patterns)
3. **#577 - Employee Bonus** (LEFT JOIN with NULL filtering)
4. **#1148 - Article Views I** (Basic filtering)
5. **#1407 - Top Travellers** (NULL handling in aggregation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Three-Valued Logic**: SQL uses TRUE, FALSE, and UNKNOWN (NULL)
2. **NULL Handling**: NULL != value is UNKNOWN, not TRUE
3. **OR with IS NULL**: Standard pattern for including NULL values
4. **COALESCE/ISNULL**: Alternative approaches for NULL handling

### 🚀 **Amazon Interview Tips**
1. **Explain NULL behavior**: "NULL values need special handling in SQL conditions"
2. **Discuss alternatives**: "I could use COALESCE to provide a default value"
3. **Consider business logic**: "NULLs here mean self-acquired customers"
4. **Think about edge cases**: "What about invalid referee IDs?"

### 🔧 **Common Patterns**
- `WHERE column != value OR column IS NULL` for excluding specific values
- Use COALESCE/ISNULL for simpler NULL handling when appropriate
- Always consider three-valued logic in SQL conditions
- Test NULL handling explicitly in your queries

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting NULL handling**: `WHERE referee_id != 2` excludes NULLs incorrectly
2. **NOT IN with NULLs**: Can produce unexpected results
3. **Assuming NULL equals anything**: NULL != NULL is UNKNOWN, not TRUE
4. **Not testing edge cases**: Always test with NULL values

### 🎯 **Amazon Leadership Principles Applied**
- **Dive Deep**: Understand SQL's three-valued logic thoroughly
- **Customer Obsession**: Accurate customer segmentation improves targeting
- **Ownership**: Responsible for handling edge cases properly
- **Learn and Be Curious**: Understanding NULL behavior is fundamental to SQL mastery

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find customers referred by customers 1, 2, or 3**
2. **Find customers who have never made a referral**
3. **Calculate referral depth for each customer**
4. **Find customers referred by customers whose names start with 'J'**

Remember: NULL handling is one of the most common sources of bugs in SQL - mastering this concept is crucial for any data engineer role at Amazon!