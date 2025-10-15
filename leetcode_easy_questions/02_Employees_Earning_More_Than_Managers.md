# LeetCode Easy #181: Employees Earning More Than Their Managers

## 📋 Problem Statement

Write a SQL query to find the employees who earn more than their managers.

Return the result table in **any order**.

## 🗄️ Table Schema

**Employee Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
| salary      | int     |
| managerId   | int     |
+-------------+---------+
```
- `id` is the primary key column for this table.
- Each row indicates the ID of an employee, their name, salary, and the ID of their manager.

## 📊 Sample Data

**Employee Table:**
| id | name  | salary | managerId |
|----|-------|--------|-----------|
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | null      |
| 4  | Max   | 90000  | null      |

**Expected Output:**
| Employee |
|----------|
| Joe      |

**Explanation:** Joe is the only employee who earns more than his manager.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to compare employee salary with their manager's salary
- This requires joining the table with itself (self-join)
- Manager information is in the same table, referenced by `managerId`
- Only return employees where employee.salary > manager.salary

### 2. **Key Insights**
- This is a **self-join** problem
- Employee table serves as both employee and manager data source
- Need to handle NULL managerId (top-level managers)
- Join condition: employee.managerId = manager.id

### 3. **Interview Discussion Points**
- "I need to compare each employee's salary with their manager's salary"
- "Since managers are also in the same table, this requires a self-join"
- "I'll alias the table twice to represent employees and managers separately"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Self-Join Pattern
```sql
-- Need to join Employee table with itself
-- One instance for employees, another for managers
```

### Step 2: Determine Join Condition
```sql
-- Employee's managerId should match Manager's id
-- employee.managerId = manager.id
```

### Step 3: Apply Salary Comparison
```sql
-- Filter where employee.salary > manager.salary
```

## ✅ Optimized SQL Solution

```sql
SELECT e.name AS Employee
FROM Employee e
INNER JOIN Employee m ON e.managerId = m.id
WHERE e.salary > m.salary;
```

### Alternative Solutions

**Using Table Aliases for Clarity:**
```sql
SELECT emp.name AS Employee
FROM Employee emp
INNER JOIN Employee mgr ON emp.managerId = mgr.id
WHERE emp.salary > mgr.salary;
```

**With Subquery Approach:**
```sql
SELECT name AS Employee
FROM Employee e1
WHERE salary > (
    SELECT salary 
    FROM Employee e2 
    WHERE e2.id = e1.managerId
);
```

**Including Additional Details:**
```sql
SELECT 
    e.name AS Employee,
    e.salary AS EmployeeSalary,
    m.name AS Manager,
    m.salary AS ManagerSalary,
    (e.salary - m.salary) AS SalaryDifference
FROM Employee e
INNER JOIN Employee m ON e.managerId = m.id
WHERE e.salary > m.salary
ORDER BY SalaryDifference DESC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Performance Optimization**
```sql
-- "How would you optimize this for millions of employees?"

-- Add indexes for better performance
CREATE INDEX idx_employee_manager_id ON Employee(managerId);
CREATE INDEX idx_employee_id ON Employee(id);

-- Consider partitioning for very large datasets
-- Partition by department or region if applicable
```

#### 2. **Handle Multiple Management Levels**
```sql
-- "What if you want to find employees earning more than any manager in their chain?"

WITH RECURSIVE ManagerChain AS (
    -- Base case: direct reports
    SELECT e.id, e.name, e.salary, e.managerId, 1 as level
    FROM Employee e
    WHERE e.managerId IS NOT NULL
    
    UNION ALL
    
    -- Recursive case: climb up the management chain
    SELECT mc.id, mc.name, mc.salary, m.managerId, mc.level + 1
    FROM ManagerChain mc
    INNER JOIN Employee m ON mc.managerId = m.id
    WHERE m.managerId IS NOT NULL AND mc.level < 10  -- Prevent infinite recursion
)
SELECT DISTINCT emp.name AS Employee
FROM Employee emp
INNER JOIN ManagerChain mc ON emp.id = mc.id
INNER JOIN Employee mgr ON mc.managerId = mgr.id
WHERE emp.salary > mgr.salary;
```

#### 3. **Department-wise Analysis**
```sql
-- "Find employees earning more than their managers by department"

SELECT 
    d.department_name,
    e.name AS Employee,
    e.salary AS EmployeeSalary,
    m.name AS Manager,
    m.salary AS ManagerSalary
FROM Employee e
INNER JOIN Employee m ON e.managerId = m.id
INNER JOIN Department d ON e.department_id = d.id
WHERE e.salary > m.salary
ORDER BY d.department_name, (e.salary - m.salary) DESC;
```

#### 4. **Data Quality and Edge Cases**
```sql
-- "How would you handle data quality issues?"

-- Check for circular management relationships
WITH RECURSIVE CircularCheck AS (
    SELECT id, managerId, name, 1 as depth, CAST(id AS VARCHAR(1000)) as path
    FROM Employee
    WHERE managerId IS NOT NULL
    
    UNION ALL
    
    SELECT e.id, e.managerId, e.name, cc.depth + 1, 
           cc.path + '->' + CAST(e.id AS VARCHAR)
    FROM Employee e
    INNER JOIN CircularCheck cc ON e.id = cc.managerId
    WHERE cc.depth < 20  -- Prevent infinite recursion
    AND CHARINDEX(CAST(e.id AS VARCHAR), cc.path) = 0  -- Check for cycles
)
SELECT * FROM CircularCheck WHERE depth > 10;  -- Potential circular references

-- Check for orphaned employees (managerId doesn't exist)
SELECT e.name, e.managerId
FROM Employee e
LEFT JOIN Employee m ON e.managerId = m.id
WHERE e.managerId IS NOT NULL AND m.id IS NULL;
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (Basic JOIN)
2. **#183 - Customers Who Never Order** (LEFT JOIN with NULL)
3. **#577 - Employee Bonus** (LEFT JOIN with conditions)
4. **#584 - Find Customer Referee** (NULL handling)
5. **#1179 - Reformat Department Table** (Self-join with aggregation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Join**: Joining a table with itself using different aliases
2. **Primary/Foreign Key Relationships**: Understanding parent-child relationships
3. **NULL Handling**: Managers at the top level have NULL managerId
4. **Alias Usage**: Essential for readability in self-joins

### 🚀 **Amazon Interview Tips**
1. **Clarify edge cases**: "What about employees without managers?"
2. **Discuss performance**: "For large tables, I'd add indexes on id and managerId"
3. **Consider data integrity**: "How do we handle circular management chains?"
4. **Think scalability**: "For millions of employees, consider partitioning strategies"

### 🔧 **Common Patterns**
- Use descriptive aliases (emp/mgr or e/m) for self-joins
- INNER JOIN excludes employees without managers automatically
- Consider LEFT JOIN if you need to include all employees
- Index both the primary key and foreign key columns

### ⚠️ **Common Mistakes to Avoid**
1. Forgetting table aliases in self-joins
2. Using same alias for both instances of the table
3. Not considering NULL managerId values
4. Confusing the join direction (employee to manager vs manager to employee)

### 🎯 **Amazon Leadership Principles Applied**
- **Dive Deep**: Understand organizational hierarchy relationships
- **Think Big**: Consider how this scales to large organizations
- **Customer Obsession**: Ensure accurate reporting for HR analytics
- **Operational Excellence**: Consider data quality and edge cases

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find managers who earn less than all their direct reports**
2. **Calculate average salary difference between employees and managers**
3. **Find the highest-earning employee in each management chain**
4. **Identify employees with no direct reports (leaf nodes)**

Remember: Self-joins are powerful for hierarchical data analysis - a common pattern in Amazon's organizational analytics!