# LeetCode Easy #182: Duplicate Emails

## 📋 Problem Statement

Write a SQL query to report all the duplicate emails. Note that it's guaranteed that the email field is not NULL.

Return the result table in **any order**.

## 🗄️ Table Schema

**Person Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| email       | varchar |
+-------------+---------+
```
- `id` is the primary key column for this table.
- Each row contains an email. The emails will not contain uppercase letters.

## 📊 Sample Data

**Person Table:**
| id | email             |
|----|-------------------|
| 1  | a@b.com          |
| 2  | c@d.com          |
| 3  | a@b.com          |

**Expected Output:**
| Email   |
|---------|
| a@b.com |

**Explanation:** a@b.com appears twice in the table.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find emails that appear more than once
- This is a classic GROUP BY with HAVING problem
- Count occurrences of each email and filter where count > 1
- Return only the email addresses, not the count

### 2. **Key Insights**
- Use GROUP BY to group identical emails together
- Use COUNT() to count occurrences per email
- Use HAVING to filter groups (not individual rows)
- HAVING comes after GROUP BY, WHERE comes before

### 3. **Interview Discussion Points**
- "I need to group by email and count occurrences"
- "HAVING clause filters groups, not individual rows"
- "Only return emails that appear more than once"

## 🔧 Step-by-Step Solution Logic

### Step 1: Group by Email
```sql
-- Group all rows by email address
GROUP BY email
```

### Step 2: Count Occurrences
```sql
-- Count how many times each email appears
COUNT(*) 
-- or COUNT(id) or COUNT(email) - all equivalent here
```

### Step 3: Filter Duplicates
```sql
-- Only keep groups where count > 1
HAVING COUNT(*) > 1
```

## ✅ Optimized SQL Solution

```sql
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

### Alternative Solutions

**Using COUNT with Explicit Column:**
```sql
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(id) > 1;
```

**Using Window Function (More Complex, but Shows Advanced Knowledge):**
```sql
SELECT DISTINCT email AS Email
FROM (
    SELECT email,
           COUNT(*) OVER (PARTITION BY email) as email_count
    FROM Person
) t
WHERE email_count > 1;
```

**Self-Join Approach (Less Efficient):**
```sql
SELECT DISTINCT p1.email AS Email
FROM Person p1
INNER JOIN Person p2 ON p1.email = p2.email AND p1.id != p2.id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Return Additional Information**
```sql
-- "Show the duplicate emails with their count and IDs"

SELECT 
    email,
    COUNT(*) as duplicate_count,
    GROUP_CONCAT(id ORDER BY id) as ids  -- MySQL
    -- STRING_AGG(CAST(id AS VARCHAR), ',') as ids  -- SQL Server/PostgreSQL
FROM Person
GROUP BY email
HAVING COUNT(*) > 1
ORDER BY duplicate_count DESC;
```

#### 2. **Find All Rows with Duplicate Emails**
```sql
-- "Return all person records that have duplicate emails"

SELECT p.*
FROM Person p
INNER JOIN (
    SELECT email
    FROM Person
    GROUP BY email
    HAVING COUNT(*) > 1
) duplicates ON p.email = duplicates.email
ORDER BY p.email, p.id;
```

#### 3. **Performance Optimization**
```sql
-- "How would you optimize this for millions of records?"

-- Add index on email column
CREATE INDEX idx_person_email ON Person(email);

-- The query remains the same but will use the index
SELECT email AS Email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

#### 4. **Data Cleaning Scenarios**
```sql
-- "Keep only the first occurrence of each duplicate email"

-- Option 1: Using ROW_NUMBER() to identify which to keep
DELETE p1
FROM Person p1
INNER JOIN (
    SELECT id,
           ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn
    FROM Person
) p2 ON p1.id = p2.id
WHERE p2.rn > 1;

-- Option 2: Keep minimum ID for each email (MySQL)
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
```

#### 5. **Advanced Duplicate Analysis**
```sql
-- "Analyze duplicate patterns and provide statistics"

SELECT 
    'Total Records' as metric,
    COUNT(*) as value
FROM Person

UNION ALL

SELECT 
    'Unique Emails' as metric,
    COUNT(DISTINCT email) as value
FROM Person

UNION ALL

SELECT 
    'Duplicate Emails' as metric,
    COUNT(*) as value
FROM (
    SELECT email
    FROM Person
    GROUP BY email
    HAVING COUNT(*) > 1
) dup

UNION ALL

SELECT 
    'Records with Duplicates' as metric,
    COUNT(*) as value
FROM Person p
WHERE EXISTS (
    SELECT 1
    FROM Person p2
    WHERE p2.email = p.email
    GROUP BY p2.email
    HAVING COUNT(*) > 1
);
```

## 🔗 Related LeetCode Questions

1. **#196 - Delete Duplicate Emails** (Duplicate removal)
2. **#183 - Customers Who Never Order** (GROUP BY with filtering)
3. **#511 - Game Play Analysis I** (GROUP BY with aggregation)
4. **#586 - Customer Placing the Largest Number of Orders** (GROUP BY with MAX)
5. **#1050 - Actors and Directors Who Cooperated At Least Three Times** (GROUP BY with HAVING)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY**: Groups identical values together
2. **HAVING**: Filters groups (use after GROUP BY)
3. **WHERE vs HAVING**: WHERE filters rows, HAVING filters groups
4. **COUNT()**: Counts non-NULL values in each group

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I return just emails or include counts?"
2. **Discuss performance**: "For large datasets, I'd add an index on email"
3. **Consider data quality**: "How should we handle case sensitivity?"
4. **Think about next steps**: "Would you want to clean these duplicates?"

### 🔧 **Common Patterns**
- GROUP BY + HAVING for finding duplicates
- COUNT(*) vs COUNT(column) - both work for non-NULL columns
- Always consider indexing the GROUP BY column for performance
- Use window functions for more complex duplicate analysis

### ⚠️ **Common Mistakes to Avoid**
1. Using WHERE instead of HAVING to filter groups
2. Forgetting to use GROUP BY before HAVING
3. Not considering case sensitivity in email comparison
4. Using inefficient self-joins instead of GROUP BY

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Clean data improves customer experience
- **Operational Excellence**: Implement monitoring for data quality
- **Frugality**: Efficient queries save computational resources
- **Dive Deep**: Understand root causes of duplicate data

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find emails that appear exactly 3 times**
2. **Find the most frequently occurring email**
3. **Count total duplicate records (not unique emails)**
4. **Find duplicates case-insensitively**

Remember: Data quality is crucial at Amazon - duplicate detection is a fundamental skill for data engineers!