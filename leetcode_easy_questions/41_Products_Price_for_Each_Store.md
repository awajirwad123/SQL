# LeetCode Easy #1777: Product's Price for Each Store

## 📋 Problem Statement

Write a SQL query to find the **price** of each product in each store.

Return the result table in **any order**.

## 🗄️ Table Schema

**Products Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| store       | enum    |
| price       | int     |
+-------------+---------+
```
- (product_id, store) is the primary key for this table.
- store is an ENUM of type ('store1', 'store2', 'store3') where each represents the store this product is available at.
- price is the price of the product at this store.

## 📊 Sample Data

**Products Table:**
| product_id | store  | price |
|------------|--------|-------|
| 0          | store1 | 95    |
| 0          | store3 | 105   |
| 0          | store2 | 100   |
| 1          | store1 | 70    |
| 1          | store3 | 80    |

**Expected Output:**
| product_id | store1 | store2 | store3 |
|------------|--------|--------|--------|
| 0          | 95     | 100    | 105    |
| 1          | 70     | null   | 80     |

**Explanation:**
- Product 0 costs 95 at store1, 100 at store2, and 105 at store3.
- Product 1 costs 70 at store1, null at store2 (not available), and 80 at store3.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Convert vertical data (multiple rows per product) to horizontal (one row per product)
- Need to PIVOT data from row format to column format
- Each store becomes a separate column
- Use aggregation with CASE statements for pivoting

### 2. **Key Insights**
- Classic PIVOT operation or manual pivoting with CASE
- GROUP BY product_id to aggregate stores into columns
- Use SUM/MAX with CASE WHEN to extract store-specific prices
- NULL values for stores where product isn't available

### 3. **Interview Discussion Points**
- "This is a pivot operation, converting rows to columns"
- "Use CASE WHEN with aggregation to create store columns"
- "GROUP BY product_id to combine store data per product"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Pivot Requirements
```sql
-- Need to convert store values to columns
-- Each store becomes a separate column
```

### Step 2: Use CASE WHEN for Conditional Aggregation
```sql
-- Extract price for each specific store
CASE WHEN store = 'store1' THEN price END as store1_price
```

### Step 3: Apply Aggregation Function
```sql
-- Use SUM or MAX to get single value per product per store
SUM(CASE WHEN store = 'store1' THEN price END) as store1
```

### Step 4: Group by Product
```sql
-- Group by product_id to aggregate stores
GROUP BY product_id
```

## ✅ Optimized SQL Solution

**Solution 1: Manual PIVOT with CASE WHEN**
```sql
SELECT 
    product_id,
    SUM(CASE WHEN store = 'store1' THEN price END) as store1,
    SUM(CASE WHEN store = 'store2' THEN price END) as store2,
    SUM(CASE WHEN store = 'store3' THEN price END) as store3
FROM Products
GROUP BY product_id;
```

### Alternative Solutions

**Solution 2: Using MAX Instead of SUM**
```sql
SELECT 
    product_id,
    MAX(CASE WHEN store = 'store1' THEN price END) as store1,
    MAX(CASE WHEN store = 'store2' THEN price END) as store2,
    MAX(CASE WHEN store = 'store3' THEN price END) as store3
FROM Products
GROUP BY product_id;
```

**Solution 3: MySQL PIVOT Syntax (if supported)**
```sql
SELECT product_id, store1, store2, store3
FROM Products
PIVOT (
    MAX(price)
    FOR store IN ('store1' as store1, 'store2' as store2, 'store3' as store3)
);
```

**Solution 4: Subquery Approach**
```sql
SELECT 
    p1.product_id,
    (SELECT price FROM Products p2 WHERE p2.product_id = p1.product_id AND p2.store = 'store1') as store1,
    (SELECT price FROM Products p3 WHERE p3.product_id = p1.product_id AND p3.store = 'store2') as store2,
    (SELECT price FROM Products p4 WHERE p4.product_id = p1.product_id AND p4.store = 'store3') as store3
FROM Products p1
GROUP BY p1.product_id;
```

**Solution 5: Comprehensive Multi-Store Retail Analytics System**
```sql
WITH product_store_analysis AS (
    SELECT 
        product_id,
        store,
        price,
        
        -- Price tier classification
        CASE 
            WHEN price >= 100 THEN 'Premium Price'
            WHEN price >= 80 THEN 'Mid-Range Price'
            WHEN price >= 60 THEN 'Standard Price'
            ELSE 'Budget Price'
        END as price_tier,
        
        -- Store performance simulation
        CASE 
            WHEN store = 'store1' THEN 'Flagship Store'
            WHEN store = 'store2' THEN 'Regional Store'
            ELSE 'Outlet Store'
        END as store_type,
        
        -- Market positioning simulation
        CASE 
            WHEN store = 'store1' AND price >= 90 THEN 'Premium Market Position'
            WHEN store = 'store2' AND price BETWEEN 70 AND 100 THEN 'Mid-Market Position'
            WHEN store = 'store3' AND price <= 85 THEN 'Value Market Position'
            ELSE 'Standard Market Position'
        END as market_position,
        
        -- Competitive analysis
        CASE 
            WHEN price >= 90 THEN 'High Competition Segment'
            WHEN price >= 70 THEN 'Moderate Competition Segment'
            ELSE 'Low Competition Segment'
        END as competition_level,
        
        -- Product category simulation
        CASE 
            WHEN product_id % 5 = 0 THEN 'Electronics'
            WHEN product_id % 5 = 1 THEN 'Clothing'
            WHEN product_id % 5 = 2 THEN 'Home & Garden'
            WHEN product_id % 5 = 3 THEN 'Sports & Outdoors'
            ELSE 'Books & Media'
        END as product_category,
        
        -- Inventory velocity simulation
        CASE 
            WHEN store = 'store1' AND price <= 80 THEN 'Fast Moving'
            WHEN store = 'store2' AND price BETWEEN 60 AND 90 THEN 'Steady Moving'
            WHEN store = 'store3' AND price >= 85 THEN 'Slow Moving'
            ELSE 'Standard Moving'
        END as inventory_velocity,
        
        -- Profit margin simulation
        CASE 
            WHEN price >= 100 THEN 'High Margin'
            WHEN price >= 80 THEN 'Good Margin'
            WHEN price >= 60 THEN 'Standard Margin'
            ELSE 'Low Margin'
        END as profit_margin_category
    FROM Products
),
multi_store_pricing_analysis AS (
    SELECT 
        product_id,
        product_category,
        
        -- Core pricing pivot
        SUM(CASE WHEN store = 'store1' THEN price END) as store1,
        SUM(CASE WHEN store = 'store2' THEN price END) as store2,
        SUM(CASE WHEN store = 'store3' THEN price END) as store3,
        
        -- Store availability
        COUNT(CASE WHEN store = 'store1' THEN 1 END) as store1_available,
        COUNT(CASE WHEN store = 'store2' THEN 1 END) as store2_available,
        COUNT(CASE WHEN store = 'store3' THEN 1 END) as store3_available,
        COUNT(*) as total_stores_available,
        
        -- Price analysis
        MIN(price) as min_price,
        MAX(price) as max_price,
        AVG(price) as avg_price,
        ROUND(STDDEV(price), 2) as price_variance,
        MAX(price) - MIN(price) as price_spread,
        
        -- Store type distribution
        COUNT(CASE WHEN store_type = 'Flagship Store' THEN 1 END) as flagship_presence,
        COUNT(CASE WHEN store_type = 'Regional Store' THEN 1 END) as regional_presence,
        COUNT(CASE WHEN store_type = 'Outlet Store' THEN 1 END) as outlet_presence,
        
        -- Price tier distribution
        COUNT(CASE WHEN price_tier = 'Premium Price' THEN 1 END) as premium_stores,
        COUNT(CASE WHEN price_tier = 'Mid-Range Price' THEN 1 END) as midrange_stores,
        COUNT(CASE WHEN price_tier = 'Standard Price' THEN 1 END) as standard_stores,
        COUNT(CASE WHEN price_tier = 'Budget Price' THEN 1 END) as budget_stores,
        
        -- Market position analysis
        COUNT(CASE WHEN market_position = 'Premium Market Position' THEN 1 END) as premium_positions,
        COUNT(CASE WHEN market_position = 'Mid-Market Position' THEN 1 END) as midmarket_positions,
        COUNT(CASE WHEN market_position = 'Value Market Position' THEN 1 END) as value_positions,
        
        -- Inventory and competition insights
        COUNT(CASE WHEN inventory_velocity = 'Fast Moving' THEN 1 END) as fast_moving_stores,
        COUNT(CASE WHEN competition_level = 'High Competition Segment' THEN 1 END) as high_competition_stores,
        COUNT(CASE WHEN profit_margin_category = 'High Margin' THEN 1 END) as high_margin_stores,
        
        -- Pricing strategy assessment
        CASE 
            WHEN COUNT(*) = 3 AND STDDEV(price) <= 10 THEN 'Uniform Pricing Strategy'
            WHEN COUNT(*) = 3 AND MAX(price) - MIN(price) >= 20 THEN 'Differentiated Pricing Strategy'
            WHEN COUNT(*) = 3 THEN 'Moderate Pricing Strategy'
            WHEN COUNT(*) = 2 THEN 'Limited Availability Strategy'
            ELSE 'Exclusive Availability Strategy'
        END as pricing_strategy,
        
        -- Market coverage assessment
        CASE 
            WHEN COUNT(*) = 3 THEN 'Full Market Coverage'
            WHEN COUNT(*) = 2 THEN 'Partial Market Coverage'
            ELSE 'Limited Market Coverage'
        END as market_coverage,
        
        -- Competitive positioning
        CASE 
            WHEN AVG(price) >= 90 AND COUNT(*) >= 2 THEN 'Premium Competitive Position'
            WHEN AVG(price) BETWEEN 70 AND 90 AND COUNT(*) = 3 THEN 'Strong Competitive Position'
            WHEN AVG(price) < 70 AND COUNT(*) >= 2 THEN 'Value Competitive Position'
            ELSE 'Niche Competitive Position'
        END as competitive_positioning,
        
        -- Revenue optimization insights
        CASE 
            WHEN COUNT(*) = 3 AND premium_stores >= 2 THEN 'Revenue Optimization: Premium focus + high-margin emphasis + luxury positioning'
            WHEN COUNT(*) = 3 AND midrange_stores >= 2 THEN 'Revenue Optimization: Volume focus + market penetration + broad appeal'
            WHEN COUNT(*) = 3 AND standard_stores >= 2 THEN 'Revenue Optimization: Value focus + competitive pricing + market share'
            WHEN COUNT(*) <= 2 THEN 'Revenue Optimization: Expand availability + market development + coverage improvement'
            ELSE 'Revenue Optimization: Balanced approach + diversified strategy + flexibility'
        END as revenue_optimization_strategy
    FROM product_store_analysis
    GROUP BY product_id, product_category
),
retail_intelligence_insights AS (
    SELECT 
        mspa.*,
        
        -- Retail strategy recommendations
        CASE 
            WHEN pricing_strategy = 'Uniform Pricing Strategy' AND competitive_positioning = 'Premium Competitive Position'
            THEN 'Retail Strategy: Premium brand consistency + luxury market focus + high-value customer targeting + quality emphasis'
            WHEN pricing_strategy = 'Differentiated Pricing Strategy' AND market_coverage = 'Full Market Coverage'
            THEN 'Retail Strategy: Market segmentation + tiered pricing + diverse customer base + flexible positioning'
            WHEN market_coverage = 'Limited Market Coverage' AND avg_price >= 80
            THEN 'Retail Strategy: Selective distribution + premium positioning + exclusive availability + scarcity value'
            WHEN competitive_positioning = 'Value Competitive Position'
            THEN 'Retail Strategy: Volume sales + competitive pricing + market penetration + value proposition'
            ELSE 'Retail Strategy: Balanced approach + flexible pricing + market responsive + adaptive positioning'
        END as retail_strategy_recommendation,
        
        -- Inventory management insights
        CASE 
            WHEN fast_moving_stores >= 2 AND high_margin_stores >= 1
            THEN 'Inventory: High turnover + premium stock + demand-driven + profit optimization + fast replenishment'
            WHEN total_stores_available = 3 AND price_variance <= 15
            THEN 'Inventory: Consistent stock + uniform demand + stable turnover + predictable patterns + standard management'
            WHEN market_coverage = 'Partial Market Coverage'
            THEN 'Inventory: Selective stock + targeted demand + focused turnover + specialized patterns + strategic management'
            ELSE 'Inventory: Standard management + regular turnover + balanced demand + flexible patterns + adaptive approach'
        END as inventory_management_strategy,
        
        -- Customer segmentation insights
        CASE 
            WHEN premium_positions >= 2 AND max_price >= 100
            THEN 'Customer Segments: Luxury buyers + premium customers + high-value clients + quality seekers + brand conscious'
            WHEN midmarket_positions >= 2 AND avg_price BETWEEN 70 AND 90
            THEN 'Customer Segments: Middle-class families + value-conscious buyers + quality-price balanced + mainstream customers'
            WHEN value_positions >= 2 AND min_price <= 70
            THEN 'Customer Segments: Budget shoppers + price-sensitive customers + value seekers + cost-conscious buyers'
            WHEN total_stores_available = 1
            THEN 'Customer Segments: Niche customers + specialized buyers + exclusive clientele + targeted market'
            ELSE 'Customer Segments: Diverse customer base + mixed segments + broad appeal + varied preferences'
        END as customer_segmentation_insights,
        
        -- Channel optimization recommendations
        CASE 
            WHEN flagship_presence = 1 AND outlet_presence = 1 AND premium_stores >= 1
            THEN 'Channel Optimization: Premium flagship + value outlet + market range coverage + customer choice + positioning flexibility'
            WHEN regional_presence = 1 AND total_stores_available >= 2
            THEN 'Channel Optimization: Regional focus + local market + community presence + accessibility + convenience'
            WHEN total_stores_available = 3 AND price_spread >= 25
            THEN 'Channel Optimization: Full spectrum + price ladder + market segmentation + choice diversity + comprehensive coverage'
            ELSE 'Channel Optimization: Strategic selection + targeted presence + focused coverage + efficient distribution + market alignment'
        END as channel_optimization,
        
        -- Pricing optimization recommendations
        CASE 
            WHEN price_variance >= 20 AND total_stores_available = 3
            THEN 'Pricing Optimization: Dynamic pricing + store-specific strategy + market-based adjustment + competitive response + value capture'
            WHEN price_variance <= 10 AND avg_price >= 85
            THEN 'Pricing Optimization: Premium consistency + brand strength + value perception + quality positioning + margin protection'
            WHEN competitive_positioning = 'Value Competitive Position'
            THEN 'Pricing Optimization: Competitive pricing + market penetration + volume focus + cost leadership + accessibility'
            ELSE 'Pricing Optimization: Balanced approach + market responsive + flexible strategy + adaptive pricing + optimal positioning'
        END as pricing_optimization,
        
        -- Business growth opportunities
        CASE 
            WHEN market_coverage = 'Limited Market Coverage' AND competitive_positioning LIKE '%Premium%'
            THEN 'Growth Opportunities: Market expansion + premium replication + luxury market development + exclusive distribution + high-value growth'
            WHEN total_stores_available = 3 AND high_margin_stores >= 2
            THEN 'Growth Opportunities: Margin expansion + premium enhancement + value proposition + profit maximization + quality focus'
            WHEN fast_moving_stores >= 1 AND market_coverage = 'Partial Market Coverage'
            THEN 'Growth Opportunities: Volume expansion + market penetration + availability increase + demand capture + scale growth'
            ELSE 'Growth Opportunities: Strategic development + market optimization + positioning enhancement + competitive strengthening + sustainable growth'
        END as growth_opportunities
    FROM multi_store_pricing_analysis mspa
)
SELECT 
    product_id,
    store1,
    store2,
    store3,
    product_category,
    pricing_strategy,
    market_coverage,
    competitive_positioning,
    retail_strategy_recommendation,
    inventory_management_strategy,
    customer_segmentation_insights,
    channel_optimization,
    pricing_optimization,
    growth_opportunities,
    
    -- Key metrics
    total_stores_available,
    min_price,
    max_price,
    avg_price,
    price_spread,
    price_variance,
    
    -- Store analysis
    premium_stores,
    midrange_stores,
    standard_stores,
    fast_moving_stores,
    high_margin_stores
FROM retail_intelligence_insights
ORDER BY 
    CASE 
        WHEN competitive_positioning = 'Premium Competitive Position' THEN 1
        WHEN competitive_positioning = 'Strong Competitive Position' THEN 2
        WHEN competitive_positioning = 'Value Competitive Position' THEN 3
        ELSE 4
    END,
    avg_price DESC,
    product_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Multi-Store and Multi-Channel Pricing Strategy Platform**
```sql
-- "Build comprehensive multi-channel pricing and inventory optimization system for Amazon's retail ecosystem"

WITH amazon_multi_channel_analysis AS (
    SELECT 
        product_id,
        store,
        price,
        
        -- Amazon channel mapping
        CASE 
            WHEN store = 'store1' THEN 'Amazon.com Direct'
            WHEN store = 'store2' THEN 'Amazon Fresh/Whole Foods'
            WHEN store = 'store3' THEN 'Third-Party Marketplace'
        END as amazon_channel,
        
        -- Amazon business model simulation
        CASE 
            WHEN store = 'store1' THEN 'First-Party Retail'
            WHEN store = 'store2' THEN 'Amazon Retail Subsidiary'
            WHEN store = 'store3' THEN 'Third-Party Seller'
        END as business_model,
        
        -- Fulfillment method simulation
        CASE 
            WHEN store = 'store1' AND price >= 90 THEN 'Amazon Prime Same-Day'
            WHEN store = 'store1' THEN 'Amazon Prime 2-Day'
            WHEN store = 'store2' THEN 'Amazon Fresh Delivery'
            WHEN store = 'store3' AND price <= 80 THEN 'FBA Standard'
            WHEN store = 'store3' THEN 'FBA Premium'
        END as fulfillment_method,
        
        -- Customer segment targeting
        CASE 
            WHEN store = 'store1' AND price >= 100 THEN 'Prime Premium Customers'
            WHEN store = 'store1' THEN 'Prime Standard Customers'
            WHEN store = 'store2' THEN 'Fresh/Grocery Customers'
            WHEN store = 'store3' AND price <= 70 THEN 'Value-Seeking Customers'
            WHEN store = 'store3' THEN 'Marketplace Shoppers'
        END as customer_segment,
        
        -- Amazon fee structure simulation
        CASE 
            WHEN store = 'store1' THEN price * 0.15  -- Amazon retail margin
            WHEN store = 'store2' THEN price * 0.12  -- Fresh/WF margin
            WHEN store = 'store3' THEN price * 0.08  -- Marketplace fee
        END as amazon_fee,
        
        -- Competition analysis within Amazon ecosystem
        CASE 
            WHEN store = 'store1' AND price >= 90 THEN 'Premium Amazon Direct Competition'
            WHEN store = 'store1' THEN 'Standard Amazon Direct Competition'
            WHEN store = 'store2' THEN 'Fresh/Grocery Competition'
            WHEN store = 'store3' THEN 'Marketplace Competition'
        END as competition_context,
        
        -- Prime benefit eligibility
        CASE 
            WHEN store = 'store1' THEN 'Prime Free Shipping + Prime Benefits + Amazon Choice'
            WHEN store = 'store2' THEN 'Prime Fresh Delivery + Same-Day Available + Grocery Benefits'
            WHEN store = 'store3' AND price >= 75 THEN 'Prime Eligible + FBA Benefits + Marketplace Prime'
            WHEN store = 'store3' THEN 'Standard Marketplace + Basic Benefits'
        END as prime_benefits
    FROM Products
),
amazon_pricing_intelligence AS (
    SELECT 
        product_id,
        
        -- Core pricing pivot for Amazon channels
        SUM(CASE WHEN amazon_channel = 'Amazon.com Direct' THEN price END) as amazon_direct,
        SUM(CASE WHEN amazon_channel = 'Amazon Fresh/Whole Foods' THEN price END) as amazon_fresh,
        SUM(CASE WHEN amazon_channel = 'Third-Party Marketplace' THEN price END) as marketplace,
        
        -- Channel availability
        COUNT(CASE WHEN amazon_channel = 'Amazon.com Direct' THEN 1 END) as direct_available,
        COUNT(CASE WHEN amazon_channel = 'Amazon Fresh/Whole Foods' THEN 1 END) as fresh_available,
        COUNT(CASE WHEN amazon_channel = 'Third-Party Marketplace' THEN 1 END) as marketplace_available,
        
        -- Business model analysis
        COUNT(CASE WHEN business_model = 'First-Party Retail' THEN 1 END) as first_party_presence,
        COUNT(CASE WHEN business_model = 'Third-Party Seller' THEN 1 END) as third_party_presence,
        
        -- Customer segment coverage
        COUNT(CASE WHEN customer_segment LIKE '%Prime Premium%' THEN 1 END) as premium_customer_coverage,
        COUNT(CASE WHEN customer_segment LIKE '%Prime Standard%' THEN 1 END) as standard_customer_coverage,
        COUNT(CASE WHEN customer_segment LIKE '%Value-Seeking%' THEN 1 END) as value_customer_coverage,
        
        -- Fulfillment method distribution
        COUNT(CASE WHEN fulfillment_method LIKE '%Same-Day%' THEN 1 END) as same_day_fulfillment,
        COUNT(CASE WHEN fulfillment_method LIKE '%Prime%' THEN 1 END) as prime_fulfillment,
        COUNT(CASE WHEN fulfillment_method LIKE '%FBA%' THEN 1 END) as fba_fulfillment,
        
        -- Revenue and fee analysis
        SUM(amazon_fee) as total_amazon_revenue,
        AVG(amazon_fee) as avg_amazon_revenue_per_channel,
        MAX(amazon_fee) as highest_channel_revenue,
        
        -- Price optimization insights
        MIN(price) as min_channel_price,
        MAX(price) as max_channel_price,
        AVG(price) as avg_channel_price,
        MAX(price) - MIN(price) as channel_price_spread,
        
        -- Amazon ecosystem strategy assessment
        CASE 
            WHEN COUNT(*) = 3 AND SUM(CASE WHEN amazon_channel = 'Amazon.com Direct' THEN price END) >= 
                                   SUM(CASE WHEN amazon_channel = 'Third-Party Marketplace' THEN price END)
            THEN 'Premium Direct Strategy'
            WHEN COUNT(*) = 3 AND SUM(CASE WHEN amazon_channel = 'Third-Party Marketplace' THEN price END) < 
                                   SUM(CASE WHEN amazon_channel = 'Amazon.com Direct' THEN price END)
            THEN 'Competitive Marketplace Strategy'
            WHEN COUNT(*) = 3 THEN 'Balanced Multi-Channel Strategy'
            WHEN direct_available = 1 AND fresh_available = 1 THEN 'Amazon Integrated Strategy'
            WHEN marketplace_available = 1 THEN 'Marketplace-Only Strategy'
            ELSE 'Limited Channel Strategy'
        END as amazon_ecosystem_strategy,
        
        -- Channel conflict analysis
        CASE 
            WHEN COUNT(*) >= 2 AND MAX(price) - MIN(price) >= 20 
            THEN 'High Channel Conflict Risk'
            WHEN COUNT(*) >= 2 AND MAX(price) - MIN(price) BETWEEN 10 AND 20 
            THEN 'Moderate Channel Conflict Risk'
            WHEN COUNT(*) >= 2 AND MAX(price) - MIN(price) < 10 
            THEN 'Low Channel Conflict Risk'
            ELSE 'No Channel Conflict'
        END as channel_conflict_risk
    FROM amazon_multi_channel_analysis
    GROUP BY product_id
),
amazon_strategic_recommendations AS (
    SELECT 
        api.*,
        
        -- Amazon Leadership Principles application
        CASE 
            WHEN amazon_ecosystem_strategy = 'Premium Direct Strategy' AND premium_customer_coverage > 0
            THEN 'Customer Obsession: Premium customer experience + direct relationship + high-value service + exclusive benefits'
            WHEN amazon_ecosystem_strategy = 'Balanced Multi-Channel Strategy' AND COUNT(*) = 3
            THEN 'Customer Obsession: Comprehensive choice + channel preference + accessibility + convenience optimization'
            WHEN marketplace_available = 1 AND value_customer_coverage > 0
            THEN 'Customer Obsession: Value accessibility + price competitiveness + marketplace choice + cost-conscious service'
            ELSE 'Customer Obsession: Channel-specific optimization + targeted service + customer preference alignment'
        END as customer_obsession_application,
        
        -- Ownership in channel management
        CASE 
            WHEN first_party_presence > 0 AND third_party_presence > 0
            THEN 'Ownership: Full ecosystem responsibility + channel coordination + conflict resolution + optimization balance'
            WHEN first_party_presence > 0
            THEN 'Ownership: Direct retail responsibility + inventory management + customer experience + margin optimization'
            WHEN third_party_presence > 0
            THEN 'Ownership: Marketplace facilitation + seller support + platform excellence + fee optimization'
            ELSE 'Ownership: Channel-specific accountability + focused responsibility + specialized management'
        END as ownership_channel_management,
        
        -- Invent and Simplify opportunities
        CASE 
            WHEN COUNT(*) = 3 AND channel_conflict_risk = 'High Channel Conflict Risk'
            THEN 'Invent and Simplify: Unified pricing system + dynamic optimization + conflict resolution + simplified management'
            WHEN same_day_fulfillment > 0 AND prime_fulfillment > 0
            THEN 'Invent and Simplify: Integrated fulfillment + logistics optimization + delivery innovation + seamless experience'
            WHEN total_amazon_revenue >= 50
            THEN 'Invent and Simplify: Revenue optimization + automated pricing + intelligent recommendations + streamlined operations'
            ELSE 'Invent and Simplify: Channel efficiency + process optimization + simplified coordination + enhanced automation'
        END as invent_simplify_opportunities,
        
        -- Think Big vision for multi-channel retail
        CASE 
            WHEN amazon_ecosystem_strategy = 'Balanced Multi-Channel Strategy' AND total_amazon_revenue >= 60
            THEN 'Think Big: Omnichannel retail leadership + global marketplace + integrated ecosystem + future commerce'
            WHEN premium_customer_coverage > 0 AND same_day_fulfillment > 0
            THEN 'Think Big: Premium service excellence + logistics innovation + customer experience revolution + market leadership'
            WHEN marketplace_available = 1 AND fba_fulfillment > 0
            THEN 'Think Big: Marketplace ecosystem expansion + seller empowerment + global commerce + platform evolution'
            ELSE 'Think Big: Channel innovation + retail transformation + customer-centric commerce + strategic expansion'
        END as think_big_vision,
        
        -- Bias for Action in pricing and channel decisions
        CASE 
            WHEN channel_conflict_risk = 'High Channel Conflict Risk'
            THEN 'Bias for Action: Immediate pricing optimization + conflict resolution + rapid adjustment + proactive management'
            WHEN channel_price_spread >= 25
            THEN 'Bias for Action: Dynamic pricing implementation + real-time optimization + competitive response + agile adjustments'
            WHEN amazon_ecosystem_strategy = 'Limited Channel Strategy'
            THEN 'Bias for Action: Channel expansion + opportunity capture + market development + rapid deployment'
            ELSE 'Bias for Action: Continuous optimization + performance monitoring + incremental improvements + responsive adjustments'
        END as bias_for_action_application
    FROM amazon_pricing_intelligence api
)
SELECT 
    product_id,
    amazon_direct,
    amazon_fresh,
    marketplace,
    amazon_ecosystem_strategy,
    channel_conflict_risk,
    customer_obsession_application,
    ownership_channel_management,
    invent_simplify_opportunities,
    think_big_vision,
    bias_for_action_application,
    
    -- Channel availability
    direct_available,
    fresh_available,
    marketplace_available,
    
    -- Financial metrics
    total_amazon_revenue,
    avg_amazon_revenue_per_channel,
    min_channel_price,
    max_channel_price,
    channel_price_spread,
    
    -- Customer and fulfillment coverage
    premium_customer_coverage,
    standard_customer_coverage,
    value_customer_coverage,
    same_day_fulfillment,
    prime_fulfillment,
    fba_fulfillment
FROM amazon_strategic_recommendations
ORDER BY 
    total_amazon_revenue DESC,
    CASE amazon_ecosystem_strategy
        WHEN 'Premium Direct Strategy' THEN 1
        WHEN 'Balanced Multi-Channel Strategy' THEN 2
        WHEN 'Amazon Integrated Strategy' THEN 3
        WHEN 'Competitive Marketplace Strategy' THEN 4
        ELSE 5
    END,
    product_id;
```

I'll create one more brief extension and then move to the final questions to complete the collection:

#### 2. **Real-time Dynamic Pricing and Competitive Intelligence**
```sql
-- "Real-time pricing optimization with competitive analysis and dynamic adjustment system"

WITH real_time_pricing_intelligence AS (
    SELECT 
        product_id,
        SUM(CASE WHEN store = 'store1' THEN price END) as store1,
        SUM(CASE WHEN store = 'store2' THEN price END) as store2,
        SUM(CASE WHEN store = 'store3' THEN price END) as store3,
        
        -- Real-time pricing alerts
        CASE 
            WHEN MAX(price) - MIN(price) >= 30 THEN 'ALERT: High price variance detected'
            WHEN COUNT(*) = 1 THEN 'ALERT: Limited availability detected'
            WHEN AVG(price) >= 100 THEN 'ALERT: Premium pricing detected'
            ELSE 'NORMAL: Standard pricing pattern'
        END as pricing_alert,
        
        -- Dynamic pricing recommendations
        CASE 
            WHEN MAX(price) - MIN(price) >= 25 
            THEN 'DYNAMIC: Implement price harmonization + competitive adjustment + market alignment'
            WHEN COUNT(*) < 3 
            THEN 'DYNAMIC: Expand availability + market penetration + channel optimization'
            WHEN AVG(price) >= 90 
            THEN 'DYNAMIC: Premium strategy + value justification + luxury positioning'
            ELSE 'DYNAMIC: Maintain current strategy + monitor competition + optimize margins'
        END as dynamic_pricing_recommendation,
        
        -- Competitive positioning score
        (CASE WHEN COUNT(*) = 3 THEN 30 ELSE COUNT(*) * 8 END) +
        (CASE WHEN AVG(price) >= 90 THEN 25 WHEN AVG(price) >= 70 THEN 15 ELSE 5 END) +
        (CASE WHEN MAX(price) - MIN(price) <= 15 THEN 20 ELSE 10 END) as competitive_score
    FROM Products
    GROUP BY product_id
)
SELECT 
    product_id,
    store1,
    store2,
    store3,
    pricing_alert,
    dynamic_pricing_recommendation,
    competitive_score
FROM real_time_pricing_intelligence
ORDER BY competitive_score DESC;
```

## 🔗 Related LeetCode Questions

1. **#1795 - Rearrange Products Table** (Data transformation and pivoting)
2. **#1757 - Recyclable and Low Fat Products** (Product filtering operations)
3. **#1789 - Primary Department for Each Employee** (Employee-department relationships)
4. **#1741 - Find Total Time Spent by Each Employee** (Aggregation with GROUP BY)
5. **#1683 - Invalid Tweets** (Simple data validation and filtering)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **PIVOT Operation**: Convert rows to columns using CASE WHEN with aggregation
2. **Conditional Aggregation**: SUM/MAX with CASE WHEN for selective data extraction
3. **GROUP BY**: Essential for aggregating related data into single rows
4. **NULL Handling**: Missing store data appears as NULL in result columns

### 🚀 **Amazon Interview Tips**
1. **Explain pivot concept**: "Converting vertical data (rows) to horizontal (columns)"
2. **Discuss aggregation choice**: "SUM or MAX work since only one price per product per store"
3. **Address NULL values**: "NULL indicates product not available in that store"
4. **Consider scalability**: "CASE WHEN approach scales to any number of stores"

### 🔧 **Common Patterns**
- CASE WHEN with aggregation functions for pivoting
- GROUP BY for combining related rows
- Conditional aggregation for selective data extraction
- NULL handling in pivot operations

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting GROUP BY** (would produce incorrect aggregation)
2. **Using wrong aggregation function** (COUNT instead of SUM/MAX)
3. **Hardcoding store names** (not scalable to dynamic stores)
4. **Not handling NULL values properly** (missing stores should show NULL)

### 🔍 **Performance Considerations**
- INDEX on (product_id, store) for efficient grouping
- Consider pivoting at application level for very wide results
- CASE WHEN operations are generally efficient
- Large pivots might benefit from dynamic SQL generation

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Multi-channel pricing serves diverse customer preferences
- **Ownership**: Complete responsibility for pricing strategy across all channels
- **Think Big**: Scale pricing intelligence across Amazon's vast product catalog
- **Deliver Results**: Optimize pricing for revenue and customer satisfaction

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Add a fourth store column and handle dynamic store additions**
2. **Calculate price differences between stores for each product**
3. **Find products with maximum price variance across stores**
4. **Create a summary showing average price per store across all products**

Remember: Pivot operations are essential for Amazon's multi-channel retail analytics, enabling comprehensive pricing strategies across direct sales, marketplace, and subsidiary channels like Whole Foods and Amazon Fresh!