# LeetCode Easy #1251: Average Selling Price

## 📋 Problem Statement

Write a SQL query to find the **average selling price** for each product. `average_price` should be **rounded to 2 decimal places**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Prices Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| start_date    | date    |
| end_date      | date    |
| price         | int     |
+---------------+---------+
```
- (product_id, start_date, end_date) is the primary key for this table.
- Each row indicates the price of the product_id in the period from start_date to end_date.
- For each product_id there will be no two overlapping periods. That means there will be no two intersecting periods for the same product_id.

**UnitsSold Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| product_id    | int     |
| purchase_date | date    |
| units         | int     |
+---------------+---------+
```
- There is no primary key for this table, it may have duplicate rows.
- Each row indicates the date, units, and product_id of each product sold.

## 📊 Sample Data

**Prices Table:**
| product_id | start_date | end_date   | price |
|------------|------------|------------|-------|
| 1          | 2019-02-17 | 2019-02-28 | 5     |
| 1          | 2019-03-01 | 2019-03-22 | 20    |
| 2          | 2019-02-01 | 2019-02-20 | 15    |
| 2          | 2019-02-21 | 2019-03-31 | 30    |

**UnitsSold Table:**
| product_id | purchase_date | units |
|------------|---------------|-------|
| 1          | 2019-02-25    | 100   |
| 1          | 2019-03-01    | 15    |
| 2          | 2019-02-10    | 200   |
| 2          | 2019-03-22    | 30    |

**Expected Output:**
| product_id | average_price |
|------------|---------------|
| 1          | 6.96          |
| 2          | 16.96         |

**Explanation:**
- **Product 1**: 
  - 100 units sold at price 5 (2019-02-25)
  - 15 units sold at price 20 (2019-03-01)
  - Average = (100×5 + 15×20) / (100+15) = 800/115 = 6.96

- **Product 2**:
  - 200 units sold at price 15 (2019-02-10)
  - 30 units sold at price 30 (2019-03-22)
  - Average = (200×15 + 30×30) / (200+30) = 3900/230 = 16.96

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to join sales data with pricing data based on date ranges
- Calculate weighted average price (price × units sold)
- Handle time period matching between tables
- Round result to 2 decimal places

### 2. **Key Insights**
- JOIN condition needs date range logic (BETWEEN)
- Weighted average requires SUM(price × units) / SUM(units)
- Each sale must match the correct price period
- ROUND function for final formatting

### 3. **Interview Discussion Points**
- "This is a weighted average calculation with date range joins"
- "Need to match each sale to the correct pricing period"
- "The key is understanding the time-based relationship between tables"

## 🔧 Step-by-Step Solution Logic

### Step 1: Join Tables with Date Logic
```sql
-- JOIN ON product_id AND purchase_date BETWEEN start_date AND end_date
-- This matches each sale to its applicable price period
```

### Step 2: Calculate Revenue per Transaction
```sql
-- price * units gives revenue for each sale
-- SUM(price * units) gives total revenue per product
```

### Step 3: Calculate Total Units per Product
```sql
-- SUM(units) gives total units sold per product
-- This is the denominator for weighted average
```

### Step 4: Compute Weighted Average
```sql
-- SUM(price * units) / SUM(units) = weighted average price
-- ROUND(..., 2) formats to 2 decimal places
```

## ✅ Optimized SQL Solution

**Solution 1: Basic JOIN with Weighted Average**
```sql
SELECT 
    p.product_id,
    ROUND(SUM(p.price * u.units) * 1.0 / SUM(u.units), 2) as average_price
FROM Prices p
JOIN UnitsSold u ON p.product_id = u.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

### Alternative Solutions

**Solution 2: Using CTE for Clarity**
```sql
WITH sales_with_prices AS (
    SELECT 
        p.product_id,
        p.price,
        u.units,
        u.purchase_date,
        p.price * u.units as revenue
    FROM Prices p
    JOIN UnitsSold u ON p.product_id = u.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
)
SELECT 
    product_id,
    ROUND(SUM(revenue) * 1.0 / SUM(units), 2) as average_price
FROM sales_with_prices
GROUP BY product_id;
```

**Solution 3: Including Products with No Sales**
```sql
SELECT 
    p.product_id,
    CASE 
        WHEN SUM(u.units) IS NULL OR SUM(u.units) = 0 THEN 0.00
        ELSE ROUND(SUM(p.price * u.units) * 1.0 / SUM(u.units), 2)
    END as average_price
FROM Prices p
LEFT JOIN UnitsSold u ON p.product_id = u.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

**Solution 4: Detailed Analysis with Metrics**
```sql
SELECT 
    p.product_id,
    COUNT(u.units) as total_transactions,
    SUM(u.units) as total_units_sold,
    SUM(p.price * u.units) as total_revenue,
    ROUND(SUM(p.price * u.units) * 1.0 / SUM(u.units), 2) as average_price,
    MIN(p.price) as min_price_period,
    MAX(p.price) as max_price_period,
    MIN(u.purchase_date) as first_sale_date,
    MAX(u.purchase_date) as last_sale_date
FROM Prices p
JOIN UnitsSold u ON p.product_id = u.product_id
    AND u.purchase_date BETWEEN p.start_date AND p.end_date
GROUP BY p.product_id;
```

**Solution 5: Window Function Analysis**
```sql
WITH sales_analysis AS (
    SELECT 
        p.product_id,
        p.price,
        u.units,
        u.purchase_date,
        p.price * u.units as revenue,
        SUM(u.units) OVER (PARTITION BY p.product_id) as total_product_units,
        SUM(p.price * u.units) OVER (PARTITION BY p.product_id) as total_product_revenue,
        ROW_NUMBER() OVER (PARTITION BY p.product_id ORDER BY u.purchase_date) as sale_sequence
    FROM Prices p
    JOIN UnitsSold u ON p.product_id = u.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
)
SELECT 
    product_id,
    ROUND(total_product_revenue * 1.0 / total_product_units, 2) as average_price,
    total_product_units,
    total_product_revenue,
    COUNT(*) as total_sales_transactions
FROM sales_analysis
GROUP BY product_id, total_product_units, total_product_revenue;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Product Pricing Analytics**
```sql
-- "Analyze pricing trends, sales performance, and revenue optimization"

WITH product_pricing_periods AS (
    SELECT 
        product_id,
        start_date,
        end_date,
        price,
        DATEDIFF(end_date, start_date) + 1 as period_duration_days,
        LAG(price) OVER (PARTITION BY product_id ORDER BY start_date) as previous_price,
        LEAD(price) OVER (PARTITION BY product_id ORDER BY start_date) as next_price
    FROM Prices
),
sales_by_period AS (
    SELECT 
        p.product_id,
        p.start_date,
        p.end_date,
        p.price,
        p.period_duration_days,
        p.previous_price,
        p.next_price,
        COUNT(u.units) as transactions_in_period,
        SUM(u.units) as units_sold_in_period,
        SUM(p.price * u.units) as revenue_in_period,
        AVG(u.units) as avg_units_per_transaction,
        MIN(u.purchase_date) as first_sale_in_period,
        MAX(u.purchase_date) as last_sale_in_period
    FROM product_pricing_periods p
    LEFT JOIN UnitsSold u ON p.product_id = u.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
    GROUP BY p.product_id, p.start_date, p.end_date, p.price, 
             p.period_duration_days, p.previous_price, p.next_price
),
pricing_impact_analysis AS (
    SELECT 
        *,
        CASE 
            WHEN previous_price IS NULL THEN 'Initial Price'
            WHEN price > previous_price THEN 'Price Increase'
            WHEN price < previous_price THEN 'Price Decrease'
            ELSE 'Price Maintained'
        END as price_change_type,
        CASE 
            WHEN previous_price IS NOT NULL 
            THEN ROUND((price - previous_price) * 100.0 / previous_price, 2)
            ELSE 0
        END as price_change_percentage,
        ROUND(revenue_in_period * 1.0 / NULLIF(period_duration_days, 0), 2) as daily_revenue_rate,
        ROUND(units_sold_in_period * 1.0 / NULLIF(period_duration_days, 0), 2) as daily_sales_rate,
        CASE 
            WHEN units_sold_in_period = 0 THEN 0
            ELSE ROUND(revenue_in_period * 1.0 / units_sold_in_period, 2)
        END as realized_average_price
    FROM sales_by_period
),
product_summary AS (
    SELECT 
        product_id,
        COUNT(*) as total_pricing_periods,
        SUM(units_sold_in_period) as total_units_sold,
        SUM(revenue_in_period) as total_revenue,
        ROUND(SUM(revenue_in_period) * 1.0 / NULLIF(SUM(units_sold_in_period), 0), 2) as overall_average_price,
        MIN(price) as lowest_price_ever,
        MAX(price) as highest_price_ever,
        AVG(daily_revenue_rate) as avg_daily_revenue,
        AVG(daily_sales_rate) as avg_daily_sales,
        SUM(CASE WHEN price_change_type = 'Price Increase' THEN 1 ELSE 0 END) as price_increases,
        SUM(CASE WHEN price_change_type = 'Price Decrease' THEN 1 ELSE 0 END) as price_decreases
    FROM pricing_impact_analysis
    GROUP BY product_id
)
SELECT 
    pia.product_id,
    pia.start_date,
    pia.end_date,
    pia.price,
    pia.price_change_type,
    pia.price_change_percentage,
    pia.units_sold_in_period,
    pia.revenue_in_period,
    pia.daily_revenue_rate,
    pia.daily_sales_rate,
    pia.realized_average_price,
    ps.overall_average_price,
    ps.total_units_sold,
    ps.total_revenue,
    CASE 
        WHEN pia.price_change_type = 'Price Increase' AND pia.units_sold_in_period > 
             LAG(pia.units_sold_in_period) OVER (PARTITION BY pia.product_id ORDER BY pia.start_date)
        THEN 'Inelastic Demand'
        WHEN pia.price_change_type = 'Price Increase' AND pia.units_sold_in_period < 
             LAG(pia.units_sold_in_period) OVER (PARTITION BY pia.product_id ORDER BY pia.start_date)
        THEN 'Elastic Demand'
        WHEN pia.price_change_type = 'Price Decrease' AND pia.units_sold_in_period > 
             LAG(pia.units_sold_in_period) OVER (PARTITION BY pia.product_id ORDER BY pia.start_date)
        THEN 'Price Sensitive Increase'
        ELSE 'Neutral Response'
    END as demand_elasticity_analysis,
    CASE 
        WHEN pia.daily_revenue_rate > ps.avg_daily_revenue * 1.2 THEN 'High Performance Period'
        WHEN pia.daily_revenue_rate < ps.avg_daily_revenue * 0.8 THEN 'Low Performance Period'
        ELSE 'Average Performance Period'
    END as period_performance_category
FROM pricing_impact_analysis pia
JOIN product_summary ps ON pia.product_id = ps.product_id
ORDER BY pia.product_id, pia.start_date;
```

#### 2. **Revenue Optimization and Price Elasticity**
```sql
-- "Determine optimal pricing strategies based on historical performance"

WITH price_elasticity_analysis AS (
    SELECT 
        p1.product_id,
        p1.price as current_price,
        p1.start_date as current_period_start,
        p1.end_date as current_period_end,
        SUM(u1.units) as current_period_units,
        SUM(p1.price * u1.units) as current_period_revenue,
        
        p2.price as previous_price,
        p2.start_date as previous_period_start,
        p2.end_date as previous_period_end,
        SUM(u2.units) as previous_period_units,
        SUM(p2.price * u2.units) as previous_period_revenue,
        
        DATEDIFF(p1.end_date, p1.start_date) + 1 as current_period_days,
        DATEDIFF(p2.end_date, p2.start_date) + 1 as previous_period_days
    FROM Prices p1
    LEFT JOIN Prices p2 ON p1.product_id = p2.product_id 
        AND p2.end_date < p1.start_date
        AND p2.start_date = (
            SELECT MAX(p3.start_date) 
            FROM Prices p3 
            WHERE p3.product_id = p1.product_id 
            AND p3.end_date < p1.start_date
        )
    LEFT JOIN UnitsSold u1 ON p1.product_id = u1.product_id
        AND u1.purchase_date BETWEEN p1.start_date AND p1.end_date
    LEFT JOIN UnitsSold u2 ON p2.product_id = u2.product_id
        AND u2.purchase_date BETWEEN p2.start_date AND p2.end_date
    GROUP BY p1.product_id, p1.price, p1.start_date, p1.end_date,
             p2.price, p2.start_date, p2.end_date, current_period_days, previous_period_days
),
elasticity_calculations AS (
    SELECT 
        *,
        ROUND(current_period_units * 1.0 / NULLIF(current_period_days, 0), 2) as current_daily_units,
        ROUND(previous_period_units * 1.0 / NULLIF(previous_period_days, 0), 2) as previous_daily_units,
        ROUND(current_period_revenue * 1.0 / NULLIF(current_period_days, 0), 2) as current_daily_revenue,
        ROUND(previous_period_revenue * 1.0 / NULLIF(previous_period_days, 0), 2) as previous_daily_revenue,
        
        CASE 
            WHEN previous_price IS NOT NULL AND previous_price != 0
            THEN ROUND((current_price - previous_price) * 100.0 / previous_price, 2)
            ELSE 0
        END as price_change_percentage,
        
        CASE 
            WHEN previous_period_units IS NOT NULL AND previous_period_units != 0
            THEN ROUND((current_period_units - previous_period_units) * 100.0 / previous_period_units, 2)
            ELSE 0
        END as units_change_percentage
    FROM price_elasticity_analysis
    WHERE previous_price IS NOT NULL
),
demand_elasticity AS (
    SELECT 
        *,
        CASE 
            WHEN price_change_percentage != 0
            THEN ROUND(units_change_percentage / price_change_percentage, 2)
            ELSE 0
        END as price_elasticity_coefficient,
        
        CASE 
            WHEN price_change_percentage > 0 AND units_change_percentage <= -20 THEN 'Highly Elastic'
            WHEN price_change_percentage > 0 AND units_change_percentage <= -10 THEN 'Moderately Elastic'
            WHEN price_change_percentage > 0 AND units_change_percentage > -10 THEN 'Inelastic'
            WHEN price_change_percentage < 0 AND units_change_percentage >= 20 THEN 'Highly Price Sensitive'
            WHEN price_change_percentage < 0 AND units_change_percentage >= 10 THEN 'Price Sensitive'
            ELSE 'Price Neutral'
        END as demand_elasticity_category,
        
        ROUND((current_daily_revenue - previous_daily_revenue) * 100.0 / NULLIF(previous_daily_revenue, 0), 2) as revenue_change_percentage
    FROM elasticity_calculations
),
optimization_recommendations AS (
    SELECT 
        *,
        CASE 
            WHEN demand_elasticity_category IN ('Inelastic', 'Price Neutral') AND revenue_change_percentage > 0 
            THEN 'Consider further price increase'
            WHEN demand_elasticity_category = 'Highly Elastic' AND revenue_change_percentage < 0 
            THEN 'Consider price decrease'
            WHEN demand_elasticity_category = 'Highly Price Sensitive' AND revenue_change_percentage > 10 
            THEN 'Maintain current price strategy'
            WHEN revenue_change_percentage < -10 
            THEN 'Review pricing strategy'
            ELSE 'Monitor performance'
        END as pricing_recommendation,
        
        CASE 
            WHEN current_price < previous_price * 0.9 THEN current_price * 1.1  -- Suggest 10% increase
            WHEN current_price > previous_price * 1.1 AND demand_elasticity_category = 'Highly Elastic' 
            THEN current_price * 0.95  -- Suggest 5% decrease
            ELSE current_price  -- Keep current price
        END as suggested_optimal_price
    FROM demand_elasticity
)
SELECT 
    product_id,
    current_price,
    previous_price,
    price_change_percentage,
    current_daily_units,
    previous_daily_units,
    units_change_percentage,
    current_daily_revenue,
    previous_daily_revenue,
    revenue_change_percentage,
    price_elasticity_coefficient,
    demand_elasticity_category,
    pricing_recommendation,
    ROUND(suggested_optimal_price, 2) as suggested_optimal_price,
    ROUND((suggested_optimal_price - current_price) * current_daily_units * 30, 2) as projected_monthly_revenue_impact
FROM optimization_recommendations
ORDER BY ABS(price_elasticity_coefficient) DESC, revenue_change_percentage DESC;
```

#### 3. **Time-Series Sales and Pricing Analysis**
```sql
-- "Analyze sales patterns and seasonality effects on pricing"

WITH daily_sales_metrics AS (
    SELECT 
        u.purchase_date,
        u.product_id,
        p.price,
        SUM(u.units) as daily_units,
        SUM(p.price * u.units) as daily_revenue,
        COUNT(*) as daily_transactions,
        AVG(u.units) as avg_units_per_transaction,
        DAYOFWEEK(u.purchase_date) as day_of_week,
        MONTH(u.purchase_date) as month_num,
        QUARTER(u.purchase_date) as quarter_num,
        YEAR(u.purchase_date) as year_num
    FROM UnitsSold u
    JOIN Prices p ON u.product_id = p.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
    GROUP BY u.purchase_date, u.product_id, p.price,
             DAYOFWEEK(u.purchase_date), MONTH(u.purchase_date), 
             QUARTER(u.purchase_date), YEAR(u.purchase_date)
),
time_series_analysis AS (
    SELECT 
        *,
        LAG(daily_units, 1) OVER (PARTITION BY product_id ORDER BY purchase_date) as prev_day_units,
        LAG(daily_revenue, 1) OVER (PARTITION BY product_id ORDER BY purchase_date) as prev_day_revenue,
        LAG(price, 1) OVER (PARTITION BY product_id ORDER BY purchase_date) as prev_day_price,
        
        AVG(daily_units) OVER (PARTITION BY product_id ORDER BY purchase_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as rolling_7day_avg_units,
        AVG(daily_revenue) OVER (PARTITION BY product_id ORDER BY purchase_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as rolling_7day_avg_revenue,
        
        AVG(daily_units) OVER (PARTITION BY product_id, day_of_week) as dow_avg_units,
        AVG(daily_revenue) OVER (PARTITION BY product_id, day_of_week) as dow_avg_revenue,
        
        AVG(daily_units) OVER (PARTITION BY product_id, month_num) as monthly_avg_units,
        AVG(daily_revenue) OVER (PARTITION BY product_id, month_num) as monthly_avg_revenue
    FROM daily_sales_metrics
),
seasonal_patterns AS (
    SELECT 
        product_id,
        day_of_week,
        CASE day_of_week
            WHEN 1 THEN 'Sunday'
            WHEN 2 THEN 'Monday'
            WHEN 3 THEN 'Tuesday'
            WHEN 4 THEN 'Wednesday'
            WHEN 5 THEN 'Thursday'
            WHEN 6 THEN 'Friday'
            WHEN 7 THEN 'Saturday'
        END as day_name,
        AVG(daily_units) as avg_dow_units,
        AVG(daily_revenue) as avg_dow_revenue,
        COUNT(*) as days_with_sales,
        ROUND(STDDEV(daily_units), 2) as units_volatility
    FROM time_series_analysis
    GROUP BY product_id, day_of_week
),
monthly_seasonality AS (
    SELECT 
        product_id,
        month_num,
        CASE month_num
            WHEN 1 THEN 'January' WHEN 2 THEN 'February' WHEN 3 THEN 'March'
            WHEN 4 THEN 'April' WHEN 5 THEN 'May' WHEN 6 THEN 'June'
            WHEN 7 THEN 'July' WHEN 8 THEN 'August' WHEN 9 THEN 'September'
            WHEN 10 THEN 'October' WHEN 11 THEN 'November' WHEN 12 THEN 'December'
        END as month_name,
        AVG(daily_units) as avg_monthly_units,
        AVG(daily_revenue) as avg_monthly_revenue,
        AVG(price) as avg_monthly_price,
        COUNT(DISTINCT purchase_date) as active_days_in_month
    FROM time_series_analysis
    GROUP BY product_id, month_num
),
trend_analysis AS (
    SELECT 
        tsa.*,
        CASE 
            WHEN daily_units > rolling_7day_avg_units * 1.2 THEN 'Above Trend'
            WHEN daily_units < rolling_7day_avg_units * 0.8 THEN 'Below Trend'
            ELSE 'On Trend'
        END as daily_trend_status,
        
        CASE 
            WHEN daily_units > dow_avg_units * 1.3 THEN 'High for Day of Week'
            WHEN daily_units < dow_avg_units * 0.7 THEN 'Low for Day of Week'
            ELSE 'Normal for Day of Week'
        END as dow_comparison,
        
        CASE 
            WHEN daily_units > monthly_avg_units * 1.5 THEN 'Exceptional Daily Performance'
            WHEN daily_units < monthly_avg_units * 0.5 THEN 'Poor Daily Performance'
            ELSE 'Normal Daily Performance'
        END as monthly_comparison,
        
        ROUND((daily_units - prev_day_units) * 100.0 / NULLIF(prev_day_units, 0), 2) as day_over_day_units_change,
        ROUND((daily_revenue - prev_day_revenue) * 100.0 / NULLIF(prev_day_revenue, 0), 2) as day_over_day_revenue_change
    FROM time_series_analysis tsa
)
SELECT 
    ta.product_id,
    ta.purchase_date,
    ta.price,
    ta.daily_units,
    ta.daily_revenue,
    ta.daily_transactions,
    ROUND(ta.rolling_7day_avg_units, 2) as rolling_7day_avg_units,
    ROUND(ta.rolling_7day_avg_revenue, 2) as rolling_7day_avg_revenue,
    ta.daily_trend_status,
    ta.dow_comparison,
    ta.monthly_comparison,
    ta.day_over_day_units_change,
    ta.day_over_day_revenue_change,
    sp.day_name,
    ROUND(sp.avg_dow_units, 2) as typical_dow_units,
    ms.month_name,
    ROUND(ms.avg_monthly_units, 2) as typical_monthly_units,
    CASE 
        WHEN ta.daily_trend_status = 'Above Trend' AND ta.dow_comparison = 'High for Day of Week' 
        THEN 'Exceptional Performance Day'
        WHEN ta.daily_trend_status = 'Below Trend' AND ta.dow_comparison = 'Low for Day of Week' 
        THEN 'Concerning Performance Day'
        WHEN ABS(ta.day_over_day_units_change) > 50 
        THEN 'Volatile Sales Day'
        ELSE 'Standard Sales Day'
    END as performance_classification
FROM trend_analysis ta
JOIN seasonal_patterns sp ON ta.product_id = sp.product_id AND ta.day_of_week = sp.day_of_week
JOIN monthly_seasonality ms ON ta.product_id = ms.product_id AND ta.month_num = ms.month_num
ORDER BY ta.product_id, ta.purchase_date;
```

#### 4. **Inventory and Demand Forecasting**
```sql
-- "Predict future demand and optimize inventory based on pricing and sales patterns"

WITH product_lifecycle_metrics AS (
    SELECT 
        u.product_id,
        MIN(u.purchase_date) as first_sale_date,
        MAX(u.purchase_date) as last_sale_date,
        DATEDIFF(MAX(u.purchase_date), MIN(u.purchase_date)) + 1 as product_lifecycle_days,
        COUNT(DISTINCT u.purchase_date) as active_sales_days,
        COUNT(*) as total_transactions,
        SUM(u.units) as total_units_sold,
        AVG(u.units) as avg_units_per_transaction,
        STDDEV(u.units) as units_volatility,
        
        -- Price history
        COUNT(DISTINCT p.price) as unique_price_points,
        MIN(p.price) as lowest_price_ever,
        MAX(p.price) as highest_price_ever,
        AVG(p.price) as avg_historical_price,
        
        -- Sales consistency
        COUNT(DISTINCT u.purchase_date) * 1.0 / NULLIF(DATEDIFF(MAX(u.purchase_date), MIN(u.purchase_date)) + 1, 0) as sales_consistency_ratio
    FROM UnitsSold u
    JOIN Prices p ON u.product_id = p.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
    GROUP BY u.product_id
),
demand_forecasting_base AS (
    SELECT 
        u.product_id,
        u.purchase_date,
        p.price,
        u.units,
        p.price * u.units as revenue,
        
        -- Time-based features
        DAYOFWEEK(u.purchase_date) as day_of_week,
        DAYOFMONTH(u.purchase_date) as day_of_month,
        MONTH(u.purchase_date) as month,
        QUARTER(u.purchase_date) as quarter,
        YEAR(u.purchase_date) as year,
        
        -- Historical context
        ROW_NUMBER() OVER (PARTITION BY u.product_id ORDER BY u.purchase_date) as sale_sequence,
        DATEDIFF(u.purchase_date, MIN(u.purchase_date) OVER (PARTITION BY u.product_id)) + 1 as days_since_launch,
        
        -- Moving averages for trend analysis
        AVG(u.units) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as ma_7day_units,
        AVG(u.units) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) as ma_14day_units,
        AVG(u.units) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date ROWS BETWEEN 29 PRECEDING AND CURRENT ROW) as ma_30day_units,
        
        -- Price relationship features
        LAG(p.price, 1) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date) as prev_price,
        LAG(u.units, 1) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date) as prev_units,
        LAG(u.units, 7) OVER (PARTITION BY u.product_id ORDER BY u.purchase_date) as units_7days_ago
    FROM UnitsSold u
    JOIN Prices p ON u.product_id = p.product_id
        AND u.purchase_date BETWEEN p.start_date AND p.end_date
),
advanced_forecasting_features AS (
    SELECT 
        *,
        -- Trend indicators
        CASE 
            WHEN ma_7day_units > ma_14day_units * 1.1 THEN 'Upward Trend'
            WHEN ma_7day_units < ma_14day_units * 0.9 THEN 'Downward Trend'
            ELSE 'Stable Trend'
        END as trend_direction,
        
        -- Seasonality indicators
        AVG(units) OVER (PARTITION BY product_id, day_of_week) as seasonal_dow_avg,
        AVG(units) OVER (PARTITION BY product_id, month) as seasonal_monthly_avg,
        
        -- Price elasticity features
        CASE 
            WHEN prev_price IS NOT NULL AND prev_price != price
            THEN (units - prev_units) * 1.0 / NULLIF((price - prev_price), 0)
            ELSE 0
        END as price_elasticity_indicator,
        
        -- Growth metrics
        CASE 
            WHEN units_7days_ago IS NOT NULL AND units_7days_ago > 0
            THEN ROUND((units - units_7days_ago) * 100.0 / units_7days_ago, 2)
            ELSE 0
        END as week_over_week_growth,
        
        -- Lifecycle stage
        CASE 
            WHEN days_since_launch <= 30 THEN 'Introduction'
            WHEN days_since_launch <= 90 THEN 'Growth'
            WHEN days_since_launch <= 180 THEN 'Maturity'
            ELSE 'Decline/Maintenance'
        END as lifecycle_stage
    FROM demand_forecasting_base
),
forecasting_models AS (
    SELECT 
        product_id,
        
        -- Simple moving average forecasts
        ROUND(AVG(units), 2) as forecast_simple_avg,
        ROUND(AVG(CASE WHEN day_of_week = DAYOFWEEK(CURRENT_DATE + INTERVAL 1 DAY) THEN units END), 2) as forecast_seasonal_dow,
        ROUND(AVG(CASE WHEN month = MONTH(CURRENT_DATE + INTERVAL 1 DAY) THEN units END), 2) as forecast_seasonal_monthly,
        
        -- Trend-adjusted forecasts
        ROUND(AVG(ma_7day_units), 2) as forecast_trend_adjusted,
        
        -- Price-sensitive forecasts
        ROUND(AVG(CASE 
            WHEN price <= (SELECT AVG(price) FROM advanced_forecasting_features af2 WHERE af2.product_id = aff.product_id)
            THEN units * 1.1  -- Assume 10% boost for below-average pricing
            ELSE units * 0.95  -- Assume 5% reduction for above-average pricing
        END), 2) as forecast_price_adjusted,
        
        -- Lifecycle-adjusted forecasts
        ROUND(AVG(CASE 
            WHEN lifecycle_stage = 'Introduction' THEN units * 1.2
            WHEN lifecycle_stage = 'Growth' THEN units * 1.1
            WHEN lifecycle_stage = 'Maturity' THEN units
            ELSE units * 0.9
        END), 2) as forecast_lifecycle_adjusted,
        
        -- Confidence metrics
        STDDEV(units) as forecast_volatility,
        COUNT(*) as data_points_available,
        MAX(purchase_date) as last_sale_date,
        AVG(price) as avg_price_level
    FROM advanced_forecasting_features aff
    GROUP BY product_id
),
inventory_recommendations AS (
    SELECT 
        plm.product_id,
        plm.total_units_sold,
        plm.product_lifecycle_days,
        plm.sales_consistency_ratio,
        
        fm.forecast_simple_avg,
        fm.forecast_seasonal_dow,
        fm.forecast_trend_adjusted,
        fm.forecast_price_adjusted,
        fm.forecast_lifecycle_adjusted,
        fm.forecast_volatility,
        
        -- Composite forecast (weighted average)
        ROUND((fm.forecast_simple_avg * 0.2 + 
               COALESCE(fm.forecast_seasonal_dow, fm.forecast_simple_avg) * 0.15 +
               fm.forecast_trend_adjusted * 0.25 +
               fm.forecast_price_adjusted * 0.2 +
               fm.forecast_lifecycle_adjusted * 0.2), 2) as composite_forecast,
        
        -- Safety stock recommendations
        ROUND(fm.forecast_volatility * 2, 0) as safety_stock_recommendation,
        
        -- Reorder point calculation
        ROUND((fm.forecast_simple_avg * 7) + (fm.forecast_volatility * 2), 0) as reorder_point,
        
        -- Economic order quantity estimation
        ROUND(SQRT(2 * fm.forecast_simple_avg * 365 * 50 / 10), 0) as economic_order_quantity,  -- Assuming order cost $50, holding cost $10
        
        CASE 
            WHEN fm.forecast_volatility > fm.forecast_simple_avg * 0.5 THEN 'High Uncertainty'
            WHEN fm.forecast_volatility > fm.forecast_simple_avg * 0.3 THEN 'Moderate Uncertainty'
            ELSE 'Low Uncertainty'
        END as forecast_reliability,
        
        CASE 
            WHEN plm.sales_consistency_ratio > 0.8 THEN 'Consistent Demand'
            WHEN plm.sales_consistency_ratio > 0.5 THEN 'Irregular Demand'
            ELSE 'Sporadic Demand'
        END as demand_pattern_classification
    FROM product_lifecycle_metrics plm
    JOIN forecasting_models fm ON plm.product_id = fm.product_id
)
SELECT 
    product_id,
    composite_forecast as predicted_daily_demand,
    safety_stock_recommendation,
    reorder_point,
    economic_order_quantity,
    forecast_reliability,
    demand_pattern_classification,
    ROUND(composite_forecast * 30, 0) as predicted_monthly_demand,
    ROUND(composite_forecast * 7, 0) as predicted_weekly_demand,
    CASE 
        WHEN forecast_reliability = 'High Uncertainty' THEN 'Increase safety stock by 50%'
        WHEN demand_pattern_classification = 'Sporadic Demand' THEN 'Consider just-in-time ordering'
        WHEN composite_forecast > total_units_sold / product_lifecycle_days * 1.5 THEN 'Demand acceleration detected'
        ELSE 'Standard inventory management'
    END as inventory_strategy_recommendation
FROM inventory_recommendations
ORDER BY composite_forecast DESC, forecast_reliability;
```

## 🔗 Related LeetCode Questions

1. **#577 - Employee Bonus** (Basic JOIN operations)
2. **#1068 - Product Sales Analysis I** (Simple product-sales joins)
3. **#1075 - Project Employees I** (JOIN with aggregation)
4. **#1141 - User Activity for the Past 30 Days I** (Date range filtering)
5. **#1158 - Market Analysis I** (Multiple table joins with aggregation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Date Range JOINs**: Matching data based on date intervals
2. **Weighted Averages**: Calculating SUM(value × weight) / SUM(weight)
3. **BETWEEN Operator**: Efficient date range filtering
4. **Mathematical Precision**: Using proper data types for calculations

### 🚀 **Amazon Interview Tips**
1. **Clarify date logic**: "Should I include start_date and end_date in the range?"
2. **Handle edge cases**: "What about products with no sales or overlapping price periods?"
3. **Discuss performance**: "Date range joins need proper indexing strategy"
4. **Business context**: "This enables dynamic pricing and revenue analysis"

### 🔧 **Common Patterns**
- Date range joins with BETWEEN operator
- Weighted average calculations
- GROUP BY with multiple aggregation functions
- Handling division by zero in averages

### ⚠️ **Common Mistakes to Avoid**
1. **Wrong date logic** (using = instead of BETWEEN)
2. **Integer division** (forgetting to cast to decimal)
3. **Missing edge cases** (products with no sales)
4. **Performance issues** (not indexing date columns)

### 🔍 **Performance Considerations**
- Index on (product_id, start_date, end_date) for efficient range queries
- Index on (product_id, purchase_date) for join optimization
- Consider partitioning by date for large datasets
- Use EXPLAIN PLAN to verify join strategy

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Accurate pricing analysis improves customer value
- **Data-Driven Decisions**: Using sales data to optimize pricing strategies
- **Operational Excellence**: Efficient revenue calculation and reporting
- **Innovation**: Advanced pricing analytics for competitive advantage

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate profit margins using cost data**
2. **Find products with most volatile pricing**
3. **Analyze seasonal price variations**
4. **Calculate revenue by time periods**

Remember: Dynamic pricing analysis is essential for Amazon's marketplace operations, seller analytics, inventory management, and revenue optimization across millions of products and price changes daily!