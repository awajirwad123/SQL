# LeetCode Easy #607: Sales Person

## 📋 Problem Statement

Write a SQL query to report the names of all the salespersons who **did not have any orders** related to the company with the name **"RED"**.

Return the result table in **any order**.

## 🗄️ Table Schema

**SalesPerson Table:**
```
+-----------------+---------+
| Column Name     | Type    |
+-----------------+---------+
| sales_id        | int     |
| name            | varchar |
| salary          | int     |
| commission_rate | int     |
| hire_date       | date    |
+-----------------+---------+
```
- `sales_id` is the primary key column for this table.
- Each row of this table indicates the name and the ID of a salesperson alongside their salary, commission rate, and hire date.

**Company Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| com_id      | int     |
| name        | varchar |
| city        | varchar |
+-------------+---------+
```
- `com_id` is the primary key column for this table.
- Each row of this table indicates the name and the ID of a company and the city in which the company is located.

**Orders Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| order_id    | int  |
| order_date  | date |
| com_id      | int  |
| sales_id    | int  |
+-------------+------+
```
- `order_id` is the primary key column for this table.
- `com_id` is a foreign key to `com_id` from the Company table.
- `sales_id` is a foreign key to `sales_id` from the SalesPerson table.
- Each row of this table contains information about one order.

## 📊 Sample Data

**SalesPerson Table:**
| sales_id | name | salary | commission_rate | hire_date  |
|----------|------|--------|-----------------|------------|
| 1        | John | 100000 | 6               | 4/1/2006   |
| 2        | Amy  | 12000  | 5               | 5/1/2010   |
| 3        | Mark | 65000  | 12              | 12/25/2008 |
| 4        | Pam  | 25000  | 25              | 1/1/2005   |
| 5        | Alex | 5000   | 10              | 2/3/2007   |

**Company Table:**
| com_id | name   | city     |
|--------|--------|----------|
| 1      | RED    | Boston   |
| 2      | ORANGE | New York |
| 3      | YELLOW | Boston   |
| 4      | GREEN  | Austin   |

**Orders Table:**
| order_id | order_date | com_id | sales_id |
|----------|------------|--------|----------|
| 1        | 1/1/2014   | 3      | 4        |
| 2        | 2/1/2014   | 4      | 5        |
| 3        | 3/1/2014   | 1      | 1        |
| 4        | 4/1/2014   | 1      | 4        |

**Expected Output:**
| name |
|------|
| Amy  |
| Mark |
| Alex |

**Explanation:** 
- John and Pam had orders related to company "RED" (com_id = 1)
- Amy, Mark, and Alex never had orders with company "RED"

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find salespersons who NEVER sold to company "RED"
- This is a "NOT EXISTS" or "NOT IN" problem
- Must join multiple tables: SalesPerson → Orders → Company
- Key challenge: handling negation logic correctly

### 2. **Key Insights**
- First identify all salespersons who DID sell to "RED"
- Then find all salespersons NOT in that group
- Alternative: Use LEFT JOIN with NULL check
- Be careful with NOT IN and NULL values

### 3. **Interview Discussion Points**
- "This is a negation problem - finding who did NOT do something"
- "I need to identify the company 'RED' first, then find related orders"
- "NOT EXISTS is typically safer than NOT IN for NULL handling"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Company "RED"
```sql
-- Find com_id for company named "RED"
-- This will be used to filter orders
```

### Step 2: Find Salespersons Who Sold to RED
```sql
-- Join Orders with Company to find RED orders
-- Get sales_id of salespersons who had RED orders
```

### Step 3: Find Salespersons Who Did NOT Sell to RED
```sql
-- Use NOT EXISTS or NOT IN to exclude RED salespersons
-- Return names from SalesPerson table
```

## ✅ Optimized SQL Solution

**Solution 1: NOT EXISTS Approach (Recommended)**
```sql
SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE o.sales_id = s.sales_id 
    AND c.name = 'RED'
);
```

### Alternative Solutions

**Solution 2: NOT IN Approach**
```sql
SELECT name
FROM SalesPerson
WHERE sales_id NOT IN (
    SELECT DISTINCT o.sales_id
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE c.name = 'RED'
    AND o.sales_id IS NOT NULL  -- Important for NOT IN
);
```

**Solution 3: LEFT JOIN with NULL Check**
```sql
SELECT DISTINCT s.name
FROM SalesPerson s
LEFT JOIN (
    SELECT DISTINCT o.sales_id
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE c.name = 'RED'
) red_sales ON s.sales_id = red_sales.sales_id
WHERE red_sales.sales_id IS NULL;
```

**Solution 4: Using EXCEPT (Where Supported)**
```sql
SELECT name FROM SalesPerson
EXCEPT
SELECT s.name
FROM SalesPerson s
JOIN Orders o ON s.sales_id = o.sales_id
JOIN Company c ON o.com_id = c.com_id
WHERE c.name = 'RED';
```

**Solution 5: Subquery with NOT IN (Direct)**
```sql
SELECT name
FROM SalesPerson
WHERE sales_id NOT IN (
    SELECT o.sales_id
    FROM Orders o
    WHERE o.com_id = (
        SELECT com_id 
        FROM Company 
        WHERE name = 'RED'
    )
    AND o.sales_id IS NOT NULL
);
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Performance Optimization**
```sql
-- "How would you optimize this for millions of records?"

-- Create indexes for efficient joins
CREATE INDEX idx_orders_sales_id ON Orders(sales_id);
CREATE INDEX idx_orders_com_id ON Orders(com_id);
CREATE INDEX idx_company_name ON Company(name);

-- Original NOT EXISTS query remains efficient
SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE o.sales_id = s.sales_id 
    AND c.name = 'RED'
);

-- Query execution plan analysis
EXPLAIN SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE o.sales_id = s.sales_id 
    AND c.name = 'RED'
);
```

#### 2. **Multiple Company Exclusions**
```sql
-- "Find salespersons who never sold to RED, ORANGE, or YELLOW"

SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE o.sales_id = s.sales_id 
    AND c.name IN ('RED', 'ORANGE', 'YELLOW')
);

-- Or with configurable exclusion list
WITH excluded_companies AS (
    SELECT com_id
    FROM Company
    WHERE name IN ('RED', 'ORANGE', 'YELLOW')
)
SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    WHERE o.sales_id = s.sales_id 
    AND o.com_id IN (SELECT com_id FROM excluded_companies)
);
```

#### 3. **Sales Performance Analysis**
```sql
-- "Analyze sales performance excluding RED company orders"

WITH non_red_sales AS (
    SELECT 
        s.sales_id,
        s.name,
        s.salary,
        s.commission_rate,
        COUNT(o.order_id) as non_red_orders,
        COUNT(DISTINCT o.com_id) as companies_served
    FROM SalesPerson s
    LEFT JOIN Orders o ON s.sales_id = o.sales_id
    LEFT JOIN Company c ON o.com_id = c.com_id
    WHERE c.name != 'RED' OR c.name IS NULL
    GROUP BY s.sales_id, s.name, s.salary, s.commission_rate
)
SELECT 
    name,
    salary,
    commission_rate,
    non_red_orders,
    companies_served,
    CASE 
        WHEN non_red_orders = 0 THEN 'No Sales'
        WHEN non_red_orders <= 2 THEN 'Low Performer'
        WHEN non_red_orders <= 5 THEN 'Average Performer'
        ELSE 'High Performer'
    END as performance_category,
    ROUND(non_red_orders * salary * commission_rate / 100.0, 2) as estimated_commission
FROM non_red_sales
ORDER BY non_red_orders DESC, salary DESC;
```

#### 4. **Company Relationship Analysis**
```sql
-- "Analyze which companies each salesperson works with"

WITH salesperson_companies AS (
    SELECT 
        s.name as salesperson_name,
        c.name as company_name,
        COUNT(o.order_id) as order_count,
        MIN(o.order_date) as first_order,
        MAX(o.order_date) as last_order
    FROM SalesPerson s
    JOIN Orders o ON s.sales_id = o.sales_id
    JOIN Company c ON o.com_id = c.com_id
    GROUP BY s.name, c.name
),
red_relationships AS (
    SELECT salesperson_name
    FROM salesperson_companies
    WHERE company_name = 'RED'
)
SELECT 
    sc.salesperson_name,
    STRING_AGG(sc.company_name, ', ') as companies_served,
    SUM(sc.order_count) as total_orders,
    CASE 
        WHEN rr.salesperson_name IS NOT NULL THEN 'Has RED Orders'
        ELSE 'No RED Orders'
    END as red_status
FROM salesperson_companies sc
LEFT JOIN red_relationships rr ON sc.salesperson_name = rr.salesperson_name
GROUP BY sc.salesperson_name, rr.salesperson_name
ORDER BY total_orders DESC;
```

#### 5. **Time-Based Analysis**
```sql
-- "Find salespersons who haven't sold to RED in the last 6 months"

SELECT s.name
FROM SalesPerson s
WHERE NOT EXISTS (
    SELECT 1
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE o.sales_id = s.sales_id 
    AND c.name = 'RED'
    AND o.order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH)
);

-- Comprehensive time-based analysis
WITH red_sales_timeline AS (
    SELECT 
        s.name,
        MAX(o.order_date) as last_red_order,
        COUNT(o.order_id) as total_red_orders
    FROM SalesPerson s
    JOIN Orders o ON s.sales_id = o.sales_id
    JOIN Company c ON o.com_id = c.com_id
    WHERE c.name = 'RED'
    GROUP BY s.name
)
SELECT 
    s.name,
    COALESCE(rst.last_red_order, 'Never') as last_red_order,
    COALESCE(rst.total_red_orders, 0) as total_red_orders,
    CASE 
        WHEN rst.last_red_order IS NULL THEN 'Never Sold to RED'
        WHEN rst.last_red_order < DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH) THEN 'No Recent RED Sales'
        WHEN rst.last_red_order < DATE_SUB(CURRENT_DATE, INTERVAL 6 MONTH) THEN 'No RED Sales in 6 Months'
        ELSE 'Recent RED Sales'
    END as red_sales_status
FROM SalesPerson s
LEFT JOIN red_sales_timeline rst ON s.name = rst.name
ORDER BY s.name;
```

#### 6. **Data Quality and Edge Cases**
```sql
-- "Handle data quality issues and edge cases"

-- Check for data inconsistencies
SELECT 
    'Data Quality Check' as check_type,
    COUNT(*) as total_orders,
    COUNT(CASE WHEN o.sales_id IS NULL THEN 1 END) as orders_without_salesperson,
    COUNT(CASE WHEN o.com_id IS NULL THEN 1 END) as orders_without_company,
    COUNT(CASE WHEN s.sales_id IS NULL THEN 1 END) as invalid_salesperson_refs,
    COUNT(CASE WHEN c.com_id IS NULL THEN 1 END) as invalid_company_refs
FROM Orders o
LEFT JOIN SalesPerson s ON o.sales_id = s.sales_id
LEFT JOIN Company c ON o.com_id = c.com_id;

-- Robust query handling NULL values and edge cases
SELECT s.name
FROM SalesPerson s
WHERE s.sales_id NOT IN (
    SELECT COALESCE(o.sales_id, -1)  -- Handle NULL sales_id
    FROM Orders o
    JOIN Company c ON o.com_id = c.com_id
    WHERE c.name = 'RED'
    AND o.sales_id IS NOT NULL
)
AND s.name IS NOT NULL  -- Ensure salesperson has a name
ORDER BY s.name;
```

## 🔗 Related LeetCode Questions

1. **#183 - Customers Who Never Order** (Similar NOT EXISTS pattern)
2. **#1179 - Reformat Department Table** (Complex joins)
3. **#1241 - Number of Comments per Post** (Multi-table analysis)
4. **#1355 - Activity Participants** (Finding users who did/didn't participate)
5. **#1393 - Capital Gain/Loss** (Transaction analysis)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **NOT EXISTS vs NOT IN**: NOT EXISTS is safer with NULL values
2. **Multi-table Joins**: Connecting data across related tables
3. **Negation Logic**: Finding records that DON'T meet criteria
4. **NULL Handling**: Critical for correct NOT IN behavior

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I exclude all orders or just recent ones?"
2. **Discuss performance**: "NOT EXISTS vs NOT IN vs LEFT JOIN performance trade-offs"
3. **Handle edge cases**: "What about NULL values in the data?"
4. **Think business context**: "This helps identify sales conflicts or compliance issues"

### 🔧 **Common Patterns**
- NOT EXISTS for finding missing relationships
- Multi-table JOINs for complex filtering
- LEFT JOIN with IS NULL for exclusion logic
- Subqueries for complex filtering criteria

### ⚠️ **Common Mistakes to Avoid**
1. **Using NOT IN without NULL checking** (can return unexpected results)
2. **Incorrect JOIN logic** leading to Cartesian products
3. **Missing DISTINCT** when needed in subqueries
4. **Performance issues** with large datasets and complex joins

### 🔍 **Performance Considerations**
- Index foreign key columns (sales_id, com_id)
- NOT EXISTS often performs better than NOT IN
- Consider materialized views for complex repeated queries
- Monitor query execution plans for optimization opportunities

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Ensuring sales teams don't conflict with customer relationships
- **Ownership**: Sales accountability and territory management
- **Data-Driven Decisions**: Using sales data for territory and account management
- **Operational Excellence**: Maintaining clean sales processes and avoiding conflicts

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find salespersons who sold to ALL companies except RED**
2. **Find the most profitable salesperson excluding RED sales**
3. **Calculate commission loss from not selling to RED**
4. **Find salespersons who only sell to companies in specific cities**

Remember: Sales territory management and conflict avoidance are crucial for Amazon's partner relationships, vendor management, and market segmentation strategies!