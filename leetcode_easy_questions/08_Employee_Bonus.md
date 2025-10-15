# LeetCode Easy #577: Employee Bonus

## 📋 Problem Statement

Write a SQL query to report the name and bonus amount of each employee with a bonus **less than 1000**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Employee Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| empId       | int     |
| name        | varchar |
| supervisor  | int     |
| salary      | int     |
+-------------+---------+
```
- `empId` is the primary key column for this table.
- Each row indicates an employee's name and their supervisor's id.

**Bonus Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| empId       | int  |
| bonus       | int  |
+-------------+------+
```
- `empId` is the primary key column for this table.
- Each row contains the bonus information for an employee.

## 📊 Sample Data

**Employee Table:**
| empId | name   | supervisor | salary |
|-------|--------|------------|--------|
| 3     | Brad   | null       | 4000   |
| 1     | John   | 3          | 1000   |
| 2     | Dan    | 3          | 2000   |
| 4     | Thomas | 3          | 4000   |

**Bonus Table:**
| empId | bonus |
|-------|-------|
| 2     | 500   |
| 4     | 2000  |

**Expected Output:**
| name   | bonus |
|--------|-------|
| Brad   | null  |
| John   | null  |
| Dan    | 500   |

**Explanation:** 
- Brad and John have no bonus records, so their bonus is null (< 1000).
- Dan has a bonus of 500 (< 1000).
- Thomas has a bonus of 2000 (>= 1000), so he's excluded.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to include employees with bonus < 1000 OR no bonus at all
- This requires LEFT JOIN to preserve all employees
- Filter where bonus IS NULL or bonus < 1000
- Return employee name and bonus amount

### 2. **Key Insights**
- LEFT JOIN preserves all employees, even those without bonuses
- NULL bonus should be treated as < 1000 (included in results)
- Need to handle NULL values properly in WHERE clause
- Some employees might not have bonus records at all

### 3. **Interview Discussion Points**
- "I need all employees with bonus < 1000, including those with no bonus"
- "LEFT JOIN ensures I don't lose employees without bonus records"
- "NULL bonus should be included since it's effectively < 1000"

## 🔧 Step-by-Step Solution Logic

### Step 1: Join Tables
```sql
-- LEFT JOIN to preserve all employees
-- Even those without bonus records
```

### Step 2: Handle NULL Bonuses
```sql
-- Include employees where bonus IS NULL (no bonus record)
-- Also include employees where bonus < 1000
```

### Step 3: Select Required Columns
```sql
-- Return employee name and bonus amount
```

## ✅ Optimized SQL Solution

```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

### Alternative Solutions

**Using COALESCE to Handle NULLs:**
```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE COALESCE(b.bonus, 0) < 1000;
```

**Using ISNULL (SQL Server):**
```sql
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE ISNULL(b.bonus, 0) < 1000;
```

**Using CASE Statement for Clarity:**
```sql
SELECT 
    e.name, 
    CASE 
        WHEN b.bonus IS NULL THEN NULL
        ELSE b.bonus 
    END AS bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

**Including Additional Employee Information:**
```sql
SELECT 
    e.name, 
    e.salary,
    COALESCE(b.bonus, 0) AS bonus,
    e.salary + COALESCE(b.bonus, 0) AS total_compensation
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE COALESCE(b.bonus, 0) < 1000
ORDER BY total_compensation DESC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Compensation Analysis**
```sql
-- "Analyze total compensation including salary and bonus"

SELECT 
    e.name,
    e.salary,
    COALESCE(b.bonus, 0) AS bonus,
    e.salary + COALESCE(b.bonus, 0) AS total_compensation,
    CASE 
        WHEN b.bonus IS NULL THEN 'No Bonus'
        WHEN b.bonus < 500 THEN 'Low Bonus'
        WHEN b.bonus < 1000 THEN 'Medium Bonus'
        ELSE 'High Bonus'
    END AS bonus_category
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE COALESCE(b.bonus, 0) < 1000
ORDER BY total_compensation DESC;
```

#### 2. **Supervisor Analysis**
```sql
-- "Include supervisor information in the analysis"

SELECT 
    e.name AS employee_name,
    COALESCE(b.bonus, 0) AS bonus,
    sup.name AS supervisor_name,
    sup_bonus.bonus AS supervisor_bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
LEFT JOIN Employee sup ON e.supervisor = sup.empId
LEFT JOIN Bonus sup_bonus ON sup.empId = sup_bonus.empId
WHERE COALESCE(b.bonus, 0) < 1000
ORDER BY e.name;
```

#### 3. **Performance Optimization**
```sql
-- "How would you optimize this for large employee datasets?"

-- Add indexes for better JOIN performance
CREATE INDEX idx_employee_id ON Employee(empId);
CREATE INDEX idx_bonus_empid ON Bonus(empId);
CREATE INDEX idx_employee_supervisor ON Employee(supervisor);

-- Consider partitioning for very large datasets
-- The basic query remains efficient with proper indexing
SELECT e.name, b.bonus
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000 OR b.bonus IS NULL;
```

#### 4. **Bonus Distribution Analysis**
```sql
-- "Provide statistics on bonus distribution"

SELECT 
    'Total Employees' AS metric,
    COUNT(*) AS count,
    NULL AS avg_amount
FROM Employee

UNION ALL

SELECT 
    'Employees with Bonus' AS metric,
    COUNT(*) AS count,
    AVG(bonus) AS avg_amount
FROM Employee e
INNER JOIN Bonus b ON e.empId = b.empId

UNION ALL

SELECT 
    'Employees with Low Bonus (<1000)' AS metric,
    COUNT(*) AS count,
    AVG(bonus) AS avg_amount
FROM Employee e
INNER JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus < 1000

UNION ALL

SELECT 
    'Employees with No Bonus' AS metric,
    COUNT(*) AS count,
    NULL AS avg_amount
FROM Employee e
LEFT JOIN Bonus b ON e.empId = b.empId
WHERE b.bonus IS NULL;
```

#### 5. **Data Quality and Validation**
```sql
-- "Check for data quality issues in bonus allocation"

-- Check for employees in Bonus table but not in Employee table
SELECT 'Orphaned Bonus Records' AS issue,
       COUNT(*) AS count
FROM Bonus b
LEFT JOIN Employee e ON b.empId = e.empId
WHERE e.empId IS NULL

UNION ALL

-- Check for negative bonuses
SELECT 'Negative Bonus Values' AS issue,
       COUNT(*) AS count
FROM Bonus
WHERE bonus < 0

UNION ALL

-- Check for extremely high bonuses (potential data errors)
SELECT 'Potentially Invalid High Bonuses' AS issue,
       COUNT(*) AS count
FROM Bonus b
INNER JOIN Employee e ON b.empId = e.empId
WHERE b.bonus > e.salary * 2;  -- Bonus more than double the salary
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (Basic LEFT JOIN)
2. **#181 - Employees Earning More Than Their Managers** (Employee relationships)
3. **#183 - Customers Who Never Order** (LEFT JOIN with NULL filtering)
4. **#184 - Department Highest Salary** (Employee salary analysis)
5. **#626 - Exchange Seats** (Employee data manipulation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **LEFT JOIN**: Preserves all records from the left table
2. **NULL Handling**: Critical when dealing with optional relationships
3. **OR Conditions**: Include both NULL and specific value conditions
4. **Data Relationships**: Understanding optional foreign keys

### 🚀 **Amazon Interview Tips**
1. **Clarify NULL handling**: "Should employees with no bonus be included?"
2. **Discuss edge cases**: "What about negative bonuses or data quality issues?"
3. **Consider performance**: "For large datasets, proper indexing on join columns is crucial"
4. **Think business context**: "Low bonus employees might need attention for retention"

### 🔧 **Common Patterns**
- LEFT JOIN for optional relationships
- OR conditions with IS NULL for filtering
- COALESCE/ISNULL for default value handling
- Always consider data quality in real-world scenarios

### ⚠️ **Common Mistakes to Avoid**
1. Using INNER JOIN instead of LEFT JOIN (loses employees without bonuses)
2. Forgetting to handle NULL values in WHERE clause
3. Not considering data quality issues (orphaned records, negative values)
4. Missing indexes on foreign key columns

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Fair compensation analysis affects employee satisfaction
- **Dive Deep**: Understand compensation structures and equity
- **Frugality**: Identify cost optimization opportunities in bonus allocation
- **Ownership**: Ensure data quality for HR decision-making

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find employees with bonus greater than their monthly salary**
2. **Calculate average bonus by department**
3. **Find supervisors whose team members have higher bonuses**
4. **Identify employees eligible for bonus but haven't received one**

Remember: Compensation analysis is crucial for Amazon's talent retention and fair pay initiatives!