# LeetCode Easy #1068: Product Sales Analysis I

## 📋 Problem Statement

Write a SQL query that reports the **product_name**, **year**, and **price** for each **sale_id** in the **Sales** table.

Return the result table in **any order**.

## 🗄️ Table Schema

**Sales Table:**
```
+-------------+-------+
| Column Name | Type  |
+-------------+-------+
| sale_id     | int   |
| product_id  | int   |
| year        | int   |
| quantity    | int   |
| price       | int   |
+-------------+-------+
```
- `(sale_id, year)` is the primary key of this table.
- `product_id` is a foreign key to Product table.
- Each row of this table shows a sale on the product `product_id` in a certain `year`.
- Note that the `price` is per unit.

**Product Table:**
```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| product_id   | int     |
| product_name | varchar |
+--------------+---------+
```
- `product_id` is the primary key of this table.
- Each row of this table indicates the product name of each product.

## 📊 Sample Data

**Sales Table:**
| sale_id | product_id | year | quantity | price |
|---------|------------|------|----------|-------|
| 1       | 100        | 2008 | 10       | 5000  |
| 2       | 100        | 2009 | 12       | 5000  |
| 7       | 200        | 2011 | 15       | 9000  |

**Product Table:**
| product_id | product_name |
|------------|--------------|
| 100        | Nokia        |
| 200        | Apple        |
| 300        | Samsung      |

**Expected Output:**
| product_name | year | price |
|--------------|------|-------|
| Nokia        | 2008 | 5000  |
| Nokia        | 2009 | 5000  |
| Apple        | 2011 | 9000  |

**Explanation:** 
- Sale 1: Nokia sold in 2008 for 5000 per unit
- Sale 2: Nokia sold in 2009 for 5000 per unit  
- Sale 7: Apple sold in 2011 for 9000 per unit

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to join Sales and Product tables
- Get product_name from Product table
- Get year and price from Sales table
- Simple INNER JOIN on product_id

### 2. **Key Insights**
- Sales table has the transaction data
- Product table has the product names
- JOIN on product_id to connect the data
- Select required columns from both tables

### 3. **Interview Discussion Points**
- "This is a basic JOIN problem to combine sales and product data"
- "I need product_name from Product table and year, price from Sales"
- "INNER JOIN ensures only valid product sales are included"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Join Relationship
```sql
-- Sales.product_id = Product.product_id
-- This connects sales transactions to product details
```

### Step 2: Select Required Columns
```sql
-- product_name from Product table
-- year and price from Sales table
```

### Step 3: Perform JOIN
```sql
-- INNER JOIN to match sales with products
-- Return only valid combinations
```

## ✅ Optimized SQL Solution

**Solution 1: INNER JOIN**
```sql
SELECT 
    p.product_name,
    s.year,
    s.price
FROM Sales s
INNER JOIN Product p ON s.product_id = p.product_id;
```

### Alternative Solutions

**Solution 2: Using Table Aliases**
```sql
SELECT 
    p.product_name,
    s.year,
    s.price
FROM Sales s
JOIN Product p ON s.product_id = p.product_id;
```

**Solution 3: Explicit Column Selection**
```sql
SELECT 
    Product.product_name,
    Sales.year,
    Sales.price
FROM Sales
INNER JOIN Product ON Sales.product_id = Product.product_id;
```

**Solution 4: With Additional Sales Information**
```sql
SELECT 
    p.product_name,
    s.year,
    s.price,
    s.quantity,
    s.price * s.quantity as total_sale_value
FROM Sales s
INNER JOIN Product p ON s.product_id = p.product_id;
```

**Solution 5: Ordered by Year**
```sql
SELECT 
    p.product_name,
    s.year,
    s.price
FROM Sales s
INNER JOIN Product p ON s.product_id = p.product_id
ORDER BY s.year, p.product_name;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Sales Analytics**
```sql
-- "Provide detailed sales analysis with product information"

WITH sales_analytics AS (
    SELECT 
        p.product_name,
        s.year,
        s.price,
        s.quantity,
        s.price * s.quantity as sale_value,
        COUNT(*) OVER (PARTITION BY p.product_id) as total_sales_count,
        SUM(s.quantity) OVER (PARTITION BY p.product_id) as total_quantity_sold,
        AVG(s.price) OVER (PARTITION BY p.product_id) as avg_price_per_product,
        MAX(s.year) OVER (PARTITION BY p.product_id) as latest_sale_year,
        MIN(s.year) OVER (PARTITION BY p.product_id) as first_sale_year
    FROM Sales s
    INNER JOIN Product p ON s.product_id = p.product_id
)
SELECT 
    product_name,
    year,
    price,
    quantity,
    sale_value,
    total_sales_count,
    total_quantity_sold,
    ROUND(avg_price_per_product, 2) as avg_price,
    latest_sale_year,
    first_sale_year,
    latest_sale_year - first_sale_year + 1 as years_in_market
FROM sales_analytics
ORDER BY product_name, year;
```

#### 2. **Product Performance Comparison**
```sql
-- "Compare product performance across years"

WITH product_performance AS (
    SELECT 
        p.product_name,
        s.year,
        COUNT(*) as sales_transactions,
        SUM(s.quantity) as total_quantity,
        SUM(s.price * s.quantity) as total_revenue,
        AVG(s.price) as avg_unit_price,
        MIN(s.price) as min_price,
        MAX(s.price) as max_price
    FROM Sales s
    INNER JOIN Product p ON s.product_id = p.product_id
    GROUP BY p.product_name, s.year
),
yearly_totals AS (
    SELECT 
        year,
        SUM(total_revenue) as year_total_revenue,
        SUM(total_quantity) as year_total_quantity
    FROM product_performance
    GROUP BY year
)
SELECT 
    pp.product_name,
    pp.year,
    pp.sales_transactions,
    pp.total_quantity,
    pp.total_revenue,
    ROUND(pp.avg_unit_price, 2) as avg_price,
    ROUND(pp.total_revenue * 100.0 / yt.year_total_revenue, 2) as market_share_percentage,
    ROUND(pp.total_quantity * 100.0 / yt.year_total_quantity, 2) as quantity_share_percentage,
    CASE 
        WHEN pp.total_revenue > yt.year_total_revenue * 0.4 THEN 'Market Leader'
        WHEN pp.total_revenue > yt.year_total_revenue * 0.2 THEN 'Major Player'
        WHEN pp.total_revenue > yt.year_total_revenue * 0.1 THEN 'Significant Player'
        ELSE 'Niche Player'
    END as market_position
FROM product_performance pp
JOIN yearly_totals yt ON pp.year = yt.year
ORDER BY pp.year, pp.total_revenue DESC;
```

#### 3. **Time Series Analysis**
```sql
-- "Analyze sales trends over time"

WITH time_series_analysis AS (
    SELECT 
        p.product_name,
        s.year,
        SUM(s.price * s.quantity) as annual_revenue,
        SUM(s.quantity) as annual_quantity,
        AVG(s.price) as avg_annual_price,
        LAG(SUM(s.price * s.quantity)) OVER (PARTITION BY p.product_name ORDER BY s.year) as prev_year_revenue,
        LEAD(SUM(s.price * s.quantity)) OVER (PARTITION BY p.product_name ORDER BY s.year) as next_year_revenue
    FROM Sales s
    INNER JOIN Product p ON s.product_id = p.product_id
    GROUP BY p.product_name, s.year
),
growth_analysis AS (
    SELECT 
        *,
        CASE 
            WHEN prev_year_revenue IS NOT NULL THEN
                ROUND((annual_revenue - prev_year_revenue) * 100.0 / prev_year_revenue, 2)
            ELSE NULL
        END as yoy_growth_percentage,
        CASE 
            WHEN prev_year_revenue IS NOT NULL THEN
                CASE 
                    WHEN annual_revenue > prev_year_revenue * 1.2 THEN 'High Growth'
                    WHEN annual_revenue > prev_year_revenue * 1.05 THEN 'Moderate Growth'
                    WHEN annual_revenue > prev_year_revenue * 0.95 THEN 'Stable'
                    WHEN annual_revenue > prev_year_revenue * 0.8 THEN 'Declining'
                    ELSE 'Significant Decline'
                END
            ELSE 'New Product'
        END as growth_category
    FROM time_series_analysis
)
SELECT 
    product_name,
    year,
    annual_revenue,
    annual_quantity,
    ROUND(avg_annual_price, 2) as avg_price,
    yoy_growth_percentage,
    growth_category,
    CASE 
        WHEN next_year_revenue IS NULL THEN 'Last Year in Data'
        WHEN next_year_revenue > annual_revenue THEN 'Continued Growth Expected'
        ELSE 'Growth Concerns'
    END as outlook
FROM growth_analysis
ORDER BY product_name, year;
```

#### 4. **Product Lifecycle Analysis**
```sql
-- "Analyze product lifecycle stages"

WITH product_lifecycle AS (
    SELECT 
        p.product_name,
        MIN(s.year) as launch_year,
        MAX(s.year) as latest_year,
        COUNT(DISTINCT s.year) as years_active,
        SUM(s.price * s.quantity) as total_lifetime_revenue,
        SUM(s.quantity) as total_lifetime_quantity,
        AVG(s.price) as avg_lifetime_price,
        MAX(s.price) as peak_price,
        MIN(s.price) as lowest_price
    FROM Sales s
    INNER JOIN Product p ON s.product_id = p.product_id
    GROUP BY p.product_name
),
lifecycle_metrics AS (
    SELECT 
        *,
        latest_year - launch_year + 1 as potential_years,
        ROUND(years_active * 100.0 / (latest_year - launch_year + 1), 2) as market_presence_percentage,
        ROUND(total_lifetime_revenue / years_active, 2) as avg_annual_revenue,
        CASE 
            WHEN years_active = 1 THEN 'New/Single Year'
            WHEN years_active <= 2 THEN 'Early Stage'
            WHEN years_active <= 5 THEN 'Growth Stage'
            WHEN years_active <= 10 THEN 'Mature Stage'
            ELSE 'Legacy Stage'
        END as lifecycle_stage
    FROM product_lifecycle
)
SELECT 
    product_name,
    launch_year,
    latest_year,
    years_active,
    lifecycle_stage,
    ROUND(total_lifetime_revenue, 2) as lifetime_revenue,
    total_lifetime_quantity,
    ROUND(avg_lifetime_price, 2) as avg_price,
    peak_price,
    lowest_price,
    ROUND(avg_annual_revenue, 2) as avg_annual_revenue,
    market_presence_percentage,
    CASE 
        WHEN avg_annual_revenue > 50000 THEN 'High Value Product'
        WHEN avg_annual_revenue > 20000 THEN 'Medium Value Product'
        ELSE 'Low Value Product'
    END as value_category
FROM lifecycle_metrics
ORDER BY total_lifetime_revenue DESC;
```

#### 5. **Data Quality and Validation**
```sql
-- "Validate data quality and identify potential issues"

WITH data_quality_checks AS (
    SELECT 
        'Sales Data Quality' as check_type,
        COUNT(*) as total_sales_records,
        COUNT(CASE WHEN s.product_id IS NULL THEN 1 END) as null_product_ids,
        COUNT(CASE WHEN s.year IS NULL THEN 1 END) as null_years,
        COUNT(CASE WHEN s.price IS NULL OR s.price <= 0 THEN 1 END) as invalid_prices,
        COUNT(CASE WHEN s.quantity IS NULL OR s.quantity <= 0 THEN 1 END) as invalid_quantities,
        MIN(s.year) as earliest_year,
        MAX(s.year) as latest_year,
        COUNT(DISTINCT s.product_id) as unique_products_in_sales
    FROM Sales s
),
product_quality_checks AS (
    SELECT 
        'Product Data Quality' as check_type,
        COUNT(*) as total_product_records,
        COUNT(CASE WHEN p.product_id IS NULL THEN 1 END) as null_product_ids,
        COUNT(CASE WHEN p.product_name IS NULL OR TRIM(p.product_name) = '' THEN 1 END) as invalid_names,
        COUNT(DISTINCT p.product_id) as unique_products
    FROM Product p
),
join_quality_checks AS (
    SELECT 
        'Join Quality' as check_type,
        COUNT(*) as successful_joins,
        COUNT(CASE WHEN p.product_name IS NULL THEN 1 END) as orphaned_sales,
        COUNT(DISTINCT s.product_id) as products_with_sales,
        (SELECT COUNT(*) FROM Product) as total_products,
        ROUND(COUNT(DISTINCT s.product_id) * 100.0 / (SELECT COUNT(*) FROM Product), 2) as product_utilization_rate
    FROM Sales s
    LEFT JOIN Product p ON s.product_id = p.product_id
)
SELECT check_type, 
       total_sales_records, null_product_ids, null_years, invalid_prices, invalid_quantities,
       earliest_year, latest_year, unique_products_in_sales
FROM data_quality_checks
UNION ALL
SELECT check_type, 
       total_product_records, null_product_ids, invalid_names, unique_products, NULL, NULL, NULL, NULL
FROM product_quality_checks
UNION ALL
SELECT check_type, 
       successful_joins, orphaned_sales, products_with_sales, total_products, 
       product_utilization_rate, NULL, NULL, NULL
FROM join_quality_checks;
```

#### 6. **Performance Optimization**
```sql
-- "Optimize for large-scale sales data"

-- Create indexes for better JOIN performance
-- CREATE INDEX idx_sales_product_id ON Sales(product_id);
-- CREATE INDEX idx_sales_year ON Sales(year);
-- CREATE INDEX idx_product_name ON Product(product_name);

-- Optimized query with execution plan
EXPLAIN ANALYZE
SELECT 
    p.product_name,
    s.year,
    s.price
FROM Sales s
INNER JOIN Product p ON s.product_id = p.product_id;

-- For very large datasets, consider partitioning
WITH partitioned_analysis AS (
    SELECT 
        p.product_name,
        s.year,
        s.price,
        ROW_NUMBER() OVER (PARTITION BY s.year ORDER BY s.sale_id) as yearly_rank
    FROM Sales s
    INNER JOIN Product p ON s.product_id = p.product_id
    WHERE s.year >= 2020  -- Filter to recent years for performance
)
SELECT 
    product_name,
    year,
    price
FROM partitioned_analysis
WHERE yearly_rank <= 1000;  -- Limit results for performance

-- Materialized view approach for frequent queries
-- CREATE MATERIALIZED VIEW sales_product_summary AS
-- SELECT 
--     p.product_name,
--     s.year,
--     s.price,
--     s.quantity,
--     s.price * s.quantity as sale_value
-- FROM Sales s
-- INNER JOIN Product p ON s.product_id = p.product_id;

-- Usage of materialized view:
-- SELECT product_name, year, price FROM sales_product_summary;
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (Basic JOIN operations)
2. **#181 - Employees Earning More Than Managers** (Self-join patterns)
3. **#577 - Employee Bonus** (LEFT JOIN with NULL handling)
4. **#1075 - Project Employees I** (JOIN with aggregation)
5. **#1084 - Sales Analysis III** (Complex sales analysis)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **INNER JOIN**: Combining related tables on foreign keys
2. **Table Aliases**: Simplifying complex queries with short names
3. **Column Selection**: Choosing specific columns from joined tables
4. **Primary Key Relationships**: Understanding table connections

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I include all sales or filter by specific criteria?"
2. **Discuss performance**: "INNER JOIN with proper indexing is efficient"
3. **Consider data quality**: "What if there are orphaned sales without products?"
4. **Think business context**: "This helps analyze product sales performance"

### 🔧 **Common Patterns**
- INNER JOIN on foreign key relationships
- SELECT specific columns from multiple tables
- Table aliases for readability
- ORDER BY for result organization

### ⚠️ **Common Mistakes to Avoid**
1. **Missing JOIN conditions** (Cartesian products)
2. **Wrong JOIN type** (LEFT JOIN when INNER JOIN is needed)
3. **Ambiguous column references** (not using table aliases)
4. **Performance issues** (missing indexes on JOIN columns)

### 🔍 **Performance Considerations**
- Index foreign key columns for efficient JOINs
- Use INNER JOIN when only matching records are needed
- Consider query execution plans for optimization
- Partition large tables by date for better performance

### 🎯 **Amazon Leadership Principles Applied**
- **Data-Driven Decisions**: Combining sales and product data for insights
- **Customer Obsession**: Understanding product performance for customers
- **Operational Excellence**: Efficient data retrieval and analysis
- **Frugality**: Optimized queries for cost-effective data processing

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Include products that have never been sold**
2. **Calculate total revenue per product**
3. **Find the best-selling product by year**
4. **Identify products with price changes over time**

Remember: JOIN operations and sales analysis are fundamental to Amazon's product analytics, inventory management, pricing strategies, and business intelligence systems!