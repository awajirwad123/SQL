# LeetCode Easy #183: Customers Who Never Order

## 📋 Problem Statement

Write a SQL query to report all customers who never order anything.

Return the result table in **any order**.

## 🗄️ Table Schema

**Customers Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
```
- `id` is the primary key column for this table.
- This table contains information about customers.

**Orders Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| customerId  | int  |
+-------------+------+
```
- `id` is the primary key column for this table.
- `customerId` is a foreign key of the ID from the Customers table.
- This table contains information about the orders made by customers.

## 📊 Sample Data

**Customers Table:**
| id | name  |
|----|-------|
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |

**Orders Table:**
| id | customerId |
|----|------------|
| 1  | 3          |
| 2  | 1          |

**Expected Output:**
| Customers |
|-----------|
| Henry     |
| Max       |

**Explanation:** Henry (id=2) and Max (id=4) never placed any orders.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find customers who don't appear in the Orders table
- This is a classic "find missing records" problem
- Two main approaches: LEFT JOIN with NULL check or NOT EXISTS subquery
- Must return customer names, not IDs

### 2. **Key Insights**
- Customers table has all customers
- Orders table only has customers who placed orders
- Need to find customers NOT IN the orders table
- LEFT JOIN will show NULL for customers without orders

### 3. **Interview Discussion Points**
- "I need to find customers who don't exist in the orders table"
- "LEFT JOIN will preserve all customers, showing NULL for those without orders"
- "Alternative approach would be NOT EXISTS or NOT IN subqueries"

## 🔧 Step-by-Step Solution Logic

### Step 1: Choose Join Strategy
```sql
-- LEFT JOIN customers with orders
-- Keep all customers, show NULL where no orders exist
```

### Step 2: Identify Missing Records
```sql
-- Filter WHERE orders.customerId IS NULL
-- These are customers who never ordered
```

### Step 3: Select Required Output
```sql
-- Return customer names only
```

## ✅ Optimized SQL Solution

**Solution 1: LEFT JOIN (Most Common)**
```sql
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.customerId IS NULL;
```

### Alternative Solutions

**Solution 2: NOT EXISTS (Often More Efficient)**
```sql
SELECT name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customerId = c.id
);
```

**Solution 3: NOT IN (Be Careful with NULLs)**
```sql
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (
    SELECT customerId
    FROM Orders
    WHERE customerId IS NOT NULL  -- Important: handle NULLs
);
```

**Solution 4: Using EXCEPT (SQL Server/PostgreSQL)**
```sql
SELECT name AS Customers
FROM Customers
WHERE id NOT IN (
    SELECT DISTINCT customerId
    FROM Orders
    WHERE customerId IS NOT NULL
);
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Performance Comparison**
```sql
-- "Which approach is most efficient for large datasets?"

-- Add indexes for better performance
CREATE INDEX idx_orders_customer_id ON Orders(customerId);
CREATE INDEX idx_customers_id ON Customers(id);

-- NOT EXISTS is often fastest as it can short-circuit
-- LEFT JOIN with IS NULL is also very efficient
-- NOT IN can be slowest due to NULL handling
```

#### 2. **Include Additional Customer Information**
```sql
-- "Include customer details and registration date in the output"

SELECT 
    c.id,
    c.name AS Customers,
    c.email,
    c.registration_date,
    DATEDIFF(CURRENT_DATE, c.registration_date) AS days_since_registration
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.customerId IS NULL
ORDER BY c.registration_date DESC;
```

#### 3. **Time-Based Analysis**
```sql
-- "Find customers who haven't ordered in the last 6 months"

SELECT c.name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.customerId = c.id
    AND o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
);
```

#### 4. **Segmentation Analysis**
```sql
-- "Categorize customers by their ordering behavior"

SELECT 
    CASE 
        WHEN o.customerId IS NULL THEN 'Never Ordered'
        WHEN order_count = 1 THEN 'Single Order'
        WHEN order_count BETWEEN 2 AND 5 THEN 'Occasional'
        WHEN order_count > 5 THEN 'Frequent'
    END AS customer_segment,
    COUNT(*) AS customer_count
FROM Customers c
LEFT JOIN (
    SELECT customerId, COUNT(*) as order_count
    FROM Orders
    GROUP BY customerId
) o ON c.id = o.customerId
GROUP BY 
    CASE 
        WHEN o.customerId IS NULL THEN 'Never Ordered'
        WHEN order_count = 1 THEN 'Single Order'
        WHEN order_count BETWEEN 2 AND 5 THEN 'Occasional'
        WHEN order_count > 5 THEN 'Frequent'
    END;
```

#### 5. **Business Impact Analysis**
```sql
-- "Calculate potential revenue loss from non-ordering customers"

WITH AvgOrderValue AS (
    SELECT AVG(total_amount) as avg_order_value
    FROM Orders
),
NonOrderingCustomers AS (
    SELECT COUNT(*) as non_ordering_count
    FROM Customers c
    LEFT JOIN Orders o ON c.id = o.customerId
    WHERE o.customerId IS NULL
)
SELECT 
    non_ordering_count,
    avg_order_value,
    (non_ordering_count * avg_order_value) AS potential_revenue_loss
FROM NonOrderingCustomers, AvgOrderValue;
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (LEFT JOIN basics)
2. **#181 - Employees Earning More Than Their Managers** (Self JOIN)
3. **#577 - Employee Bonus** (LEFT JOIN with conditions)
4. **#584 - Find Customer Referee** (NULL handling)
5. **#1068 - Product Sales Analysis I** (INNER vs LEFT JOIN)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **LEFT JOIN**: Preserves all records from the left table
2. **IS NULL**: Checks for missing relationships
3. **NOT EXISTS**: Often more efficient than NOT IN
4. **NULL Handling**: Critical in NOT IN clauses

### 🚀 **Amazon Interview Tips**
1. **Discuss performance**: "NOT EXISTS often performs better than NOT IN"
2. **Consider scale**: "For millions of customers, indexing is crucial"
3. **Think business impact**: "These customers represent growth opportunities"
4. **Handle edge cases**: "What about customers with cancelled orders?"

### 🔧 **Common Patterns**
- LEFT JOIN + IS NULL for finding missing relationships
- NOT EXISTS for existence checks (often more efficient)
- Be careful with NOT IN when NULLs are possible
- Always consider indexing foreign key columns

### ⚠️ **Common Mistakes to Avoid**
1. Using INNER JOIN instead of LEFT JOIN
2. Forgetting to handle NULLs in NOT IN clauses
3. Not considering performance implications of different approaches
4. Missing indexes on join columns

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Identify customers who need re-engagement
- **Dive Deep**: Understand customer behavior patterns
- **Frugality**: Choose efficient query patterns
- **Think Big**: Consider how this scales for millions of customers

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find customers who placed only one order**
2. **Find products that were never ordered**
3. **Find customers with orders but no recent activity**
4. **Calculate the percentage of customers who never order**

Remember: Understanding customer behavior is crucial for Amazon's recommendation systems and marketing strategies!