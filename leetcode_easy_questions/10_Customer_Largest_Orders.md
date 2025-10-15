# LeetCode Easy #586: Customer Placing the Largest Number of Orders

## 📋 Problem Statement

Write a SQL query to find the customer who has placed the **largest number of orders**.

**Note:** It is guaranteed that exactly one customer has placed more orders than any other customer.

## 🗄️ Table Schema

**Orders Table:**
```
+-----------------+----------+
| Column Name     | Type     |
+-----------------+----------+
| order_number    | int      |
| customer_number | int      |
+-----------------+----------+
```
- `order_number` is the primary key column for this table.
- This table contains information about the order ID and the customer ID.

## 📊 Sample Data

**Orders Table:**
| order_number | customer_number |
|--------------|-----------------|
| 1            | 1               |
| 2            | 2               |
| 3            | 3               |
| 4            | 3               |

**Expected Output:**
| customer_number |
|-----------------|
| 3               |

**Explanation:** Customer 3 has placed 2 orders, which is more than any other customer.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to count orders per customer
- Find the customer with the maximum count
- This is a GROUP BY with ORDER BY LIMIT or MAX aggregation problem
- Guaranteed unique answer simplifies the solution

### 2. **Key Insights**
- GROUP BY customer_number to count orders per customer
- Use COUNT(*) to count orders for each customer
- ORDER BY count DESC LIMIT 1 to get the top customer
- Alternative: Use subquery to find MAX count

### 3. **Interview Discussion Points**
- "I need to count orders per customer and find the maximum"
- "GROUP BY customer_number with COUNT(*) gives me order counts"
- "Since it's guaranteed unique, I can use LIMIT 1 safely"

## 🔧 Step-by-Step Solution Logic

### Step 1: Count Orders per Customer
```sql
-- GROUP BY customer_number
-- COUNT(*) to count orders for each customer
```

### Step 2: Find Maximum Count
```sql
-- ORDER BY count DESC to get highest first
-- Use LIMIT 1 to get just the top customer
```

### Step 3: Return Customer Number
```sql
-- Select only the customer_number column
```

## ✅ Optimized SQL Solution

**Solution 1: ORDER BY with LIMIT**
```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

### Alternative Solutions

**Solution 2: Using Subquery with MAX**
```sql
SELECT customer_number
FROM Orders
GROUP BY customer_number
HAVING COUNT(*) = (
    SELECT MAX(order_count)
    FROM (
        SELECT COUNT(*) as order_count
        FROM Orders
        GROUP BY customer_number
    ) as counts
);
```

**Solution 3: Using Window Functions**
```sql
SELECT customer_number
FROM (
    SELECT customer_number,
           RANK() OVER (ORDER BY COUNT(*) DESC) as rnk
    FROM Orders
    GROUP BY customer_number
) ranked
WHERE rnk = 1;
```

**Solution 4: Using TOP (SQL Server)**
```sql
SELECT TOP 1 customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC;
```

**Solution 5: Including Order Count in Output**
```sql
SELECT 
    customer_number,
    COUNT(*) as order_count
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Handle Ties**
```sql
-- "What if multiple customers have the same highest order count?"

-- Return all customers with maximum order count
SELECT customer_number
FROM Orders
GROUP BY customer_number
HAVING COUNT(*) = (
    SELECT MAX(order_count)
    FROM (
        SELECT COUNT(*) as order_count
        FROM Orders
        GROUP BY customer_number
    ) as customer_counts
);

-- Or using window functions
SELECT customer_number
FROM (
    SELECT customer_number,
           COUNT(*) as order_count,
           RANK() OVER (ORDER BY COUNT(*) DESC) as rnk
    FROM Orders
    GROUP BY customer_number
) ranked
WHERE rnk = 1;
```

#### 2. **Top N Customers**
```sql
-- "Find the top 3 customers by order count"

SELECT 
    customer_number,
    COUNT(*) as order_count,
    RANK() OVER (ORDER BY COUNT(*) DESC) as rank_position
FROM Orders
GROUP BY customer_number
ORDER BY order_count DESC
LIMIT 3;
```

#### 3. **Customer Order Analysis**
```sql
-- "Provide comprehensive customer ordering statistics"

WITH CustomerStats AS (
    SELECT 
        customer_number,
        COUNT(*) as order_count,
        MIN(order_number) as first_order,
        MAX(order_number) as latest_order
    FROM Orders
    GROUP BY customer_number
)
SELECT 
    customer_number,
    order_count,
    first_order,
    latest_order,
    CASE 
        WHEN order_count >= 5 THEN 'VIP'
        WHEN order_count >= 3 THEN 'Frequent'
        WHEN order_count >= 2 THEN 'Regular'
        ELSE 'New'
    END as customer_tier
FROM CustomerStats
ORDER BY order_count DESC;
```

#### 4. **Performance Optimization**
```sql
-- "How would you optimize this for millions of orders?"

-- Add index on customer_number for efficient grouping
CREATE INDEX idx_orders_customer ON Orders(customer_number);

-- The basic query remains efficient with proper indexing
SELECT customer_number
FROM Orders
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;

-- For very large datasets, consider pre-aggregated tables
CREATE TABLE customer_order_summary AS
SELECT 
    customer_number,
    COUNT(*) as order_count,
    MAX(order_number) as last_order_number
FROM Orders
GROUP BY customer_number;
```

#### 5. **Time-Based Analysis**
```sql
-- "Find the customer with most orders in the last 30 days"

-- Assuming Orders table has an order_date column
SELECT customer_number
FROM Orders
WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 30 DAY)
GROUP BY customer_number
ORDER BY COUNT(*) DESC
LIMIT 1;

-- Monthly top customer analysis
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    customer_number,
    COUNT(*) as monthly_orders
FROM Orders
WHERE order_date >= DATE_SUB(CURRENT_DATE, INTERVAL 12 MONTH)
GROUP BY YEAR(order_date), MONTH(order_date), customer_number
HAVING COUNT(*) = (
    SELECT MAX(monthly_count)
    FROM (
        SELECT COUNT(*) as monthly_count
        FROM Orders o2
        WHERE YEAR(o2.order_date) = YEAR(Orders.order_date)
        AND MONTH(o2.order_date) = MONTH(Orders.order_date)
        GROUP BY customer_number
    ) monthly_max
)
ORDER BY year DESC, month DESC;
```

#### 6. **Business Intelligence Insights**
```sql
-- "Provide customer segmentation based on order frequency"

WITH CustomerMetrics AS (
    SELECT 
        customer_number,
        COUNT(*) as order_count
    FROM Orders
    GROUP BY customer_number
),
OrderDistribution AS (
    SELECT 
        order_count,
        COUNT(*) as customers_with_this_count
    FROM CustomerMetrics
    GROUP BY order_count
)
SELECT 
    'Order Frequency Distribution' as analysis_type,
    order_count,
    customers_with_this_count,
    ROUND(customers_with_this_count * 100.0 / SUM(customers_with_this_count) OVER(), 2) as percentage
FROM OrderDistribution
ORDER BY order_count DESC;
```

## 🔗 Related LeetCode Questions

1. **#511 - Game Play Analysis I** (Basic GROUP BY aggregation)
2. **#1084 - Sales Analysis III** (GROUP BY with filtering)
3. **#1141 - User Activity for the Past 30 Days I** (Time-based aggregation)
4. **#1393 - Capital Gain/Loss** (GROUP BY with calculations)
5. **#1484 - Group Sold Products By The Date** (GROUP BY with string aggregation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY + COUNT**: Standard pattern for counting occurrences
2. **ORDER BY + LIMIT**: Simple way to get top N results
3. **Aggregation Functions**: COUNT, MAX, MIN for statistical analysis
4. **Window Functions**: Advanced ranking and analysis capabilities

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I handle ties or is the answer guaranteed unique?"
2. **Discuss alternatives**: "I could use window functions for more complex ranking needs"
3. **Consider performance**: "For large datasets, indexing customer_number is crucial"
4. **Think business value**: "This helps identify VIP customers for special treatment"

### 🔧 **Common Patterns**
- GROUP BY + ORDER BY + LIMIT for finding top records
- Use HAVING instead of WHERE for filtering grouped results
- Consider window functions for complex ranking scenarios
- Index columns used in GROUP BY for better performance

### ⚠️ **Common Mistakes to Avoid**
1. **Using WHERE instead of HAVING** for filtering grouped results
2. **Forgetting to handle ties** when they're possible
3. **Not considering performance** implications of GROUP BY on large datasets
4. **Missing the business context** of customer analysis

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Identifying top customers enables better service
- **Data-Driven Decisions**: Use customer behavior data for business strategy
- **Think Big**: Scale analysis to millions of customers and orders
- **Frugality**: Efficient queries save computational resources

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find customers who have placed exactly 2 orders**
2. **Find the customer with the highest average order value**
3. **Find customers who haven't placed orders in the last 90 days**
4. **Calculate the percentage of total orders each customer represents**

Remember: Customer analysis is fundamental to Amazon's recommendation systems, inventory management, and customer retention strategies!