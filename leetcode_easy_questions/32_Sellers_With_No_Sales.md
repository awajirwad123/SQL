# LeetCode Easy #1607: Sellers With No Sales

## 📋 Problem Statement

Write a SQL query to report the names of all sellers who did not make any sales in **2020**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Customer Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| customer_id   | int     |
| customer_name | varchar |
+---------------+---------+
```
- customer_id is the primary key for this table.
- customer_name is the name of the customer.

**Orders Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| order_id      | int     |
| sale_date     | date    |
| order_cost    | int     |
| customer_id   | int     |
| seller_id     | int     |
+---------------+---------+
```
- order_id is the primary key for this table.
- Each row of this table contains all orders made by customer_id to the seller seller_id.

**Seller Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| seller_id     | int     |
| seller_name   | varchar |
+---------------+---------+
```
- seller_id is the primary key for this table.
- seller_name is the name of the seller.

## 📊 Sample Data

**Customer Table:**
| customer_id | customer_name |
|-------------|---------------|
| 101         | Alice         |
| 102         | Bob           |
| 103         | Charlie       |

**Seller Table:**
| seller_id | seller_name |
|-----------|-------------|
| 1         | Daniel      |
| 2         | Elizabeth   |
| 3         | Frank       |
| 4         | George      |
| 5         | Hannah      |

**Orders Table:**
| order_id | sale_date  | order_cost | customer_id | seller_id |
|----------|------------|------------|-------------|-----------|
| 1        | 2020-03-01 | 1500       | 101         | 1         |
| 2        | 2020-05-25 | 2400       | 102         | 2         |
| 3        | 2019-12-25 | 2000       | 101         | 2         |
| 4        | 2020-06-02 | 800        | 103         | 3         |
| 5        | 2021-02-10 | 1800       | 101         | 2         |

**Expected Output:**
| seller_name |
|-------------|
| George      |
| Hannah      |

**Explanation:**
- **Daniel (seller_id=1)**: ✅ Made sale on 2020-03-01 → Has sales in 2020
- **Elizabeth (seller_id=2)**: ✅ Made sale on 2020-05-25 → Has sales in 2020 (also 2019, 2021)
- **Frank (seller_id=3)**: ✅ Made sale on 2020-06-02 → Has sales in 2020
- **George (seller_id=4)**: ❌ No sales records → No sales in 2020
- **Hannah (seller_id=5)**: ❌ No sales records → No sales in 2020

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find sellers who made NO sales in 2020
- Need to check for absence of records, not presence
- Must filter orders by year 2020 first
- Use LEFT JOIN to identify sellers without 2020 sales

### 2. **Key Insights**
- LEFT JOIN Seller with Orders filtered by 2020
- WHERE Orders.seller_id IS NULL identifies no sales
- Date filtering for 2020: YEAR(sale_date) = 2020
- Alternative: sale_date BETWEEN '2020-01-01' AND '2020-12-31'

### 3. **Interview Discussion Points**
- "This is a LEFT JOIN with NULL filtering on filtered dataset"
- "Must filter orders by year 2020 before joining"
- "Different from finding sellers with no sales ever"

## 🔧 Step-by-Step Solution Logic

### Step 1: Filter Orders by 2020
```sql
-- Filter orders to only 2020 sales
FROM Orders 
WHERE YEAR(sale_date) = 2020
```

### Step 2: LEFT JOIN with Filtered Orders
```sql
-- Join sellers with 2020 orders
FROM Seller s
LEFT JOIN (Orders filtered by 2020) o ON s.seller_id = o.seller_id
```

### Step 3: Find Sellers with No 2020 Sales
```sql
-- WHERE o.seller_id IS NULL
-- Identifies sellers without 2020 sales
```

### Step 4: Select Seller Names
```sql
-- SELECT seller_name
-- Return names of sellers without 2020 sales
```

## ✅ Optimized SQL Solution

**Solution 1: LEFT JOIN with Subquery**
```sql
SELECT s.seller_name
FROM Seller s
LEFT JOIN (
    SELECT DISTINCT seller_id 
    FROM Orders 
    WHERE YEAR(sale_date) = 2020
) o2020 ON s.seller_id = o2020.seller_id
WHERE o2020.seller_id IS NULL;
```

### Alternative Solutions

**Solution 2: NOT EXISTS**
```sql
SELECT seller_name
FROM Seller s
WHERE NOT EXISTS (
    SELECT 1 
    FROM Orders o 
    WHERE o.seller_id = s.seller_id 
      AND YEAR(o.sale_date) = 2020
);
```

**Solution 3: NOT IN (with NULL handling)**
```sql
SELECT seller_name
FROM Seller
WHERE seller_id NOT IN (
    SELECT seller_id 
    FROM Orders 
    WHERE YEAR(sale_date) = 2020 
      AND seller_id IS NOT NULL
);
```

**Solution 4: Using Date Range (More Portable)**
```sql
SELECT s.seller_name
FROM Seller s
LEFT JOIN (
    SELECT DISTINCT seller_id 
    FROM Orders 
    WHERE sale_date >= '2020-01-01' 
      AND sale_date <= '2020-12-31'
) o2020 ON s.seller_id = o2020.seller_id
WHERE o2020.seller_id IS NULL;
```

**Solution 5: Comprehensive Analysis with Performance Metrics**
```sql
SELECT 
    s.seller_name,
    s.seller_id,
    
    -- Additional context for analysis
    CASE 
        WHEN EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id AND YEAR(o.sale_date) = 2019)
        THEN 'Had 2019 Sales'
        ELSE 'No 2019 Sales'
    END as sales_2019_status,
    
    CASE 
        WHEN EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id AND YEAR(o.sale_date) = 2021)
        THEN 'Had 2021 Sales'
        ELSE 'No 2021 Sales'
    END as sales_2021_status,
    
    CASE 
        WHEN EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id)
        THEN 'Has Historical Sales'
        ELSE 'No Sales Ever'
    END as overall_sales_status,
    
    -- Count total historical sales
    COALESCE((SELECT COUNT(*) FROM Orders o WHERE o.seller_id = s.seller_id), 0) as total_historical_orders,
    
    -- Performance categorization
    CASE 
        WHEN NOT EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id)
        THEN 'Never Sold - Training Needed'
        WHEN EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id AND YEAR(o.sale_date) IN (2019, 2021))
        THEN 'Active in Other Years - 2020 Gap'
        ELSE 'Historical Seller - Inactive'
    END as seller_performance_category
FROM Seller s
WHERE NOT EXISTS (
    SELECT 1 
    FROM Orders o 
    WHERE o.seller_id = s.seller_id 
      AND YEAR(o.sale_date) = 2020
)
ORDER BY 
    CASE 
        WHEN NOT EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id) THEN 1
        WHEN EXISTS (SELECT 1 FROM Orders o WHERE o.seller_id = s.seller_id AND YEAR(o.sale_date) IN (2019, 2021)) THEN 2
        ELSE 3
    END,
    s.seller_name;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Sales Performance Analytics and Seller Management System**
```sql
-- "Build a comprehensive sales performance analytics system for seller management and optimization"

WITH seller_performance_analysis AS (
    SELECT 
        s.seller_id,
        s.seller_name,
        
        -- 2020 sales performance (target year)
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2020 THEN o.order_cost END), 0) as revenue_2020,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2020 THEN 1 END) as orders_2020,
        COUNT(DISTINCT CASE WHEN YEAR(o.sale_date) = 2020 THEN o.customer_id END) as customers_2020,
        
        -- Historical performance (all years)
        COALESCE(SUM(o.order_cost), 0) as total_revenue,
        COUNT(o.order_id) as total_orders,
        COUNT(DISTINCT o.customer_id) as total_customers,
        
        -- Year-by-year breakdown
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2019 THEN o.order_cost END), 0) as revenue_2019,
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2021 THEN o.order_cost END), 0) as revenue_2021,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2019 THEN 1 END) as orders_2019,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2021 THEN 1 END) as orders_2021,
        
        -- Performance consistency
        COUNT(DISTINCT YEAR(o.sale_date)) as active_years,
        MIN(o.sale_date) as first_sale_date,
        MAX(o.sale_date) as last_sale_date,
        
        -- Average performance metrics
        COALESCE(AVG(o.order_cost), 0) as avg_order_value,
        COALESCE(AVG(CASE WHEN YEAR(o.sale_date) = 2020 THEN o.order_cost END), 0) as avg_order_value_2020,
        
        -- Customer relationship strength
        COALESCE(MAX(customer_order_count.orders_per_customer), 0) as max_orders_per_customer,
        COALESCE(AVG(customer_order_count.orders_per_customer), 0) as avg_orders_per_customer
    FROM Seller s
    LEFT JOIN Orders o ON s.seller_id = o.seller_id
    LEFT JOIN (
        SELECT seller_id, customer_id, COUNT(*) as orders_per_customer
        FROM Orders
        GROUP BY seller_id, customer_id
    ) customer_order_count ON s.seller_id = customer_order_count.seller_id
    GROUP BY s.seller_id, s.seller_name
),
seller_categorization AS (
    SELECT 
        spa.*,
        
        -- Primary performance classification
        CASE 
            WHEN orders_2020 = 0 AND total_orders = 0 THEN 'New Seller - No Sales History'
            WHEN orders_2020 = 0 AND total_orders > 0 THEN 'Inactive in 2020 - Has History'
            WHEN orders_2020 > 0 AND orders_2020 >= 10 THEN 'High Performer 2020'
            WHEN orders_2020 > 0 AND orders_2020 >= 5 THEN 'Medium Performer 2020'
            WHEN orders_2020 > 0 THEN 'Low Performer 2020'
            ELSE 'Unclassified'
        END as performance_category_2020,
        
        -- Seller lifecycle stage
        CASE 
            WHEN total_orders = 0 THEN 'Onboarding'
            WHEN active_years = 1 AND YEAR(first_sale_date) = 2020 THEN 'New Seller 2020'
            WHEN active_years >= 3 THEN 'Veteran Seller'
            WHEN active_years = 2 THEN 'Experienced Seller'
            ELSE 'Junior Seller'
        END as seller_lifecycle_stage,
        
        -- Revenue tier classification
        CASE 
            WHEN revenue_2020 >= 5000 THEN 'Top Revenue 2020'
            WHEN revenue_2020 >= 2000 THEN 'High Revenue 2020'
            WHEN revenue_2020 >= 1000 THEN 'Medium Revenue 2020'
            WHEN revenue_2020 > 0 THEN 'Low Revenue 2020'
            ELSE 'No Revenue 2020'
        END as revenue_tier_2020,
        
        -- Customer relationship strength
        CASE 
            WHEN customers_2020 >= 10 THEN 'Strong Customer Base 2020'
            WHEN customers_2020 >= 5 THEN 'Medium Customer Base 2020'
            WHEN customers_2020 >= 1 THEN 'Small Customer Base 2020'
            ELSE 'No Customers 2020'
        END as customer_base_strength_2020,
        
        -- Year-over-year performance trend
        CASE 
            WHEN revenue_2019 > 0 AND revenue_2020 > revenue_2019 * 1.2 THEN 'Growing YoY'
            WHEN revenue_2019 > 0 AND revenue_2020 > revenue_2019 * 0.8 THEN 'Stable YoY'
            WHEN revenue_2019 > 0 AND revenue_2020 > 0 THEN 'Declining YoY'
            WHEN revenue_2019 > 0 AND revenue_2020 = 0 THEN 'Stopped Selling in 2020'
            WHEN revenue_2019 = 0 AND revenue_2020 > 0 THEN 'Started Selling in 2020'
            ELSE 'No 2019-2020 Comparison Available'
        END as yoy_trend_2019_2020,
        
        -- Recovery potential (2020 to 2021)
        CASE 
            WHEN revenue_2020 = 0 AND revenue_2021 > 0 THEN 'Recovered in 2021'
            WHEN revenue_2020 = 0 AND revenue_2021 = 0 AND total_revenue > 0 THEN 'Still Inactive in 2021'
            WHEN revenue_2020 = 0 AND total_revenue = 0 THEN 'Never Active'
            ELSE 'Was Active in 2020'
        END as recovery_status_2021
    FROM seller_performance_analysis spa
),
intervention_strategy_analysis AS (
    SELECT 
        sc.*,
        
        -- Risk assessment for sellers without 2020 sales
        CASE 
            WHEN orders_2020 = 0 AND performance_category_2020 = 'New Seller - No Sales History'
            THEN 'High Risk - Immediate Training and Support Needed'
            WHEN orders_2020 = 0 AND yoy_trend_2019_2020 = 'Stopped Selling in 2020'
            THEN 'High Risk - Investigate Reasons for Stopping'
            WHEN orders_2020 = 0 AND recovery_status_2021 = 'Recovered in 2021'
            THEN 'Medium Risk - 2020 was Anomaly, Monitor Performance'
            WHEN orders_2020 = 0 AND recovery_status_2021 = 'Still Inactive in 2021'
            THEN 'High Risk - Extended Inactivity, Consider Termination'
            WHEN orders_2020 = 0 AND recovery_status_2021 = 'Never Active'
            THEN 'Critical Risk - Complete Non-Performance'
            ELSE 'Low Risk - Active in 2020'
        END as risk_assessment,
        
        -- Recommended interventions
        CASE 
            WHEN orders_2020 = 0 AND total_orders = 0
            THEN 'Intensive Training Program + Mentorship + Performance Monitoring'
            WHEN orders_2020 = 0 AND total_revenue > 3000
            THEN 'Performance Review + Root Cause Analysis + Re-engagement Plan'
            WHEN orders_2020 = 0 AND revenue_2021 > 0
            THEN 'Monitor Closely + Understand 2020 Gap + Prevent Future Lapses'
            WHEN orders_2020 = 0 AND active_years >= 2
            THEN 'Veteran Re-activation Program + Incentive Alignment'
            WHEN orders_2020 = 0
            THEN 'Standard Re-engagement + Skills Assessment'
            ELSE 'Continue Standard Support'
        END as recommended_intervention,
        
        -- Investment priority for improvement
        CASE 
            WHEN orders_2020 = 0 AND total_revenue >= 5000
            THEN 'High Priority - Proven Seller, High Recovery Value'
            WHEN orders_2020 = 0 AND revenue_2021 >= 2000
            THEN 'High Priority - Active Recovery, Prevent Relapse'
            WHEN orders_2020 = 0 AND active_years >= 3
            THEN 'Medium Priority - Veteran Seller, Worth Investment'
            WHEN orders_2020 = 0 AND total_orders >= 5
            THEN 'Medium Priority - Has Experience, Likely Recoverable'
            WHEN orders_2020 = 0 AND total_orders > 0
            THEN 'Low Priority - Limited History, Lower Investment'
            WHEN orders_2020 = 0
            THEN 'Minimal Priority - No Proven Track Record'
            ELSE 'Maintenance - Currently Performing'
        END as investment_priority,
        
        -- Success probability for re-activation
        CASE 
            WHEN orders_2020 = 0 AND recovery_status_2021 = 'Recovered in 2021' THEN 85
            WHEN orders_2020 = 0 AND total_revenue >= 3000 THEN 75
            WHEN orders_2020 = 0 AND active_years >= 2 THEN 65
            WHEN orders_2020 = 0 AND total_orders >= 3 THEN 55
            WHEN orders_2020 = 0 AND total_orders > 0 THEN 35
            WHEN orders_2020 = 0 THEN 15
            ELSE 95  -- Already active
        END as reactivation_success_probability
    FROM seller_categorization sc
),
organizational_insights AS (
    SELECT 
        -- Overall seller portfolio metrics
        COUNT(*) as total_sellers,
        SUM(CASE WHEN orders_2020 = 0 THEN 1 ELSE 0 END) as sellers_no_2020_sales,
        SUM(CASE WHEN orders_2020 > 0 THEN 1 ELSE 0 END) as sellers_with_2020_sales,
        
        -- Performance distribution
        SUM(CASE WHEN performance_category_2020 = 'High Performer 2020' THEN 1 ELSE 0 END) as high_performers,
        SUM(CASE WHEN performance_category_2020 = 'Medium Performer 2020' THEN 1 ELSE 0 END) as medium_performers,
        SUM(CASE WHEN performance_category_2020 = 'Low Performer 2020' THEN 1 ELSE 0 END) as low_performers,
        
        -- Investment allocation
        SUM(CASE WHEN investment_priority = 'High Priority - Proven Seller, High Recovery Value' THEN 1 ELSE 0 END) as high_investment_priority,
        SUM(CASE WHEN investment_priority = 'High Priority - Active Recovery, Prevent Relapse' THEN 1 ELSE 0 END) as recovery_investment_priority,
        
        -- Financial impact
        SUM(revenue_2020) as total_revenue_2020,
        SUM(CASE WHEN orders_2020 = 0 THEN total_revenue ELSE 0 END) as potential_lost_revenue_2020,
        AVG(reactivation_success_probability) as avg_reactivation_probability,
        
        -- Risk assessment summary
        SUM(CASE WHEN risk_assessment LIKE 'High Risk%' THEN 1 ELSE 0 END) as high_risk_sellers,
        SUM(CASE WHEN risk_assessment LIKE 'Critical Risk%' THEN 1 ELSE 0 END) as critical_risk_sellers
    FROM intervention_strategy_analysis
)
SELECT 
    isa.seller_name,
    isa.seller_id,
    isa.performance_category_2020,
    isa.seller_lifecycle_stage,
    isa.yoy_trend_2019_2020,
    isa.recovery_status_2021,
    isa.risk_assessment,
    isa.recommended_intervention,
    isa.investment_priority,
    isa.reactivation_success_probability,
    
    -- Historical context
    isa.total_revenue,
    isa.total_orders,
    isa.active_years,
    isa.revenue_2019,
    isa.revenue_2021,
    
    -- Organizational benchmarks
    oi.total_sellers,
    oi.sellers_no_2020_sales,
    ROUND(isa.reactivation_success_probability - oi.avg_reactivation_probability, 1) as success_prob_vs_avg
FROM intervention_strategy_analysis isa
CROSS JOIN organizational_insights oi
WHERE isa.orders_2020 = 0  -- Focus on sellers with no 2020 sales
ORDER BY 
    CASE isa.investment_priority 
        WHEN 'High Priority - Proven Seller, High Recovery Value' THEN 1
        WHEN 'High Priority - Active Recovery, Prevent Relapse' THEN 2
        WHEN 'Medium Priority - Veteran Seller, Worth Investment' THEN 3
        WHEN 'Medium Priority - Has Experience, Likely Recoverable' THEN 4
        WHEN 'Low Priority - Limited History, Lower Investment' THEN 5
        ELSE 6
    END,
    isa.reactivation_success_probability DESC,
    isa.total_revenue DESC;
```

#### 2. **Sales Territory and Market Performance Analysis**
```sql
-- "Analyze sales territory performance and market coverage gaps for sellers without 2020 sales"

WITH territory_performance_analysis AS (
    SELECT 
        s.seller_id,
        s.seller_name,
        
        -- Sales performance by year
        SUM(CASE WHEN YEAR(o.sale_date) = 2020 THEN o.order_cost ELSE 0 END) as revenue_2020,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2020 THEN 1 END) as orders_2020,
        
        -- Customer geographic distribution simulation
        COUNT(DISTINCT o.customer_id) as total_unique_customers,
        COUNT(DISTINCT CASE WHEN YEAR(o.sale_date) = 2020 THEN o.customer_id END) as customers_2020,
        
        -- Market penetration metrics (simulated)
        CASE 
            WHEN s.seller_id % 4 = 0 THEN 'Northeast'
            WHEN s.seller_id % 4 = 1 THEN 'Southeast'
            WHEN s.seller_id % 4 = 2 THEN 'Midwest'
            ELSE 'West'
        END as assigned_territory,
        
        -- Simulate territory size and potential
        CASE 
            WHEN s.seller_id % 4 = 0 THEN 150  -- Northeast - high potential
            WHEN s.seller_id % 4 = 1 THEN 120  -- Southeast - medium-high potential
            WHEN s.seller_id % 4 = 2 THEN 100  -- Midwest - medium potential
            ELSE 180  -- West - highest potential
        END as territory_market_size,
        
        -- Competition level simulation
        CASE 
            WHEN s.seller_id % 3 = 0 THEN 'High Competition'
            WHEN s.seller_id % 3 = 1 THEN 'Medium Competition'
            ELSE 'Low Competition'
        END as competition_level,
        
        -- Product category focus simulation
        CASE 
            WHEN s.seller_id % 5 = 0 THEN 'Electronics'
            WHEN s.seller_id % 5 = 1 THEN 'Clothing'
            WHEN s.seller_id % 5 = 2 THEN 'Home & Garden'
            WHEN s.seller_id % 5 = 3 THEN 'Sports & Outdoors'
            ELSE 'Books & Media'
        END as primary_product_category,
        
        -- Experience level simulation
        CASE 
            WHEN COUNT(o.order_id) >= 20 THEN 'Senior Seller'
            WHEN COUNT(o.order_id) >= 10 THEN 'Experienced Seller'
            WHEN COUNT(o.order_id) >= 5 THEN 'Junior Seller'
            WHEN COUNT(o.order_id) > 0 THEN 'New Seller'
            ELSE 'No Sales History'
        END as experience_level
    FROM Seller s
    LEFT JOIN Orders o ON s.seller_id = o.seller_id
    GROUP BY s.seller_id, s.seller_name
),
market_gap_analysis AS (
    SELECT 
        tpa.*,
        
        -- Market penetration calculation
        CASE 
            WHEN territory_market_size > 0 THEN 
                ROUND((customers_2020 * 100.0) / territory_market_size, 2)
            ELSE 0
        END as market_penetration_2020,
        
        -- Territory performance vs potential
        CASE 
            WHEN orders_2020 = 0 AND territory_market_size >= 150 
            THEN 'High Opportunity - Large Market, No Sales'
            WHEN orders_2020 = 0 AND territory_market_size >= 120 
            THEN 'Medium Opportunity - Good Market, No Sales'
            WHEN orders_2020 = 0 AND territory_market_size >= 100 
            THEN 'Standard Opportunity - Average Market, No Sales'
            WHEN orders_2020 = 0 
            THEN 'Limited Opportunity - Small Market, No Sales'
            ELSE 'Has Sales in 2020'
        END as market_opportunity_assessment,
        
        -- Competitive positioning
        CASE 
            WHEN orders_2020 = 0 AND competition_level = 'Low Competition'
            THEN 'Easy Market Entry - Low Competition'
            WHEN orders_2020 = 0 AND competition_level = 'Medium Competition'
            THEN 'Moderate Market Entry - Medium Competition'
            WHEN orders_2020 = 0 AND competition_level = 'High Competition'
            THEN 'Difficult Market Entry - High Competition'
            ELSE 'Currently Active in Market'
        END as competitive_positioning,
        
        -- Product category performance
        CASE 
            WHEN orders_2020 = 0 AND primary_product_category = 'Electronics'
            THEN 'High-Value Category Gap'
            WHEN orders_2020 = 0 AND primary_product_category IN ('Clothing', 'Sports & Outdoors')
            THEN 'Medium-Value Category Gap'
            WHEN orders_2020 = 0 AND primary_product_category IN ('Home & Garden', 'Books & Media')
            THEN 'Standard-Value Category Gap'
            ELSE 'Category Coverage Active'
        END as category_gap_analysis,
        
        -- Experience vs performance alignment
        CASE 
            WHEN orders_2020 = 0 AND experience_level = 'Senior Seller'
            THEN 'Concerning - Experienced Seller Not Performing'
            WHEN orders_2020 = 0 AND experience_level = 'Experienced Seller'
            THEN 'Surprising - Good Experience, No 2020 Sales'
            WHEN orders_2020 = 0 AND experience_level = 'Junior Seller'
            THEN 'Expected - Junior Seller Development Needed'
            WHEN orders_2020 = 0 AND experience_level = 'New Seller'
            THEN 'Normal - New Seller Onboarding Required'
            WHEN orders_2020 = 0 AND experience_level = 'No Sales History'
            THEN 'Critical - Complete Non-Performance'
            ELSE 'Performance Aligned with Experience'
        END as experience_performance_alignment
    FROM territory_performance_analysis tpa
),
strategic_recommendations AS (
    SELECT 
        mga.*,
        
        -- Territory reallocation recommendations
        CASE 
            WHEN orders_2020 = 0 AND market_opportunity_assessment LIKE 'High Opportunity%'
            THEN 'Priority Territory Reallocation - High Potential Market'
            WHEN orders_2020 = 0 AND competitive_positioning = 'Easy Market Entry - Low Competition'
            THEN 'Fast Track Market Entry - Low Competition Advantage'
            WHEN orders_2020 = 0 AND experience_level = 'Senior Seller'
            THEN 'Investigate Territory Barriers - Experienced Seller Struggling'
            WHEN orders_2020 = 0 AND category_gap_analysis = 'High-Value Category Gap'
            THEN 'Critical Category Coverage - High Revenue Impact'
            ELSE 'Standard Territory Management'
        END as territory_strategy,
        
        -- Training and development focus
        CASE 
            WHEN orders_2020 = 0 AND competition_level = 'High Competition'
            THEN 'Advanced Competitive Strategy Training'
            WHEN orders_2020 = 0 AND market_penetration_2020 = 0 AND territory_market_size >= 150
            THEN 'Market Entry and Customer Acquisition Training'
            WHEN orders_2020 = 0 AND experience_level = 'New Seller'
            THEN 'Comprehensive Sales Fundamentals Training'
            WHEN orders_2020 = 0 AND primary_product_category = 'Electronics'
            THEN 'Technical Product Knowledge Enhancement'
            ELSE 'Standard Ongoing Training'
        END as training_focus,
        
        -- Resource allocation priority
        CASE 
            WHEN orders_2020 = 0 AND territory_market_size >= 150 AND competition_level != 'High Competition'
            THEN 'High Resource Priority - Large Market, Manageable Competition'
            WHEN orders_2020 = 0 AND experience_level = 'Senior Seller' AND territory_market_size >= 120
            THEN 'High Resource Priority - Experienced Seller in Good Market'
            WHEN orders_2020 = 0 AND category_gap_analysis = 'High-Value Category Gap'
            THEN 'Medium Resource Priority - Important Category Coverage'
            WHEN orders_2020 = 0 AND market_opportunity_assessment LIKE 'Medium Opportunity%'
            THEN 'Medium Resource Priority - Moderate Market Potential'
            WHEN orders_2020 = 0
            THEN 'Low Resource Priority - Limited Market Potential'
            ELSE 'Maintenance Resources - Currently Performing'
        END as resource_allocation_priority,
        
        -- Market coverage risk assessment
        CASE 
            WHEN orders_2020 = 0 AND territory_market_size >= 150
            THEN 'High Coverage Risk - Large Market Uncovered'
            WHEN orders_2020 = 0 AND primary_product_category = 'Electronics'
            THEN 'High Coverage Risk - High-Value Category Uncovered'
            WHEN orders_2020 = 0 AND competition_level = 'Low Competition'
            THEN 'Medium Coverage Risk - Missing Easy Market Opportunity'
            WHEN orders_2020 = 0
            THEN 'Low Coverage Risk - Limited Market Impact'
            ELSE 'No Coverage Risk - Market Covered'
        END as market_coverage_risk,
        
        -- Performance improvement timeline
        CASE 
            WHEN orders_2020 = 0 AND experience_level = 'Senior Seller'
            THEN '30-60 Days - Should Respond Quickly to Intervention'
            WHEN orders_2020 = 0 AND competitive_positioning = 'Easy Market Entry - Low Competition'
            THEN '60-90 Days - Good Market Conditions for Quick Results'
            WHEN orders_2020 = 0 AND experience_level = 'Experienced Seller'
            THEN '90-120 Days - Moderate Ramp-up Expected'
            WHEN orders_2020 = 0 AND experience_level = 'Junior Seller'
            THEN '120-180 Days - Development Timeline Required'
            WHEN orders_2020 = 0 AND experience_level = 'New Seller'
            THEN '180-240 Days - Full Training and Development Cycle'
            ELSE 'Ongoing - Maintain Current Performance'
        END as improvement_timeline
    FROM market_gap_analysis mga
),
organizational_territory_insights AS (
    SELECT 
        assigned_territory,
        COUNT(*) as sellers_in_territory,
        SUM(CASE WHEN orders_2020 = 0 THEN 1 ELSE 0 END) as underperforming_sellers,
        AVG(territory_market_size) as avg_market_size,
        AVG(market_penetration_2020) as avg_market_penetration,
        
        -- Territory health metrics
        ROUND(SUM(CASE WHEN orders_2020 = 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) as underperformance_rate,
        COUNT(CASE WHEN resource_allocation_priority = 'High Resource Priority - Large Market, Manageable Competition' THEN 1 END) as high_priority_opportunities
    FROM strategic_recommendations
    GROUP BY assigned_territory
)
SELECT 
    sr.seller_name,
    sr.assigned_territory,
    sr.territory_market_size,
    sr.market_penetration_2020,
    sr.primary_product_category,
    sr.experience_level,
    sr.competition_level,
    sr.market_opportunity_assessment,
    sr.competitive_positioning,
    sr.territory_strategy,
    sr.training_focus,
    sr.resource_allocation_priority,
    sr.market_coverage_risk,
    sr.improvement_timeline,
    
    -- Territory context
    oti.sellers_in_territory,
    oti.underperforming_sellers,
    oti.underperformance_rate as territory_underperformance_rate
FROM strategic_recommendations sr
LEFT JOIN organizational_territory_insights oti ON sr.assigned_territory = oti.assigned_territory
WHERE sr.orders_2020 = 0  -- Focus on sellers with no 2020 sales
ORDER BY 
    CASE sr.resource_allocation_priority 
        WHEN 'High Resource Priority - Large Market, Manageable Competition' THEN 1
        WHEN 'High Resource Priority - Experienced Seller in Good Market' THEN 2
        WHEN 'Medium Resource Priority - Important Category Coverage' THEN 3
        WHEN 'Medium Resource Priority - Moderate Market Potential' THEN 4
        ELSE 5
    END,
    sr.territory_market_size DESC,
    sr.seller_name;
```

#### 3. **Predictive Analytics and Retention Model for At-Risk Sellers**
```sql
-- "Build predictive analytics model to identify and retain at-risk sellers before they become non-performers"

WITH seller_risk_factors AS (
    SELECT 
        s.seller_id,
        s.seller_name,
        
        -- Current performance (2020)
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2020 THEN o.order_cost END), 0) as revenue_2020,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2020 THEN 1 END) as orders_2020,
        
        -- Historical performance patterns
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2019 THEN o.order_cost END), 0) as revenue_2019,
        COALESCE(SUM(CASE WHEN YEAR(o.sale_date) = 2018 THEN o.order_cost END), 0) as revenue_2018,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2019 THEN 1 END) as orders_2019,
        COUNT(CASE WHEN YEAR(o.sale_date) = 2018 THEN 1 END) as orders_2018,
        
        -- Performance volatility indicators
        COUNT(DISTINCT YEAR(o.sale_date)) as active_years,
        COALESCE(STDDEV(yearly_revenue.revenue), 0) as revenue_volatility,
        COALESCE(AVG(yearly_revenue.revenue), 0) as avg_yearly_revenue,
        
        -- Customer relationship indicators
        COUNT(DISTINCT o.customer_id) as total_customers,
        COUNT(DISTINCT CASE WHEN YEAR(o.sale_date) = 2020 THEN o.customer_id END) as customers_2020,
        COALESCE(MAX(customer_metrics.max_customer_orders), 0) as largest_customer_relationship,
        COALESCE(AVG(customer_metrics.customer_orders), 0) as avg_customer_relationship_strength,
        
        -- Temporal performance patterns
        MIN(o.sale_date) as first_sale_date,
        MAX(o.sale_date) as last_sale_date,
        DATEDIFF(MAX(o.sale_date), MIN(o.sale_date)) as selling_tenure_days,
        
        -- Seasonal performance (simulated)
        COUNT(CASE WHEN MONTH(o.sale_date) IN (12, 1, 2) THEN 1 END) as winter_sales,
        COUNT(CASE WHEN MONTH(o.sale_date) IN (3, 4, 5) THEN 1 END) as spring_sales,
        COUNT(CASE WHEN MONTH(o.sale_date) IN (6, 7, 8) THEN 1 END) as summer_sales,
        COUNT(CASE WHEN MONTH(o.sale_date) IN (9, 10, 11) THEN 1 END) as fall_sales
    FROM Seller s
    LEFT JOIN Orders o ON s.seller_id = o.seller_id
    LEFT JOIN (
        SELECT 
            seller_id,
            YEAR(sale_date) as year,
            SUM(order_cost) as revenue
        FROM Orders
        GROUP BY seller_id, YEAR(sale_date)
    ) yearly_revenue ON s.seller_id = yearly_revenue.seller_id
    LEFT JOIN (
        SELECT 
            seller_id,
            customer_id,
            COUNT(*) as customer_orders,
            MAX(COUNT(*)) OVER (PARTITION BY seller_id) as max_customer_orders
        FROM Orders
        GROUP BY seller_id, customer_id
    ) customer_metrics ON s.seller_id = customer_metrics.seller_id
    GROUP BY s.seller_id, s.seller_name
),
risk_scoring_model AS (
    SELECT 
        srf.*,
        
        -- Performance decline risk factors
        CASE 
            WHEN revenue_2019 > 0 AND revenue_2020 = 0 THEN 50  -- Complete stop
            WHEN revenue_2019 > 0 AND revenue_2020 < revenue_2019 * 0.5 THEN 35  -- Major decline
            WHEN revenue_2019 > 0 AND revenue_2020 < revenue_2019 * 0.8 THEN 20  -- Moderate decline
            WHEN revenue_2020 = 0 AND revenue_2019 = 0 THEN 45  -- Never started
            ELSE 0
        END as performance_decline_score,
        
        -- Customer concentration risk
        CASE 
            WHEN total_customers > 0 AND largest_customer_relationship / total_customers >= 0.8 THEN 25  -- High concentration
            WHEN total_customers > 0 AND largest_customer_relationship / total_customers >= 0.6 THEN 15  -- Medium concentration
            WHEN total_customers > 0 AND largest_customer_relationship / total_customers >= 0.4 THEN 8   -- Low concentration
            WHEN total_customers = 0 THEN 30  -- No customers
            ELSE 0
        END as customer_concentration_risk_score,
        
        -- Tenure and experience risk
        CASE 
            WHEN selling_tenure_days < 90 THEN 20  -- Very new seller
            WHEN selling_tenure_days < 180 THEN 15  -- New seller
            WHEN selling_tenure_days < 365 THEN 10  -- Junior seller
            WHEN selling_tenure_days >= 365 AND orders_2020 = 0 THEN 25  -- Experienced but not performing
            ELSE 0
        END as tenure_risk_score,
        
        -- Volatility and consistency risk
        CASE 
            WHEN avg_yearly_revenue > 0 AND revenue_volatility / avg_yearly_revenue > 1.5 THEN 20  -- High volatility
            WHEN avg_yearly_revenue > 0 AND revenue_volatility / avg_yearly_revenue > 1.0 THEN 12  -- Medium volatility
            WHEN avg_yearly_revenue > 0 AND revenue_volatility / avg_yearly_revenue > 0.5 THEN 6   -- Low volatility
            WHEN active_years <= 1 THEN 15  -- Insufficient data
            ELSE 0
        END as volatility_risk_score,
        
        -- Seasonal dependency risk
        CASE 
            WHEN (winter_sales + spring_sales + summer_sales + fall_sales) > 0 THEN
                CASE 
                    WHEN GREATEST(winter_sales, spring_sales, summer_sales, fall_sales) / 
                         (winter_sales + spring_sales + summer_sales + fall_sales) >= 0.7 THEN 15  -- High seasonal dependency
                    WHEN GREATEST(winter_sales, spring_sales, summer_sales, fall_sales) / 
                         (winter_sales + spring_sales + summer_sales + fall_sales) >= 0.5 THEN 8   -- Medium seasonal dependency
                    ELSE 0
                END
            ELSE 10  -- No sales data
        END as seasonal_dependency_risk_score,
        
        -- Customer base erosion risk
        CASE 
            WHEN customers_2020 = 0 AND total_customers > 0 THEN 30  -- Lost all customers
            WHEN total_customers > 0 AND customers_2020 < total_customers * 0.5 THEN 20  -- Lost majority of customers
            WHEN total_customers > 0 AND customers_2020 < total_customers * 0.8 THEN 10  -- Lost some customers
            WHEN total_customers = 0 THEN 25  -- Never had customers
            ELSE 0
        END as customer_erosion_risk_score
    FROM seller_risk_factors srf
),
predictive_risk_assessment AS (
    SELECT 
        rsm.*,
        
        -- Combined risk score
        performance_decline_score + 
        customer_concentration_risk_score + 
        tenure_risk_score + 
        volatility_risk_score + 
        seasonal_dependency_risk_score + 
        customer_erosion_risk_score as total_risk_score,
        
        -- Risk categorization
        CASE 
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 80 
            THEN 'Critical Risk'
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 60 
            THEN 'High Risk'
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 40 
            THEN 'Medium Risk'
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 20 
            THEN 'Low Risk'
            ELSE 'Minimal Risk'
        END as risk_category,
        
        -- Predictive churn probability
        CASE 
            WHEN orders_2020 = 0 THEN 90  -- Already churned in 2020
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 80 
            THEN 75  -- Very high churn probability
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 60 
            THEN 55  -- High churn probability
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 40 
            THEN 35  -- Medium churn probability
            WHEN (performance_decline_score + customer_concentration_risk_score + tenure_risk_score + 
                  volatility_risk_score + seasonal_dependency_risk_score + customer_erosion_risk_score) >= 20 
            THEN 15  -- Low churn probability
            ELSE 5   -- Very low churn probability
        END as churn_probability_percent,
        
        -- Recovery potential
        CASE 
            WHEN orders_2020 = 0 AND revenue_2019 >= 3000 AND tenure_risk_score <= 10 
            THEN 'High Recovery Potential'
            WHEN orders_2020 = 0 AND revenue_2019 >= 1000 AND customer_concentration_risk_score <= 15 
            THEN 'Medium Recovery Potential'
            WHEN orders_2020 = 0 AND revenue_2019 > 0 
            THEN 'Low Recovery Potential'
            WHEN orders_2020 = 0 
            THEN 'Minimal Recovery Potential'
            ELSE 'Not Applicable - Currently Active'
        END as recovery_potential,
        
        -- Early warning indicators
        CASE 
            WHEN orders_2020 > 0 AND (performance_decline_score >= 20 OR customer_erosion_risk_score >= 20)
            THEN 'Early Warning - Monitor Closely'
            WHEN orders_2020 > 0 AND total_risk_score >= 40
            THEN 'Watch List - Preventive Action Needed'
            WHEN orders_2020 > 0 AND total_risk_score >= 20
            THEN 'Standard Monitoring - Some Risk Factors Present'
            WHEN orders_2020 > 0
            THEN 'Healthy Seller - Continue Standard Support'
            ELSE 'Already At-Risk or Churned'
        END as early_warning_status
    FROM risk_scoring_model rsm
),
intervention_recommendations AS (
    SELECT 
        pra.*,
        
        -- Immediate intervention recommendations
        CASE 
            WHEN orders_2020 = 0 AND recovery_potential = 'High Recovery Potential'
            THEN 'Urgent: Executive Intervention + Personalized Recovery Plan + Dedicated Support'
            WHEN orders_2020 = 0 AND recovery_potential = 'Medium Recovery Potential'
            THEN 'High Priority: Account Manager Assignment + Structured Re-engagement Program'
            WHEN orders_2020 = 0 AND recovery_potential = 'Low Recovery Potential'
            THEN 'Standard: Automated Re-engagement Campaign + Training Opportunities'
            WHEN orders_2020 = 0
            THEN 'Minimal: Basic Reactivation Attempt + Consider Termination Timeline'
            WHEN early_warning_status = 'Early Warning - Monitor Closely'
            THEN 'Preventive: Weekly Check-ins + Address Risk Factors Immediately'
            WHEN early_warning_status = 'Watch List - Preventive Action Needed'
            THEN 'Proactive: Monthly Reviews + Support Enhancement + Performance Goals'
            ELSE 'Standard: Regular Business Reviews + Ongoing Support'
        END as intervention_recommendation,
        
        -- Resource allocation for intervention
        CASE 
            WHEN orders_2020 = 0 AND recovery_potential = 'High Recovery Potential'
            THEN 'High Investment: $2000-5000 recovery budget per seller'
            WHEN orders_2020 = 0 AND recovery_potential = 'Medium Recovery Potential'
            THEN 'Medium Investment: $500-2000 recovery budget per seller'
            WHEN orders_2020 = 0 AND recovery_potential = 'Low Recovery Potential'
            THEN 'Low Investment: $100-500 recovery budget per seller'
            WHEN orders_2020 = 0
            THEN 'Minimal Investment: $50-100 basic reactivation budget per seller'
            WHEN churn_probability_percent >= 55
            THEN 'Prevention Investment: $300-1000 retention budget per seller'
            WHEN churn_probability_percent >= 35
            THEN 'Monitoring Investment: $100-300 monitoring budget per seller'
            ELSE 'Standard Investment: Regular operational costs only'
        END as investment_recommendation,
        
        -- Success metrics and timeline
        CASE 
            WHEN orders_2020 = 0 AND recovery_potential = 'High Recovery Potential'
            THEN 'Target: First sale within 30 days, $1000+ revenue in 90 days'
            WHEN orders_2020 = 0 AND recovery_potential = 'Medium Recovery Potential'
            THEN 'Target: First sale within 60 days, $500+ revenue in 120 days'
            WHEN orders_2020 = 0 AND recovery_potential = 'Low Recovery Potential'
            THEN 'Target: Any sale within 90 days, sustained activity in 180 days'
            WHEN orders_2020 = 0
            THEN 'Target: Engagement response within 30 days, decide by 60 days'
            WHEN churn_probability_percent >= 55
            THEN 'Target: Maintain current performance, reduce risk factors by 50%'
            WHEN churn_probability_percent >= 35
            THEN 'Target: 10%+ performance improvement, risk score reduction'
            ELSE 'Target: Maintain or improve current performance levels'
        END as success_metrics_timeline
    FROM predictive_risk_assessment pra
)
SELECT 
    seller_name,
    orders_2020,
    revenue_2020,
    risk_category,
    total_risk_score,
    churn_probability_percent,
    recovery_potential,
    early_warning_status,
    intervention_recommendation,
    investment_recommendation,
    success_metrics_timeline,
    
    -- Risk factor breakdown
    performance_decline_score,
    customer_concentration_risk_score,
    tenure_risk_score,
    volatility_risk_score,
    customer_erosion_risk_score,
    
    -- Historical context
    revenue_2019,
    total_customers,
    selling_tenure_days
FROM intervention_recommendations
WHERE orders_2020 = 0 OR churn_probability_percent >= 35  -- Focus on at-risk sellers
ORDER BY 
    CASE 
        WHEN orders_2020 = 0 AND recovery_potential = 'High Recovery Potential' THEN 1
        WHEN orders_2020 = 0 AND recovery_potential = 'Medium Recovery Potential' THEN 2
        WHEN churn_probability_percent >= 55 THEN 3
        WHEN orders_2020 = 0 AND recovery_potential = 'Low Recovery Potential' THEN 4
        WHEN churn_probability_percent >= 35 THEN 5
        ELSE 6
    END,
    total_risk_score DESC,
    revenue_2019 DESC;
```

## 🔗 Related LeetCode Questions

1. **#1581 - Customer Who Visited but Did Not Make Any Transactions** (LEFT JOIN with NULL filtering)
2. **#1378 - Replace Employee ID With The Unique Identifier** (LEFT JOIN patterns)
3. **#1280 - Students and Examinations** (LEFT JOIN with missing data)
4. **#1965 - Employees With Missing Information** (Finding missing relationships)
5. **#1795 - Rearrange Products Table** (Data analysis with JOINs)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Filtered LEFT JOIN**: First filter data, then join to find missing relationships
2. **Date Filtering**: YEAR() function vs date range comparisons
3. **Subquery in JOIN**: Using filtered subquery for specific year analysis
4. **NULL Detection**: IS NULL to identify missing relationships

### 🚀 **Amazon Interview Tips**
1. **Explain filtering logic**: "Filter orders by 2020 first, then find sellers without matches"
2. **Discuss date functions**: "YEAR() is readable but date ranges might be more portable"
3. **Consider alternatives**: "NOT EXISTS often performs better than LEFT JOIN"
4. **Address edge cases**: "What if seller has no orders in any year?"

### 🔧 **Common Patterns**
- Filter data before joining for performance
- Use DISTINCT in subquery when appropriate
- Date range filtering for year-specific analysis
- LEFT JOIN with IS NULL for missing data analysis

### ⚠️ **Common Mistakes to Avoid**
1. **Filtering after JOIN** (includes all years instead of just 2020)
2. **Missing DISTINCT** (duplicate seller_ids in subquery)
3. **Wrong date filtering** (not handling date boundaries correctly)
4. **Performance issues** (not considering index usage on date columns)

### 🔍 **Performance Considerations**
- Index on sale_date for date filtering performance
- DISTINCT in subquery prevents duplicate processing
- NOT EXISTS can be faster for large datasets
- Consider query execution plan for optimization

### 🎯 **Amazon Leadership Principles Applied**
- **Operational Excellence**: Systematic identification of underperforming sellers
- **Ownership**: Taking responsibility for seller success and market coverage
- **Deliver Results**: Data-driven approach to sales team optimization
- **Earn Trust**: Transparent performance analysis builds seller confidence

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find sellers with declining year-over-year performance**
2. **Identify sellers who stopped selling in specific quarters**
3. **Find product categories with no sales in 2020**
4. **Analyze customer segments with no purchases from specific sellers**

Remember: Sales performance analysis is crucial for Amazon's marketplace management, seller ecosystem optimization, revenue forecasting, and strategic business development across all product categories and geographic markets!