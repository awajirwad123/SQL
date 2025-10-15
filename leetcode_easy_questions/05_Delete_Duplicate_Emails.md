# LeetCode Easy #196: Delete Duplicate Emails

## 📋 Problem Statement

Write a SQL query to **delete** all the duplicate emails, keeping only one unique email with the smallest `id`.

Note that you are supposed to write a DELETE statement, not a SELECT statement.

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

**Before Deletion:**
| id | email             |
|----|-------------------|
| 1  | john@example.com  |
| 2  | bob@example.com   |
| 3  | john@example.com  |

**After Deletion:**
| id | email             |
|----|-------------------|
| 1  | john@example.com  |
| 2  | bob@example.com   |

**Explanation:** john@example.com appears twice. We keep the one with the smallest id (id = 1) and delete the other (id = 3).

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to DELETE records, not just SELECT them
- Keep the record with the smallest ID for each email
- Delete all other duplicate records
- This requires identifying which records to delete

### 2. **Key Insights**
- Use self-join to compare records with the same email
- Delete records where id is NOT the minimum for that email
- Alternative: Use window functions to rank records
- Be careful with DELETE syntax and safety

### 3. **Interview Discussion Points**
- "I need to identify duplicates and keep only the one with smallest ID"
- "Self-join approach: delete where current ID > minimum ID for same email"
- "Let me be careful with DELETE syntax to avoid deleting everything"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Duplicates
```sql
-- Find emails that appear more than once
-- For each duplicate email, identify which record to keep (min ID)
```

### Step 2: Determine Records to Delete
```sql
-- Delete records where ID is NOT the minimum for that email
-- Use self-join to compare current record with minimum ID record
```

### Step 3: Execute Safe DELETE
```sql
-- Use proper DELETE syntax with JOIN conditions
-- Test with SELECT first in production environments
```

## ✅ Optimized SQL Solution

**Solution 1: Self-Join with DELETE (MySQL)**
```sql
DELETE p1 
FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email 
  AND p1.id > p2.id;
```

### Alternative Solutions

**Solution 2: Using Subquery to Find Min IDs**
```sql
DELETE FROM Person
WHERE id NOT IN (
    SELECT min_id FROM (
        SELECT MIN(id) as min_id
        FROM Person
        GROUP BY email
    ) as temp
);
```

**Solution 3: Using Window Function (PostgreSQL/SQL Server)**
```sql
WITH DuplicateRanking AS (
    SELECT id,
           ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) as rn
    FROM Person
)
DELETE FROM Person
WHERE id IN (
    SELECT id 
    FROM DuplicateRanking 
    WHERE rn > 1
);
```

**Solution 4: Safe Approach with Temporary Table**
```sql
-- Step 1: Create table with records to keep
CREATE TEMPORARY TABLE PersonToKeep AS
SELECT MIN(id) as id, email
FROM Person
GROUP BY email;

-- Step 2: Delete records not in the keep list
DELETE FROM Person
WHERE id NOT IN (SELECT id FROM PersonToKeep);

-- Step 3: Clean up
DROP TEMPORARY TABLE PersonToKeep;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Safety and Rollback Strategy**
```sql
-- "How would you ensure this DELETE is safe in production?"

-- Step 1: Always start with SELECT to verify
SELECT COUNT(*) as total_records FROM Person;
SELECT COUNT(DISTINCT email) as unique_emails FROM Person;

-- Step 2: Test DELETE with LIMIT first (MySQL)
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id
LIMIT 10;

-- Step 3: Use transactions for safety
BEGIN;
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
-- Verify results before committing
SELECT COUNT(*) FROM Person;
COMMIT; -- or ROLLBACK if something looks wrong
```

#### 2. **Performance Optimization**
```sql
-- "How would you optimize this for millions of records?"

-- Add indexes before running DELETE
CREATE INDEX idx_person_email ON Person(email);
CREATE INDEX idx_person_id_email ON Person(id, email);

-- For very large tables, process in batches
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email 
  AND p1.id > p2.id
LIMIT 1000; -- Process in chunks

-- Monitor progress and repeat until no more rows affected
```

#### 3. **Alternative Business Logic**
```sql
-- "What if we want to keep the most recent record instead of smallest ID?"

-- Assuming there's a created_date column
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email 
  AND p1.created_date < p2.created_date;

-- Or keep the one with latest activity
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email 
  AND p1.last_login_date < p2.last_login_date;
```

#### 4. **Audit Trail for Deleted Records**
```sql
-- "How would you maintain an audit trail of deleted records?"

-- Step 1: Create audit table
CREATE TABLE PersonDeleteAudit (
    original_id INT,
    email VARCHAR(255),
    deleted_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_reason VARCHAR(100)
);

-- Step 2: Insert records that will be deleted
INSERT INTO PersonDeleteAudit (original_id, email, deleted_reason)
SELECT p1.id, p1.email, 'Duplicate email cleanup'
FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;

-- Step 3: Then perform the deletion
DELETE p1 FROM Person p1
INNER JOIN Person p2
WHERE p1.email = p2.email AND p1.id > p2.id;
```

#### 5. **Data Quality Validation**
```sql
-- "How would you validate the cleanup was successful?"

-- Before cleanup metrics
SELECT 
    COUNT(*) as total_records,
    COUNT(DISTINCT email) as unique_emails,
    COUNT(*) - COUNT(DISTINCT email) as duplicate_count
FROM Person;

-- After cleanup validation
SELECT 
    COUNT(*) as total_records,
    COUNT(DISTINCT email) as unique_emails,
    CASE 
        WHEN COUNT(*) = COUNT(DISTINCT email) THEN 'SUCCESS: No duplicates remain'
        ELSE 'ERROR: Duplicates still exist'
    END as validation_result
FROM Person;

-- Check for any remaining duplicates
SELECT email, COUNT(*) as count
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```

## 🔗 Related LeetCode Questions

1. **#182 - Duplicate Emails** (Finding duplicates)
2. **#197 - Rising Temperature** (Self-join comparisons)
3. **#1667 - Fix Names in a Table** (UPDATE operations)
4. **#1484 - Group Sold Products By The Date** (Data deduplication)
5. **#1527 - Patients With a Condition** (Data cleaning)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **DELETE with JOIN**: Deleting based on relationships between records
2. **Self-Join Patterns**: Comparing records within the same table
3. **Data Deduplication**: Common data cleaning operation
4. **Transaction Safety**: Critical for destructive operations

### 🚀 **Amazon Interview Tips**
1. **Safety First**: "I'd always test with SELECT first and use transactions"
2. **Performance**: "For large tables, I'd process in batches and monitor locks"
3. **Business Logic**: "Clarify which record to keep - smallest ID, most recent, etc."
4. **Audit Trail**: "Important to track what was deleted for compliance"

### 🔧 **Common Patterns**
- Self-join with DELETE for duplicate removal
- Always use transactions for safety
- Create indexes before large DELETE operations
- Consider batching for very large tables

### ⚠️ **Common Mistakes to Avoid**
1. Forgetting to test DELETE with SELECT first
2. Not using transactions for rollback capability
3. Deleting all records accidentally (missing WHERE clause)
4. Not considering performance impact on large tables

### 🎯 **Amazon Leadership Principles Applied**
- **Operational Excellence**: Implement safe DELETE procedures
- **Customer Obsession**: Clean data improves customer experience
- **Ownership**: Take responsibility for data integrity
- **Dive Deep**: Understand root causes of duplicate data

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Delete all but the latest record for each email**
2. **Delete duplicates while preserving specific metadata**
3. **Create a stored procedure for safe duplicate removal**
4. **Implement duplicate prevention with constraints**

Remember: Data integrity is paramount at Amazon - destructive operations require careful planning and testing!