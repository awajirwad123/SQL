# LeetCode Easy #1757: Recyclable and Low Fat Products

## 📋 Problem Statement

Write a SQL query to find the **ids** of products that are both **low fat** and **recyclable**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Products Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| product_id  | int     |
| low_fats    | enum    |
| recyclable  | enum    |
+-------------+---------+
```
- product_id is the primary key for this table.
- low_fats is an ENUM of type ('Y', 'N') where 'Y' means this product is low fat and 'N' means it is not.
- recyclable is an ENUM of type ('Y', 'N') where 'Y' means this product is recyclable and 'N' means it is not.

## 📊 Sample Data

**Products Table:**
| product_id | low_fats | recyclable |
|------------|----------|------------|
| 0          | Y        | N          |
| 1          | Y        | Y          |
| 2          | N        | Y          |
| 3          | Y        | Y          |
| 4          | N        | N          |

**Expected Output:**
| product_id |
|------------|
| 1          |
| 3          |

**Explanation:**
Only products 1 and 3 are both low fat (Y) and recyclable (Y).

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Filter products with TWO conditions simultaneously
- Both low_fats = 'Y' AND recyclable = 'Y'
- Simple WHERE clause with multiple conditions
- Return only product_id column

### 2. **Key Insights**
- Use AND operator to combine conditions
- ENUM values are string literals, need quotes
- Both conditions must be true (intersection, not union)
- Basic filtering operation

### 3. **Interview Discussion Points**
- "Need both conditions to be true, so use AND operator"
- "ENUM values are stored as strings, so compare with 'Y'"
- "Simple filtering problem, straightforward WHERE clause"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Required Columns
```sql
-- Need only product_id in result
SELECT product_id
```

### Step 2: Apply First Condition
```sql
-- Filter for low fat products
WHERE low_fats = 'Y'
```

### Step 3: Apply Second Condition
```sql
-- Also filter for recyclable products
AND recyclable = 'Y'
```

## ✅ Optimized SQL Solution

**Solution 1: Basic AND Condition**
```sql
SELECT product_id
FROM Products
WHERE low_fats = 'Y' 
  AND recyclable = 'Y';
```

### Alternative Solutions

**Solution 2: Using IN Operator (Overkill but Valid)**
```sql
SELECT product_id
FROM Products
WHERE low_fats IN ('Y')
  AND recyclable IN ('Y');
```

**Solution 3: With Explicit Boolean Logic**
```sql
SELECT product_id
FROM Products
WHERE (low_fats = 'Y')
  AND (recyclable = 'Y');
```

**Solution 4: Using CASE for Demonstration**
```sql
SELECT product_id
FROM Products
WHERE CASE WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 1 ELSE 0 END = 1;
```

**Solution 5: Comprehensive Product Analytics System**
```sql
WITH product_sustainability_analysis AS (
    SELECT 
        product_id,
        low_fats,
        recyclable,
        
        -- Sustainability scoring
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Excellent Sustainability'
            WHEN low_fats = 'Y' OR recyclable = 'Y' THEN 'Good Sustainability'
            ELSE 'Standard Product'
        END as sustainability_rating,
        
        -- Environmental impact simulation
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Low Environmental Impact'
            WHEN low_fats = 'Y' AND recyclable = 'N' THEN 'Medium Environmental Impact - Packaging Concern'
            WHEN low_fats = 'N' AND recyclable = 'Y' THEN 'Medium Environmental Impact - Health Concern'
            ELSE 'High Environmental Impact'
        END as environmental_impact,
        
        -- Product category simulation
        CASE 
            WHEN product_id % 10 = 0 THEN 'Beverages'
            WHEN product_id % 10 = 1 THEN 'Snacks'
            WHEN product_id % 10 = 2 THEN 'Dairy'
            WHEN product_id % 10 = 3 THEN 'Frozen Foods'
            WHEN product_id % 10 = 4 THEN 'Bakery'
            WHEN product_id % 10 = 5 THEN 'Meat & Seafood'
            WHEN product_id % 10 = 6 THEN 'Fresh Produce'
            WHEN product_id % 10 = 7 THEN 'Pantry Staples'
            WHEN product_id % 10 = 8 THEN 'Health & Wellness'
            ELSE 'Personal Care'
        END as product_category,
        
        -- Market positioning
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Premium Eco-Friendly'
            WHEN low_fats = 'Y' THEN 'Health-Focused'
            WHEN recyclable = 'Y' THEN 'Eco-Conscious'
            ELSE 'Standard Market'
        END as market_positioning,
        
        -- Consumer target segment
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Health & Environment Conscious'
            WHEN low_fats = 'Y' THEN 'Health-Focused Consumers'
            WHEN recyclable = 'Y' THEN 'Environmentally Aware Consumers'
            ELSE 'General Market'
        END as target_segment,
        
        -- Compliance and certification potential
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Eligible for: Organic + Green + Health Certifications'
            WHEN low_fats = 'Y' THEN 'Eligible for: Health + Nutritional Certifications'
            WHEN recyclable = 'Y' THEN 'Eligible for: Environmental + Sustainability Certifications'
            ELSE 'Standard Compliance Requirements'
        END as certification_eligibility,
        
        -- Marketing message potential
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Good for You, Good for Planet'
            WHEN low_fats = 'Y' THEN 'Healthy Choice for Better Living'
            WHEN recyclable = 'Y' THEN 'Environmentally Responsible Choice'
            ELSE 'Quality You Can Trust'
        END as marketing_message
    FROM Products
),
product_portfolio_analysis AS (
    SELECT 
        psa.*,
        
        -- Portfolio optimization insights
        CASE 
            WHEN sustainability_rating = 'Excellent Sustainability' 
            THEN 'Portfolio Optimization: Feature prominently + premium pricing + sustainability marketing + eco-friendly placement'
            WHEN sustainability_rating = 'Good Sustainability'
            THEN 'Portfolio Optimization: Highlight single benefit + targeted marketing + improvement opportunities + niche positioning'
            ELSE 'Portfolio Optimization: Standard positioning + cost optimization + potential reformulation + market research'
        END as portfolio_strategy,
        
        -- Supply chain considerations
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' 
            THEN 'Supply Chain: Sustainable sourcing + eco-friendly packaging + health-conscious suppliers + green logistics'
            WHEN low_fats = 'Y'
            THEN 'Supply Chain: Health-focused sourcing + nutritional suppliers + quality control + wellness partnerships'
            WHEN recyclable = 'Y'
            THEN 'Supply Chain: Eco-friendly packaging + sustainable materials + recycling partnerships + green distribution'
            ELSE 'Supply Chain: Cost-efficient sourcing + standard packaging + traditional suppliers + optimized logistics'
        END as supply_chain_strategy,
        
        -- Innovation opportunities
        CASE 
            WHEN low_fats = 'N' AND recyclable = 'N'
            THEN 'Innovation Opportunity: Complete product redesign + health improvement + eco-friendly packaging + market repositioning'
            WHEN low_fats = 'N' AND recyclable = 'Y'
            THEN 'Innovation Opportunity: Health improvement + low-fat formulation + maintain eco-packaging + wellness focus'
            WHEN low_fats = 'Y' AND recyclable = 'N'
            THEN 'Innovation Opportunity: Sustainable packaging + maintain health benefits + eco-friendly materials + green credentials'
            ELSE 'Innovation Opportunity: Maintain excellence + premium enhancements + advanced sustainability + market leadership'
        END as innovation_opportunity,
        
        -- Competitive positioning
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Competitive Position: Market leader + differentiation advantage + premium command + sustainability champion'
            WHEN low_fats = 'Y' OR recyclable = 'Y'
            THEN 'Competitive Position: Niche advantage + single benefit leader + targeted differentiation + improvement potential'
            ELSE 'Competitive Position: Price competitor + volume play + improvement required + catch-up strategy'
        END as competitive_positioning,
        
        -- Customer value proposition
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Value Proposition: Complete wellness + environmental responsibility + future-ready choice + conscious consumption'
            WHEN low_fats = 'Y'
            THEN 'Value Proposition: Health benefits + nutritional value + wellness support + healthy lifestyle'
            WHEN recyclable = 'Y'
            THEN 'Value Proposition: Environmental impact + sustainable choice + future generations + responsible consumption'
            ELSE 'Value Proposition: Quality product + reliable choice + affordability + traditional value'
        END as value_proposition
    FROM product_sustainability_analysis psa
)
SELECT 
    product_id,
    product_category,
    sustainability_rating,
    environmental_impact,
    market_positioning,
    target_segment,
    certification_eligibility,
    marketing_message,
    portfolio_strategy,
    supply_chain_strategy,
    innovation_opportunity,
    competitive_positioning,
    value_proposition
FROM product_portfolio_analysis
WHERE low_fats = 'Y' AND recyclable = 'Y'
ORDER BY product_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Fresh and Whole Foods Sustainable Product Strategy**
```sql
-- "Design comprehensive sustainable product management system for Amazon's grocery platforms"

WITH amazon_grocery_sustainability AS (
    SELECT 
        product_id,
        low_fats,
        recyclable,
        
        -- Amazon grocery platform assignment
        CASE 
            WHEN product_id % 3 = 0 THEN 'Amazon Fresh'
            WHEN product_id % 3 = 1 THEN 'Whole Foods Market'
            ELSE 'Amazon Pantry'
        END as amazon_platform,
        
        -- Prime member benefit tier
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Prime Exclusive Sustainable'
            WHEN low_fats = 'Y' OR recyclable = 'Y' THEN 'Prime Preferred Eco'
            ELSE 'Prime Standard'
        END as prime_tier,
        
        -- Amazon sustainability program eligibility
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Climate Pledge Friendly + Health Conscious'
            WHEN recyclable = 'Y' THEN 'Climate Pledge Friendly'
            WHEN low_fats = 'Y' THEN 'Health Conscious Program'
            ELSE 'Standard Amazon Product'
        END as amazon_sustainability_program,
        
        -- Fulfillment optimization
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Premium Fulfillment + Eco-Packaging + Temperature Control'
            WHEN low_fats = 'Y' THEN 'Health-Safe Fulfillment + Temperature Control + Quality Assurance'
            WHEN recyclable = 'Y' THEN 'Eco-Friendly Fulfillment + Sustainable Packaging + Green Logistics'
            ELSE 'Standard Fulfillment + Cost Optimization + Efficiency Focus'
        END as fulfillment_strategy,
        
        -- Amazon recommendation engine weighting
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'High Recommendation Weight + Featured Placement + Customer Preference'
            WHEN low_fats = 'Y' OR recyclable = 'Y' THEN 'Medium Recommendation Weight + Category Preference + Targeted Suggestions'
            ELSE 'Standard Recommendation Weight + Price-Based Suggestions + Volume Preference'
        END as recommendation_weighting,
        
        -- Alexa integration features
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Alexa Featured: "Best sustainable choice" + voice shopping priority + smart suggestions'
            WHEN low_fats = 'Y' THEN 'Alexa Featured: "Healthy option" + nutrition voice info + wellness integration'
            WHEN recyclable = 'Y' THEN 'Alexa Featured: "Eco-friendly choice" + sustainability voice info + green shopping'
            ELSE 'Alexa Standard: Basic product info + standard voice ordering + price comparisons'
        END as alexa_integration
    FROM Products
),
amazon_grocery_operations AS (
    SELECT 
        ags.*,
        
        -- Amazon Fresh specific optimization
        CASE 
            WHEN amazon_platform = 'Amazon Fresh' AND low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Fresh Strategy: Premium positioning + same-day delivery + eco-packaging + health focus + urban targeting'
            WHEN amazon_platform = 'Amazon Fresh' AND (low_fats = 'Y' OR recyclable = 'Y')
            THEN 'Fresh Strategy: Category leadership + fast delivery + targeted marketing + benefit highlighting'
            WHEN amazon_platform = 'Amazon Fresh'
            THEN 'Fresh Strategy: Competitive pricing + reliable delivery + volume optimization + market share focus'
            ELSE 'N/A'
        END as amazon_fresh_strategy,
        
        -- Whole Foods integration
        CASE 
            WHEN amazon_platform = 'Whole Foods Market' AND low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Whole Foods: Premium organic + store prominence + quality standards + sustainability showcase + health leadership'
            WHEN amazon_platform = 'Whole Foods Market' AND low_fats = 'Y'
            THEN 'Whole Foods: Health section + nutritional focus + wellness promotion + quality assurance + organic potential'
            WHEN amazon_platform = 'Whole Foods Market' AND recyclable = 'Y'
            THEN 'Whole Foods: Sustainability section + eco-friendly promotion + green positioning + environmental leadership'
            WHEN amazon_platform = 'Whole Foods Market'
            THEN 'Whole Foods: Quality standards + natural positioning + premium placement + brand excellence'
            ELSE 'N/A'
        END as whole_foods_strategy,
        
        -- Prime member personalization
        CASE 
            WHEN prime_tier = 'Prime Exclusive Sustainable'
            THEN 'Prime Personalization: Exclusive early access + sustainability dashboard + health tracking + eco-rewards program'
            WHEN prime_tier = 'Prime Preferred Eco'
            THEN 'Prime Personalization: Preferred placement + targeted notifications + benefit highlighting + category suggestions'
            ELSE 'Prime Personalization: Standard benefits + price advantages + fast delivery + basic recommendations'
        END as prime_personalization,
        
        -- Amazon advertising strategy
        CASE 
            WHEN amazon_sustainability_program LIKE '%Climate Pledge Friendly + Health Conscious%'
            THEN 'Advertising: Premium campaign + sustainability + health dual-messaging + lifestyle targeting + premium budget'
            WHEN amazon_sustainability_program LIKE '%Climate Pledge Friendly%'
            THEN 'Advertising: Environmental campaign + green messaging + eco-conscious targeting + sustainability budget'
            WHEN amazon_sustainability_program LIKE '%Health Conscious%'
            THEN 'Advertising: Health campaign + wellness messaging + health-focused targeting + nutrition budget'
            ELSE 'Advertising: Standard campaign + price messaging + broad targeting + efficiency budget'
        END as advertising_strategy,
        
        -- Customer review and rating strategy
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Review Strategy: Highlight sustainability + health benefits + eco-credentials + wellness impact + customer testimonials'
            WHEN low_fats = 'Y'
            THEN 'Review Strategy: Focus on health benefits + nutritional value + wellness outcomes + healthy lifestyle support'
            WHEN recyclable = 'Y'
            THEN 'Review Strategy: Emphasize environmental impact + sustainability + eco-friendly packaging + green values'
            ELSE 'Review Strategy: Focus on quality + value + reliability + customer satisfaction + price point'
        END as review_strategy,
        
        -- Cross-selling and bundling opportunities
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Cross-Sell: Eco-wellness bundle + sustainable lifestyle package + health-conscious meal kits + green living products'
            WHEN low_fats = 'Y'
            THEN 'Cross-Sell: Health bundle + wellness products + fitness accessories + nutritional supplements + healthy snacks'
            WHEN recyclable = 'Y'
            THEN 'Cross-Sell: Eco-friendly bundle + sustainable products + green cleaning + environmental lifestyle products'
            ELSE 'Cross-Sell: Value bundle + related products + frequently bought together + price-optimized packages'
        END as cross_selling_strategy
    FROM amazon_grocery_sustainability ags
),
amazon_leadership_principles_application AS (
    SELECT 
        ago.*,
        
        -- Customer Obsession in grocery context
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Customer Obsession: Anticipate health + environmental needs + future-conscious choices + holistic wellness + sustainable living'
            WHEN low_fats = 'Y'
            THEN 'Customer Obsession: Health-first decisions + nutritional well-being + wellness support + healthy lifestyle enablement'
            WHEN recyclable = 'Y'
            THEN 'Customer Obsession: Environmental responsibility + sustainable choices + future generations + planet-conscious decisions'
            ELSE 'Customer Obsession: Value delivery + quality assurance + reliable choices + customer satisfaction + everyday needs'
        END as customer_obsession_grocery,
        
        -- Ownership in product management
        CASE 
            WHEN amazon_sustainability_program LIKE '%Climate Pledge Friendly + Health Conscious%'
            THEN 'Ownership: Lead market transformation + sustainability innovation + health advancement + long-term impact + industry leadership'
            WHEN amazon_sustainability_program LIKE '%Climate Pledge Friendly%'
            THEN 'Ownership: Environmental stewardship + sustainability leadership + green innovation + climate impact + eco-responsibility'
            WHEN amazon_sustainability_program LIKE '%Health Conscious%'
            THEN 'Ownership: Health advocacy + wellness promotion + nutritional advancement + customer well-being + health innovation'
            ELSE 'Ownership: Quality delivery + customer satisfaction + operational excellence + reliable service + continuous improvement'
        END as ownership_product_management,
        
        -- Invent and Simplify opportunities
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Invent and Simplify: One-click sustainable shopping + AI health recommendations + eco-score simplification + sustainability dashboard'
            WHEN low_fats = 'Y'
            THEN 'Invent and Simplify: Nutrition scoring + health goal integration + wellness tracking + simplified healthy choices'
            WHEN recyclable = 'Y'
            THEN 'Invent and Simplify: Eco-impact scoring + sustainability tracking + green choice recommendations + environmental dashboard'
            ELSE 'Invent and Simplify: Value optimization + price comparison + quality assurance + simplified shopping experience'
        END as invent_simplify_opportunities,
        
        -- Think Big vision
        CASE 
            WHEN amazon_platform = 'Whole Foods Market' AND low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Think Big: Transform grocery industry + sustainable food system + health revolution + environmental leadership + global impact'
            WHEN amazon_platform = 'Amazon Fresh' AND low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Think Big: Reinvent food delivery + sustainable logistics + health-focused urban living + eco-conscious convenience'
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Think Big: Lead sustainable commerce + health-conscious marketplace + environmental responsibility + wellness economy'
            ELSE 'Think Big: Grocery innovation + customer convenience + market expansion + operational excellence + scalable solutions'
        END as think_big_vision,
        
        -- Deliver Results metrics
        CASE 
            WHEN prime_tier = 'Prime Exclusive Sustainable'
            THEN 'Deliver Results: Premium customer satisfaction + sustainability metrics + health outcomes + retention rates + advocacy scores'
            WHEN prime_tier = 'Prime Preferred Eco'
            THEN 'Deliver Results: Category leadership + customer preference + benefit recognition + satisfaction improvement'
            ELSE 'Deliver Results: Market share growth + operational efficiency + customer acquisition + cost optimization + profitability'
        END as deliver_results_metrics
    FROM amazon_grocery_operations ago
)
SELECT 
    product_id,
    amazon_platform,
    prime_tier,
    amazon_sustainability_program,
    customer_obsession_grocery,
    ownership_product_management,
    invent_simplify_opportunities,
    think_big_vision,
    deliver_results_metrics,
    amazon_fresh_strategy,
    whole_foods_strategy,
    prime_personalization,
    advertising_strategy,
    review_strategy,
    cross_selling_strategy,
    fulfillment_strategy,
    recommendation_weighting,
    alexa_integration
FROM amazon_leadership_principles_application
WHERE low_fats = 'Y' AND recyclable = 'Y'
ORDER BY 
    CASE amazon_platform
        WHEN 'Whole Foods Market' THEN 1
        WHEN 'Amazon Fresh' THEN 2
        ELSE 3
    END,
    product_id;
```

#### 2. **Climate Pledge Friendly Product Intelligence and Environmental Impact Analytics**
```sql
-- "Comprehensive environmental impact assessment and sustainability intelligence system"

WITH environmental_intelligence_system AS (
    SELECT 
        product_id,
        low_fats,
        recyclable,
        
        -- Environmental impact scoring
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 95  -- Excellent environmental score
            WHEN recyclable = 'Y' AND low_fats = 'N' THEN 75  -- Good environmental score
            WHEN low_fats = 'Y' AND recyclable = 'N' THEN 60  -- Moderate environmental score
            ELSE 30  -- Basic environmental score
        END as environmental_score,
        
        -- Carbon footprint simulation
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Low Carbon Footprint'
            WHEN recyclable = 'Y' THEN 'Medium Carbon Footprint - Packaging Optimized'
            WHEN low_fats = 'Y' THEN 'Medium Carbon Footprint - Health Optimized'
            ELSE 'High Carbon Footprint'
        END as carbon_footprint_category,
        
        -- Packaging sustainability assessment
        CASE 
            WHEN recyclable = 'Y' THEN 'Sustainable Packaging + Recyclable Materials + Circular Economy + Waste Reduction'
            ELSE 'Standard Packaging + Improvement Opportunity + Sustainability Potential + Eco-Innovation Needed'
        END as packaging_sustainability,
        
        -- Life cycle assessment simulation
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' 
            THEN 'Excellent LCA: Sustainable sourcing + eco-processing + green packaging + responsible disposal + circular lifecycle'
            WHEN low_fats = 'Y' OR recyclable = 'Y'
            THEN 'Good LCA: Partial sustainability + improvement areas + optimization potential + targeted enhancements'
            ELSE 'Standard LCA: Traditional approach + sustainability opportunities + redesign potential + eco-improvement needed'
        END as lifecycle_assessment,
        
        -- Climate pledge friendly eligibility
        CASE 
            WHEN recyclable = 'Y' THEN 'Climate Pledge Friendly Eligible'
            ELSE 'Climate Pledge Friendly - Improvement Required'
        END as climate_pledge_status,
        
        -- Sustainability certification potential
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' 
            THEN 'Multi-Certification Ready: USDA Organic + Rainforest Alliance + Carbon Neutral + Health Certified + Sustainable'
            WHEN recyclable = 'Y'
            THEN 'Environmental Certification Ready: FSC Certified + Recyclable + Sustainable Packaging + Green Certified'
            WHEN low_fats = 'Y'
            THEN 'Health Certification Ready: Heart Healthy + Low Fat + Nutritional + Wellness Certified'
            ELSE 'Standard Certification: Quality + Safety + Basic Compliance + Traditional Standards'
        END as certification_readiness,
        
        -- Environmental innovation opportunities
        CASE 
            WHEN low_fats = 'N' AND recyclable = 'N'
            THEN 'High Innovation Potential: Complete sustainability redesign + health improvement + eco-packaging + green formulation'
            WHEN low_fats = 'N' AND recyclable = 'Y'
            THEN 'Health Innovation Potential: Nutritional improvement + maintain sustainability + wellness focus + healthy reformulation'
            WHEN low_fats = 'Y' AND recyclable = 'N'
            THEN 'Packaging Innovation Potential: Sustainable packaging + maintain health benefits + eco-materials + circular design'
            ELSE 'Enhancement Innovation: Advanced sustainability + premium eco-features + market leadership + next-generation green'
        END as innovation_potential
    FROM Products
),
sustainability_analytics_dashboard AS (
    SELECT 
        eis.*,
        
        -- Environmental impact categories
        CASE 
            WHEN environmental_score >= 90 THEN 'Category A: Exceptional Environmental Performance'
            WHEN environmental_score >= 70 THEN 'Category B: Strong Environmental Performance'
            WHEN environmental_score >= 50 THEN 'Category C: Moderate Environmental Performance'
            ELSE 'Category D: Environmental Improvement Required'
        END as environmental_category,
        
        -- Sustainability action plan
        CASE 
            WHEN environmental_score >= 90
            THEN 'Action Plan: Market leadership + innovation showcase + sustainability advocacy + industry standard setting'
            WHEN environmental_score >= 70
            THEN 'Action Plan: Continuous improvement + certification pursuit + performance optimization + competitive advantage'
            WHEN environmental_score >= 50
            THEN 'Action Plan: Targeted improvements + sustainability initiatives + gradual enhancement + performance upgrading'
            ELSE 'Action Plan: Comprehensive redesign + sustainability transformation + major improvements + eco-innovation required'
        END as sustainability_action_plan,
        
        -- Consumer environmental education
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Education: Dual impact awareness + health-environment connection + sustainable living + conscious consumption + lifestyle change'
            WHEN recyclable = 'Y'
            THEN 'Education: Environmental impact + recycling importance + waste reduction + planet protection + future generations'
            WHEN low_fats = 'Y'
            THEN 'Education: Health benefits + wellness value + nutritional advantages + personal well-being + healthy choices'
            ELSE 'Education: Product quality + value proposition + basic benefits + customer satisfaction + reliable choice'
        END as consumer_education_strategy,
        
        -- Supply chain sustainability requirements
        CASE 
            WHEN environmental_score >= 90
            THEN 'Supply Chain: 100% sustainable sourcing + carbon neutral + renewable energy + zero waste + circular economy'
            WHEN environmental_score >= 70
            THEN 'Supply Chain: Majority sustainable sourcing + carbon reduction + eco-efficiency + waste minimization + green logistics'
            WHEN environmental_score >= 50
            THEN 'Supply Chain: Partial sustainable sourcing + environmental monitoring + improvement programs + gradual transition'
            ELSE 'Supply Chain: Traditional sourcing + sustainability assessment + improvement planning + eco-transformation roadmap'
        END as supply_chain_requirements,
        
        -- Competitive environmental advantage
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Competitive Advantage: Market differentiation + premium positioning + sustainability leadership + health focus + future-ready'
            WHEN recyclable = 'Y'
            THEN 'Competitive Advantage: Environmental leadership + eco-conscious appeal + green positioning + sustainable choice + planet-friendly'
            WHEN low_fats = 'Y'
            THEN 'Competitive Advantage: Health leadership + wellness positioning + nutritional benefits + healthy lifestyle + well-being focus'
            ELSE 'Competitive Advantage: Quality focus + value proposition + reliability + customer satisfaction + traditional benefits'
        END as competitive_environmental_advantage,
        
        -- Long-term sustainability roadmap
        CASE 
            WHEN environmental_score >= 90
            THEN 'Roadmap: Innovation leadership + next-gen sustainability + industry transformation + global impact + future standards'
            WHEN environmental_score >= 70
            THEN 'Roadmap: Performance optimization + certification expansion + market leadership + continuous innovation + excellence pursuit'
            WHEN environmental_score >= 50
            THEN 'Roadmap: Systematic improvement + sustainability integration + performance upgrading + competitive positioning + gradual excellence'
            ELSE 'Roadmap: Foundation building + sustainability adoption + performance improvement + market catching-up + transformation journey'
        END as sustainability_roadmap
    FROM environmental_intelligence_system eis
),
amazon_environmental_leadership AS (
    SELECT 
        sad.*,
        
        -- Amazon Climate Pledge integration
        CASE 
            WHEN climate_pledge_status = 'Climate Pledge Friendly Eligible' AND environmental_score >= 90
            THEN 'Climate Pledge: Featured sustainability champion + carbon negative showcase + environmental leadership + net-zero pioneer'
            WHEN climate_pledge_status = 'Climate Pledge Friendly Eligible'
            THEN 'Climate Pledge: Certified sustainable product + environmental benefits + carbon reduction + sustainability choice'
            ELSE 'Climate Pledge: Improvement opportunity + sustainability roadmap + environmental enhancement + future eligibility'
        END as climate_pledge_integration,
        
        -- Amazon sustainability innovation
        CASE 
            WHEN environmental_score >= 90
            THEN 'Sustainability Innovation: Showcase product + R&D investment + innovation lab + industry collaboration + thought leadership'
            WHEN environmental_score >= 70
            THEN 'Sustainability Innovation: Development priority + improvement investment + enhancement focus + innovation opportunity'
            ELSE 'Sustainability Innovation: Transformation candidate + redesign opportunity + innovation potential + future development'
        END as amazon_sustainability_innovation,
        
        -- Environmental leadership principles
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y'
            THEN 'Environmental Leadership: Holistic sustainability + health-planet connection + future responsibility + comprehensive stewardship'
            WHEN recyclable = 'Y'
            THEN 'Environmental Leadership: Circular economy + waste reduction + resource optimization + environmental stewardship'
            WHEN low_fats = 'Y'
            THEN 'Environmental Leadership: Health sustainability + wellness responsibility + nutritional stewardship + personal well-being'
            ELSE 'Environmental Leadership: Quality stewardship + customer responsibility + operational excellence + basic environmental care'
        END as environmental_leadership_principles
    FROM sustainability_analytics_dashboard sad
)
SELECT 
    product_id,
    environmental_score,
    environmental_category,
    carbon_footprint_category,
    climate_pledge_status,
    climate_pledge_integration,
    certification_readiness,
    packaging_sustainability,
    lifecycle_assessment,
    sustainability_action_plan,
    innovation_potential,
    amazon_sustainability_innovation,
    consumer_education_strategy,
    supply_chain_requirements,
    competitive_environmental_advantage,
    sustainability_roadmap,
    environmental_leadership_principles
FROM amazon_environmental_leadership
WHERE low_fats = 'Y' AND recyclable = 'Y'
ORDER BY environmental_score DESC, product_id;
```

I'll create one more concise extension and then continue with the remaining questions:

#### 3. **Enterprise Sustainability Compliance and Regulatory Framework**
```sql
-- "Global sustainability compliance and regulatory adherence system"

WITH global_sustainability_compliance AS (
    SELECT 
        product_id,
        low_fats,
        recyclable,
        
        -- Regulatory compliance simulation
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Full Compliance: EU Green Deal + US EPA + FDA Health Standards + Global Sustainability'
            WHEN recyclable = 'Y' THEN 'Environmental Compliance: EU Circular Economy + US Recycling Standards + Global Environmental'
            WHEN low_fats = 'Y' THEN 'Health Compliance: FDA Nutritional + WHO Health Standards + Global Wellness Requirements'
            ELSE 'Basic Compliance: Standard regulatory requirements + minimum standards + traditional compliance'
        END as regulatory_compliance,
        
        -- ESG scoring and reporting
        (CASE WHEN low_fats = 'Y' THEN 30 ELSE 0 END) +
        (CASE WHEN recyclable = 'Y' THEN 40 ELSE 0 END) +
        20 as esg_score,  -- Base score of 20 for all products
        
        -- Sustainability reporting requirements
        CASE 
            WHEN low_fats = 'Y' AND recyclable = 'Y' THEN 'Comprehensive ESG + Carbon Disclosure + Health Impact + Sustainability Report'
            WHEN recyclable = 'Y' THEN 'Environmental Report + Carbon Footprint + Recycling Impact + Green Metrics'
            WHEN low_fats = 'Y' THEN 'Health Report + Nutritional Impact + Wellness Metrics + Health Outcomes'
            ELSE 'Standard Report + Basic Metrics + Traditional Reporting + Minimum Requirements'
        END as reporting_requirements
    FROM Products
)
SELECT 
    product_id,
    regulatory_compliance,
    esg_score,
    reporting_requirements
FROM global_sustainability_compliance
WHERE low_fats = 'Y' AND recyclable = 'Y'
ORDER BY esg_score DESC;
```

## 🔗 Related LeetCode Questions

1. **#1683 - Invalid Tweets** (Simple WHERE clause filtering)
2. **#1729 - Find Followers Count** (Basic filtering and aggregation)
3. **#1978 - Employees Whose Manager Left the Company** (Multiple condition filtering)
4. **#1789 - Primary Department for Each Employee** (Filtering with employee data)
5. **#1795 - Rearrange Products Table** (Product data manipulation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **AND Operator**: Both conditions must be true simultaneously
2. **ENUM Comparison**: Compare ENUM values with string literals ('Y', 'N')
3. **WHERE Clause**: Filter rows based on multiple column conditions
4. **Simple Filtering**: Basic boolean logic for data filtering

### 🚀 **Amazon Interview Tips**
1. **Explain boolean logic**: "Need both conditions true, so use AND operator"
2. **Discuss ENUM handling**: "ENUM values stored as strings, compare with 'Y'"
3. **Address business requirements**: "Both sustainability criteria must be met"
4. **Consider scalability**: "Simple filter easily handles large product catalogs"

### 🔧 **Common Patterns**
- Multiple WHERE conditions with AND operator
- ENUM value comparison with string literals
- Single table filtering operations
- Boolean logic in SQL conditions

### ⚠️ **Common Mistakes to Avoid**
1. **Using OR instead of AND** (would include products with only one property)
2. **Forgetting quotes around ENUM values** ('Y' not Y)
3. **Case sensitivity** (ensuring correct 'Y' vs 'y')
4. **Null handling** (though not applicable in this problem)

### 🔍 **Performance Considerations**
- INDEX on low_fats for fast filtering
- INDEX on recyclable for efficient lookups
- Composite index on (low_fats, recyclable) for optimal performance
- Simple conditions are highly optimizable

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Focus on products that meet health and environmental customer needs
- **Ownership**: Take responsibility for promoting sustainable and healthy products
- **Think Big**: Scale sustainable product offerings across Amazon's platform
- **Deliver Results**: Efficiently identify products that meet multiple customer criteria

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find products that are either low fat OR recyclable (not both)**
2. **Count total products by sustainability category**
3. **Calculate percentage of products that meet both criteria**
4. **Find products that need improvement (neither criteria met)**

Remember: Simple filtering operations like this form the foundation for complex product analytics that power Amazon's massive e-commerce platform and sustainability initiatives!