# LeetCode Easy #1795: Rearrange Products Table

## 📋 Problem Statement

Write a SQL query to **rearrange** the Products table so that each row has **(product_id, price)** for each of the three stores. Each product in the original Products table is sold in one of the three stores: store1, store2, or store3.

Return the result table in **any order**.

## 🗄️ Table Schema

**Products Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| store1      | int     |
| store2      | int     |
| store3      | int     |
+-------------+---------+
```
- product_id is the primary key for this table.
- Each row in this table indicates the product's price in the three different stores: store1, store2, and store3.
- If the product is not available in a store, then the price will be null in that store's column.

## 📊 Sample Data

**Products Table:**
| product_id | store1 | store2 | store3 |
|------------|--------|--------|--------|
| 0          | 95     | 100    | 105    |
| 1          | 70     | null   | 80     |

**Expected Output:**
| product_id | store | price |
|------------|-------|-------|
| 0          | store1| 95    |
| 0          | store2| 100   |
| 0          | store3| 105   |
| 1          | store1| 70    |
| 1          | store3| 80    |

**Explanation:**
- Product 0 is available in all three stores with prices 95, 100, and 105 respectively.
- Product 1 is available only in store1 (70) and store3 (80). It's not available in store2 (null).
- We exclude null prices from the result.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Convert horizontal data (columns) to vertical data (rows)
- This is the OPPOSITE of a PIVOT operation - it's an UNPIVOT
- Each store column becomes multiple rows
- Exclude NULL values (unavailable products)

### 2. **Key Insights**
- Use UNION ALL to combine store data into separate rows
- Each UNION section handles one store column
- Filter out NULL values with WHERE clauses
- Transform column names into data values

### 3. **Interview Discussion Points**
- "This is an UNPIVOT operation, opposite of the previous pivot problem"
- "Use UNION ALL to stack store data vertically"
- "Need to filter NULL values since they represent unavailable products"

## 🔧 Step-by-Step Solution Logic

### Step 1: Create Rows for Store1
```sql
SELECT product_id, 'store1' as store, store1 as price
FROM Products
WHERE store1 IS NOT NULL
```

### Step 2: Create Rows for Store2
```sql
SELECT product_id, 'store2' as store, store2 as price
FROM Products
WHERE store2 IS NOT NULL
```

### Step 3: Create Rows for Store3
```sql
SELECT product_id, 'store3' as store, store3 as price
FROM Products
WHERE store3 IS NOT NULL
```

### Step 4: Combine All Stores
```sql
-- UNION ALL to combine all store data
UNION ALL
```

## ✅ Optimized SQL Solution

**Solution 1: Basic UNION ALL Approach**
```sql
SELECT product_id, 'store1' as store, store1 as price
FROM Products
WHERE store1 IS NOT NULL

UNION ALL

SELECT product_id, 'store2' as store, store2 as price
FROM Products
WHERE store2 IS NOT NULL

UNION ALL

SELECT product_id, 'store3' as store, store3 as price
FROM Products
WHERE store3 IS NOT NULL;
```

### Alternative Solutions

**Solution 2: Using CROSS JOIN with VALUES**
```sql
SELECT 
    p.product_id,
    s.store,
    CASE s.store
        WHEN 'store1' THEN p.store1
        WHEN 'store2' THEN p.store2
        WHEN 'store3' THEN p.store3
    END as price
FROM Products p
CROSS JOIN (
    SELECT 'store1' as store
    UNION ALL SELECT 'store2'
    UNION ALL SELECT 'store3'
) s
WHERE CASE s.store
    WHEN 'store1' THEN p.store1
    WHEN 'store2' THEN p.store2
    WHEN 'store3' THEN p.store3
END IS NOT NULL;
```

**Solution 3: MySQL UNPIVOT Syntax (if supported)**
```sql
SELECT product_id, store, price
FROM Products
UNPIVOT (
    price FOR store IN (store1, store2, store3)
) AS unpivoted_products
WHERE price IS NOT NULL;
```

**Solution 4: Using JSON Functions (Advanced)**
```sql
SELECT 
    product_id,
    JSON_UNQUOTE(JSON_EXTRACT(store_data.store_info, '$.store')) as store,
    JSON_UNQUOTE(JSON_EXTRACT(store_data.store_info, '$.price')) as price
FROM Products p
CROSS JOIN JSON_TABLE(
    JSON_ARRAY(
        JSON_OBJECT('store', 'store1', 'price', store1),
        JSON_OBJECT('store', 'store2', 'price', store2),
        JSON_OBJECT('store', 'store3', 'price', store3)
    ),
    '$[*]' COLUMNS (
        store_info JSON PATH '$'
    )
) as store_data
WHERE JSON_UNQUOTE(JSON_EXTRACT(store_data.store_info, '$.price')) IS NOT NULL;
```

**Solution 5: Comprehensive E-commerce Product Distribution Analytics System**
```sql
WITH product_store_unpivot AS (
    -- Store1 data
    SELECT 
        product_id, 
        'store1' as store, 
        store1 as price,
        1 as store_sequence
    FROM Products
    WHERE store1 IS NOT NULL
    
    UNION ALL
    
    -- Store2 data
    SELECT 
        product_id, 
        'store2' as store, 
        store2 as price,
        2 as store_sequence
    FROM Products
    WHERE store2 IS NOT NULL
    
    UNION ALL
    
    -- Store3 data
    SELECT 
        product_id, 
        'store3' as store, 
        store3 as price,
        3 as store_sequence
    FROM Products
    WHERE store3 IS NOT NULL
),
product_distribution_analysis AS (
    SELECT 
        psu.*,
        
        -- Price tier classification
        CASE 
            WHEN price >= 100 THEN 'Premium Tier'
            WHEN price >= 80 THEN 'Mid-Range Tier'
            WHEN price >= 60 THEN 'Standard Tier'
            ELSE 'Budget Tier'
        END as price_tier,
        
        -- Store characteristics simulation
        CASE 
            WHEN store = 'store1' THEN 'Flagship Store'
            WHEN store = 'store2' THEN 'Regional Store'
            ELSE 'Outlet Store'
        END as store_type,
        
        -- Market positioning
        CASE 
            WHEN store = 'store1' AND price >= 90 THEN 'Premium Market Position'
            WHEN store = 'store2' AND price BETWEEN 70 AND 100 THEN 'Mid-Market Position'
            WHEN store = 'store3' AND price <= 85 THEN 'Value Market Position'
            ELSE 'Standard Market Position'
        END as market_position,
        
        -- Product availability analysis
        COUNT(*) OVER(PARTITION BY product_id) as total_stores_available,
        ROW_NUMBER() OVER(PARTITION BY product_id ORDER BY price) as price_rank_asc,
        ROW_NUMBER() OVER(PARTITION BY product_id ORDER BY price DESC) as price_rank_desc,
        
        -- Store performance simulation
        CASE 
            WHEN store = 'store1' THEN 'High Traffic'
            WHEN store = 'store2' THEN 'Medium Traffic'
            ELSE 'Standard Traffic'
        END as store_traffic,
        
        -- Competition level
        CASE 
            WHEN price >= 90 THEN 'High Competition'
            WHEN price >= 70 THEN 'Medium Competition'
            ELSE 'Low Competition'
        END as competition_level,
        
        -- Customer segment targeting
        CASE 
            WHEN store = 'store1' AND price >= 90 THEN 'Premium Customers'
            WHEN store = 'store2' AND price BETWEEN 60 AND 90 THEN 'Mid-Market Customers'
            WHEN store = 'store3' AND price <= 80 THEN 'Value-Conscious Customers'
            ELSE 'General Market Customers'
        END as target_customer_segment,
        
        -- Product category simulation
        CASE 
            WHEN product_id % 6 = 0 THEN 'Electronics'
            WHEN product_id % 6 = 1 THEN 'Fashion & Apparel'
            WHEN product_id % 6 = 2 THEN 'Home & Garden'
            WHEN product_id % 6 = 3 THEN 'Sports & Recreation'
            WHEN product_id % 6 = 4 THEN 'Books & Media'
            ELSE 'Health & Beauty'
        END as product_category,
        
        -- Inventory velocity simulation
        CASE 
            WHEN store = 'store1' AND price <= 80 THEN 'Fast Moving'
            WHEN store = 'store2' AND price BETWEEN 60 AND 90 THEN 'Steady Moving'
            WHEN store = 'store3' AND price >= 85 THEN 'Slow Moving'
            ELSE 'Standard Moving'
        END as inventory_velocity,
        
        -- Profit margin estimation
        CASE 
            WHEN price >= 100 THEN 'High Margin'
            WHEN price >= 80 THEN 'Good Margin'
            WHEN price >= 60 THEN 'Standard Margin'
            ELSE 'Low Margin'
        END as profit_margin_category
    FROM product_store_unpivot psu
),
comprehensive_distribution_insights AS (
    SELECT 
        pda.*,
        
        -- Distribution strategy assessment
        CASE 
            WHEN total_stores_available = 3 AND price_rank_asc = 1 AND store_type = 'Outlet Store'
            THEN 'Distribution Strategy: Full coverage + competitive pricing + value leadership + market penetration'
            WHEN total_stores_available = 3 AND price_rank_desc = 1 AND store_type = 'Flagship Store'
            THEN 'Distribution Strategy: Premium positioning + flagship excellence + luxury market + brand leadership'
            WHEN total_stores_available = 3
            THEN 'Distribution Strategy: Complete market coverage + tiered pricing + diverse customer base + market dominance'
            WHEN total_stores_available = 2 AND store_type = 'Flagship Store'
            THEN 'Distribution Strategy: Premium focus + selective distribution + quality positioning + targeted reach'
            WHEN total_stores_available = 1
            THEN 'Distribution Strategy: Exclusive distribution + specialized market + focused positioning + niche strategy'
            ELSE 'Distribution Strategy: Balanced approach + strategic positioning + market optimization'
        END as distribution_strategy,
        
        -- Pricing strategy insights
        CASE 
            WHEN total_stores_available = 3 AND price_rank_asc = 1 AND price_tier = 'Budget Tier'
            THEN 'Pricing Strategy: Value leadership + competitive pricing + market share focus + accessibility priority'
            WHEN total_stores_available = 3 AND price_rank_desc = 1 AND price_tier = 'Premium Tier'
            THEN 'Pricing Strategy: Premium positioning + value justification + quality emphasis + margin optimization'
            WHEN total_stores_available >= 2 AND market_position = 'Mid-Market Position'
            THEN 'Pricing Strategy: Balanced positioning + competitive advantage + value proposition + market alignment'
            WHEN store_traffic = 'High Traffic' AND competition_level = 'High Competition'
            THEN 'Pricing Strategy: Dynamic pricing + competitive response + traffic monetization + market responsiveness'
            ELSE 'Pricing Strategy: Standard positioning + stable pricing + consistent value + market maintenance'
        END as pricing_strategy,
        
        -- Customer acquisition strategy
        CASE 
            WHEN target_customer_segment = 'Premium Customers' AND store_traffic = 'High Traffic'
            THEN 'Customer Strategy: Premium acquisition + luxury experience + high-value retention + exclusive services'
            WHEN target_customer_segment = 'Value-Conscious Customers' AND inventory_velocity = 'Fast Moving'
            THEN 'Customer Strategy: Volume acquisition + value communication + cost leadership + mass market appeal'
            WHEN target_customer_segment = 'Mid-Market Customers' AND total_stores_available = 3
            THEN 'Customer Strategy: Broad acquisition + choice offering + convenience + comprehensive service'
            WHEN store_type = 'Regional Store' AND competition_level = 'Medium Competition'
            THEN 'Customer Strategy: Local market focus + community engagement + regional excellence + competitive differentiation'
            ELSE 'Customer Strategy: Targeted acquisition + segment focus + specialized service + customer satisfaction'
        END as customer_acquisition_strategy,
        
        -- Inventory and supply chain optimization
        CASE 
            WHEN inventory_velocity = 'Fast Moving' AND total_stores_available >= 2
            THEN 'Supply Chain: High turnover + rapid replenishment + demand-driven + multi-location coordination + efficiency optimization'
            WHEN inventory_velocity = 'Slow Moving' AND profit_margin_category = 'High Margin'
            THEN 'Supply Chain: Premium inventory + selective stocking + quality focus + margin protection + luxury supply chain'
            WHEN total_stores_available = 3 AND competition_level = 'High Competition'
            THEN 'Supply Chain: Responsive logistics + competitive supply + market agility + cost optimization + rapid deployment'
            WHEN store_type = 'Flagship Store' AND market_position = 'Premium Market Position'
            THEN 'Supply Chain: Premium supply chain + quality assurance + brand excellence + flagship support + luxury logistics'
            ELSE 'Supply Chain: Standard operations + reliable delivery + cost efficiency + quality maintenance + steady supply'
        END as supply_chain_optimization,
        
        -- Marketing and promotional strategies
        CASE 
            WHEN price_tier = 'Premium Tier' AND store_type = 'Flagship Store'
            THEN 'Marketing: Luxury marketing + brand prestige + exclusive promotions + high-value messaging + premium positioning'
            WHEN price_tier = 'Budget Tier' AND store_traffic = 'High Traffic'
            THEN 'Marketing: Value marketing + volume promotions + accessibility messaging + cost leadership + mass appeal'
            WHEN total_stores_available = 3 AND market_position = 'Mid-Market Position'
            THEN 'Marketing: Comprehensive campaigns + multi-channel promotion + broad appeal + convenience messaging + choice emphasis'
            WHEN competition_level = 'High Competition' AND inventory_velocity = 'Fast Moving'
            THEN 'Marketing: Competitive marketing + aggressive promotion + market share focus + rapid response + dynamic campaigns'
            ELSE 'Marketing: Targeted promotion + segment focus + brand building + customer education + steady growth'
        END as marketing_strategy,
        
        -- Business growth and expansion opportunities
        CASE 
            WHEN total_stores_available = 1 AND profit_margin_category = 'High Margin'
            THEN 'Growth Opportunity: Market expansion + additional channels + premium replication + coverage increase + strategic growth'
            WHEN total_stores_available = 3 AND price_rank_asc = 1 AND inventory_velocity = 'Fast Moving'
            THEN 'Growth Opportunity: Volume expansion + market penetration + scale optimization + efficiency gains + market domination'
            WHEN store_type = 'Flagship Store' AND target_customer_segment = 'Premium Customers'
            THEN 'Growth Opportunity: Premium expansion + luxury market development + brand extension + high-value growth + exclusive markets'
            WHEN market_position = 'Value Market Position' AND competition_level = 'Low Competition'
            THEN 'Growth Opportunity: Market leadership + competitive advantage + share capture + strategic positioning + sustainable growth'
            ELSE 'Growth Opportunity: Strategic development + optimization focus + market enhancement + competitive strengthening + sustainable expansion'
        END as growth_opportunities
    FROM product_distribution_analysis pda
)
SELECT 
    product_id,
    store,
    price,
    product_category,
    price_tier,
    store_type,
    market_position,
    distribution_strategy,
    pricing_strategy,
    customer_acquisition_strategy,
    supply_chain_optimization,
    marketing_strategy,
    growth_opportunities,
    
    -- Key metrics
    total_stores_available,
    price_rank_asc,
    target_customer_segment,
    inventory_velocity,
    competition_level,
    profit_margin_category,
    store_traffic
FROM comprehensive_distribution_insights
ORDER BY 
    product_id,
    price DESC,
    store_sequence;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Multi-Channel Inventory and Availability Management System**
```sql
-- "Design comprehensive inventory management system for Amazon's complex multi-channel distribution network"

WITH amazon_channel_unpivot AS (
    -- Amazon.com Direct
    SELECT 
        product_id, 
        'Amazon Direct' as channel, 
        store1 as price,
        'First-Party Retail' as business_model,
        'Prime 2-Day Delivery' as fulfillment_type
    FROM Products
    WHERE store1 IS NOT NULL
    
    UNION ALL
    
    -- Amazon Fresh/Whole Foods
    SELECT 
        product_id, 
        'Amazon Fresh' as channel, 
        store2 as price,
        'Amazon Subsidiary' as business_model,
        'Same-Day Grocery Delivery' as fulfillment_type
    FROM Products
    WHERE store2 IS NOT NULL
    
    UNION ALL
    
    -- Third-Party Marketplace
    SELECT 
        product_id, 
        'Marketplace' as channel, 
        store3 as price,
        'Third-Party Seller' as business_model,
        'FBA or FBM' as fulfillment_type
    FROM Products
    WHERE store3 IS NOT NULL
),
amazon_availability_intelligence AS (
    SELECT 
        acu.*,
        
        -- Amazon customer segment mapping
        CASE 
            WHEN channel = 'Amazon Direct' AND price >= 90 THEN 'Prime Premium Customers'
            WHEN channel = 'Amazon Direct' THEN 'Prime Standard Customers'
            WHEN channel = 'Amazon Fresh' THEN 'Grocery & Fresh Customers'
            WHEN channel = 'Marketplace' AND price <= 70 THEN 'Value-Seeking Customers'
            WHEN channel = 'Marketplace' THEN 'Marketplace Shoppers'
        END as amazon_customer_segment,
        
        -- Prime benefit tier
        CASE 
            WHEN channel = 'Amazon Direct' AND price >= 90 THEN 'Prime Exclusive Benefits'
            WHEN channel = 'Amazon Direct' THEN 'Standard Prime Benefits'
            WHEN channel = 'Amazon Fresh' THEN 'Prime Fresh Benefits'
            WHEN channel = 'Marketplace' AND price >= 75 THEN 'Prime Eligible'
            ELSE 'Standard Marketplace'
        END as prime_tier,
        
        -- Amazon fee structure simulation
        CASE 
            WHEN business_model = 'First-Party Retail' THEN price * 0.15
            WHEN business_model = 'Amazon Subsidiary' THEN price * 0.12
            WHEN business_model = 'Third-Party Seller' AND price >= 75 THEN price * 0.10
            ELSE price * 0.08
        END as amazon_fee_revenue,
        
        -- Inventory management complexity
        CASE 
            WHEN business_model = 'First-Party Retail' THEN 'Amazon Managed Inventory'
            WHEN business_model = 'Amazon Subsidiary' THEN 'Subsidiary Managed Inventory'
            WHEN fulfillment_type = 'FBA or FBM' AND price >= 75 THEN 'FBA Premium Inventory'
            ELSE 'Third-Party Managed Inventory'
        END as inventory_management_type,
        
        -- Customer experience optimization
        CASE 
            WHEN channel = 'Amazon Direct' AND prime_tier = 'Prime Exclusive Benefits'
            THEN 'Premium Experience: Exclusive access + premium support + luxury service + VIP treatment'
            WHEN channel = 'Amazon Fresh' AND fulfillment_type = 'Same-Day Grocery Delivery'
            THEN 'Fresh Experience: Same-day delivery + grocery convenience + fresh guarantee + meal solutions'
            WHEN channel = 'Marketplace' AND amazon_fee_revenue >= 7
            THEN 'Marketplace Premium: Quality sellers + reliable delivery + customer protection + competitive prices'
            ELSE 'Standard Experience: Reliable service + customer support + competitive pricing + standard delivery'
        END as customer_experience_level,
        
        -- Availability across Amazon ecosystem
        COUNT(*) OVER(PARTITION BY product_id) as total_amazon_channels,
        ROW_NUMBER() OVER(PARTITION BY product_id ORDER BY amazon_fee_revenue DESC) as revenue_priority,
        
        -- Strategic channel importance
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 3 THEN 'Full Amazon Ecosystem'
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 2 AND channel = 'Amazon Direct' THEN 'Amazon Core Channels'
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 1 AND channel = 'Amazon Direct' THEN 'Amazon Exclusive'
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 1 AND channel = 'Marketplace' THEN 'Marketplace Only'
            ELSE 'Partial Amazon Presence'
        END as amazon_ecosystem_strategy
    FROM amazon_channel_unpivot acu
),
amazon_operational_excellence AS (
    SELECT 
        aai.*,
        
        -- Operational excellence assessment based on Amazon Leadership Principles
        CASE 
            WHEN amazon_ecosystem_strategy = 'Full Amazon Ecosystem' AND total_amazon_channels = 3
            THEN 'Operational Excellence: Complete ecosystem integration + multi-channel optimization + comprehensive customer reach'
            WHEN business_model = 'First-Party Retail' AND inventory_management_type = 'Amazon Managed Inventory'
            THEN 'Operational Excellence: Direct control + inventory optimization + customer experience + quality assurance'
            WHEN channel = 'Amazon Fresh' AND customer_experience_level LIKE '%Fresh Experience%'
            THEN 'Operational Excellence: Fresh logistics + same-day delivery + grocery expertise + meal solution integration'
            WHEN amazon_ecosystem_strategy = 'Marketplace Only' AND amazon_fee_revenue >= 6
            THEN 'Operational Excellence: Marketplace efficiency + seller support + customer protection + competitive pricing'
            ELSE 'Operational Excellence: Standard operations + reliable service + continuous improvement + customer satisfaction'
        END as operational_excellence_level,
        
        -- Customer obsession application
        CASE 
            WHEN amazon_customer_segment = 'Prime Premium Customers' AND customer_experience_level LIKE '%Premium%'
            THEN 'Customer Obsession: Premium customer focus + exclusive benefits + luxury experience + personalized service'
            WHEN amazon_customer_segment = 'Grocery & Fresh Customers' AND fulfillment_type = 'Same-Day Grocery Delivery'
            THEN 'Customer Obsession: Fresh convenience + grocery solutions + meal planning + family needs + time savings'
            WHEN amazon_customer_segment = 'Value-Seeking Customers' AND channel = 'Marketplace'
            THEN 'Customer Obsession: Value delivery + competitive pricing + choice variety + accessible options + cost savings'
            WHEN prime_tier LIKE '%Prime%' AND total_amazon_channels >= 2
            THEN 'Customer Obsession: Prime member benefits + multi-channel choice + convenience + fast delivery + exclusive access'
            ELSE 'Customer Obsession: Customer satisfaction + reliable service + quality products + competitive value + support excellence'
        END as customer_obsession_implementation,
        
        -- Ownership and accountability
        CASE 
            WHEN business_model = 'First-Party Retail' AND inventory_management_type = 'Amazon Managed Inventory'
            THEN 'Ownership: Full product responsibility + inventory control + customer experience + end-to-end accountability'
            WHEN amazon_ecosystem_strategy = 'Full Amazon Ecosystem'
            THEN 'Ownership: Multi-channel coordination + ecosystem optimization + customer experience + strategic management'
            WHEN business_model = 'Amazon Subsidiary' AND channel = 'Amazon Fresh'
            THEN 'Ownership: Fresh market responsibility + grocery expertise + subsidiary excellence + specialized accountability'
            WHEN business_model = 'Third-Party Seller' AND amazon_fee_revenue >= 7
            THEN 'Ownership: Marketplace facilitation + seller support + customer protection + platform excellence'
            ELSE 'Ownership: Channel responsibility + service quality + customer satisfaction + operational excellence'
        END as ownership_demonstration,
        
        -- Think Big vision for product distribution
        CASE 
            WHEN amazon_ecosystem_strategy = 'Full Amazon Ecosystem' AND amazon_fee_revenue >= 25
            THEN 'Think Big: Global marketplace dominance + ecosystem leadership + customer experience revolution + retail transformation'
            WHEN channel = 'Amazon Direct' AND prime_tier = 'Prime Exclusive Benefits'
            THEN 'Think Big: Premium market leadership + luxury commerce + exclusive experiences + high-value customer focus'
            WHEN channel = 'Amazon Fresh' AND customer_experience_level LIKE '%Fresh%'
            THEN 'Think Big: Grocery retail transformation + fresh delivery innovation + meal solution revolution + lifestyle convenience'
            WHEN total_amazon_channels >= 2 AND amazon_fee_revenue >= 15
            THEN 'Think Big: Multi-channel excellence + integrated commerce + customer choice + market expansion + scale optimization'
            ELSE 'Think Big: Channel innovation + market development + customer experience + competitive advantage + strategic growth'
        END as think_big_implementation
    FROM amazon_availability_intelligence aai
)
SELECT 
    product_id,
    channel,
    price,
    business_model,
    fulfillment_type,
    amazon_customer_segment,
    prime_tier,
    customer_experience_level,
    amazon_ecosystem_strategy,
    operational_excellence_level,
    customer_obsession_implementation,
    ownership_demonstration,
    think_big_implementation,
    
    -- Financial and operational metrics
    amazon_fee_revenue,
    total_amazon_channels,
    revenue_priority,
    inventory_management_type
FROM amazon_operational_excellence
ORDER BY 
    amazon_fee_revenue DESC,
    CASE amazon_ecosystem_strategy
        WHEN 'Full Amazon Ecosystem' THEN 1
        WHEN 'Amazon Core Channels' THEN 2
        WHEN 'Amazon Exclusive' THEN 3
        ELSE 4
    END,
    product_id,
    revenue_priority;
```

I'll create one final brief extension and then complete the last two questions:

#### 2. **Dynamic Product Catalog Management and Real-time Availability System**
```sql
-- "Real-time product catalog management with dynamic availability tracking and automated alerts"

WITH real_time_catalog_management AS (
    SELECT 
        product_id,
        'store1' as store,
        store1 as price,
        CASE WHEN store1 IS NOT NULL THEN 'Available' ELSE 'Out of Stock' END as availability_status
    FROM Products
    WHERE store1 IS NOT NULL
    
    UNION ALL
    
    SELECT 
        product_id,
        'store2' as store,
        store2 as price,
        CASE WHEN store2 IS NOT NULL THEN 'Available' ELSE 'Out of Stock' END as availability_status
    FROM Products
    WHERE store2 IS NOT NULL
    
    UNION ALL
    
    SELECT 
        product_id,
        'store3' as store,
        store3 as price,
        CASE WHEN store3 IS NOT NULL THEN 'Available' ELSE 'Out of Stock' END as availability_status
    FROM Products
    WHERE store3 IS NOT NULL
),
catalog_intelligence_dashboard AS (
    SELECT 
        product_id,
        store,
        price,
        availability_status,
        COUNT(*) OVER(PARTITION BY product_id) as store_availability_count,
        
        -- Real-time alerts
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 1 THEN 'ALERT: Limited availability'
            WHEN price >= 100 THEN 'ALERT: Premium pricing detected'
            WHEN price <= 60 THEN 'ALERT: Value pricing opportunity'
            ELSE 'NORMAL: Standard availability and pricing'
        END as real_time_alert,
        
        -- Dynamic catalog recommendations
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 1 
            THEN 'RECOMMENDATION: Expand availability + increase distribution + market penetration'
            WHEN price >= 100 
            THEN 'RECOMMENDATION: Premium strategy + luxury positioning + value justification'
            WHEN COUNT(*) OVER(PARTITION BY product_id) = 3 
            THEN 'RECOMMENDATION: Optimize pricing + channel strategy + competitive positioning'
            ELSE 'RECOMMENDATION: Maintain current strategy + monitor performance + optimize margins'
        END as catalog_recommendation
    FROM real_time_catalog_management
)
SELECT 
    product_id,
    store,
    price,
    store_availability_count,
    real_time_alert,
    catalog_recommendation
FROM catalog_intelligence_dashboard
ORDER BY store_availability_count ASC, price DESC;
```

## 🔗 Related LeetCode Questions

1. **#1777 - Product's Price for Each Store** (The reverse operation - PIVOT)
2. **#1757 - Recyclable and Low Fat Products** (Product filtering operations)
3. **#1741 - Find Total Time Spent by Each Employee** (Data aggregation patterns)
4. **#1789 - Primary Department for Each Employee** (Conditional data selection)
5. **#1978 - Employees Whose Manager Left the Company** (Complex data relationships)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **UNPIVOT Operation**: Convert columns to rows using UNION ALL
2. **NULL Filtering**: Use WHERE clauses to exclude unavailable products
3. **Data Transformation**: Transform column names into data values
4. **Vertical Data Stacking**: Combine multiple SELECT statements with UNION ALL

### 🚀 **Amazon Interview Tips**
1. **Explain unpivot concept**: "Converting horizontal data (columns) to vertical (rows)"
2. **Discuss NULL handling**: "Filter out NULL values since they represent unavailable products"
3. **Address data integrity**: "Each UNION section handles one store column consistently"
4. **Consider scalability**: "Easy to add more stores by adding more UNION sections"

### 🔧 **Common Patterns**
- UNION ALL for combining similar data structures
- Column-to-value transformation with literal strings
- NULL filtering with WHERE clauses
- Consistent SELECT structure across UNION sections

### ⚠️ **Common Mistakes to Avoid**
1. **Using UNION instead of UNION ALL** (removes duplicates unnecessarily)
2. **Forgetting NULL filters** (would include unavailable products)
3. **Inconsistent column names** (UNION requires matching column structure)
4. **Wrong literal values** (store names must match business requirements)

### 🔍 **Performance Considerations**
- UNION ALL is faster than UNION (no duplicate elimination)
- INDEX on product_id for efficient filtering across sections
- Consider using UNPIVOT if database supports it natively
- Large tables benefit from parallel execution of UNION sections

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Provide comprehensive product availability information
- **Invent and Simplify**: Transform complex data structures into simple, usable formats
- **Think Big**: Scale data transformation across Amazon's massive product catalog
- **Deliver Results**: Enable efficient inventory management and customer experience

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Add a fourth store column and include it in the unpivot**
2. **Include a calculated column showing price differences from average**
3. **Add store characteristics (location, type) to the result**
4. **Create a summary showing total available stores per product**

Remember: Data transformation operations like UNPIVOT are essential for Amazon's dynamic product catalog management, enabling flexible inventory tracking across multiple channels and real-time availability updates!