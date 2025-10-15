# LeetCode Easy #175: Combine Two Tables

## 📋 Problem Statement

Write a SQL query to report the first name, last name, city, and state of each person in the `Person` table. If the address of a `personId` is not present in the `Address` table, report `null` instead.

Return the result table in **any order**.

## 🗄️ Table Schema

**Person Table:**
```
+----------+---------+-----------+
| Column   | Type    | Key       |
+----------+---------+-----------+
| personId | int     | Primary   |
| lastName | varchar |           |
| firstName| varchar |           |
+----------+---------+-----------+
```

**Address Table:**
```
+-----------+---------+-----------+
| Column    | Type    | Key       |
+-----------+---------+-----------+
| addressId | int     | Primary   |
| personId  | int     | Foreign   |
| city      | varchar |           |
| state     | varchar |           |
+-----------+---------+-----------+
```

## 📊 Sample Data

### Person Table
| personId | lastName | firstName |
|----------|----------|-----------|
| 1        | Wang     | Allen     |
| 2        | Alice    | Bob       |

### Address Table
| addressId | personId | city          | state    |
|-----------|----------|---------------|----------|
| 1         | 2        | New York City | New York |
| 2         | 3        | Leetcode      | CA       |

### Expected Output
| firstName | lastName | city          | state    |
|-----------|----------|---------------|----------|
| Allen     | Wang     | null          | null     |
| Bob       | Alice    | New York City | New York |

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to combine data from two tables
- Some people might not have addresses (need to handle missing data)
- Must include ALL people, even without addresses
- This is a classic LEFT JOIN scenario

### 2. **Key Insights**
- Person table is the "driving" table (we want all persons)
- Address table has optional information
- Need to preserve all rows from Person table
- Missing address data should show as NULL

### 3. **Interview Discussion Points**
- "I need to join these tables, but preserve all people even without addresses"
- "This sounds like a LEFT JOIN where Person is the left table"
- "Let me verify the expected output matches this approach"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify the Join Type
```sql
-- We need ALL persons, some may not have addresses
-- This requires LEFT JOIN with Person as the left table
```

### Step 2: Determine Join Condition
```sql
-- Join on personId which exists in both tables
-- Person.personId = Address.personId
```

### Step 3: Select Required Columns
```sql
-- firstName, lastName from Person table
-- city, state from Address table (will be NULL if no match)
```

## ✅ Optimized SQL Solution

```sql
SELECT 
    p.firstName,
    p.lastName,
    a.city,
    a.state
FROM Person p
LEFT JOIN Address a ON p.personId = a.personId;
```

### Alternative Solutions

**Using Table Aliases for Clarity:**
```sql
SELECT 
    person.firstName,
    person.lastName,
    address.city,
    address.state
FROM Person person
LEFT JOIN Address address ON person.personId = address.personId;
```

**With Explicit NULL Handling (PostgreSQL/SQL Server):**
```sql
SELECT 
    p.firstName,
    p.lastName,
    COALESCE(a.city, NULL) AS city,    -- Explicit NULL (optional)
    COALESCE(a.state, NULL) AS state
FROM Person p
LEFT JOIN Address a ON p.personId = a.personId;
```

## 🔗 Related LeetCode Questions

1. **#181 - Employees Earning More Than Their Managers** (Self JOIN)
2. **#183 - Customers Who Never Order** (LEFT JOIN with NULL check)
3. **#197 - Rising Temperature** (Self JOIN with date comparison)
4. **#577 - Employee Bonus** (LEFT JOIN with conditional logic)
5. **#584 - Find Customer Referee** (NULL handling and filtering)

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Performance Optimization**
```sql
-- "How would you optimize this for a table with millions of records?"

-- Add indexes on join columns
CREATE INDEX idx_person_id ON Person(personId);
CREATE INDEX idx_address_person_id ON Address(personId);

-- The optimized query remains the same
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person p
LEFT JOIN Address a ON p.personId = a.personId;
```

#### 2. **Multiple Addresses Scenario**
```sql
-- "What if a person could have multiple addresses?"

-- Option 1: Get all address combinations
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person p
LEFT JOIN Address a ON p.personId = a.personId
ORDER BY p.personId, a.addressId;

-- Option 2: Get only the most recent address
SELECT p.firstName, p.lastName, a.city, a.state
FROM Person p
LEFT JOIN (
    SELECT personId, city, state,
           ROW_NUMBER() OVER (PARTITION BY personId ORDER BY addressId DESC) as rn
    FROM Address
) a ON p.personId = a.personId AND a.rn = 1;
```

#### 3. **Data Quality Concerns**
```sql
-- "How would you handle data quality issues?"

-- Check for orphaned addresses
SELECT COUNT(*) as orphaned_addresses
FROM Address a
LEFT JOIN Person p ON a.personId = p.personId
WHERE p.personId IS NULL;

-- Check for duplicate persons
SELECT personId, COUNT(*) as duplicate_count
FROM Person
GROUP BY personId
HAVING COUNT(*) > 1;
```

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **LEFT JOIN** preserves all rows from the left table
2. **RIGHT JOIN** preserves all rows from the right table  
3. **INNER JOIN** only returns matching rows
4. **FULL OUTER JOIN** preserves all rows from both tables

### 🚀 **Amazon Interview Tips**
1. **Always clarify requirements**: "Should I include people without addresses?"
2. **Discuss edge cases**: "What if someone has multiple addresses?"
3. **Consider performance**: "For large datasets, I'd add indexes on join columns"
4. **Think about data quality**: "I'd validate referential integrity"

### 🔧 **Common Patterns**
- Use LEFT JOIN when you need all records from the primary table
- Always use table aliases in joins for readability
- Consider NULL handling explicitly when business logic requires it
- Index foreign key columns for better join performance

### ⚠️ **Common Mistakes to Avoid**
1. Using INNER JOIN instead of LEFT JOIN (loses people without addresses)
2. Forgetting table aliases in complex queries
3. Not considering NULL values in WHERE clauses
4. Ignoring performance implications of joins

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Ensure data accuracy for downstream analytics
- **Ownership**: Consider data quality and edge cases
- **Dive Deep**: Understand the business impact of missing data
- **Invent and Simplify**: Choose the most readable and maintainable solution

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Include only people who have addresses**
2. **Include people and count their total addresses**
3. **Find people with addresses in specific states**
4. **Join three tables: Person, Address, and Phone**

Remember: In Amazon interviews, they value candidates who think about scalability, data quality, and business impact, not just getting the right query result!