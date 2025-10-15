# LeetCode Easy #1084: Sales Analysis III

## 📋 Problem Statement

Write a SQL query that reports the products that were **only sold in the first quarter of 2019**. That is, between **2019-01-01** and **2019-03-31** inclusive.

Return the result table in **any order**.

## 🗄️ Table Schema

**Product Table:**
```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
| unit_price   | int     |
+--------------+---------+
```
- `product_id` is the primary key of this table.
- Each row of this table indicates the name and the price of each product.

**Sales Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| seller_id   | int     |
| product_id  | int     |
| buyer_id    | int     |
| sale_date   | date    |
| quantity    | int     |
| price       | int     |
+-------------+---------+
```
- This table has no primary key, it can have repeated rows.
- `product_id` is a foreign key to the Product table.
- Each row of this table contains some information about one sale.

## 📊 Sample Data

**Product Table:**
| product_id | product_name | unit_price |
|------------|--------------|------------|
| 1          | S8           | 1000       |
| 2          | G4           | 800        |
| 3          | iPhone       | 1400       |

**Sales Table:**
| seller_id | product_id | buyer_id | sale_date  | quantity | price |
|-----------|------------|----------|------------|----------|-------|
| 1         | 1          | 1        | 2019-01-21 | 2        | 2000  |
| 1         | 2          | 2        | 2019-02-17 | 1        | 800   |
| 2         | 2          | 3        | 2019-06-02 | 1        | 800   |
| 1         | 3          | 4        | 2019-05-13 | 2        | 2800  |

**Expected Output:**
| product_id | product_name |
|------------|--------------|
| 1          | S8           |

**Explanation:** 
- Product S8 (ID=1) was only sold on 2019-01-21 (Q1 2019)
- Product G4 (ID=2) was sold on 2019-02-17 (Q1) AND 2019-06-02 (Q2) - excluded
- Product iPhone (ID=3) was only sold on 2019-05-13 (Q2) - excluded

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find products sold ONLY in Q1 2019 (Jan 1 - Mar 31, 2019)
- Products sold in Q1 AND other periods should be excluded
- Products NOT sold in Q1 should be excluded
- Need to check all sales dates for each product

### 2. **Key Insights**
- Use date filtering to identify Q1 2019 sales
- GROUP BY product to check all sales dates
- Use HAVING with date conditions to ensure exclusivity
- Alternative: Use NOT EXISTS to exclude products sold outside Q1

### 3. **Interview Discussion Points**
- "This is a conditional aggregation problem with date filtering"
- "I need to ensure products were sold ONLY in Q1, not just sold in Q1"
- "NOT EXISTS or HAVING can filter products with sales outside the period"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Q1 2019 Sales
```sql
-- Filter sales between 2019-01-01 and 2019-03-31
-- These are the valid sales we're interested in
```

### Step 2: Find Products with ALL Sales in Q1
```sql
-- GROUP BY product_id and use HAVING to check all dates
-- Ensure no sales exist outside Q1 2019
```

### Step 3: Join with Product Table
```sql
-- Get product names for qualifying products
-- Return product_id and product_name
```

## ✅ Optimized SQL Solution

**Solution 1: Using NOT EXISTS**
```sql
SELECT 
    p.product_id,
    p.product_name
FROM Product p
WHERE EXISTS (
    SELECT 1 
    FROM Sales s 
    WHERE s.product_id = p.product_id 
    AND s.sale_date BETWEEN '2019-01-01' AND '2019-03-31'
)
AND NOT EXISTS (
    SELECT 1 
    FROM Sales s 
    WHERE s.product_id = p.product_id 
    AND s.sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31'
);
```

### Alternative Solutions

**Solution 2: Using HAVING with GROUP BY**
```sql
SELECT 
    p.product_id,
    p.product_name
FROM Product p
INNER JOIN Sales s ON p.product_id = s.product_id
GROUP BY p.product_id, p.product_name
HAVING MIN(s.sale_date) >= '2019-01-01' 
   AND MAX(s.sale_date) <= '2019-03-31';
```

**Solution 3: Using Conditional Counting**
```sql
SELECT 
    p.product_id,
    p.product_name
FROM Product p
INNER JOIN Sales s ON p.product_id = s.product_id
GROUP BY p.product_id, p.product_name
HAVING COUNT(CASE WHEN s.sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) > 0
   AND COUNT(CASE WHEN s.sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) = 0;
```

**Solution 4: Using Subquery with Set Operations**
```sql
SELECT 
    p.product_id,
    p.product_name
FROM Product p
WHERE p.product_id IN (
    SELECT product_id 
    FROM Sales 
    WHERE sale_date BETWEEN '2019-01-01' AND '2019-03-31'
)
AND p.product_id NOT IN (
    SELECT product_id 
    FROM Sales 
    WHERE sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31'
);
```

**Solution 5: Using Window Functions**
```sql
WITH sales_analysis AS (
    SELECT 
        p.product_id,
        p.product_name,
        COUNT(CASE WHEN s.sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) as q1_sales,
        COUNT(CASE WHEN s.sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) as non_q1_sales
    FROM Product p
    INNER JOIN Sales s ON p.product_id = s.product_id
    GROUP BY p.product_id, p.product_name
)
SELECT product_id, product_name
FROM sales_analysis
WHERE q1_sales > 0 AND non_q1_sales = 0;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Quarterly Sales Analysis**
```sql
-- "Analyze sales patterns across all quarters"

WITH quarterly_sales AS (
    SELECT 
        p.product_id,
        p.product_name,
        CASE 
            WHEN s.sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 'Q1'
            WHEN s.sale_date BETWEEN '2019-04-01' AND '2019-06-30' THEN 'Q2'
            WHEN s.sale_date BETWEEN '2019-07-01' AND '2019-09-30' THEN 'Q3'
            WHEN s.sale_date BETWEEN '2019-10-01' AND '2019-12-31' THEN 'Q4'
            ELSE 'Other Year'
        END as quarter,
        COUNT(*) as sales_count,
        SUM(s.quantity) as total_quantity,
        SUM(s.price) as total_revenue
    FROM Product p
    INNER JOIN Sales s ON p.product_id = s.product_id
    WHERE YEAR(s.sale_date) = 2019
    GROUP BY p.product_id, p.product_name, 
        CASE 
            WHEN s.sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 'Q1'
            WHEN s.sale_date BETWEEN '2019-04-01' AND '2019-06-30' THEN 'Q2'
            WHEN s.sale_date BETWEEN '2019-07-01' AND '2019-09-30' THEN 'Q3'
            WHEN s.sale_date BETWEEN '2019-10-01' AND '2019-12-31' THEN 'Q4'
            ELSE 'Other Year'
        END
),
product_quarter_summary AS (
    SELECT 
        product_id,
        product_name,
        COUNT(DISTINCT quarter) as quarters_active,
        GROUP_CONCAT(quarter ORDER BY quarter) as active_quarters,
        SUM(sales_count) as total_annual_sales,
        SUM(total_revenue) as total_annual_revenue
    FROM quarterly_sales
    GROUP BY product_id, product_name
)
SELECT 
    product_id,
    product_name,
    quarters_active,
    active_quarters,
    total_annual_sales,
    total_annual_revenue,
    CASE 
        WHEN active_quarters = 'Q1' THEN 'Q1 Only Product'
        WHEN quarters_active = 1 THEN 'Single Quarter Product'
        WHEN quarters_active = 2 THEN 'Two Quarter Product'
        WHEN quarters_active = 3 THEN 'Three Quarter Product'
        WHEN quarters_active = 4 THEN 'Year-Round Product'
        ELSE 'Irregular Pattern'
    END as sales_pattern,
    CASE 
        WHEN active_quarters = 'Q1' THEN 'Seasonal Launch'
        WHEN active_quarters LIKE '%Q1%' AND quarters_active > 1 THEN 'Extended Product'
        WHEN quarters_active = 4 THEN 'Core Product'
        ELSE 'Limited Run'
    END as product_category
FROM product_quarter_summary
ORDER BY 
    CASE WHEN active_quarters = 'Q1' THEN 1 ELSE 2 END,
    total_annual_revenue DESC;
```

#### 2. **Seasonal Product Performance Analysis**
```sql
-- "Identify seasonal patterns and product lifecycle stages"

WITH product_timeline AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.unit_price,
        MIN(s.sale_date) as first_sale_date,
        MAX(s.sale_date) as last_sale_date,
        COUNT(DISTINCT s.sale_date) as unique_sale_days,
        COUNT(*) as total_transactions,
        SUM(s.quantity) as total_units_sold,
        SUM(s.price) as total_revenue,
        AVG(s.price / s.quantity) as avg_selling_price
    FROM Product p
    INNER JOIN Sales s ON p.product_id = s.product_id
    GROUP BY p.product_id, p.product_name, p.unit_price
),
seasonal_analysis AS (
    SELECT 
        *,
        DATEDIFF(last_sale_date, first_sale_date) + 1 as sales_period_days,
        CASE 
            WHEN first_sale_date BETWEEN '2019-01-01' AND '2019-03-31' 
             AND last_sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 'Q1 Exclusive'
            WHEN first_sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 'Q1 Launch'
            WHEN last_sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 'Q1 Ended'
            ELSE 'Non-Q1 Product'
        END as q1_relationship,
        ROUND(total_revenue / total_units_sold, 2) as avg_price_per_unit,
        ROUND(avg_selling_price / unit_price, 2) as pricing_ratio
    FROM product_timeline
),
performance_metrics AS (
    SELECT 
        *,
        CASE 
            WHEN sales_period_days <= 90 THEN 'Short Lifecycle'
            WHEN sales_period_days <= 180 THEN 'Medium Lifecycle'
            ELSE 'Long Lifecycle'
        END as lifecycle_stage,
        CASE 
            WHEN total_revenue >= 5000 THEN 'High Revenue'
            WHEN total_revenue >= 2000 THEN 'Medium Revenue'
            ELSE 'Low Revenue'
        END as revenue_tier,
        CASE 
            WHEN pricing_ratio > 1.1 THEN 'Premium Pricing'
            WHEN pricing_ratio > 0.9 THEN 'Standard Pricing'
            ELSE 'Discounted Pricing'
        END as pricing_strategy
    FROM seasonal_analysis
)
SELECT 
    product_id,
    product_name,
    q1_relationship,
    lifecycle_stage,
    revenue_tier,
    pricing_strategy,
    total_revenue,
    total_units_sold,
    sales_period_days,
    unique_sale_days,
    first_sale_date,
    last_sale_date,
    ROUND(total_revenue / sales_period_days, 2) as daily_revenue_rate,
    CASE 
        WHEN q1_relationship = 'Q1 Exclusive' AND revenue_tier = 'High Revenue' THEN 'Successful Q1 Launch'
        WHEN q1_relationship = 'Q1 Exclusive' AND revenue_tier = 'Low Revenue' THEN 'Limited Q1 Product'
        WHEN q1_relationship = 'Q1 Launch' THEN 'Q1 Extended Product'
        ELSE 'Non-Q1 Focus'
    END as business_assessment
FROM performance_metrics
ORDER BY 
    CASE q1_relationship 
        WHEN 'Q1 Exclusive' THEN 1 
        WHEN 'Q1 Launch' THEN 2 
        ELSE 3 
    END,
    total_revenue DESC;
```

#### 3. **Data Quality and Edge Cases**
```sql
-- "Handle data quality issues and edge cases"

WITH data_quality_analysis AS (
    SELECT 
        'Sales Data Quality' as check_type,
        COUNT(*) as total_sales_records,
        COUNT(CASE WHEN sale_date IS NULL THEN 1 END) as null_sale_dates,
        COUNT(CASE WHEN product_id IS NULL THEN 1 END) as null_product_ids,
        COUNT(CASE WHEN price <= 0 THEN 1 END) as invalid_prices,
        COUNT(CASE WHEN quantity <= 0 THEN 1 END) as invalid_quantities,
        MIN(sale_date) as earliest_sale,
        MAX(sale_date) as latest_sale,
        COUNT(DISTINCT product_id) as unique_products_sold
    FROM Sales
),
product_quality_analysis AS (
    SELECT 
        'Product Data Quality' as check_type,
        COUNT(*) as total_product_records,
        COUNT(CASE WHEN product_id IS NULL THEN 1 END) as null_product_ids,
        COUNT(CASE WHEN product_name IS NULL OR TRIM(product_name) = '' THEN 1 END) as invalid_names,
        COUNT(CASE WHEN unit_price <= 0 THEN 1 END) as invalid_unit_prices,
        COUNT(DISTINCT product_id) as unique_products
    FROM Product
),
q1_specific_analysis AS (
    SELECT 
        'Q1 2019 Analysis' as check_type,
        COUNT(*) as q1_sales_count,
        COUNT(DISTINCT product_id) as q1_products_count,
        COUNT(DISTINCT sale_date) as q1_unique_days,
        SUM(quantity) as q1_total_quantity,
        SUM(price) as q1_total_revenue
    FROM Sales
    WHERE sale_date BETWEEN '2019-01-01' AND '2019-03-31'
),
q1_exclusive_validation AS (
    SELECT 
        p.product_id,
        p.product_name,
        COUNT(CASE WHEN s.sale_date BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) as q1_sales,
        COUNT(CASE WHEN s.sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31' THEN 1 END) as non_q1_sales,
        COUNT(*) as total_sales,
        MIN(s.sale_date) as first_sale,
        MAX(s.sale_date) as last_sale
    FROM Product p
    LEFT JOIN Sales s ON p.product_id = s.product_id
    GROUP BY p.product_id, p.product_name
)
SELECT 
    'Q1 Exclusive Products Validation' as analysis_type,
    COUNT(CASE WHEN q1_sales > 0 AND non_q1_sales = 0 THEN 1 END) as q1_exclusive_count,
    COUNT(CASE WHEN q1_sales > 0 AND non_q1_sales > 0 THEN 1 END) as q1_plus_other_periods,
    COUNT(CASE WHEN q1_sales = 0 AND non_q1_sales > 0 THEN 1 END) as non_q1_only,
    COUNT(CASE WHEN total_sales = 0 THEN 1 END) as never_sold,
    ROUND(COUNT(CASE WHEN q1_sales > 0 AND non_q1_sales = 0 THEN 1 END) * 100.0 / 
          COUNT(CASE WHEN total_sales > 0 THEN 1 END), 2) as q1_exclusive_percentage
FROM q1_exclusive_validation;
```

#### 4. **Business Impact Analysis**
```sql
-- "Analyze business impact of Q1-only products"

WITH q1_exclusive_products AS (
    SELECT 
        p.product_id,
        p.product_name,
        p.unit_price
    FROM Product p
    WHERE EXISTS (
        SELECT 1 FROM Sales s 
        WHERE s.product_id = p.product_id 
        AND s.sale_date BETWEEN '2019-01-01' AND '2019-03-31'
    )
    AND NOT EXISTS (
        SELECT 1 FROM Sales s 
        WHERE s.product_id = p.product_id 
        AND s.sale_date NOT BETWEEN '2019-01-01' AND '2019-03-31'
    )
),
q1_sales_metrics AS (
    SELECT 
        qep.product_id,
        qep.product_name,
        qep.unit_price,
        COUNT(s.seller_id) as total_transactions,
        COUNT(DISTINCT s.seller_id) as unique_sellers,
        COUNT(DISTINCT s.buyer_id) as unique_buyers,
        SUM(s.quantity) as total_units_sold,
        SUM(s.price) as total_revenue,
        AVG(s.price / s.quantity) as avg_selling_price,
        MIN(s.sale_date) as first_sale_date,
        MAX(s.sale_date) as last_sale_date
    FROM q1_exclusive_products qep
    INNER JOIN Sales s ON qep.product_id = s.product_id
    GROUP BY qep.product_id, qep.product_name, qep.unit_price
),
market_comparison AS (
    SELECT 
        'Q1 Exclusive Products' as product_category,
        COUNT(*) as product_count,
        SUM(total_revenue) as category_revenue,
        AVG(total_revenue) as avg_revenue_per_product,
        SUM(total_units_sold) as category_units_sold,
        AVG(total_units_sold) as avg_units_per_product
    FROM q1_sales_metrics
    
    UNION ALL
    
    SELECT 
        'All Products (2019)' as product_category,
        COUNT(DISTINCT p.product_id) as product_count,
        SUM(s.price) as category_revenue,
        AVG(product_revenue.revenue) as avg_revenue_per_product,
        SUM(s.quantity) as category_units_sold,
        AVG(product_units.units) as avg_units_per_product
    FROM Product p
    INNER JOIN Sales s ON p.product_id = s.product_id
    CROSS JOIN (
        SELECT AVG(SUM(price)) as revenue
        FROM Sales 
        WHERE YEAR(sale_date) = 2019
        GROUP BY product_id
    ) product_revenue
    CROSS JOIN (
        SELECT AVG(SUM(quantity)) as units
        FROM Sales 
        WHERE YEAR(sale_date) = 2019
        GROUP BY product_id
    ) product_units
    WHERE YEAR(s.sale_date) = 2019
)
SELECT 
    qsm.product_id,
    qsm.product_name,
    qsm.total_revenue,
    qsm.total_units_sold,
    qsm.unique_buyers,
    qsm.unique_sellers,
    ROUND(qsm.avg_selling_price, 2) as avg_price,
    ROUND(qsm.avg_selling_price / qsm.unit_price, 2) as price_ratio,
    DATEDIFF(qsm.last_sale_date, qsm.first_sale_date) + 1 as active_days,
    ROUND(qsm.total_revenue / (DATEDIFF(qsm.last_sale_date, qsm.first_sale_date) + 1), 2) as daily_revenue,
    CASE 
        WHEN qsm.total_revenue > 3000 THEN 'High Impact Q1 Product'
        WHEN qsm.total_revenue > 1500 THEN 'Medium Impact Q1 Product'
        ELSE 'Low Impact Q1 Product'
    END as business_impact,
    CASE 
        WHEN DATEDIFF(qsm.last_sale_date, qsm.first_sale_date) <= 30 THEN 'Flash Sale Product'
        WHEN DATEDIFF(qsm.last_sale_date, qsm.first_sale_date) <= 60 THEN 'Limited Run Product'
        ELSE 'Quarter-Long Product'
    END as product_type
FROM q1_sales_metrics qsm
ORDER BY qsm.total_revenue DESC, qsm.total_units_sold DESC;
```

#### 5. **Performance Optimization**
```sql
-- "Optimize query performance for large datasets"

-- Create indexes for efficient date range queries
-- CREATE INDEX idx_sales_date_product ON Sales(sale_date, product_id);
-- CREATE INDEX idx_sales_product_date ON Sales(product_id, sale_date);

-- Optimized version using indexed date ranges
EXPLAIN ANALYZE
SELECT 
    p.product_id,
    p.product_name
FROM Product p
WHERE EXISTS (
    SELECT 1 
    FROM Sales s 
    WHERE s.product_id = p.product_id 
    AND s.sale_date >= '2019-01-01' 
    AND s.sale_date <= '2019-03-31'
)
AND NOT EXISTS (
    SELECT 1 
    FROM Sales s 
    WHERE s.product_id = p.product_id 
    AND (s.sale_date < '2019-01-01' OR s.sale_date > '2019-03-31')
);

-- Alternative optimized approach using window functions
WITH sales_date_analysis AS (
    SELECT 
        product_id,
        MIN(sale_date) as min_sale_date,
        MAX(sale_date) as max_sale_date,
        COUNT(*) as total_sales
    FROM Sales
    GROUP BY product_id
)
SELECT 
    p.product_id,
    p.product_name
FROM Product p
INNER JOIN sales_date_analysis sda ON p.product_id = sda.product_id
WHERE sda.min_sale_date >= '2019-01-01' 
  AND sda.max_sale_date <= '2019-03-31';

-- Materialized view approach for repeated queries
-- CREATE MATERIALIZED VIEW quarterly_product_sales AS
-- SELECT 
--     product_id,
--     QUARTER(sale_date) as quarter,
--     YEAR(sale_date) as year,
--     COUNT(*) as sales_count,
--     SUM(quantity) as total_quantity,
--     SUM(price) as total_revenue
-- FROM Sales
-- GROUP BY product_id, QUARTER(sale_date), YEAR(sale_date);

-- Query using materialized view:
-- SELECT DISTINCT p.product_id, p.product_name
-- FROM Product p
-- INNER JOIN quarterly_product_sales qps ON p.product_id = qps.product_id
-- WHERE qps.year = 2019 AND qps.quarter = 1
-- AND p.product_id NOT IN (
--     SELECT product_id FROM quarterly_product_sales 
--     WHERE year != 2019 OR quarter != 1
-- );
```

## 🔗 Related LeetCode Questions

1. **#1068 - Product Sales Analysis I** (Basic product-sales JOIN)
2. **#1141 - User Activity for the Past 30 Days I** (Date filtering with GROUP BY)
3. **#1393 - Capital Gain/Loss** (Complex conditional aggregation)
4. **#586 - Customer Placing the Largest Number of Orders** (GROUP BY with conditions)
5. **#1179 - Reformat Department Table** (Advanced conditional logic)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Date Range Filtering**: Using BETWEEN for inclusive date ranges
2. **EXISTS vs NOT EXISTS**: Checking for presence/absence of related data
3. **Conditional Aggregation**: Using HAVING with GROUP BY for complex filtering
4. **Set Operations**: Using IN/NOT IN for membership testing

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I include products never sold, or only those with sales?"
2. **Discuss alternatives**: "EXISTS vs HAVING vs conditional counting trade-offs"
3. **Consider performance**: "Date range indexes are crucial for large sales tables"
4. **Think business context**: "This helps identify seasonal or promotional products"

### 🔧 **Common Patterns**
- Date range filtering with BETWEEN
- EXISTS/NOT EXISTS for conditional filtering
- GROUP BY with HAVING for aggregate conditions
- Conditional counting with CASE statements

### ⚠️ **Common Mistakes to Avoid**
1. **Incorrect date ranges** (using exclusive vs inclusive boundaries)
2. **Missing NOT EXISTS logic** (only checking for Q1 sales, not exclusivity)
3. **Performance issues** (not indexing date columns properly)
4. **Edge case handling** (products with no sales at all)

### 🔍 **Performance Considerations**
- Index on (sale_date, product_id) for efficient date range queries
- Consider partitioning large sales tables by date
- Use EXISTS instead of IN for better performance with NULLs
- Monitor query execution plans for optimization opportunities

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding seasonal product preferences
- **Data-Driven Decisions**: Using sales timing for inventory and marketing planning
- **Operational Excellence**: Efficient seasonal product identification
- **Frugality**: Optimized queries for large-scale sales analysis

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find products sold only in Q4 2019**
2. **Find products sold in exactly two consecutive quarters**
3. **Calculate revenue contribution of Q1-only products**
4. **Identify products with seasonal sales patterns**

Remember: Seasonal analysis and time-based product segmentation are essential for Amazon's inventory planning, promotional strategies, seasonal recommendations, and supply chain optimization!