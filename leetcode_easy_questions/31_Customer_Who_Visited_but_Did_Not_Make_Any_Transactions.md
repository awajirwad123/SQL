# LeetCode Easy #1581: Customer Who Visited but Did Not Make Any Transactions

## 📋 Problem Statement

Write a SQL query to find the IDs of the users who visited without making any transactions and the number of times they made these types of visits.

Return the result table sorted in **any order**.

## 🗄️ Table Schema

**Visits Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| visit_id    | int     |
| customer_id | int     |
+-------------+---------+
```
- visit_id is the primary key for this table.
- This table contains information about the visits of the customers.

**Transactions Table:**
```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| transaction_id | int     |
| visit_id       | int     |
| amount         | int     |
+----------------+---------+
```
- transaction_id is the primary key for this table.
- visit_id is a foreign key (reference column) to the Visits table.
- This table contains information about the transactions made during the visits.

## 📊 Sample Data

**Visits Table:**
| visit_id | customer_id |
|----------|-------------|
| 1        | 23          |
| 2        | 9           |
| 4        | 30          |
| 5        | 54          |
| 6        | 96          |
| 7        | 54          |
| 8        | 54          |

**Transactions Table:**
| transaction_id | visit_id | amount |
|----------------|----------|--------|
| 2              | 5        | 310    |
| 3              | 5        | 300    |
| 9              | 5        | 200    |
| 12             | 1        | 910    |
| 13             | 2        | 970    |

**Expected Output:**
| customer_id | count_no_trans |
|-------------|----------------|
| 54          | 2              |
| 30          | 1              |
| 96          | 1              |

**Explanation:**
- **Customer 23**: Made 1 visit (visit_id=1) ✅ Had transaction (transaction_id=12) → Not included
- **Customer 9**: Made 1 visit (visit_id=2) ✅ Had transaction (transaction_id=13) → Not included
- **Customer 30**: Made 1 visit (visit_id=4) ❌ No transactions → Included (count=1)
- **Customer 54**: Made 3 visits (visit_id=5,7,8) ✅ Had transactions only for visit_id=5 → 2 visits without transactions
- **Customer 96**: Made 1 visit (visit_id=6) ❌ No transactions → Included (count=1)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find customers who visited but didn't make transactions
- Count how many such visits each customer made
- Need to identify visits without corresponding transactions
- Use LEFT JOIN to find visits without transactions

### 2. **Key Insights**
- LEFT JOIN Visits with Transactions on visit_id
- Where Transactions.visit_id IS NULL indicates no transaction
- GROUP BY customer_id to count visits per customer
- COUNT(*) gives number of visits without transactions

### 3. **Interview Discussion Points**
- "This is a classic LEFT JOIN with NULL filtering problem"
- "Need to count visits per customer where no transaction occurred"
- "LEFT JOIN preserves all visits, NULL indicates missing transactions"

## 🔧 Step-by-Step Solution Logic

### Step 1: Join Tables
```sql
-- LEFT JOIN to keep all visits
-- Even those without transactions
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
```

### Step 2: Filter Non-Transaction Visits
```sql
-- WHERE t.visit_id IS NULL
-- Identifies visits without transactions
```

### Step 3: Group and Count
```sql
-- GROUP BY customer_id
-- COUNT(*) for visits per customer
```

### Step 4: Select Results
```sql
-- customer_id and count of visits without transactions
SELECT customer_id, COUNT(*) as count_no_trans
```

## ✅ Optimized SQL Solution

**Solution 1: LEFT JOIN with NULL Filtering**
```sql
SELECT 
    v.customer_id,
    COUNT(*) as count_no_trans
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.visit_id IS NULL
GROUP BY v.customer_id;
```

### Alternative Solutions

**Solution 2: Using NOT EXISTS**
```sql
SELECT 
    customer_id,
    COUNT(*) as count_no_trans
FROM Visits v
WHERE NOT EXISTS (
    SELECT 1 
    FROM Transactions t 
    WHERE t.visit_id = v.visit_id
)
GROUP BY customer_id;
```

**Solution 3: Using NOT IN (with NULL handling)**
```sql
SELECT 
    customer_id,
    COUNT(*) as count_no_trans
FROM Visits
WHERE visit_id NOT IN (
    SELECT visit_id 
    FROM Transactions 
    WHERE visit_id IS NOT NULL
)
GROUP BY customer_id;
```

**Solution 4: Using EXCEPT/MINUS (PostgreSQL/Oracle)**
```sql
-- PostgreSQL version
SELECT 
    customer_id,
    COUNT(*) as count_no_trans
FROM Visits
WHERE visit_id IN (
    SELECT visit_id FROM Visits
    EXCEPT
    SELECT visit_id FROM Transactions
)
GROUP BY customer_id;
```

**Solution 5: Comprehensive Analysis with Visit Details**
```sql
SELECT 
    v.customer_id,
    COUNT(*) as count_no_trans,
    GROUP_CONCAT(v.visit_id ORDER BY v.visit_id) as visits_without_transactions,
    
    -- Additional analytics
    COUNT(DISTINCT v.visit_id) as total_visits_no_trans,
    
    -- Visit pattern analysis
    CASE 
        WHEN COUNT(*) = 1 THEN 'Single Non-Transaction Visit'
        WHEN COUNT(*) <= 3 THEN 'Few Non-Transaction Visits'
        WHEN COUNT(*) <= 5 THEN 'Multiple Non-Transaction Visits'
        ELSE 'Frequent Non-Transaction Visits'
    END as visit_pattern,
    
    -- Customer behavior insight
    CASE 
        WHEN COUNT(*) >= 3 THEN 'High Browse Behavior'
        WHEN COUNT(*) = 2 THEN 'Moderate Browse Behavior'
        ELSE 'Low Browse Behavior'
    END as customer_behavior_type
FROM Visits v
LEFT JOIN Transactions t ON v.visit_id = t.visit_id
WHERE t.visit_id IS NULL
GROUP BY v.customer_id
ORDER BY count_no_trans DESC, v.customer_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Customer Journey Analytics and Conversion Optimization**
```sql
-- "Analyze complete customer journey including conversion patterns and optimization opportunities"

WITH customer_visit_analysis AS (
    SELECT 
        v.customer_id,
        v.visit_id,
        t.transaction_id,
        t.amount,
        
        -- Visit outcome classification
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 'Transaction Visit'
            ELSE 'Non-Transaction Visit'
        END as visit_outcome,
        
        -- Simulate visit timestamps and other attributes
        CASE 
            WHEN v.visit_id % 10 = 0 THEN '2024-01-15 09:00:00'
            WHEN v.visit_id % 10 = 1 THEN '2024-01-15 14:30:00'
            WHEN v.visit_id % 10 = 2 THEN '2024-01-16 10:15:00'
            WHEN v.visit_id % 10 = 3 THEN '2024-01-16 16:45:00'
            WHEN v.visit_id % 10 = 4 THEN '2024-01-17 11:20:00'
            WHEN v.visit_id % 10 = 5 THEN '2024-01-17 15:10:00'
            WHEN v.visit_id % 10 = 6 THEN '2024-01-18 12:30:00'
            WHEN v.visit_id % 10 = 7 THEN '2024-01-18 17:00:00'
            WHEN v.visit_id % 10 = 8 THEN '2024-01-19 13:45:00'
            ELSE '2024-01-19 18:20:00'
        END as simulated_visit_timestamp,
        
        -- Simulate visit duration and page views
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 
                ROUND(5 + (v.visit_id % 15), 0)  -- Transaction visits longer
            ELSE 
                ROUND(1 + (v.visit_id % 8), 0)   -- Non-transaction visits shorter
        END as simulated_duration_minutes,
        
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 
                ROUND(8 + (v.visit_id % 12), 0)  -- Transaction visits more pages
            ELSE 
                ROUND(2 + (v.visit_id % 6), 0)   -- Non-transaction visits fewer pages
        END as simulated_page_views,
        
        -- Simulate visit source
        CASE 
            WHEN v.visit_id % 5 = 0 THEN 'Organic Search'
            WHEN v.visit_id % 5 = 1 THEN 'Paid Search'
            WHEN v.visit_id % 5 = 2 THEN 'Social Media'
            WHEN v.visit_id % 5 = 3 THEN 'Direct'
            ELSE 'Email Campaign'
        END as simulated_traffic_source
    FROM Visits v
    LEFT JOIN Transactions t ON v.visit_id = t.visit_id
),
customer_journey_metrics AS (
    SELECT 
        customer_id,
        
        -- Visit summary metrics
        COUNT(*) as total_visits,
        SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END) as transaction_visits,
        SUM(CASE WHEN visit_outcome = 'Non-Transaction Visit' THEN 1 ELSE 0 END) as non_transaction_visits,
        
        -- Conversion rate calculation
        ROUND(
            SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 
            2
        ) as conversion_rate_percent,
        
        -- Revenue metrics
        COALESCE(SUM(amount), 0) as total_revenue,
        COALESCE(AVG(amount), 0) as avg_transaction_value,
        COALESCE(SUM(amount) / NULLIF(SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END), 0), 0) as revenue_per_transaction_visit,
        
        -- Engagement metrics
        AVG(simulated_duration_minutes) as avg_visit_duration,
        AVG(simulated_page_views) as avg_page_views_per_visit,
        
        -- Visit pattern analysis
        MIN(simulated_visit_timestamp) as first_visit,
        MAX(simulated_visit_timestamp) as last_visit,
        
        -- Traffic source diversity
        COUNT(DISTINCT simulated_traffic_source) as traffic_sources_used,
        
        -- Behavioral segmentation
        CASE 
            WHEN COUNT(*) >= 5 THEN 'High Frequency Visitor'
            WHEN COUNT(*) >= 3 THEN 'Medium Frequency Visitor'
            WHEN COUNT(*) = 2 THEN 'Low Frequency Visitor'
            ELSE 'One-Time Visitor'
        END as visit_frequency_segment,
        
        CASE 
            WHEN ROUND(SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) >= 80 
            THEN 'High Converter'
            WHEN ROUND(SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) >= 50 
            THEN 'Medium Converter'
            WHEN ROUND(SUM(CASE WHEN visit_outcome = 'Transaction Visit' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) > 0 
            THEN 'Low Converter'
            ELSE 'Browser Only'
        END as conversion_segment
    FROM customer_visit_analysis
    GROUP BY customer_id
),
conversion_optimization_analysis AS (
    SELECT 
        cjm.*,
        
        -- Customer value classification
        CASE 
            WHEN total_revenue >= 1000 THEN 'High Value Customer'
            WHEN total_revenue >= 500 THEN 'Medium Value Customer'
            WHEN total_revenue > 0 THEN 'Low Value Customer'
            ELSE 'No Revenue Customer'
        END as customer_value_tier,
        
        -- Optimization opportunity assessment
        CASE 
            WHEN conversion_segment = 'Browser Only' AND visit_frequency_segment = 'High Frequency Visitor'
            THEN 'High Priority - Frequent Browser, Never Converts'
            WHEN conversion_segment = 'Browser Only' AND visit_frequency_segment = 'Medium Frequency Visitor'
            THEN 'Medium Priority - Regular Browser, Never Converts'
            WHEN conversion_segment = 'Low Converter' AND avg_visit_duration >= 5
            THEN 'High Priority - Engaged but Low Converting'
            WHEN conversion_segment = 'Medium Converter' AND total_revenue < 500
            THEN 'Medium Priority - Converting but Low Value'
            WHEN conversion_segment = 'High Converter' AND avg_transaction_value < 300
            THEN 'Low Priority - High Conversion, Increase AOV'
            ELSE 'Monitor - Standard Performance'
        END as optimization_priority,
        
        -- Personalization strategy
        CASE 
            WHEN conversion_segment = 'Browser Only' AND avg_page_views_per_visit >= 5
            THEN 'Show targeted offers, implement exit-intent popups'
            WHEN conversion_segment = 'Browser Only' AND avg_visit_duration >= 3
            THEN 'Retargeting campaigns, abandoned browse remarketing'
            WHEN conversion_segment = 'Low Converter' AND avg_transaction_value > 0
            THEN 'Upselling campaigns, loyalty program enrollment'
            WHEN conversion_segment = 'Medium Converter'
            THEN 'Cross-selling opportunities, premium product recommendations'
            WHEN conversion_segment = 'High Converter'
            THEN 'VIP treatment, early access to new products'
            ELSE 'Standard engagement strategy'
        END as personalization_strategy,
        
        -- Predicted lifetime value
        CASE 
            WHEN conversion_segment = 'High Converter' AND visit_frequency_segment = 'High Frequency Visitor'
            THEN total_revenue * 3.5  -- High multiplier for best customers
            WHEN conversion_segment = 'Medium Converter' AND visit_frequency_segment IN ('High Frequency Visitor', 'Medium Frequency Visitor')
            THEN total_revenue * 2.2  -- Medium multiplier
            WHEN conversion_segment = 'Low Converter' AND visit_frequency_segment != 'One-Time Visitor'
            THEN total_revenue * 1.8  -- Low multiplier but some potential
            WHEN conversion_segment = 'Browser Only' AND visit_frequency_segment = 'High Frequency Visitor'
            THEN 200  -- Potential value if converted
            WHEN conversion_segment = 'Browser Only' AND visit_frequency_segment = 'Medium Frequency Visitor'
            THEN 100  -- Lower potential value
            ELSE total_revenue * 1.1  -- Minimal growth expected
        END as predicted_clv
    FROM customer_journey_metrics cjm
),
actionable_insights AS (
    SELECT 
        coa.*,
        
        -- Immediate action recommendations
        CASE 
            WHEN optimization_priority = 'High Priority - Frequent Browser, Never Converts'
            THEN 'Immediate Action: Launch personalized retargeting campaign with 15% discount'
            WHEN optimization_priority = 'High Priority - Engaged but Low Converting'
            THEN 'Immediate Action: A/B test checkout flow optimization and payment options'
            WHEN optimization_priority = 'Medium Priority - Regular Browser, Never Converts'
            THEN 'Immediate Action: Implement browse abandonment email sequence'
            WHEN optimization_priority = 'Medium Priority - Converting but Low Value'
            THEN 'Immediate Action: Introduce bundle offers and free shipping thresholds'
            WHEN optimization_priority = 'Low Priority - High Conversion, Increase AOV'
            THEN 'Immediate Action: Implement cross-sell recommendations at checkout'
            ELSE 'Monitor and maintain current engagement'
        END as immediate_action,
        
        -- Campaign targeting
        CASE 
            WHEN non_transaction_visits >= 3 AND avg_visit_duration >= 4
            THEN 'High-Intent Browse Abandonment Campaign'
            WHEN non_transaction_visits >= 2 AND avg_page_views_per_visit >= 5
            THEN 'Product Interest Retargeting Campaign'
            WHEN non_transaction_visits >= 1 AND traffic_sources_used >= 3
            THEN 'Multi-Channel Engagement Campaign'
            ELSE 'Standard Nurture Campaign'
        END as campaign_targeting,
        
        -- Budget allocation recommendation
        CASE 
            WHEN predicted_clv >= 1000 THEN 'High Budget - $50-100 per customer'
            WHEN predicted_clv >= 500 THEN 'Medium Budget - $25-50 per customer'
            WHEN predicted_clv >= 100 THEN 'Low Budget - $10-25 per customer'
            ELSE 'Minimal Budget - $5-10 per customer'
        END as marketing_budget_allocation,
        
        -- Success metrics
        CASE 
            WHEN conversion_segment = 'Browser Only'
            THEN 'Target: First Purchase within 30 days, 15%+ conversion rate'
            WHEN conversion_segment = 'Low Converter'
            THEN 'Target: 25%+ conversion rate, 20%+ increase in AOV'
            WHEN conversion_segment = 'Medium Converter'
            THEN 'Target: 40%+ conversion rate, 30%+ increase in AOV'
            ELSE 'Target: Maintain performance, increase purchase frequency'
        END as success_metrics
    FROM conversion_optimization_analysis coa
)
SELECT 
    customer_id,
    non_transaction_visits as count_no_trans,
    total_visits,
    conversion_rate_percent,
    total_revenue,
    customer_value_tier,
    visit_frequency_segment,
    conversion_segment,
    optimization_priority,
    personalization_strategy,
    immediate_action,
    campaign_targeting,
    predicted_clv,
    marketing_budget_allocation,
    success_metrics
FROM actionable_insights
WHERE non_transaction_visits > 0  -- Focus on customers who browsed without buying
ORDER BY 
    CASE optimization_priority 
        WHEN 'High Priority - Frequent Browser, Never Converts' THEN 1
        WHEN 'High Priority - Engaged but Low Converting' THEN 2
        WHEN 'Medium Priority - Regular Browser, Never Converts' THEN 3
        WHEN 'Medium Priority - Converting but Low Value' THEN 4
        ELSE 5
    END,
    predicted_clv DESC,
    non_transaction_visits DESC;
```

#### 2. **Real-time Customer Behavior Analytics and Intervention System**
```sql
-- "Build real-time customer behavior analytics system with automated intervention triggers"

WITH real_time_visit_behavior AS (
    SELECT 
        v.customer_id,
        v.visit_id,
        t.transaction_id,
        t.amount,
        
        -- Real-time behavior flags
        CASE WHEN t.transaction_id IS NULL THEN 1 ELSE 0 END as is_browse_only_visit,
        
        -- Simulate real-time engagement metrics
        CASE 
            WHEN v.visit_id % 4 = 0 THEN 90  -- High engagement
            WHEN v.visit_id % 4 = 1 THEN 65  -- Medium engagement
            WHEN v.visit_id % 4 = 2 THEN 40  -- Low engagement
            ELSE 15  -- Very low engagement
        END as engagement_score,
        
        -- Simulate time on site (seconds)
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 
                300 + (v.visit_id % 600)  -- Transaction visits: 5-15 minutes
            ELSE 
                60 + (v.visit_id % 240)   -- Browse visits: 1-5 minutes
        END as time_on_site_seconds,
        
        -- Simulate cart interactions
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 'Purchased'
            WHEN v.visit_id % 3 = 0 THEN 'Added to Cart'
            WHEN v.visit_id % 3 = 1 THEN 'Viewed Product'
            ELSE 'Browsed Only'
        END as highest_funnel_stage,
        
        -- Simulate exit behavior
        CASE 
            WHEN t.transaction_id IS NOT NULL THEN 'Completed Purchase'
            WHEN v.visit_id % 5 = 0 THEN 'Abandoned Cart'
            WHEN v.visit_id % 5 = 1 THEN 'Exit from Product Page'
            WHEN v.visit_id % 5 = 2 THEN 'Exit from Category Page'
            WHEN v.visit_id % 5 = 3 THEN 'Exit from Homepage'
            ELSE 'Session Timeout'
        END as exit_behavior,
        
        -- Device and experience simulation
        CASE 
            WHEN v.visit_id % 3 = 0 THEN 'Mobile'
            WHEN v.visit_id % 3 = 1 THEN 'Desktop'
            ELSE 'Tablet'
        END as device_type,
        
        -- Customer journey stage
        ROW_NUMBER() OVER (PARTITION BY v.customer_id ORDER BY v.visit_id) as visit_sequence
    FROM Visits v
    LEFT JOIN Transactions t ON v.visit_id = t.visit_id
),
customer_behavior_patterns AS (
    SELECT 
        customer_id,
        
        -- Visit pattern analysis
        COUNT(*) as total_visits,
        SUM(is_browse_only_visit) as browse_only_visits,
        SUM(CASE WHEN transaction_id IS NOT NULL THEN 1 ELSE 0 END) as purchase_visits,
        
        -- Engagement pattern analysis
        AVG(engagement_score) as avg_engagement_score,
        MAX(engagement_score) as peak_engagement_score,
        AVG(time_on_site_seconds) as avg_time_on_site,
        
        -- Funnel progression analysis
        MAX(CASE 
            WHEN highest_funnel_stage = 'Purchased' THEN 4
            WHEN highest_funnel_stage = 'Added to Cart' THEN 3
            WHEN highest_funnel_stage = 'Viewed Product' THEN 2
            WHEN highest_funnel_stage = 'Browsed Only' THEN 1
        END) as max_funnel_progression,
        
        -- Exit behavior patterns
        SUM(CASE WHEN exit_behavior = 'Abandoned Cart' THEN 1 ELSE 0 END) as cart_abandonment_count,
        SUM(CASE WHEN exit_behavior LIKE 'Exit from%' THEN 1 ELSE 0 END) as premature_exit_count,
        
        -- Device usage patterns
        COUNT(DISTINCT device_type) as device_types_used,
        MAX(CASE WHEN device_type = 'Mobile' THEN 1 ELSE 0 END) as uses_mobile,
        MAX(CASE WHEN device_type = 'Desktop' THEN 1 ELSE 0 END) as uses_desktop,
        
        -- Journey progression
        COUNT(CASE WHEN visit_sequence = 1 THEN 1 END) as first_visit_exists,
        MAX(visit_sequence) as total_visit_sessions,
        
        -- Real-time intervention triggers
        CASE 
            WHEN SUM(is_browse_only_visit) >= 3 AND AVG(engagement_score) >= 60
            THEN 'High-Intent Non-Converter'
            WHEN SUM(is_browse_only_visit) >= 2 AND SUM(CASE WHEN exit_behavior = 'Abandoned Cart' THEN 1 ELSE 0 END) >= 1
            THEN 'Cart Abandoner'
            WHEN SUM(is_browse_only_visit) >= 2 AND AVG(time_on_site_seconds) >= 180
            THEN 'Engaged Browser'
            WHEN SUM(is_browse_only_visit) = 1 AND MAX(visit_sequence) = 1
            THEN 'First-Time Browser'
            ELSE 'Standard Visitor'
        END as behavior_segment
    FROM real_time_visit_behavior
    GROUP BY customer_id
),
intervention_system AS (
    SELECT 
        cbp.*,
        
        -- Real-time intervention recommendations
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter'
            THEN 'URGENT: Deploy exit-intent popup with 20% discount'
            WHEN behavior_segment = 'Cart Abandoner'
            THEN 'URGENT: Send cart abandonment email within 1 hour'
            WHEN behavior_segment = 'Engaged Browser'
            THEN 'MEDIUM: Show personalized product recommendations'
            WHEN behavior_segment = 'First-Time Browser'
            THEN 'LOW: Capture email with welcome offer'
            ELSE 'MONITOR: Standard experience'
        END as real_time_intervention,
        
        -- Intervention timing
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter'
            THEN 'Immediate (within 5 minutes)'
            WHEN behavior_segment = 'Cart Abandoner'
            THEN 'Within 1 hour'
            WHEN behavior_segment = 'Engaged Browser'
            THEN 'Within 24 hours'
            WHEN behavior_segment = 'First-Time Browser'
            THEN 'Within 3 days'
            ELSE 'No immediate intervention'
        END as intervention_timing,
        
        -- Personalization parameters
        CASE 
            WHEN max_funnel_progression >= 3
            THEN 'Product-specific offers, cart recovery'
            WHEN max_funnel_progression = 2
            THEN 'Category-based recommendations, social proof'
            WHEN avg_engagement_score >= 70
            THEN 'Premium products, exclusive offers'
            WHEN uses_mobile = 1 AND uses_desktop = 1
            THEN 'Cross-device experience optimization'
            ELSE 'General discovery content'
        END as personalization_strategy,
        
        -- Success probability scoring
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter' AND avg_engagement_score >= 80
            THEN 85  -- Very high conversion probability
            WHEN behavior_segment = 'Cart Abandoner' AND cart_abandonment_count = 1
            THEN 70  -- High conversion probability with right intervention
            WHEN behavior_segment = 'Engaged Browser' AND avg_time_on_site >= 300
            THEN 55  -- Medium conversion probability
            WHEN behavior_segment = 'First-Time Browser' AND avg_engagement_score >= 50
            THEN 35  -- Low but meaningful conversion probability
            ELSE 15  -- Low conversion probability
        END as conversion_probability_score,
        
        -- Investment justification
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter'
            THEN 'High ROI - Prevent imminent churn of engaged customers'
            WHEN behavior_segment = 'Cart Abandoner'
            THEN 'Medium ROI - Recover expressed purchase intent'
            WHEN behavior_segment = 'Engaged Browser'
            THEN 'Medium ROI - Convert high engagement to purchase'
            WHEN behavior_segment = 'First-Time Browser'
            THEN 'Low ROI - Long-term customer acquisition'
            ELSE 'Minimal ROI - Monitoring only'
        END as investment_justification
    FROM customer_behavior_patterns cbp
),
automated_campaign_triggers AS (
    SELECT 
        is_.*,
        
        -- Campaign automation triggers
        CASE 
            WHEN real_time_intervention LIKE 'URGENT:%' 
            THEN 'Auto-trigger immediate intervention campaign'
            WHEN real_time_intervention LIKE 'MEDIUM:%' 
            THEN 'Queue for next marketing automation cycle'
            WHEN real_time_intervention LIKE 'LOW:%' 
            THEN 'Add to nurture campaign sequence'
            ELSE 'No automated campaign needed'
        END as campaign_automation,
        
        -- Channel orchestration
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter'
            THEN 'Multi-channel: Website popup + Email + SMS (if opted in)'
            WHEN behavior_segment = 'Cart Abandoner'
            THEN 'Email + Push notification (if app user)'
            WHEN behavior_segment = 'Engaged Browser'
            THEN 'Email + Social media retargeting'
            WHEN behavior_segment = 'First-Time Browser'
            THEN 'Email capture + Display retargeting'
            ELSE 'Passive monitoring'
        END as channel_orchestration,
        
        -- Budget allocation per intervention
        CASE 
            WHEN conversion_probability_score >= 80 THEN '$15-25 per customer'
            WHEN conversion_probability_score >= 60 THEN '$8-15 per customer'
            WHEN conversion_probability_score >= 40 THEN '$3-8 per customer'
            WHEN conversion_probability_score >= 20 THEN '$1-3 per customer'
            ELSE '$0.50-1 per customer'
        END as intervention_budget,
        
        -- Performance tracking metrics
        CASE 
            WHEN behavior_segment = 'High-Intent Non-Converter'
            THEN 'Track: Popup engagement, email open rates, conversion within 24h'
            WHEN behavior_segment = 'Cart Abandoner'
            THEN 'Track: Email open/click rates, cart recovery rate, time to purchase'
            WHEN behavior_segment = 'Engaged Browser'
            THEN 'Track: Content engagement, email subscription rate, future visit behavior'
            WHEN behavior_segment = 'First-Time Browser'
            THEN 'Track: Email capture rate, return visit rate, long-term conversion'
            ELSE 'Track: Standard engagement metrics'
        END as success_tracking_metrics
    FROM intervention_system is_
)
SELECT 
    customer_id,
    browse_only_visits as count_no_trans,
    total_visits,
    behavior_segment,
    avg_engagement_score,
    conversion_probability_score,
    real_time_intervention,
    intervention_timing,
    personalization_strategy,
    campaign_automation,
    channel_orchestration,
    intervention_budget,
    investment_justification,
    success_tracking_metrics
FROM automated_campaign_triggers
WHERE browse_only_visits > 0
ORDER BY 
    conversion_probability_score DESC,
    CASE real_time_intervention 
        WHEN 'URGENT: Deploy exit-intent popup with 20% discount' THEN 1
        WHEN 'URGENT: Send cart abandonment email within 1 hour' THEN 2
        WHEN 'MEDIUM: Show personalized product recommendations' THEN 3
        WHEN 'LOW: Capture email with welcome offer' THEN 4
        ELSE 5
    END,
    browse_only_visits DESC;
```

#### 3. **Customer Segmentation and Lifetime Value Optimization**
```sql
-- "Create comprehensive customer segmentation and lifetime value optimization based on visit patterns"

WITH customer_visit_segmentation AS (
    SELECT 
        v.customer_id,
        COUNT(*) as total_visits,
        SUM(CASE WHEN t.transaction_id IS NULL THEN 1 ELSE 0 END) as non_transaction_visits,
        SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) as transaction_visits,
        COALESCE(SUM(t.amount), 0) as total_revenue,
        COALESCE(AVG(t.amount), 0) as avg_order_value,
        
        -- Calculate visit-to-conversion ratio
        CASE 
            WHEN COUNT(*) > 0 THEN 
                ROUND(SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2)
            ELSE 0
        END as conversion_rate,
        
        -- Behavioral segmentation factors
        CASE 
            WHEN SUM(CASE WHEN t.transaction_id IS NULL THEN 1 ELSE 0 END) >= 5 
            THEN 'High Frequency Browser'
            WHEN SUM(CASE WHEN t.transaction_id IS NULL THEN 1 ELSE 0 END) >= 3 
            THEN 'Medium Frequency Browser'
            WHEN SUM(CASE WHEN t.transaction_id IS NULL THEN 1 ELSE 0 END) >= 1 
            THEN 'Low Frequency Browser'
            ELSE 'Non-Browser'
        END as browsing_behavior,
        
        CASE 
            WHEN COALESCE(SUM(t.amount), 0) >= 1500 THEN 'High Value'
            WHEN COALESCE(SUM(t.amount), 0) >= 500 THEN 'Medium Value'
            WHEN COALESCE(SUM(t.amount), 0) > 0 THEN 'Low Value'
            ELSE 'No Value'
        END as value_tier,
        
        -- Customer lifecycle stage
        CASE 
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) = 0 
                 AND COUNT(*) >= 3 THEN 'Prospect - High Intent'
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) = 0 
                 AND COUNT(*) >= 1 THEN 'Prospect - Moderate Intent'
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) = 1 
                 AND COUNT(*) >= 4 THEN 'First-Time Buyer - High Engagement'
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) = 1 
            THEN 'First-Time Buyer'
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) >= 3 
            THEN 'Repeat Customer'
            WHEN SUM(CASE WHEN t.transaction_id IS NOT NULL THEN 1 ELSE 0 END) = 2 
            THEN 'Second-Time Buyer'
            ELSE 'Unclassified'
        END as lifecycle_stage
    FROM Visits v
    LEFT JOIN Transactions t ON v.visit_id = t.visit_id
    GROUP BY v.customer_id
),
advanced_customer_segmentation AS (
    SELECT 
        cvs.*,
        
        -- RFM-like analysis adapted for visit data
        CASE 
            WHEN total_visits >= 7 THEN 5  -- Very recent/frequent
            WHEN total_visits >= 5 THEN 4  -- Recent/frequent
            WHEN total_visits >= 3 THEN 3  -- Moderate
            WHEN total_visits >= 2 THEN 2  -- Low
            ELSE 1  -- Very low
        END as frequency_score,
        
        CASE 
            WHEN total_revenue >= 1500 THEN 5  -- Very high monetary
            WHEN total_revenue >= 800 THEN 4   -- High monetary
            WHEN total_revenue >= 400 THEN 3   -- Moderate monetary
            WHEN total_revenue > 0 THEN 2      -- Low monetary
            ELSE 1  -- No monetary
        END as monetary_score,
        
        -- Customer health score
        (CASE 
            WHEN total_visits >= 7 THEN 5
            WHEN total_visits >= 5 THEN 4
            WHEN total_visits >= 3 THEN 3
            WHEN total_visits >= 2 THEN 2
            ELSE 1
        END * 0.3) +  -- 30% weight on frequency
        (CASE 
            WHEN total_revenue >= 1500 THEN 5
            WHEN total_revenue >= 800 THEN 4
            WHEN total_revenue >= 400 THEN 3
            WHEN total_revenue > 0 THEN 2
            ELSE 1
        END * 0.4) +  -- 40% weight on monetary
        (CASE 
            WHEN conversion_rate >= 60 THEN 5
            WHEN conversion_rate >= 40 THEN 4
            WHEN conversion_rate >= 20 THEN 3
            WHEN conversion_rate > 0 THEN 2
            ELSE 1
        END * 0.3) as customer_health_score,  -- 30% weight on conversion
        
        -- Strategic customer segments
        CASE 
            WHEN total_revenue >= 1000 AND conversion_rate >= 50 
            THEN 'Champions'
            WHEN total_revenue >= 500 AND conversion_rate >= 30 
            THEN 'Loyal Customers'
            WHEN total_revenue >= 200 AND total_visits >= 5 
            THEN 'Potential Loyalists'
            WHEN conversion_rate >= 40 AND total_visits <= 3 
            THEN 'New Customers'
            WHEN total_revenue > 0 AND conversion_rate < 30 
            THEN 'Promising'
            WHEN non_transaction_visits >= 3 AND total_revenue = 0 
            THEN 'Need Attention'
            WHEN total_visits >= 2 AND total_revenue = 0 
            THEN 'About to Sleep'
            WHEN total_visits = 1 AND total_revenue = 0 
            THEN 'Lost Prospects'
            ELSE 'Cannot Lose Them'
        END as strategic_segment
    FROM customer_visit_segmentation cvs
),
lifetime_value_modeling AS (
    SELECT 
        acs.*,
        
        -- Predictive CLV calculation
        CASE 
            WHEN strategic_segment = 'Champions' 
            THEN total_revenue * 4.5 + (non_transaction_visits * 50)
            WHEN strategic_segment = 'Loyal Customers' 
            THEN total_revenue * 3.2 + (non_transaction_visits * 35)
            WHEN strategic_segment = 'Potential Loyalists' 
            THEN total_revenue * 2.8 + (non_transaction_visits * 40)
            WHEN strategic_segment = 'New Customers' 
            THEN total_revenue * 3.0 + (non_transaction_visits * 30)
            WHEN strategic_segment = 'Promising' 
            THEN total_revenue * 2.2 + (non_transaction_visits * 25)
            WHEN strategic_segment = 'Need Attention' 
            THEN (non_transaction_visits * 45) + 150  -- High browse intent value
            WHEN strategic_segment = 'About to Sleep' 
            THEN (non_transaction_visits * 25) + 75
            WHEN strategic_segment = 'Lost Prospects' 
            THEN 50  -- Minimal value but some potential
            ELSE total_revenue * 1.5
        END as predicted_clv_12months,
        
        -- Investment priority scoring
        CASE 
            WHEN strategic_segment = 'Champions' 
            THEN 95  -- Highest priority - protect and grow
            WHEN strategic_segment = 'Loyal Customers' 
            THEN 90  -- Very high priority - maintain loyalty
            WHEN strategic_segment = 'Need Attention' AND non_transaction_visits >= 4
            THEN 85  -- High priority - high intent browsers
            WHEN strategic_segment = 'Potential Loyalists' 
            THEN 80  -- High priority - growth opportunity
            WHEN strategic_segment = 'New Customers' 
            THEN 75  -- Medium-high priority - nurture conversion
            WHEN strategic_segment = 'Promising' 
            THEN 65  -- Medium priority - improve conversion
            WHEN strategic_segment = 'About to Sleep' 
            THEN 55  -- Medium-low priority - reactivation
            WHEN strategic_segment = 'Lost Prospects' 
            THEN 30  -- Low priority - limited intervention
            ELSE 40
        END as investment_priority_score,
        
        -- Churn risk assessment
        CASE 
            WHEN strategic_segment = 'About to Sleep' 
            THEN 'High Risk'
            WHEN strategic_segment = 'Lost Prospects' 
            THEN 'Already Churned'
            WHEN strategic_segment = 'Need Attention' 
            THEN 'Medium Risk'
            WHEN strategic_segment = 'Promising' AND conversion_rate < 20 
            THEN 'Medium Risk'
            WHEN strategic_segment IN ('Champions', 'Loyal Customers') 
            THEN 'Low Risk'
            ELSE 'Medium-Low Risk'
        END as churn_risk_level,
        
        -- Recommended intervention strategy
        CASE 
            WHEN strategic_segment = 'Champions' 
            THEN 'VIP Experience: Exclusive access, premium support, loyalty rewards'
            WHEN strategic_segment = 'Loyal Customers' 
            THEN 'Retention Focus: Cross-sell, upsell, referral programs'
            WHEN strategic_segment = 'Need Attention' 
            THEN 'Conversion Optimization: Targeted offers, personalized content'
            WHEN strategic_segment = 'Potential Loyalists' 
            THEN 'Loyalty Development: Progressive offers, engagement campaigns'
            WHEN strategic_segment = 'New Customers' 
            THEN 'Onboarding Excellence: Tutorial content, first purchase incentives'
            WHEN strategic_segment = 'Promising' 
            THEN 'Conversion Support: Simplified checkout, multiple payment options'
            WHEN strategic_segment = 'About to Sleep' 
            THEN 'Reactivation Campaign: Win-back offers, preference updates'
            WHEN strategic_segment = 'Lost Prospects' 
            THEN 'Minimal Touch: Quarterly newsletter, major sale notifications'
            ELSE 'Standard Engagement'
        END as intervention_strategy
    FROM advanced_customer_segmentation acs
),
portfolio_optimization AS (
    SELECT 
        lvm.*,
        
        -- Portfolio-level metrics
        COUNT(*) OVER() as total_customers_with_non_trans_visits,
        SUM(predicted_clv_12months) OVER() as total_predicted_portfolio_value,
        AVG(predicted_clv_12months) OVER() as avg_predicted_clv,
        
        -- Customer ranking within segment
        ROW_NUMBER() OVER (
            PARTITION BY strategic_segment 
            ORDER BY predicted_clv_12months DESC
        ) as segment_clv_rank,
        
        -- Investment allocation
        CASE 
            WHEN investment_priority_score >= 90 THEN 'Tier 1 - Premium Investment'
            WHEN investment_priority_score >= 75 THEN 'Tier 2 - High Investment'
            WHEN investment_priority_score >= 60 THEN 'Tier 3 - Standard Investment'
            WHEN investment_priority_score >= 45 THEN 'Tier 4 - Limited Investment'
            ELSE 'Tier 5 - Minimal Investment'
        END as investment_tier,
        
        -- Budget allocation percentage
        CASE 
            WHEN investment_priority_score >= 90 THEN '15-25% of customer marketing budget'
            WHEN investment_priority_score >= 75 THEN '8-15% of customer marketing budget'
            WHEN investment_priority_score >= 60 THEN '4-8% of customer marketing budget'
            WHEN investment_priority_score >= 45 THEN '2-4% of customer marketing budget'
            ELSE '0.5-2% of customer marketing budget'
        END as budget_allocation,
        
        -- Success metrics and KPIs
        CASE 
            WHEN strategic_segment = 'Need Attention' 
            THEN 'KPI: First purchase within 60 days, 25%+ conversion rate'
            WHEN strategic_segment = 'About to Sleep' 
            THEN 'KPI: Re-engagement within 30 days, any transaction'
            WHEN strategic_segment = 'Promising' 
            THEN 'KPI: 40%+ conversion rate improvement, 2nd purchase'
            WHEN strategic_segment = 'Champions' 
            THEN 'KPI: Maintain 80%+ satisfaction, increase purchase frequency'
            ELSE 'KPI: Improve engagement and conversion metrics'
        END as success_kpis
    FROM lifetime_value_modeling lvm
)
SELECT 
    customer_id,
    non_transaction_visits as count_no_trans,
    total_visits,
    conversion_rate,
    total_revenue,
    strategic_segment,
    customer_health_score,
    predicted_clv_12months,
    churn_risk_level,
    investment_priority_score,
    investment_tier,
    intervention_strategy,
    budget_allocation,
    success_kpis,
    segment_clv_rank,
    
    -- Portfolio context
    ROUND(predicted_clv_12months * 100.0 / SUM(predicted_clv_12months) OVER(), 2) as portfolio_value_percentage
FROM portfolio_optimization
WHERE non_transaction_visits > 0
ORDER BY 
    investment_priority_score DESC,
    predicted_clv_12months DESC,
    customer_id;
```

## 🔗 Related LeetCode Questions

1. **#1378 - Replace Employee ID With The Unique Identifier** (LEFT JOIN patterns)
2. **#1280 - Students and Examinations** (CROSS JOIN with LEFT JOIN)
3. **#1607 - Sellers With No Sales** (LEFT JOIN with NULL filtering)
4. **#1965 - Employees With Missing Information** (LEFT JOIN for missing data)
5. **#1795 - Rearrange Products Table** (Data transformation with JOINs)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **LEFT JOIN**: Preserves all records from the left table
2. **NULL Filtering**: WHERE condition to find missing relationships
3. **Aggregation**: COUNT() to summarize missing relationships
4. **GROUP BY**: Organizing results by key dimensions

### 🚀 **Amazon Interview Tips**
1. **Explain JOIN types**: "LEFT JOIN keeps all visits, even without transactions"
2. **Clarify NULL logic**: "WHERE t.visit_id IS NULL finds visits without transactions"
3. **Discuss alternatives**: "Could also use NOT EXISTS or NOT IN"
4. **Consider performance**: "LEFT JOIN with NULL filter is generally efficient"

### 🔧 **Common Patterns**
- LEFT JOIN for preserving all records from primary table
- IS NULL condition to identify missing relationships
- GROUP BY to aggregate missing relationship counts
- Alternative approaches with NOT EXISTS or NOT IN

### ⚠️ **Common Mistakes to Avoid**
1. **Using INNER JOIN** (would exclude visits without transactions)
2. **Forgetting GROUP BY** (would return individual visits, not counts)
3. **NOT IN with NULLs** (can return unexpected results with NULL values)
4. **Wrong NULL condition** (using = NULL instead of IS NULL)

### 🔍 **Performance Considerations**
- LEFT JOIN is generally efficient with proper indexing
- INDEX on join columns (visit_id) improves performance
- NOT EXISTS can be faster than NOT IN for large datasets
- Consider query plan analysis for optimization

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding customer browsing behavior improves experience
- **Data-Driven Decisions**: Analytics on non-converting visits guide optimization strategies
- **Operational Excellence**: Systematic identification of conversion opportunities
- **Innovation**: Advanced analytics enable personalized intervention strategies

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find customers who visited multiple times but never bought**
2. **Calculate average time between visits for non-purchasing customers**
3. **Identify products viewed but never purchased**
4. **Find customer segments with highest browse-to-purchase ratios**

Remember: Browse-without-purchase analysis is essential for Amazon's conversion optimization, customer retention strategies, personalization systems, and revenue maximization across all e-commerce and digital service platforms!