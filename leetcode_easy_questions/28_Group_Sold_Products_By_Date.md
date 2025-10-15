# LeetCode Easy #1484: Group Sold Products By The Date

## 📋 Problem Statement

Write a SQL query to find for each date the number of **different products sold** and their **names**.

The sold products names for each date should be sorted **lexicographically**.

Return the result table **ordered by** `sell_date`.

## 🗄️ Table Schema

**Activities Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| sell_date   | date    |
| product     | varchar |
+-------------+---------+
```
- There is no primary key for this table, it may contain duplicate rows.
- Each row contains the date and the product sold on that date.

## 📊 Sample Data

**Activities Table:**
| sell_date  | product     |
|------------|-------------|
| 2020-05-30 | Headphone   |
| 2020-06-01 | Pencil      |
| 2020-06-02 | Mask        |
| 2020-05-30 | Basketball  |
| 2020-06-01 | Bible       |
| 2020-06-02 | Mask        |
| 2020-05-30 | T-Shirt     |

**Expected Output:**
| sell_date  | num_sold | products                     |
|------------|----------|------------------------------|
| 2020-05-30 | 3        | Basketball,Headphone,T-Shirt |
| 2020-06-01 | 2        | Bible,Pencil                 |
| 2020-06-02 | 1        | Mask                         |

**Explanation:**
- **2020-05-30**: 3 different products sold (Basketball, Headphone, T-Shirt) - sorted alphabetically
- **2020-06-01**: 2 different products sold (Bible, Pencil) - sorted alphabetically  
- **2020-06-02**: 1 product sold (Mask) - duplicate entries count as 1

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Group by sell_date to aggregate by day
- Count DISTINCT products per date
- Concatenate product names, sorted alphabetically
- Handle duplicate product sales on same date

### 2. **Key Insights**
- GROUP BY for date aggregation
- COUNT(DISTINCT) for unique product counting
- GROUP_CONCAT/STRING_AGG for concatenation
- ORDER BY within aggregation for sorting

### 3. **Interview Discussion Points**
- "This combines grouping, distinct counting, and string aggregation"
- "Need to sort products alphabetically within each date"
- "Duplicates should be eliminated before counting and concatenating"

## 🔧 Step-by-Step Solution Logic

### Step 1: Group by Date
```sql
-- GROUP BY sell_date
-- This aggregates all sales for each unique date
```

### Step 2: Count Distinct Products
```sql
-- COUNT(DISTINCT product)
-- Eliminates duplicate products on same date
```

### Step 3: Concatenate Sorted Products
```sql
-- GROUP_CONCAT(DISTINCT product ORDER BY product)
-- Sorts products alphabetically and concatenates
```

### Step 4: Sort Results by Date
```sql
-- ORDER BY sell_date
-- Final result ordering by date
```

## ✅ Optimized SQL Solution

**Solution 1: Using GROUP_CONCAT (MySQL)**
```sql
SELECT 
    sell_date,
    COUNT(DISTINCT product) as num_sold,
    GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') as products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

### Alternative Solutions

**Solution 2: Using STRING_AGG (PostgreSQL/SQL Server)**
```sql
SELECT 
    sell_date,
    COUNT(DISTINCT product) as num_sold,
    STRING_AGG(DISTINCT product, ',' ORDER BY product) as products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

**Solution 3: Using LISTAGG (Oracle)**
```sql
SELECT 
    sell_date,
    COUNT(DISTINCT product) as num_sold,
    LISTAGG(DISTINCT product, ',') WITHIN GROUP (ORDER BY product) as products
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

**Solution 4: With Additional Sales Metrics**
```sql
SELECT 
    sell_date,
    COUNT(*) as total_sales_transactions,
    COUNT(DISTINCT product) as num_sold,
    GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') as products,
    AVG(CASE WHEN product IS NOT NULL THEN 1 ELSE 0 END) as avg_products_per_transaction,
    
    -- Additional analysis
    CASE 
        WHEN COUNT(DISTINCT product) >= 5 THEN 'High Variety Day'
        WHEN COUNT(DISTINCT product) >= 3 THEN 'Moderate Variety Day'
        WHEN COUNT(DISTINCT product) >= 2 THEN 'Low Variety Day'
        ELSE 'Single Product Day'
    END as variety_category
FROM Activities
GROUP BY sell_date
ORDER BY sell_date;
```

**Solution 5: Platform-Agnostic Using Subquery**
```sql
WITH distinct_daily_sales AS (
    SELECT DISTINCT 
        sell_date,
        product
    FROM Activities
    WHERE product IS NOT NULL
),
sales_summary AS (
    SELECT 
        sell_date,
        COUNT(*) as num_sold,
        GROUP_CONCAT(product ORDER BY product SEPARATOR ',') as products
    FROM distinct_daily_sales
    GROUP BY sell_date
)
SELECT 
    sell_date,
    num_sold,
    products
FROM sales_summary
ORDER BY sell_date;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Sales Analytics and Trend Analysis**
```sql
-- "Analyze sales patterns, product performance, and market trends"

WITH daily_sales_metrics AS (
    SELECT 
        sell_date,
        COUNT(*) as total_transactions,
        COUNT(DISTINCT product) as unique_products_sold,
        GROUP_CONCAT(DISTINCT product ORDER BY product SEPARATOR ',') as products_list,
        
        -- Product frequency analysis
        MAX(product_counts.product_frequency) as most_sold_product_count,
        MIN(product_counts.product_frequency) as least_sold_product_count,
        AVG(product_counts.product_frequency) as avg_product_frequency
    FROM Activities a
    JOIN (
        SELECT 
            sell_date,
            product,
            COUNT(*) as product_frequency
        FROM Activities
        GROUP BY sell_date, product
    ) product_counts ON a.sell_date = product_counts.sell_date
    GROUP BY sell_date
),
time_series_analysis AS (
    SELECT 
        dsm.*,
        LAG(unique_products_sold, 1) OVER (ORDER BY sell_date) as prev_day_products,
        LAG(total_transactions, 1) OVER (ORDER BY sell_date) as prev_day_transactions,
        
        -- Moving averages
        AVG(unique_products_sold) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as ma_7day_products,
        AVG(total_transactions) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as ma_7day_transactions,
        
        -- Day of week analysis
        DAYOFWEEK(sell_date) as day_of_week,
        DAYNAME(sell_date) as day_name,
        
        -- Weekly and monthly context
        WEEK(sell_date) as week_number,
        MONTH(sell_date) as month_number,
        MONTHNAME(sell_date) as month_name
    FROM daily_sales_metrics dsm
),
product_performance_analysis AS (
    SELECT 
        product,
        COUNT(DISTINCT sell_date) as days_sold,
        COUNT(*) as total_sales_frequency,
        MIN(sell_date) as first_sale_date,
        MAX(sell_date) as last_sale_date,
        DATEDIFF(MAX(sell_date), MIN(sell_date)) + 1 as sales_span_days,
        
        -- Product popularity metrics
        COUNT(DISTINCT sell_date) * 1.0 / (
            SELECT COUNT(DISTINCT sell_date) FROM Activities
        ) as market_presence_ratio,
        
        AVG(daily_freq.daily_frequency) as avg_daily_frequency,
        MAX(daily_freq.daily_frequency) as max_daily_frequency,
        
        CASE 
            WHEN COUNT(DISTINCT sell_date) >= (SELECT COUNT(DISTINCT sell_date) FROM Activities) * 0.8 
            THEN 'Consistent Seller'
            WHEN COUNT(DISTINCT sell_date) >= (SELECT COUNT(DISTINCT sell_date) FROM Activities) * 0.5 
            THEN 'Regular Seller'
            WHEN COUNT(DISTINCT sell_date) >= (SELECT COUNT(DISTINCT sell_date) FROM Activities) * 0.2 
            THEN 'Occasional Seller'
            ELSE 'Rare Seller'
        END as product_category
    FROM Activities a
    LEFT JOIN (
        SELECT 
            sell_date,
            product,
            COUNT(*) as daily_frequency
        FROM Activities
        GROUP BY sell_date, product
    ) daily_freq ON a.sell_date = daily_freq.sell_date AND a.product = daily_freq.product
    GROUP BY product
),
seasonal_trend_analysis AS (
    SELECT 
        tsa.*,
        
        -- Growth calculations
        CASE 
            WHEN prev_day_products IS NOT NULL AND prev_day_products > 0
            THEN ROUND((unique_products_sold - prev_day_products) * 100.0 / prev_day_products, 2)
            ELSE 0
        END as product_variety_growth_rate,
        
        CASE 
            WHEN prev_day_transactions IS NOT NULL AND prev_day_transactions > 0
            THEN ROUND((total_transactions - prev_day_transactions) * 100.0 / prev_day_transactions, 2)
            ELSE 0
        END as transaction_growth_rate,
        
        -- Trend classification
        CASE 
            WHEN unique_products_sold > ma_7day_products * 1.2 THEN 'Above Trend'
            WHEN unique_products_sold < ma_7day_products * 0.8 THEN 'Below Trend'
            ELSE 'On Trend'
        END as variety_trend_status,
        
        -- Day of week performance
        AVG(unique_products_sold) OVER (PARTITION BY day_of_week) as dow_avg_products,
        AVG(total_transactions) OVER (PARTITION BY day_of_week) as dow_avg_transactions,
        
        -- Monthly performance
        AVG(unique_products_sold) OVER (PARTITION BY month_number) as monthly_avg_products,
        AVG(total_transactions) OVER (PARTITION BY month_number) as monthly_avg_transactions
    FROM time_series_analysis tsa
),
comprehensive_insights AS (
    SELECT 
        sta.sell_date,
        sta.unique_products_sold as num_sold,
        sta.products_list as products,
        sta.total_transactions,
        
        -- Performance indicators
        ROUND(sta.ma_7day_products, 2) as week_avg_product_variety,
        ROUND(sta.ma_7day_transactions, 2) as week_avg_transactions,
        sta.product_variety_growth_rate,
        sta.transaction_growth_rate,
        sta.variety_trend_status,
        
        -- Day context
        sta.day_name,
        ROUND(sta.dow_avg_products, 2) as typical_dow_variety,
        ROUND(sta.dow_avg_transactions, 2) as typical_dow_transactions,
        
        -- Performance classification
        CASE 
            WHEN sta.unique_products_sold > sta.dow_avg_products * 1.3 THEN 'Exceptional Day for ' || sta.day_name
            WHEN sta.unique_products_sold < sta.dow_avg_products * 0.7 THEN 'Below Average for ' || sta.day_name
            ELSE 'Typical ' || sta.day_name
        END as daily_performance_assessment,
        
        -- Strategic insights
        CASE 
            WHEN sta.variety_trend_status = 'Above Trend' AND sta.product_variety_growth_rate > 20 
            THEN 'Strong product diversification'
            WHEN sta.variety_trend_status = 'Below Trend' AND sta.product_variety_growth_rate < -20 
            THEN 'Product variety decline'
            WHEN sta.total_transactions > sta.ma_7day_transactions * 1.5 
            THEN 'High transaction volume day'
            WHEN sta.unique_products_sold = 1 
            THEN 'Single product focus day'
            ELSE 'Standard sales pattern'
        END as strategic_observation
    FROM seasonal_trend_analysis sta
)
SELECT 
    sell_date,
    num_sold,
    products,
    total_transactions,
    week_avg_product_variety,
    product_variety_growth_rate,
    variety_trend_status,
    day_name,
    daily_performance_assessment,
    strategic_observation
FROM comprehensive_insights
ORDER BY sell_date;
```

#### 2. **Product Portfolio and Market Analysis**
```sql
-- "Analyze product portfolio performance and market positioning"

WITH product_market_analysis AS (
    SELECT 
        product,
        COUNT(DISTINCT sell_date) as market_days,
        COUNT(*) as total_transactions,
        MIN(sell_date) as product_launch_date,
        MAX(sell_date) as last_sale_date,
        
        -- Market share analysis
        COUNT(*) * 100.0 / (SELECT COUNT(*) FROM Activities) as market_share_percentage,
        COUNT(DISTINCT sell_date) * 100.0 / (SELECT COUNT(DISTINCT sell_date) FROM Activities) as market_presence_percentage,
        
        -- Sales consistency
        COUNT(*) * 1.0 / COUNT(DISTINCT sell_date) as avg_daily_sales_frequency,
        STDDEV(daily_sales.daily_count) as sales_volatility,
        
        -- Product lifecycle stage
        DATEDIFF(MAX(sell_date), MIN(sell_date)) + 1 as product_lifespan_days,
        CASE 
            WHEN COUNT(DISTINCT sell_date) = 1 THEN 'One-Day Product'
            WHEN DATEDIFF(MAX(sell_date), MIN(sell_date)) + 1 <= 7 THEN 'Short-Term Product'
            WHEN DATEDIFF(MAX(sell_date), MIN(sell_date)) + 1 <= 30 THEN 'Medium-Term Product'
            ELSE 'Long-Term Product'
        END as product_lifecycle_category
    FROM Activities a
    LEFT JOIN (
        SELECT 
            product,
            sell_date,
            COUNT(*) as daily_count
        FROM Activities
        GROUP BY product, sell_date
    ) daily_sales ON a.product = daily_sales.product AND a.sell_date = daily_sales.sell_date
    GROUP BY product
),
competitive_landscape AS (
    SELECT 
        pma.*,
        
        -- Competitive positioning
        ROW_NUMBER() OVER (ORDER BY market_share_percentage DESC) as market_share_rank,
        ROW_NUMBER() OVER (ORDER BY market_presence_percentage DESC) as market_presence_rank,
        ROW_NUMBER() OVER (ORDER BY total_transactions DESC) as transaction_volume_rank,
        
        -- Performance categories
        CASE 
            WHEN market_share_percentage >= 15 THEN 'Market Leader'
            WHEN market_share_percentage >= 10 THEN 'Major Player'
            WHEN market_share_percentage >= 5 THEN 'Significant Player'
            WHEN market_share_percentage >= 2 THEN 'Niche Player'
            ELSE 'Emerging Product'
        END as market_position,
        
        CASE 
            WHEN market_presence_percentage >= 80 THEN 'Ubiquitous Product'
            WHEN market_presence_percentage >= 60 THEN 'High Presence Product'
            WHEN market_presence_percentage >= 40 THEN 'Moderate Presence Product'
            WHEN market_presence_percentage >= 20 THEN 'Limited Presence Product'
            ELSE 'Rare Product'
        END as market_presence_category,
        
        -- Growth potential assessment
        CASE 
            WHEN product_lifecycle_category = 'Long-Term Product' AND market_share_percentage >= 10 
            THEN 'Established Winner'
            WHEN product_lifecycle_category IN ('Short-Term Product', 'Medium-Term Product') AND market_share_percentage >= 5 
            THEN 'Fast Growing'
            WHEN market_presence_percentage >= 50 BUT market_share_percentage < 5 
            THEN 'Wide Distribution, Low Volume'
            WHEN market_share_percentage >= 10 BUT market_presence_percentage < 50 
            THEN 'High Volume, Limited Distribution'
            ELSE 'Development Stage'
        END as growth_potential_assessment
    FROM product_market_analysis pma
),
portfolio_optimization AS (
    SELECT 
        cl.*,
        
        -- Portfolio recommendations
        CASE 
            WHEN cl.market_position = 'Market Leader' AND cl.sales_volatility <= 1 
            THEN 'Maintain and protect market position'
            WHEN cl.market_position = 'Major Player' AND cl.growth_potential_assessment = 'Fast Growing' 
            THEN 'Invest in growth acceleration'
            WHEN cl.market_position IN ('Niche Player', 'Emerging Product') AND cl.market_presence_percentage >= 40 
            THEN 'Expand market reach'
            WHEN cl.market_position = 'Emerging Product' AND cl.product_lifecycle_category = 'One-Day Product' 
            THEN 'Evaluate product viability'
            WHEN cl.sales_volatility > 2 
            THEN 'Stabilize sales pattern'
            ELSE 'Continue current strategy'
        END as strategic_recommendation,
        
        -- Investment priority
        CASE 
            WHEN cl.market_share_rank <= 3 AND cl.growth_potential_assessment = 'Fast Growing' THEN 'High Priority'
            WHEN cl.market_share_rank <= 5 OR cl.growth_potential_assessment = 'Fast Growing' THEN 'Medium Priority'
            WHEN cl.market_position != 'Emerging Product' THEN 'Low Priority'
            ELSE 'Review Required'
        END as investment_priority,
        
        -- Risk assessment
        CASE 
            WHEN cl.product_lifecycle_category = 'One-Day Product' THEN 'High Risk - Limited Data'
            WHEN cl.sales_volatility > 3 THEN 'High Risk - Unstable Sales'
            WHEN cl.market_presence_percentage < 20 AND cl.market_share_percentage < 2 THEN 'Medium Risk - Limited Market'
            WHEN cl.market_position IN ('Market Leader', 'Major Player') THEN 'Low Risk - Established'
            ELSE 'Medium Risk - Standard'
        END as risk_assessment
    FROM competitive_landscape cl
),
category_analysis AS (
    -- Simulate product categories for more realistic analysis
    SELECT 
        po.*,
        CASE 
            WHEN po.product LIKE '%book%' OR po.product LIKE '%bible%' THEN 'Books & Education'
            WHEN po.product LIKE '%phone%' OR po.product LIKE '%headphone%' THEN 'Electronics'
            WHEN po.product LIKE '%shirt%' OR po.product LIKE '%mask%' THEN 'Apparel & Accessories'
            WHEN po.product LIKE '%ball%' OR po.product LIKE '%basketball%' THEN 'Sports & Recreation'
            WHEN po.product LIKE '%pencil%' THEN 'Office & School Supplies'
            ELSE 'General Merchandise'
        END as product_category,
        
        -- Category performance metrics
        AVG(po.market_share_percentage) OVER (
            PARTITION BY CASE 
                WHEN po.product LIKE '%book%' OR po.product LIKE '%bible%' THEN 'Books & Education'
                WHEN po.product LIKE '%phone%' OR po.product LIKE '%headphone%' THEN 'Electronics'
                WHEN po.product LIKE '%shirt%' OR po.product LIKE '%mask%' THEN 'Apparel & Accessories'
                WHEN po.product LIKE '%ball%' OR po.product LIKE '%basketball%' THEN 'Sports & Recreation'
                WHEN po.product LIKE '%pencil%' THEN 'Office & School Supplies'
                ELSE 'General Merchandise'
            END
        ) as category_avg_market_share,
        
        COUNT(*) OVER (
            PARTITION BY CASE 
                WHEN po.product LIKE '%book%' OR po.product LIKE '%bible%' THEN 'Books & Education'
                WHEN po.product LIKE '%phone%' OR po.product LIKE '%headphone%' THEN 'Electronics'
                WHEN po.product LIKE '%shirt%' OR po.product LIKE '%mask%' THEN 'Apparel & Accessories'
                WHEN po.product LIKE '%ball%' OR po.product LIKE '%basketball%' THEN 'Sports & Recreation'
                WHEN po.product LIKE '%pencil%' THEN 'Office & School Supplies'
                ELSE 'General Merchandise'
            END
        ) as category_product_count
    FROM portfolio_optimization po
)
SELECT 
    product,
    product_category,
    ROUND(market_share_percentage, 2) as market_share_pct,
    ROUND(market_presence_percentage, 2) as market_presence_pct,
    market_position,
    market_presence_category,
    growth_potential_assessment,
    strategic_recommendation,
    investment_priority,
    risk_assessment,
    market_share_rank,
    total_transactions,
    market_days,
    ROUND(category_avg_market_share, 2) as category_avg_share
FROM category_analysis
ORDER BY 
    CASE investment_priority 
        WHEN 'High Priority' THEN 1 
        WHEN 'Medium Priority' THEN 2 
        WHEN 'Low Priority' THEN 3 
        ELSE 4 
    END,
    market_share_percentage DESC;
```

#### 3. **Advanced Sales Forecasting and Inventory Planning**
```sql
-- "Predict future sales patterns and optimize inventory management"

WITH sales_time_series AS (
    SELECT 
        sell_date,
        COUNT(DISTINCT product) as daily_unique_products,
        COUNT(*) as daily_total_sales,
        GROUP_CONCAT(DISTINCT product ORDER BY product) as daily_products,
        
        -- Time-based features for forecasting
        DAYOFWEEK(sell_date) as day_of_week,
        DAY(sell_date) as day_of_month,
        WEEK(sell_date) as week_of_year,
        MONTH(sell_date) as month,
        QUARTER(sell_date) as quarter,
        
        -- Holiday and special day indicators (simplified)
        CASE 
            WHEN DAYOFWEEK(sell_date) IN (1, 7) THEN 1 ELSE 0  -- Weekend
        END as is_weekend,
        
        CASE 
            WHEN DAY(sell_date) IN (1, 15, 30, 31) THEN 1 ELSE 0  -- Month start/mid/end
        END as is_special_date
    FROM Activities
    GROUP BY sell_date
),
trend_analysis AS (
    SELECT 
        sts.*,
        
        -- Moving averages for trend identification
        AVG(daily_unique_products) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as ma_7day_products,
        AVG(daily_total_sales) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as ma_7day_sales,
        
        AVG(daily_unique_products) OVER (ORDER BY sell_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) as ma_14day_products,
        AVG(daily_total_sales) OVER (ORDER BY sell_date ROWS BETWEEN 13 PRECEDING AND CURRENT ROW) as ma_14day_sales,
        
        -- Lag features for momentum analysis
        LAG(daily_unique_products, 1) OVER (ORDER BY sell_date) as lag_1day_products,
        LAG(daily_unique_products, 7) OVER (ORDER BY sell_date) as lag_7day_products,
        LAG(daily_total_sales, 1) OVER (ORDER BY sell_date) as lag_1day_sales,
        LAG(daily_total_sales, 7) OVER (ORDER BY sell_date) as lag_7day_sales,
        
        -- Seasonal patterns
        AVG(daily_unique_products) OVER (PARTITION BY day_of_week) as dow_avg_products,
        AVG(daily_total_sales) OVER (PARTITION BY day_of_week) as dow_avg_sales,
        
        AVG(daily_unique_products) OVER (PARTITION BY week_of_year) as weekly_avg_products,
        AVG(daily_total_sales) OVER (PARTITION BY week_of_year) as weekly_avg_sales
    FROM sales_time_series sts
),
forecasting_features AS (
    SELECT 
        ta.*,
        
        -- Momentum indicators
        CASE 
            WHEN lag_1day_products IS NOT NULL AND lag_1day_products > 0
            THEN (daily_unique_products - lag_1day_products) * 100.0 / lag_1day_products
            ELSE 0
        END as day_over_day_product_change,
        
        CASE 
            WHEN lag_7day_products IS NOT NULL AND lag_7day_products > 0
            THEN (daily_unique_products - lag_7day_products) * 100.0 / lag_7day_products
            ELSE 0
        END as week_over_week_product_change,
        
        -- Trend classification
        CASE 
            WHEN ma_7day_products > ma_14day_products * 1.1 THEN 'Accelerating Growth'
            WHEN ma_7day_products > ma_14day_products * 1.05 THEN 'Moderate Growth'
            WHEN ma_7day_products < ma_14day_products * 0.9 THEN 'Declining Trend'
            WHEN ma_7day_products < ma_14day_products * 0.95 THEN 'Moderate Decline'
            ELSE 'Stable Trend'
        END as trend_direction,
        
        -- Seasonality strength
        ABS(daily_unique_products - dow_avg_products) as dow_deviation,
        ABS(daily_total_sales - dow_avg_sales) as dow_sales_deviation,
        
        -- Volatility measures
        STDDEV(daily_unique_products) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as product_volatility_7day,
        STDDEV(daily_total_sales) OVER (ORDER BY sell_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as sales_volatility_7day
    FROM trend_analysis ta
),
predictive_models AS (
    SELECT 
        ff.*,
        
        -- Simple forecasting models
        
        -- 1. Trend-adjusted forecast
        ROUND(ma_7day_products + 
              CASE trend_direction 
                  WHEN 'Accelerating Growth' THEN ma_7day_products * 0.15
                  WHEN 'Moderate Growth' THEN ma_7day_products * 0.08
                  WHEN 'Moderate Decline' THEN ma_7day_products * -0.05
                  WHEN 'Declining Trend' THEN ma_7day_products * -0.12
                  ELSE 0
              END, 0) as trend_forecast_products,
        
        -- 2. Seasonal forecast (next day of week)
        ROUND(dow_avg_products, 0) as seasonal_forecast_products,
        
        -- 3. Momentum-based forecast
        ROUND(daily_unique_products + 
              (daily_unique_products * day_over_day_product_change / 100 * 0.3), 0) as momentum_forecast_products,
        
        -- 4. Hybrid forecast (weighted combination)
        ROUND((ma_7day_products * 0.4 + 
               dow_avg_products * 0.3 + 
               (daily_unique_products + daily_unique_products * day_over_day_product_change / 100 * 0.3) * 0.3), 0) as hybrid_forecast_products,
        
        -- Confidence scoring
        CASE 
            WHEN product_volatility_7day <= 1 THEN 'High Confidence'
            WHEN product_volatility_7day <= 2 THEN 'Medium Confidence'
            ELSE 'Low Confidence'
        END as forecast_confidence,
        
        -- Inventory recommendations
        CASE 
            WHEN trend_direction = 'Accelerating Growth' THEN 'Increase inventory 20%'
            WHEN trend_direction = 'Moderate Growth' THEN 'Increase inventory 10%'
            WHEN trend_direction = 'Declining Trend' THEN 'Reduce inventory 15%'
            WHEN trend_direction = 'Moderate Decline' THEN 'Reduce inventory 8%'
            WHEN product_volatility_7day > 2 THEN 'Increase safety stock'
            ELSE 'Maintain current levels'
        END as inventory_recommendation
    FROM forecasting_features ff
),
demand_planning AS (
    SELECT 
        pm.*,
        
        -- Future period forecasts
        hybrid_forecast_products as next_day_forecast,
        ROUND(hybrid_forecast_products * (1 + week_over_week_product_change / 100 * 0.1), 0) as next_week_forecast,
        ROUND(dow_avg_products * 7, 0) as weekly_demand_forecast,
        
        -- Stock recommendations
        ROUND(hybrid_forecast_products * 1.5, 0) as recommended_daily_stock,  -- 1.5x safety factor
        ROUND(dow_avg_products * 7 * 1.2, 0) as recommended_weekly_stock,    -- 1.2x safety factor
        
        -- Alerts and recommendations
        CASE 
            WHEN product_volatility_7day > 3 THEN 'HIGH ALERT: Highly volatile demand pattern'
            WHEN trend_direction = 'Declining Trend' AND ma_7day_products < 2 THEN 'ALERT: Critically low product variety'
            WHEN trend_direction = 'Accelerating Growth' AND ma_7day_products > 5 THEN 'OPPORTUNITY: High variety growth'
            WHEN forecast_confidence = 'Low Confidence' THEN 'CAUTION: Unpredictable demand pattern'
            ELSE 'NORMAL: Standard monitoring recommended'
        END as demand_planning_alert,
        
        -- Business strategy implications
        CASE 
            WHEN daily_unique_products > dow_avg_products * 1.5 AND is_weekend = 1 
            THEN 'Weekend surge - optimize weekend operations'
            WHEN daily_unique_products < dow_avg_products * 0.5 
            THEN 'Underperformance - investigate causes'
            WHEN trend_direction = 'Accelerating Growth' 
            THEN 'Growth opportunity - scale operations'
            WHEN product_volatility_7day > 2.5 
            THEN 'Demand instability - review product mix strategy'
            ELSE 'Standard operations'
        END as strategic_implication
    FROM predictive_models pm
)
SELECT 
    sell_date,
    daily_unique_products as num_sold,
    daily_products as products,
    daily_total_sales,
    ROUND(ma_7day_products, 2) as week_avg_variety,
    trend_direction,
    ROUND(day_over_day_product_change, 2) as daily_change_pct,
    next_day_forecast,
    weekly_demand_forecast,
    forecast_confidence,
    inventory_recommendation,
    demand_planning_alert,
    strategic_implication
FROM demand_planning
ORDER BY sell_date;
```

#### 4. **Cross-Selling and Market Basket Analysis**
```sql
-- "Analyze product relationships and cross-selling opportunities"

WITH daily_product_combinations AS (
    -- Get all products sold on the same day for market basket analysis
    SELECT 
        a1.sell_date,
        a1.product as product_a,
        a2.product as product_b
    FROM Activities a1
    JOIN Activities a2 ON a1.sell_date = a2.sell_date 
        AND a1.product < a2.product  -- Avoid duplicates and self-pairs
    GROUP BY a1.sell_date, a1.product, a2.product
),
product_affinity_analysis AS (
    SELECT 
        product_a,
        product_b,
        COUNT(DISTINCT sell_date) as co_occurrence_days,
        COUNT(*) as co_occurrence_frequency,
        
        -- Calculate affinity metrics
        (SELECT COUNT(DISTINCT sell_date) FROM Activities WHERE product = dpc.product_a) as product_a_total_days,
        (SELECT COUNT(DISTINCT sell_date) FROM Activities WHERE product = dpc.product_b) as product_b_total_days,
        (SELECT COUNT(DISTINCT sell_date) FROM Activities) as total_sales_days,
        
        -- Support: P(A and B)
        COUNT(DISTINCT sell_date) * 1.0 / (SELECT COUNT(DISTINCT sell_date) FROM Activities) as support,
        
        -- Confidence: P(B|A) = P(A and B) / P(A)
        COUNT(DISTINCT sell_date) * 1.0 / (SELECT COUNT(DISTINCT sell_date) FROM Activities WHERE product = dpc.product_a) as confidence_a_to_b,
        
        -- Confidence: P(A|B) = P(A and B) / P(B)
        COUNT(DISTINCT sell_date) * 1.0 / (SELECT COUNT(DISTINCT sell_date) FROM Activities WHERE product = dpc.product_b) as confidence_b_to_a
    FROM daily_product_combinations dpc
    GROUP BY product_a, product_b
),
lift_and_correlation AS (
    SELECT 
        paa.*,
        
        -- Lift: P(A and B) / (P(A) * P(B))
        support / (
            (product_a_total_days * 1.0 / total_sales_days) * 
            (product_b_total_days * 1.0 / total_sales_days)
        ) as lift,
        
        -- Conviction: (1 - P(B)) / (1 - Confidence(A->B))
        CASE 
            WHEN confidence_a_to_b = 1 THEN 9999  -- Perfect rule
            ELSE (1 - (product_b_total_days * 1.0 / total_sales_days)) / (1 - confidence_a_to_b)
        END as conviction_a_to_b,
        
        -- Classification of relationships
        CASE 
            WHEN support >= 0.1 AND confidence_a_to_b >= 0.5 THEN 'Strong Association'
            WHEN support >= 0.05 AND confidence_a_to_b >= 0.3 THEN 'Moderate Association'
            WHEN support >= 0.02 AND confidence_a_to_b >= 0.2 THEN 'Weak Association'
            ELSE 'No Significant Association'
        END as association_strength
    FROM product_affinity_analysis paa
),
cross_selling_opportunities AS (
    SELECT 
        lac.*,
        
        -- Cross-selling recommendation scoring
        CASE 
            WHEN lift >= 2.0 AND confidence_a_to_b >= 0.5 THEN 'High Priority Cross-Sell'
            WHEN lift >= 1.5 AND confidence_a_to_b >= 0.3 THEN 'Medium Priority Cross-Sell'
            WHEN lift >= 1.2 AND confidence_a_to_b >= 0.2 THEN 'Low Priority Cross-Sell'
            WHEN lift < 1.0 THEN 'Negative Association - Avoid Bundling'
            ELSE 'No Cross-Sell Opportunity'
        END as cross_sell_recommendation,
        
        -- Bundle pricing strategy
        CASE 
            WHEN lift >= 2.0 AND confidence_a_to_b >= 0.6 THEN 'Premium Bundle - 15% discount'
            WHEN lift >= 1.5 AND confidence_a_to_b >= 0.4 THEN 'Value Bundle - 10% discount'
            WHEN lift >= 1.2 AND confidence_a_to_b >= 0.3 THEN 'Standard Bundle - 5% discount'
            ELSE 'No bundle recommended'
        END as bundle_strategy,
        
        -- Marketing campaign recommendations
        CASE 
            WHEN association_strength = 'Strong Association' AND co_occurrence_days >= 3 
            THEN 'Featured product combo in marketing'
            WHEN association_strength = 'Moderate Association' 
            THEN 'Suggest in product recommendations'
            WHEN association_strength = 'Weak Association' 
            THEN 'A/B test cross-promotion'
            ELSE 'Monitor for emerging patterns'
        END as marketing_recommendation
    FROM lift_and_correlation lac
),
market_basket_insights AS (
    SELECT 
        cso.*,
        
        -- Product category relationships (simulated)
        CASE 
            WHEN product_a LIKE '%book%' OR product_a LIKE '%bible%' THEN 'Books & Education'
            WHEN product_a LIKE '%phone%' OR product_a LIKE '%headphone%' THEN 'Electronics'
            WHEN product_a LIKE '%shirt%' OR product_a LIKE '%mask%' THEN 'Apparel & Accessories'
            WHEN product_a LIKE '%ball%' OR product_a LIKE '%basketball%' THEN 'Sports & Recreation'
            WHEN product_a LIKE '%pencil%' THEN 'Office & School Supplies'
            ELSE 'General Merchandise'
        END as category_a,
        
        CASE 
            WHEN product_b LIKE '%book%' OR product_b LIKE '%bible%' THEN 'Books & Education'
            WHEN product_b LIKE '%phone%' OR product_b LIKE '%headphone%' THEN 'Electronics'
            WHEN product_b LIKE '%shirt%' OR product_b LIKE '%mask%' THEN 'Apparel & Accessories'
            WHEN product_b LIKE '%ball%' OR product_b LIKE '%basketball%' THEN 'Sports & Recreation'
            WHEN product_b LIKE '%pencil%' THEN 'Office & School Supplies'
            ELSE 'General Merchandise'
        END as category_b,
        
        -- Revenue impact estimation (simplified)
        CASE 
            WHEN cross_sell_recommendation = 'High Priority Cross-Sell' THEN co_occurrence_days * 50  -- $50 per opportunity
            WHEN cross_sell_recommendation = 'Medium Priority Cross-Sell' THEN co_occurrence_days * 30
            WHEN cross_sell_recommendation = 'Low Priority Cross-Sell' THEN co_occurrence_days * 15
            ELSE 0
        END as estimated_revenue_impact
    FROM cross_selling_opportunities cso
),
final_recommendations AS (
    SELECT 
        mbi.*,
        
        -- Cross-category vs same-category analysis
        CASE 
            WHEN category_a = category_b THEN 'Same Category Cross-Sell'
            ELSE 'Cross-Category Cross-Sell'
        END as cross_sell_type,
        
        -- Implementation priority
        CASE 
            WHEN estimated_revenue_impact >= 200 THEN 'Immediate Implementation'
            WHEN estimated_revenue_impact >= 100 THEN 'Next Quarter Implementation'
            WHEN estimated_revenue_impact >= 50 THEN 'Future Consideration'
            ELSE 'Monitor Only'
        END as implementation_priority,
        
        -- Actionable insights
        CONCAT('When customer buys ', product_a, ', recommend ', product_b, 
               ' (', ROUND(confidence_a_to_b * 100, 1), '% likelihood, ',
               ROUND(lift, 2), 'x lift)') as actionable_insight
    FROM market_basket_insights mbi
    WHERE association_strength != 'No Significant Association'
)
SELECT 
    product_a,
    product_b,
    category_a,
    category_b,
    cross_sell_type,
    co_occurrence_days,
    ROUND(support * 100, 2) as support_pct,
    ROUND(confidence_a_to_b * 100, 2) as confidence_pct,
    ROUND(lift, 2) as lift_score,
    association_strength,
    cross_sell_recommendation,
    bundle_strategy,
    marketing_recommendation,
    estimated_revenue_impact,
    implementation_priority,
    actionable_insight
FROM final_recommendations
ORDER BY 
    CASE implementation_priority 
        WHEN 'Immediate Implementation' THEN 1 
        WHEN 'Next Quarter Implementation' THEN 2 
        WHEN 'Future Consideration' THEN 3 
        ELSE 4 
    END,
    lift_score DESC,
    confidence_pct DESC;
```

## 🔗 Related LeetCode Questions

1. **#511 - Game Play Analysis I** (Basic GROUP BY with aggregation)
2. **#1693 - Daily Leads and Partners** (COUNT DISTINCT with GROUP BY)
3. **#1581 - Customer Who Visited but Did Not Make Any Transactions** (Date-based aggregation)
4. **#1141 - User Activity for the Past 30 Days I** (GROUP BY with DISTINCT counting)
5. **#1407 - Top Travellers** (GROUP BY with SUM and ORDER BY)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY with Aggregation**: Combining multiple aggregate functions
2. **COUNT DISTINCT**: Eliminating duplicates in counting
3. **String Aggregation**: GROUP_CONCAT, STRING_AGG, LISTAGG
4. **ORDER BY in Aggregation**: Sorting within grouped results

### 🚀 **Amazon Interview Tips**
1. **Know your database**: "GROUP_CONCAT is MySQL, STRING_AGG is PostgreSQL"
2. **Handle duplicates**: "DISTINCT eliminates duplicate products per date"
3. **String formatting**: "Separator and ordering are important for readability"
4. **Performance awareness**: "String aggregation can be memory-intensive"

### 🔧 **Common Patterns**
- GROUP BY with multiple aggregation functions
- String concatenation with sorting
- DISTINCT in both COUNT and string aggregation
- Date-based grouping and ordering

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting DISTINCT** (counting duplicate products multiple times)
2. **Wrong aggregation function** (using different syntax for different databases)
3. **Missing ORDER BY in aggregation** (unsorted product lists)
4. **Performance issues** (not indexing GROUP BY columns)

### 🔍 **Performance Considerations**
- Index on sell_date for efficient grouping
- Consider partial indexes for frequently queried date ranges
- Monitor memory usage with large string aggregations
- Use appropriate string aggregation function for your database

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Daily sales insights improve inventory and customer experience
- **Data-Driven Decisions**: Product variety analytics guide business strategy
- **Operational Excellence**: Efficient daily sales reporting and analysis
- **Innovation**: Advanced sales pattern recognition for competitive advantage

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Group by week instead of day**
2. **Include sales quantities and revenue**
3. **Find the most popular product combinations**
4. **Calculate running totals of unique products**

Remember: Daily sales aggregation is essential for Amazon's marketplace analytics, inventory management, seller insights, and customer behavior analysis across millions of products and transactions daily!