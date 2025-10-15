# LeetCode Easy #1729: Find Followers Count

## 📋 Problem Statement

Write a SQL query that will, for each user, return the **number of followers**.

Return the result table **ordered by user_id in ascending order**.

## 🗄️ Table Schema

**Followers Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| user_id     | int  |
| follower_id | int  |
+-------------+------+
```
- (user_id, follower_id) is the primary key for this table.
- This table contains the IDs of a user and a follower in a social media app where the follower follows the user.

## 📊 Sample Data

**Followers Table:**
| user_id | follower_id |
|---------|-------------|
| 0       | 1           |
| 1       | 0           |
| 2       | 0           |
| 2       | 1           |

**Expected Output:**
| user_id | followers_count |
|---------|-----------------|
| 0       | 1               |
| 1       | 1               |
| 2       | 2               |

**Explanation:**
- User 0 has 1 follower (user 1)
- User 1 has 1 follower (user 0)  
- User 2 has 2 followers (users 0 and 1)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Count the number of followers for each user
- Group by user_id to aggregate followers
- Use COUNT() function to count follower_id entries
- Order results by user_id

### 2. **Key Insights**
- This is a basic GROUP BY aggregation problem
- Each row represents one follower relationship
- COUNT(follower_id) gives us the follower count per user
- Need ORDER BY for consistent output

### 3. **Interview Discussion Points**
- "This is a fundamental aggregation problem using GROUP BY"
- "We count the follower relationships for each user"
- "ORDER BY ensures consistent, predictable output"

## 🔧 Step-by-Step Solution Logic

### Step 1: Group by User
```sql
-- Group followers by user_id
GROUP BY user_id
```

### Step 2: Count Followers
```sql
-- Count the number of followers for each user
COUNT(follower_id) as followers_count
```

### Step 3: Order Results
```sql
-- Order by user_id in ascending order
ORDER BY user_id
```

## ✅ Optimized SQL Solution

**Solution 1: Basic GROUP BY with COUNT**
```sql
SELECT 
    user_id,
    COUNT(follower_id) as followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

### Alternative Solutions

**Solution 2: Using COUNT(*) Instead**
```sql
SELECT 
    user_id,
    COUNT(*) as followers_count
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

**Solution 3: With Additional Analytics**
```sql
SELECT 
    user_id,
    COUNT(follower_id) as followers_count,
    MIN(follower_id) as first_follower,
    MAX(follower_id) as latest_follower
FROM Followers
GROUP BY user_id
ORDER BY user_id;
```

**Solution 4: Including Users with Zero Followers (if Users table exists)**
```sql
-- Assuming a Users table exists
SELECT 
    u.user_id,
    COALESCE(COUNT(f.follower_id), 0) as followers_count
FROM Users u
LEFT JOIN Followers f ON u.user_id = f.user_id
GROUP BY u.user_id
ORDER BY u.user_id;
```

**Solution 5: Comprehensive Social Media Analytics System**
```sql
WITH follower_analytics AS (
    SELECT 
        user_id,
        follower_id,
        
        -- Simulate timestamp for follower relationship
        CASE 
            WHEN follower_id % 3 = 0 THEN '2024-01-01'
            WHEN follower_id % 3 = 1 THEN '2024-02-01'
            ELSE '2024-03-01'
        END as follow_date,
        
        -- Simulate follower characteristics
        CASE 
            WHEN follower_id % 4 = 0 THEN 'Active Follower'
            WHEN follower_id % 4 = 1 THEN 'Casual Follower'
            WHEN follower_id % 4 = 2 THEN 'Influential Follower'
            ELSE 'New Follower'
        END as follower_type,
        
        -- Simulate engagement level
        CASE 
            WHEN follower_id % 5 = 0 THEN 'High Engagement'
            WHEN follower_id % 5 = 1 THEN 'Medium Engagement'
            WHEN follower_id % 5 = 2 THEN 'Low Engagement'
            ELSE 'Minimal Engagement'
        END as engagement_level,
        
        -- Geographic simulation
        CASE 
            WHEN follower_id % 6 = 0 THEN 'North America'
            WHEN follower_id % 6 = 1 THEN 'Europe'
            WHEN follower_id % 6 = 2 THEN 'Asia Pacific'
            WHEN follower_id % 6 = 3 THEN 'Latin America'
            WHEN follower_id % 6 = 4 THEN 'Middle East'
            ELSE 'Africa'
        END as follower_region
    FROM Followers
),
user_influence_metrics AS (
    SELECT 
        user_id,
        
        -- Basic follower metrics
        COUNT(*) as total_followers,
        COUNT(DISTINCT follower_id) as unique_followers,
        
        -- Follower type distribution
        COUNT(CASE WHEN follower_type = 'Active Follower' THEN 1 END) as active_followers,
        COUNT(CASE WHEN follower_type = 'Casual Follower' THEN 1 END) as casual_followers,
        COUNT(CASE WHEN follower_type = 'Influential Follower' THEN 1 END) as influential_followers,
        COUNT(CASE WHEN follower_type = 'New Follower' THEN 1 END) as new_followers,
        
        -- Engagement distribution
        COUNT(CASE WHEN engagement_level = 'High Engagement' THEN 1 END) as high_engagement_followers,
        COUNT(CASE WHEN engagement_level = 'Medium Engagement' THEN 1 END) as medium_engagement_followers,
        COUNT(CASE WHEN engagement_level = 'Low Engagement' THEN 1 END) as low_engagement_followers,
        COUNT(CASE WHEN engagement_level = 'Minimal Engagement' THEN 1 END) as minimal_engagement_followers,
        
        -- Geographic distribution
        COUNT(DISTINCT follower_region) as global_reach,
        COUNT(CASE WHEN follower_region = 'North America' THEN 1 END) as north_america_followers,
        COUNT(CASE WHEN follower_region = 'Europe' THEN 1 END) as europe_followers,
        COUNT(CASE WHEN follower_region = 'Asia Pacific' THEN 1 END) as asia_pacific_followers,
        
        -- Temporal analysis
        COUNT(CASE WHEN follow_date = '2024-01-01' THEN 1 END) as q1_followers,
        COUNT(CASE WHEN follow_date = '2024-02-01' THEN 1 END) as q2_followers,
        COUNT(CASE WHEN follow_date = '2024-03-01' THEN 1 END) as q3_followers,
        
        -- Influence scoring
        (COUNT(CASE WHEN follower_type = 'Influential Follower' THEN 1 END) * 5) +
        (COUNT(CASE WHEN engagement_level = 'High Engagement' THEN 1 END) * 3) +
        (COUNT(CASE WHEN follower_type = 'Active Follower' THEN 1 END) * 2) +
        COUNT(*) as influence_score,
        
        -- Growth trend simulation
        CASE 
            WHEN COUNT(CASE WHEN follow_date = '2024-03-01' THEN 1 END) > 
                 COUNT(CASE WHEN follow_date = '2024-02-01' THEN 1 END)
            THEN 'Growing'
            WHEN COUNT(CASE WHEN follow_date = '2024-03-01' THEN 1 END) = 
                 COUNT(CASE WHEN follow_date = '2024-02-01' THEN 1 END)
            THEN 'Stable'
            ELSE 'Declining'
        END as growth_trend
    FROM follower_analytics
    GROUP BY user_id
),
social_media_insights AS (
    SELECT 
        uim.*,
        
        -- User classification based on follower count
        CASE 
            WHEN total_followers >= 10 THEN 'Mega Influencer'
            WHEN total_followers >= 5 THEN 'Macro Influencer'
            WHEN total_followers >= 3 THEN 'Micro Influencer'
            WHEN total_followers >= 2 THEN 'Rising Influencer'
            WHEN total_followers = 1 THEN 'Emerging User'
            ELSE 'New User'
        END as user_tier,
        
        -- Engagement quality assessment
        CASE 
            WHEN high_engagement_followers * 100.0 / total_followers >= 60
            THEN 'Excellent Engagement Quality'
            WHEN high_engagement_followers * 100.0 / total_followers >= 40
            THEN 'Good Engagement Quality'
            WHEN high_engagement_followers * 100.0 / total_followers >= 20
            THEN 'Average Engagement Quality'
            ELSE 'Poor Engagement Quality'
        END as engagement_quality,
        
        -- Global reach assessment
        CASE 
            WHEN global_reach >= 5 THEN 'Global Presence'
            WHEN global_reach >= 3 THEN 'Multi-Regional Presence'
            WHEN global_reach >= 2 THEN 'Regional Presence'
            ELSE 'Local Presence'
        END as reach_classification,
        
        -- Growth potential assessment
        CASE 
            WHEN growth_trend = 'Growing' AND influential_followers > 0
            THEN 'High Growth Potential - Influential Network'
            WHEN growth_trend = 'Growing' AND new_followers > 0
            THEN 'High Growth Potential - New Audience'
            WHEN growth_trend = 'Stable' AND high_engagement_followers > 0
            THEN 'Medium Growth Potential - Engaged Audience'
            WHEN growth_trend = 'Declining'
            THEN 'Low Growth Potential - Audience Decline'
            ELSE 'Moderate Growth Potential - Standard Growth'
        END as growth_potential,
        
        -- Content strategy recommendations
        CASE 
            WHEN user_tier LIKE '%Influencer%' AND engagement_quality = 'Excellent Engagement Quality'
            THEN 'Premium Strategy: Thought leadership + brand partnerships + exclusive content'
            WHEN total_followers >= 3 AND global_reach >= 3
            THEN 'Global Strategy: Multi-cultural content + timezone optimization + regional customization'
            WHEN high_engagement_followers > casual_followers
            THEN 'Engagement Strategy: Interactive content + community building + audience participation'
            WHEN new_followers > 0 AND growth_trend = 'Growing'
            THEN 'Growth Strategy: Consistent posting + trending topics + audience expansion'
            ELSE 'Foundation Strategy: Quality content + audience building + platform optimization'
        END as content_strategy_recommendation,
        
        -- Monetization potential
        CASE 
            WHEN user_tier = 'Mega Influencer' AND engagement_quality = 'Excellent Engagement Quality'
            THEN 'High Monetization: Premium sponsorships + product launches + exclusive partnerships'
            WHEN user_tier LIKE '%Influencer%' AND influential_followers > 0
            THEN 'Medium Monetization: Brand collaborations + affiliate marketing + sponsored content'
            WHEN total_followers >= 2 AND high_engagement_followers > 0
            THEN 'Low Monetization: Niche partnerships + product placements + community monetization'
            ELSE 'Early Stage: Focus on audience building before monetization'
        END as monetization_potential
    FROM user_influence_metrics uim
),
competitive_analysis AS (
    SELECT 
        smi.*,
        
        -- Competitive positioning
        RANK() OVER (ORDER BY total_followers DESC) as follower_rank,
        RANK() OVER (ORDER BY influence_score DESC) as influence_rank,
        RANK() OVER (ORDER BY high_engagement_followers DESC) as engagement_rank,
        RANK() OVER (ORDER BY global_reach DESC) as reach_rank,
        
        -- Percentile analysis
        PERCENT_RANK() OVER (ORDER BY total_followers) as follower_percentile,
        PERCENT_RANK() OVER (ORDER BY influence_score) as influence_percentile,
        
        -- Market position assessment
        CASE 
            WHEN RANK() OVER (ORDER BY total_followers DESC) = 1
            THEN 'Market Leader - Highest follower count'
            WHEN RANK() OVER (ORDER BY influence_score DESC) = 1
            THEN 'Influence Leader - Highest influence score'
            WHEN RANK() OVER (ORDER BY high_engagement_followers DESC) = 1
            THEN 'Engagement Leader - Most engaged audience'
            WHEN PERCENT_RANK() OVER (ORDER BY total_followers) >= 0.8
            THEN 'Top Tier - Top 20% of users'
            WHEN PERCENT_RANK() OVER (ORDER BY total_followers) >= 0.5
            THEN 'Mid Tier - Top 50% of users'
            ELSE 'Growing Tier - Building audience base'
        END as market_position,
        
        -- Strategic recommendations
        CASE 
            WHEN follower_rank <= 2 AND growth_trend = 'Declining'
            THEN 'Strategic Priority: Audience retention + content refresh + engagement revival'
            WHEN influence_rank <= 2 AND engagement_quality != 'Excellent Engagement Quality'
            THEN 'Strategic Priority: Engagement optimization + community management + value delivery'
            WHEN growth_trend = 'Growing' AND global_reach < 3
            THEN 'Strategic Priority: Geographic expansion + international content + market penetration'
            WHEN user_tier LIKE '%Influencer%' AND monetization_potential = 'Early Stage'
            THEN 'Strategic Priority: Monetization readiness + brand partnerships + revenue diversification'
            ELSE 'Strategic Priority: Steady growth + content quality + audience satisfaction'
        END as strategic_recommendation
    FROM social_media_insights smi
)
SELECT 
    user_id,
    total_followers as followers_count,
    user_tier,
    engagement_quality,
    reach_classification,
    growth_potential,
    market_position,
    content_strategy_recommendation,
    monetization_potential,
    strategic_recommendation,
    
    -- Detailed metrics
    active_followers,
    influential_followers,
    high_engagement_followers,
    global_reach,
    influence_score,
    growth_trend,
    
    -- Competitive context
    follower_rank,
    influence_rank,
    follower_percentile,
    influence_percentile
FROM competitive_analysis
ORDER BY user_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Social Commerce and Influencer Partnership Platform**
```sql
-- "Build comprehensive social commerce platform with influencer partnership management and performance analytics"

WITH influencer_commerce_analysis AS (
    SELECT 
        f.user_id,
        COUNT(f.follower_id) as followers_count,
        
        -- Commerce potential simulation
        CASE 
            WHEN f.user_id % 3 = 0 THEN 'E-commerce Merchant'
            WHEN f.user_id % 3 = 1 THEN 'Content Creator'
            ELSE 'Consumer Influencer'
        END as user_category,
        
        -- Simulate purchase behavior influence
        CASE 
            WHEN COUNT(f.follower_id) >= 3 THEN 'High Purchase Influence'
            WHEN COUNT(f.follower_id) >= 2 THEN 'Medium Purchase Influence'
            WHEN COUNT(f.follower_id) >= 1 THEN 'Low Purchase Influence'
            ELSE 'Minimal Purchase Influence'
        END as purchase_influence_level,
        
        -- Amazon ecosystem integration potential
        CASE 
            WHEN f.user_id % 4 = 0 THEN 'Amazon Prime Video Creator'
            WHEN f.user_id % 4 = 1 THEN 'Amazon Live Streamer'
            WHEN f.user_id % 4 = 2 THEN 'Amazon Affiliate Partner'
            ELSE 'Amazon Customer Reviewer'
        END as amazon_ecosystem_role,
        
        -- Product category affinity simulation
        CASE 
            WHEN f.user_id % 5 = 0 THEN 'Electronics & Tech'
            WHEN f.user_id % 5 = 1 THEN 'Fashion & Beauty'
            WHEN f.user_id % 5 = 2 THEN 'Home & Garden'
            WHEN f.user_id % 5 = 3 THEN 'Sports & Outdoors'
            ELSE 'Books & Education'
        END as product_affinity,
        
        -- Follower purchasing power estimation
        COUNT(f.follower_id) * CASE 
            WHEN f.user_id % 3 = 0 THEN 150  -- High value followers
            WHEN f.user_id % 3 = 1 THEN 100  -- Medium value followers
            ELSE 75  -- Standard value followers
        END as estimated_follower_value_usd,
        
        -- Social commerce conversion potential
        ROUND(COUNT(f.follower_id) * 0.15, 2) as estimated_conversion_rate_percent
    FROM Followers f
    GROUP BY f.user_id
),
amazon_partnership_optimization AS (
    SELECT 
        ica.*,
        
        -- Amazon partnership tier classification
        CASE 
            WHEN followers_count >= 3 AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Amazon Premium Partner - Tier 1'
            WHEN followers_count >= 2 AND purchase_influence_level IN ('High Purchase Influence', 'Medium Purchase Influence')
            THEN 'Amazon Standard Partner - Tier 2'
            WHEN followers_count >= 1 AND purchase_influence_level != 'Minimal Purchase Influence'
            THEN 'Amazon Emerging Partner - Tier 3'
            ELSE 'Amazon Community Member - Standard'
        END as amazon_partnership_tier,
        
        -- Commission structure recommendations
        CASE 
            WHEN amazon_partnership_tier = 'Amazon Premium Partner - Tier 1'
            THEN 'Premium Commission: 8-12% base + performance bonuses + exclusive product access'
            WHEN amazon_partnership_tier = 'Amazon Standard Partner - Tier 2'
            THEN 'Standard Commission: 5-8% base + volume bonuses + early product access'
            WHEN amazon_partnership_tier = 'Amazon Emerging Partner - Tier 3'
            THEN 'Emerging Commission: 3-5% base + growth bonuses + training support'
            ELSE 'Community Commission: 2-3% base + standard affiliate rates'
        END as commission_structure,
        
        -- Product promotion strategy
        CASE 
            WHEN product_affinity = 'Electronics & Tech' AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Tech Strategy: Latest gadgets + exclusive launches + tech reviews + innovation showcases'
            WHEN product_affinity = 'Fashion & Beauty' AND amazon_ecosystem_role = 'Amazon Live Streamer'
            THEN 'Fashion Strategy: Live styling sessions + seasonal collections + beauty tutorials + trend showcases'
            WHEN product_affinity = 'Home & Garden' AND followers_count >= 2
            THEN 'Home Strategy: Room makeovers + seasonal decor + home improvement + lifestyle content'
            WHEN product_affinity = 'Sports & Outdoors' AND estimated_conversion_rate_percent > 0.20
            THEN 'Sports Strategy: Fitness equipment + outdoor gear + seasonal sports + health & wellness'
            ELSE 'General Strategy: Diverse product mix + seasonal promotions + customer education + value focus'
        END as product_promotion_strategy,
        
        -- Content collaboration opportunities
        CASE 
            WHEN amazon_ecosystem_role = 'Amazon Prime Video Creator' AND followers_count >= 2
            THEN 'Prime Video Collaboration: Exclusive content + behind-the-scenes + product placements + creator funds'
            WHEN amazon_ecosystem_role = 'Amazon Live Streamer'
            THEN 'Live Commerce: Real-time product demos + interactive shopping + live Q&A + instant purchases'
            WHEN amazon_ecosystem_role = 'Amazon Affiliate Partner' AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Affiliate Excellence: Advanced analytics + conversion optimization + A/B testing + performance coaching'
            WHEN amazon_ecosystem_role = 'Amazon Customer Reviewer'
            THEN 'Review Enhancement: Verified purchases + detailed reviews + video testimonials + community building'
            ELSE 'Standard Collaboration: Content guidelines + promotional materials + basic analytics + community support'
        END as collaboration_opportunity,
        
        -- Revenue projection (monthly)
        CASE 
            WHEN amazon_partnership_tier = 'Amazon Premium Partner - Tier 1'
            THEN estimated_follower_value_usd * 0.08 * estimated_conversion_rate_percent / 100
            WHEN amazon_partnership_tier = 'Amazon Standard Partner - Tier 2'
            THEN estimated_follower_value_usd * 0.06 * estimated_conversion_rate_percent / 100
            WHEN amazon_partnership_tier = 'Amazon Emerging Partner - Tier 3'
            THEN estimated_follower_value_usd * 0.04 * estimated_conversion_rate_percent / 100
            ELSE estimated_follower_value_usd * 0.025 * estimated_conversion_rate_percent / 100
        END as projected_monthly_revenue_usd,
        
        -- Growth acceleration programs
        CASE 
            WHEN followers_count >= 3 AND amazon_partnership_tier LIKE '%Premium%'
            THEN 'Premium Growth: Dedicated account manager + exclusive events + advanced training + priority support'
            WHEN followers_count >= 2 AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Accelerated Growth: Group coaching + best practice sharing + performance workshops + peer networking'
            WHEN amazon_ecosystem_role = 'Amazon Live Streamer' OR amazon_ecosystem_role = 'Amazon Prime Video Creator'
            THEN 'Creator Growth: Content optimization + production support + technical assistance + creative resources'
            ELSE 'Standard Growth: Online training + resource library + community forums + basic support'
        END as growth_acceleration_program
    FROM influencer_commerce_analysis ica
),
social_commerce_ecosystem AS (
    SELECT 
        apo.*,
        
        -- Ecosystem synergy opportunities
        CASE 
            WHEN amazon_ecosystem_role = 'Amazon Prime Video Creator' AND product_affinity = 'Electronics & Tech'
            THEN 'Tech Content Synergy: Product reviews in video content + tech tutorials + gadget unboxings + innovation discussions'
            WHEN amazon_ecosystem_role = 'Amazon Live Streamer' AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Live Commerce Synergy: Real-time product demonstrations + live shopping events + interactive commerce + instant gratification'
            WHEN amazon_ecosystem_role = 'Amazon Affiliate Partner' AND followers_count >= 3
            THEN 'Affiliate Scale Synergy: Large audience monetization + conversion optimization + multi-channel promotion + revenue maximization'
            WHEN amazon_ecosystem_role = 'Amazon Customer Reviewer' AND amazon_partnership_tier LIKE '%Premium%'
            THEN 'Review Authority Synergy: Trusted product opinions + detailed evaluations + community trust + purchase guidance'
            ELSE 'Standard Synergy: Basic affiliate integration + standard promotions + community engagement + brand awareness'
        END as ecosystem_synergy,
        
        -- Cross-platform integration strategy
        CASE 
            WHEN followers_count >= 3 AND estimated_follower_value_usd >= 450
            THEN 'Multi-Platform Integration: Amazon + Instagram + TikTok + YouTube + unified analytics + cross-promotion'
            WHEN followers_count >= 2 AND purchase_influence_level = 'High Purchase Influence'
            THEN 'Dual-Platform Integration: Amazon + primary social platform + coordinated campaigns + shared analytics'
            WHEN amazon_ecosystem_role IN ('Amazon Live Streamer', 'Amazon Prime Video Creator')
            THEN 'Amazon-Centric Integration: Deep Amazon ecosystem integration + platform-specific optimization + exclusive features'
            ELSE 'Single-Platform Focus: Amazon affiliate optimization + platform mastery + audience building + conversion focus'
        END as cross_platform_strategy,
        
        -- Customer experience enhancement
        CASE 
            WHEN purchase_influence_level = 'High Purchase Influence' AND estimated_conversion_rate_percent > 0.20
            THEN 'Premium Experience: Curated product recommendations + exclusive deals + personal shopping assistance + VIP treatment'
            WHEN amazon_partnership_tier LIKE '%Premium%' OR amazon_partnership_tier LIKE '%Standard%'
            THEN 'Enhanced Experience: Quality product selection + detailed reviews + comparison guides + educational content'
            WHEN followers_count >= 2
            THEN 'Community Experience: Group buying opportunities + follower-exclusive deals + community discussions + shared experiences'
            ELSE 'Standard Experience: Basic product promotion + standard affiliate links + general recommendations + accessible pricing'
        END as customer_experience_strategy,
        
        -- Innovation and technology integration
        CASE 
            WHEN amazon_ecosystem_role = 'Amazon Live Streamer' AND product_affinity = 'Electronics & Tech'
            THEN 'Tech Innovation: AR product visualization + AI recommendations + voice commerce + smart home integration'
            WHEN followers_count >= 3 AND amazon_partnership_tier = 'Amazon Premium Partner - Tier 1'
            THEN 'Premium Innovation: Advanced analytics + predictive recommendations + personalization + automation tools'
            WHEN purchase_influence_level = 'High Purchase Influence'
            THEN 'Influence Innovation: Social proof technology + peer recommendations + trust signals + community validation'
            ELSE 'Standard Innovation: Basic tracking + standard analytics + conventional promotion + traditional methods'
        END as innovation_integration,
        
        -- Long-term partnership development
        CASE 
            WHEN projected_monthly_revenue_usd >= 50 AND amazon_partnership_tier = 'Amazon Premium Partner - Tier 1'
            THEN 'Strategic Partnership: Exclusive brand collaborations + product development input + market research participation + executive access'
            WHEN projected_monthly_revenue_usd >= 25 AND followers_count >= 2
            THEN 'Growth Partnership: Expanded promotional opportunities + advanced tools + priority support + development programs'
            WHEN amazon_ecosystem_role IN ('Amazon Prime Video Creator', 'Amazon Live Streamer')
            THEN 'Content Partnership: Creative collaboration + content funding + production support + platform integration'
            ELSE 'Community Partnership: Standard affiliate relationship + basic support + community participation + steady growth'
        END as partnership_development_path
    FROM amazon_partnership_optimization apo
)
SELECT 
    user_id,
    followers_count,
    user_category,
    amazon_partnership_tier,
    product_affinity,
    purchase_influence_level,
    commission_structure,
    product_promotion_strategy,
    collaboration_opportunity,
    ecosystem_synergy,
    cross_platform_strategy,
    customer_experience_strategy,
    innovation_integration,
    partnership_development_path,
    
    -- Financial projections
    estimated_follower_value_usd,
    estimated_conversion_rate_percent,
    projected_monthly_revenue_usd,
    
    -- Program details
    amazon_ecosystem_role,
    growth_acceleration_program
FROM social_commerce_ecosystem
ORDER BY 
    projected_monthly_revenue_usd DESC,
    followers_count DESC,
    user_id;
```

#### 2. **Real-time Social Media Analytics and Engagement Intelligence**
```sql
-- "Implement real-time social media analytics platform with engagement intelligence and predictive insights"

WITH real_time_engagement_analysis AS (
    SELECT 
        f.user_id,
        COUNT(f.follower_id) as followers_count,
        
        -- Real-time engagement simulation
        CURRENT_TIMESTAMP as analysis_timestamp,
        
        -- Engagement velocity indicators
        CASE 
            WHEN COUNT(f.follower_id) >= 3 THEN 'High Velocity'
            WHEN COUNT(f.follower_id) >= 2 THEN 'Medium Velocity'
            WHEN COUNT(f.follower_id) >= 1 THEN 'Low Velocity'
            ELSE 'Minimal Velocity'
        END as engagement_velocity,
        
        -- Content virality potential
        CASE 
            WHEN f.user_id % 2 = 0 AND COUNT(f.follower_id) >= 2 THEN 'High Virality Potential'
            WHEN COUNT(f.follower_id) >= 2 THEN 'Medium Virality Potential'
            WHEN COUNT(f.follower_id) >= 1 THEN 'Low Virality Potential'
            ELSE 'Minimal Virality Potential'
        END as virality_potential,
        
        -- Audience quality indicators
        CASE 
            WHEN f.user_id % 3 = 0 THEN 'High Quality Audience'
            WHEN f.user_id % 3 = 1 THEN 'Medium Quality Audience'
            ELSE 'Standard Quality Audience'
        END as audience_quality,
        
        -- Real-time activity simulation
        CASE 
            WHEN f.user_id % 4 = 0 THEN 'Currently Active'
            WHEN f.user_id % 4 = 1 THEN 'Recently Active'
            WHEN f.user_id % 4 = 2 THEN 'Moderately Active'
            ELSE 'Low Activity'
        END as current_activity_level,
        
        -- Social proof strength
        COUNT(f.follower_id) * CASE 
            WHEN f.user_id % 2 = 0 THEN 1.5  -- Enhanced social proof
            ELSE 1.0  -- Standard social proof
        END as social_proof_score,
        
        -- Trend alignment simulation
        CASE 
            WHEN f.user_id % 5 = 0 THEN 'Trending Content'
            WHEN f.user_id % 5 = 1 THEN 'Rising Trend'
            WHEN f.user_id % 5 = 2 THEN 'Stable Trend'
            WHEN f.user_id % 5 = 3 THEN 'Declining Trend'
            ELSE 'Niche Content'
        END as trend_alignment
    FROM Followers f
    GROUP BY f.user_id
),
predictive_engagement_intelligence AS (
    SELECT 
        rtea.*,
        
        -- Engagement prediction algorithms
        CASE 
            WHEN engagement_velocity = 'High Velocity' AND audience_quality = 'High Quality Audience'
            THEN 'Prediction: Viral breakthrough imminent + exponential growth expected'
            WHEN virality_potential = 'High Virality Potential' AND trend_alignment = 'Trending Content'
            THEN 'Prediction: High engagement surge + trend amplification likely'
            WHEN followers_count >= 3 AND current_activity_level = 'Currently Active'
            THEN 'Prediction: Sustained engagement + community growth + influence expansion'
            WHEN social_proof_score >= 3 AND audience_quality != 'Standard Quality Audience'
            THEN 'Prediction: Steady engagement growth + organic reach increase'
            ELSE 'Prediction: Standard engagement pattern + gradual audience building'
        END as engagement_prediction,
        
        -- Content optimization recommendations
        CASE 
            WHEN virality_potential = 'High Virality Potential' AND trend_alignment = 'Trending Content'
            THEN 'Optimization: Capitalize on trending topics + increase posting frequency + leverage hashtags + time for peak hours'
            WHEN engagement_velocity = 'High Velocity' AND current_activity_level = 'Currently Active'
            THEN 'Optimization: Maintain momentum + engage with audience + create interactive content + build on current success'
            WHEN audience_quality = 'High Quality Audience' AND followers_count >= 2
            THEN 'Optimization: Premium content strategy + exclusive access + community building + value-driven approach'
            WHEN social_proof_score >= 2 AND trend_alignment != 'Declining Trend'
            THEN 'Optimization: Leverage social proof + encourage sharing + create shareable content + build advocacy'
            ELSE 'Optimization: Focus on quality + consistency + audience engagement + gradual growth'
        END as content_optimization,
        
        -- Real-time intervention triggers
        CASE 
            WHEN engagement_velocity = 'High Velocity' AND virality_potential = 'High Virality Potential'
            THEN 'IMMEDIATE ACTION: Boost content promotion + increase resources + maximize reach + capitalize on momentum'
            WHEN trend_alignment = 'Trending Content' AND current_activity_level = 'Currently Active'
            THEN 'URGENT ACTION: Join trend conversation + create timely content + engage with trend + ride the wave'
            WHEN followers_count >= 3 AND audience_quality = 'High Quality Audience'
            THEN 'PRIORITY ACTION: Nurture high-value audience + exclusive content + premium engagement + relationship building'
            WHEN social_proof_score >= 4
            THEN 'STRATEGIC ACTION: Leverage social proof + amplify reach + encourage advocacy + build authority'
            ELSE 'STANDARD ACTION: Continue consistent strategy + monitor performance + gradual optimization'
        END as real_time_intervention,
        
        -- Algorithmic boost opportunities
        CASE 
            WHEN engagement_velocity = 'High Velocity' AND current_activity_level = 'Currently Active'
            THEN 'Algorithm Boost: Maximum visibility + trending feed placement + discovery enhancement + reach amplification'
            WHEN virality_potential = 'High Virality Potential' AND trend_alignment IN ('Trending Content', 'Rising Trend')
            THEN 'Algorithm Favor: Enhanced distribution + trend algorithm inclusion + increased impressions + organic boost'
            WHEN audience_quality = 'High Quality Audience' AND social_proof_score >= 3
            THEN 'Algorithm Support: Quality signal boost + engagement prioritization + community amplification + authority recognition'
            WHEN followers_count >= 2 AND trend_alignment != 'Declining Trend'
            THEN 'Algorithm Standard: Normal distribution + standard recommendations + typical reach + baseline visibility'
            ELSE 'Algorithm Neutral: Standard processing + organic growth + community building + gradual discovery'
        END as algorithmic_treatment,
        
        -- Monetization readiness assessment
        CASE 
            WHEN followers_count >= 3 AND engagement_velocity = 'High Velocity' AND audience_quality = 'High Quality Audience'
            THEN 'Monetization Ready: High-value audience + strong engagement + premium partnerships + immediate revenue potential'
            WHEN social_proof_score >= 3 AND virality_potential = 'High Virality Potential'
            THEN 'Monetization Emerging: Growing influence + viral potential + partnership opportunities + developing revenue streams'
            WHEN followers_count >= 2 AND current_activity_level IN ('Currently Active', 'Recently Active')
            THEN 'Monetization Building: Active audience + engagement foundation + early partnerships + revenue preparation'
            WHEN audience_quality = 'High Quality Audience'
            THEN 'Monetization Potential: Quality audience + niche influence + targeted partnerships + specialized revenue'
            ELSE 'Monetization Future: Focus on growth + audience building + engagement development + foundation setting'
        END as monetization_readiness
    FROM real_time_engagement_analysis rtea
),
engagement_intelligence_dashboard AS (
    SELECT 
        pei.*,
        
        -- Real-time dashboard metrics
        COUNT(*) OVER () as total_users_analyzed,
        COUNT(*) OVER (PARTITION BY engagement_velocity) as velocity_distribution,
        COUNT(*) OVER (PARTITION BY virality_potential) as virality_distribution,
        COUNT(*) OVER (PARTITION BY current_activity_level) as activity_distribution,
        
        -- Performance benchmarking
        AVG(followers_count) OVER () as platform_avg_followers,
        AVG(social_proof_score) OVER () as platform_avg_social_proof,
        RANK() OVER (ORDER BY followers_count DESC) as follower_rank,
        RANK() OVER (ORDER BY social_proof_score DESC) as influence_rank,
        
        -- Trend analysis indicators
        CASE 
            WHEN trend_alignment = 'Trending Content' AND followers_count >= platform_avg_followers
            THEN 'Trend Leader: Above-average influence riding trending wave'
            WHEN trend_alignment = 'Rising Trend' AND engagement_velocity = 'High Velocity'
            THEN 'Early Adopter: High engagement on emerging trends'
            WHEN trend_alignment = 'Stable Trend' AND audience_quality = 'High Quality Audience'
            THEN 'Consistent Performer: Quality audience with stable content approach'
            WHEN trend_alignment = 'Declining Trend'
            THEN 'Repositioning Needed: Content strategy refresh required'
            ELSE 'Standard Performer: Typical trend engagement patterns'
        END as trend_analysis,
        
        -- Competitive intelligence
        CASE 
            WHEN RANK() OVER (ORDER BY followers_count DESC) = 1
            THEN 'Market Position: Follower count leader + competitive advantage + market dominance'
            WHEN RANK() OVER (ORDER BY social_proof_score DESC) = 1
            THEN 'Market Position: Influence leader + social proof strength + thought leadership'
            WHEN followers_count >= platform_avg_followers AND engagement_velocity = 'High Velocity'
            THEN 'Market Position: Above-average performer + growth momentum + competitive strength'
            WHEN PERCENT_RANK() OVER (ORDER BY followers_count) >= 0.7
            THEN 'Market Position: Top tier performer + strong market position + growth potential'
            ELSE 'Market Position: Building market presence + growth opportunity + development stage'
        END as competitive_position,
        
        -- Strategic priority classification
        CASE 
            WHEN engagement_prediction LIKE '%imminent%' OR engagement_prediction LIKE '%surge%'
            THEN 'STRATEGIC PRIORITY 1: High-impact opportunity + immediate action + resource allocation + maximum support'
            WHEN monetization_readiness = 'Monetization Ready' OR monetization_readiness = 'Monetization Emerging'
            THEN 'STRATEGIC PRIORITY 2: Revenue opportunity + partnership development + commercial focus + value capture'
            WHEN real_time_intervention LIKE 'IMMEDIATE ACTION%' OR real_time_intervention LIKE 'URGENT ACTION%'
            THEN 'STRATEGIC PRIORITY 3: Time-sensitive opportunity + rapid response + tactical execution + momentum capture'
            WHEN audience_quality = 'High Quality Audience' AND followers_count >= 2
            THEN 'STRATEGIC PRIORITY 4: Quality audience development + relationship building + long-term value + retention focus'
            ELSE 'STRATEGIC PRIORITY 5: Foundation building + steady growth + consistent strategy + patience and persistence'
        END as strategic_priority
    FROM predictive_engagement_intelligence pei
)
SELECT 
    user_id,
    followers_count,
    engagement_velocity,
    virality_potential,
    audience_quality,
    current_activity_level,
    social_proof_score,
    trend_alignment,
    engagement_prediction,
    content_optimization,
    real_time_intervention,
    algorithmic_treatment,
    monetization_readiness,
    trend_analysis,
    competitive_position,
    strategic_priority,
    
    -- Benchmarking context
    platform_avg_followers,
    platform_avg_social_proof,
    follower_rank,
    influence_rank,
    
    -- Distribution context
    velocity_distribution,
    virality_distribution,
    activity_distribution
FROM engagement_intelligence_dashboard
ORDER BY 
    CASE strategic_priority
        WHEN 'STRATEGIC PRIORITY 1: High-impact opportunity + immediate action + resource allocation + maximum support' THEN 1
        WHEN 'STRATEGIC PRIORITY 2: Revenue opportunity + partnership development + commercial focus + value capture' THEN 2
        WHEN 'STRATEGIC PRIORITY 3: Time-sensitive opportunity + rapid response + tactical execution + momentum capture' THEN 3
        WHEN 'STRATEGIC PRIORITY 4: Quality audience development + relationship building + long-term value + retention focus' THEN 4
        ELSE 5
    END,
    social_proof_score DESC,
    followers_count DESC;
```

#### 3. **Enterprise Social Media Management and Brand Intelligence Platform**
```sql
-- "Create enterprise social media management platform with brand intelligence, competitive analysis, and strategic insights"

WITH enterprise_social_intelligence AS (
    SELECT 
        f.user_id,
        COUNT(f.follower_id) as followers_count,
        
        -- Enterprise user classification
        CASE 
            WHEN f.user_id % 6 = 0 THEN 'Fortune 500 Brand'
            WHEN f.user_id % 6 = 1 THEN 'Emerging Brand'
            WHEN f.user_id % 6 = 2 THEN 'B2B Enterprise'
            WHEN f.user_id % 6 = 3 THEN 'Consumer Brand'
            WHEN f.user_id % 6 = 4 THEN 'Tech Startup'
            ELSE 'Local Business'
        END as enterprise_category,
        
        -- Industry vertical simulation
        CASE 
            WHEN f.user_id % 8 = 0 THEN 'Technology'
            WHEN f.user_id % 8 = 1 THEN 'Retail & E-commerce'
            WHEN f.user_id % 8 = 2 THEN 'Financial Services'
            WHEN f.user_id % 8 = 3 THEN 'Healthcare'
            WHEN f.user_id % 8 = 4 THEN 'Manufacturing'
            WHEN f.user_id % 8 = 5 THEN 'Media & Entertainment'
            WHEN f.user_id % 8 = 6 THEN 'Education'
            ELSE 'Professional Services'
        END as industry_vertical,
        
        -- Brand maturity assessment
        CASE 
            WHEN COUNT(f.follower_id) >= 3 THEN 'Established Brand'
            WHEN COUNT(f.follower_id) >= 2 THEN 'Growing Brand'
            WHEN COUNT(f.follower_id) >= 1 THEN 'Emerging Brand'
            ELSE 'Startup Brand'
        END as brand_maturity,
        
        -- Market influence scoring
        COUNT(f.follower_id) * CASE 
            WHEN f.user_id % 6 = 0 THEN 3.0  -- Fortune 500 multiplier
            WHEN f.user_id % 6 = 2 THEN 2.5  -- B2B Enterprise multiplier
            WHEN f.user_id % 6 = 4 THEN 2.0  -- Tech Startup multiplier
            ELSE 1.5  -- Standard multiplier
        END as market_influence_score,
        
        -- Digital transformation readiness
        CASE 
            WHEN f.user_id % 4 = 0 THEN 'Digital Native'
            WHEN f.user_id % 4 = 1 THEN 'Digital Advanced'
            WHEN f.user_id % 4 = 2 THEN 'Digital Adopter'
            ELSE 'Digital Learning'
        END as digital_readiness,
        
        -- Brand risk assessment
        CASE 
            WHEN COUNT(f.follower_id) >= 3 AND f.user_id % 6 = 0 THEN 'High Visibility Risk'
            WHEN COUNT(f.follower_id) >= 2 THEN 'Medium Visibility Risk'
            WHEN COUNT(f.follower_id) >= 1 THEN 'Low Visibility Risk'
            ELSE 'Minimal Visibility Risk'
        END as brand_risk_level
    FROM Followers f
    GROUP BY f.user_id
),
competitive_brand_analysis AS (
    SELECT 
        esi.*,
        
        -- Competitive positioning analysis
        COUNT(*) OVER (PARTITION BY industry_vertical) as industry_competitor_count,
        COUNT(*) OVER (PARTITION BY enterprise_category) as category_competitor_count,
        RANK() OVER (PARTITION BY industry_vertical ORDER BY followers_count DESC) as industry_rank,
        RANK() OVER (PARTITION BY enterprise_category ORDER BY market_influence_score DESC) as category_influence_rank,
        
        -- Market share estimation
        ROUND(followers_count * 100.0 / SUM(followers_count) OVER (PARTITION BY industry_vertical), 2) as industry_market_share_percent,
        ROUND(market_influence_score * 100.0 / SUM(market_influence_score) OVER (PARTITION BY industry_vertical), 2) as industry_influence_share_percent,
        
        -- Brand strategy recommendations
        CASE 
            WHEN industry_rank = 1 AND enterprise_category = 'Fortune 500 Brand'
            THEN 'Market Leader Strategy: Maintain dominance + thought leadership + industry innovation + competitive defense'
            WHEN industry_rank <= 2 AND brand_maturity = 'Established Brand'
            THEN 'Challenger Strategy: Aggressive growth + differentiation + market disruption + competitive advancement'
            WHEN enterprise_category = 'Tech Startup' AND digital_readiness = 'Digital Native'
            THEN 'Disruptor Strategy: Innovation focus + agile marketing + viral growth + market penetration'
            WHEN brand_maturity = 'Growing Brand' AND followers_count >= 2
            THEN 'Growth Strategy: Scale operations + expand reach + brand building + market development'
            ELSE 'Foundation Strategy: Build awareness + establish presence + develop audience + create value'
        END as brand_strategy_recommendation,
        
        -- Enterprise social media governance
        CASE 
            WHEN enterprise_category = 'Fortune 500 Brand' AND brand_risk_level = 'High Visibility Risk'
            THEN 'Tier 1 Governance: C-suite approval + legal review + crisis management + reputation monitoring + stakeholder communication'
            WHEN enterprise_category IN ('B2B Enterprise', 'Financial Services') AND followers_count >= 2
            THEN 'Tier 2 Governance: Department head approval + compliance review + professional standards + industry regulations'
            WHEN brand_risk_level = 'Medium Visibility Risk' AND industry_vertical IN ('Healthcare', 'Financial Services')
            THEN 'Tier 3 Governance: Manager approval + regulatory compliance + industry standards + risk mitigation'
            WHEN enterprise_category IN ('Tech Startup', 'Emerging Brand')
            THEN 'Agile Governance: Streamlined approval + innovation focus + rapid response + growth enablement'
            ELSE 'Standard Governance: Basic approval workflows + content guidelines + brand consistency + routine monitoring'
        END as governance_framework,
        
        -- Investment and resource allocation
        CASE 
            WHEN market_influence_score >= 6 AND industry_rank <= 2
            THEN 'Premium Investment: Dedicated team + advanced tools + content creators + paid promotion + executive support'
            WHEN followers_count >= 3 AND enterprise_category = 'Fortune 500 Brand'
            THEN 'High Investment: Social media manager + content budget + analytics tools + professional services'
            WHEN brand_maturity = 'Growing Brand' AND digital_readiness IN ('Digital Native', 'Digital Advanced')
            THEN 'Growth Investment: Marketing budget allocation + automation tools + training + performance tracking'
            WHEN enterprise_category = 'Tech Startup' AND industry_influence_share_percent >= 15
            THEN 'Strategic Investment: Founder involvement + viral marketing + community building + influencer partnerships'
            ELSE 'Foundation Investment: Basic tools + training + guidelines + measured approach + gradual scaling'
        END as investment_recommendation,
        
        -- Crisis management and reputation protection
        CASE 
            WHEN brand_risk_level = 'High Visibility Risk' AND enterprise_category = 'Fortune 500 Brand'
            THEN 'Advanced Crisis Management: 24/7 monitoring + rapid response team + crisis communication plan + stakeholder management + media relations'
            WHEN industry_vertical IN ('Healthcare', 'Financial Services') AND followers_count >= 2
            THEN 'Regulatory Crisis Management: Compliance protocols + industry-specific response + regulatory communication + professional standards'
            WHEN brand_risk_level = 'Medium Visibility Risk' AND market_influence_score >= 3
            THEN 'Standard Crisis Management: Monitoring tools + response procedures + brand protection + reputation management'
            WHEN enterprise_category = 'Tech Startup' AND digital_readiness = 'Digital Native'
            THEN 'Agile Crisis Management: Real-time monitoring + rapid response + community engagement + transparent communication'
            ELSE 'Basic Crisis Management: Standard monitoring + basic response plan + escalation procedures + damage control'
        END as crisis_management_protocol
    FROM enterprise_social_intelligence esi
),
strategic_business_intelligence AS (
    SELECT 
        cba.*,
        
        -- Business impact assessment
        CASE 
            WHEN industry_rank = 1 AND industry_market_share_percent >= 30
            THEN 'Business Impact: Market leadership position + significant brand value + competitive advantage + revenue driver'
            WHEN followers_count >= 3 AND enterprise_category = 'B2B Enterprise'
            THEN 'Business Impact: Lead generation + relationship building + thought leadership + sales enablement'
            WHEN industry_influence_share_percent >= 20 AND brand_maturity = 'Established Brand'
            THEN 'Business Impact: Brand recognition + customer engagement + market presence + reputation enhancement'
            WHEN enterprise_category = 'Tech Startup' AND market_influence_score >= 3
            THEN 'Business Impact: User acquisition + brand awareness + investor attraction + market validation'
            ELSE 'Business Impact: Brand building + customer connection + market presence + foundational value'
        END as business_impact_assessment,
        
        -- ROI and performance metrics
        CASE 
            WHEN enterprise_category = 'Fortune 500 Brand' AND industry_rank <= 2
            THEN 'ROI Metrics: Brand value increase + customer acquisition cost + engagement rate + share of voice + reputation score'
            WHEN enterprise_category = 'B2B Enterprise' AND followers_count >= 2
            THEN 'ROI Metrics: Lead generation + sales attribution + customer lifetime value + pipeline contribution + conversion rate'
            WHEN enterprise_category = 'Consumer Brand' AND market_influence_score >= 3
            THEN 'ROI Metrics: Brand awareness + purchase intent + customer satisfaction + social commerce + loyalty metrics'
            WHEN enterprise_category = 'Tech Startup'
            THEN 'ROI Metrics: User growth + app downloads + sign-up conversion + viral coefficient + community engagement'
            ELSE 'ROI Metrics: Follower growth + engagement rate + brand mentions + website traffic + content performance'
        END as roi_metrics_framework,
        
        -- Technology and automation recommendations
        CASE 
            WHEN digital_readiness = 'Digital Native' AND market_influence_score >= 6
            THEN 'Advanced Technology Stack: AI-powered analytics + automation platforms + predictive insights + integrated CRM + advanced reporting'
            WHEN enterprise_category = 'Fortune 500 Brand' AND followers_count >= 3
            THEN 'Enterprise Technology Stack: Social media management platform + monitoring tools + analytics dashboard + approval workflows'
            WHEN brand_maturity = 'Growing Brand' AND digital_readiness IN ('Digital Advanced', 'Digital Native')
            THEN 'Growth Technology Stack: Scheduling tools + analytics platform + content creation tools + automation workflows'
            WHEN enterprise_category = 'Tech Startup' AND industry_influence_share_percent >= 10
            THEN 'Startup Technology Stack: Cost-effective tools + growth hacking platforms + viral mechanics + community tools'
            ELSE 'Foundation Technology Stack: Basic management tools + simple analytics + content calendar + manual processes'
        END as technology_recommendation,
        
        -- Strategic partnerships and collaborations
        CASE 
            WHEN industry_rank = 1 AND enterprise_category = 'Fortune 500 Brand'
            THEN 'Premium Partnerships: Industry leaders + strategic alliances + thought leadership collaborations + exclusive partnerships'
            WHEN market_influence_score >= 4 AND industry_vertical = 'Technology'
            THEN 'Innovation Partnerships: Tech companies + startups + research institutions + innovation ecosystems'
            WHEN enterprise_category = 'B2B Enterprise' AND followers_count >= 2
            THEN 'Professional Partnerships: Industry associations + professional networks + B2B influencers + trade organizations'
            WHEN brand_maturity = 'Growing Brand' AND digital_readiness = 'Digital Advanced'
            THEN 'Growth Partnerships: Complementary brands + co-marketing opportunities + cross-promotion + community partnerships'
            ELSE 'Foundation Partnerships: Local partnerships + industry connections + networking opportunities + collaborative content'
        END as partnership_strategy,
        
        -- Future growth and expansion planning
        CASE 
            WHEN industry_rank <= 2 AND market_influence_score >= 6
            THEN 'Expansion Planning: International markets + new platforms + emerging technologies + market leadership extension'
            WHEN enterprise_category = 'Tech Startup' AND followers_count >= 3
            THEN 'Scale Planning: Rapid user acquisition + viral growth strategies + platform expansion + investor preparation'
            WHEN brand_maturity = 'Established Brand' AND industry_influence_share_percent >= 15
            THEN 'Evolution Planning: Brand refresh + digital transformation + innovation adoption + competitive advancement'
            WHEN digital_readiness = 'Digital Native' AND market_influence_score >= 3
            THEN 'Innovation Planning: Emerging platform adoption + technology integration + trend leadership + digital excellence'
            ELSE 'Development Planning: Steady growth + capability building + market presence + foundation strengthening'
        END as growth_expansion_plan
    FROM competitive_brand_analysis cba
)
SELECT 
    user_id,
    followers_count,
    enterprise_category,
    industry_vertical,
    brand_maturity,
    digital_readiness,
    brand_strategy_recommendation,
    governance_framework,
    investment_recommendation,
    crisis_management_protocol,
    business_impact_assessment,
    roi_metrics_framework,
    technology_recommendation,
    partnership_strategy,
    growth_expansion_plan,
    
    -- Competitive intelligence
    industry_rank,
    category_influence_rank,
    industry_market_share_percent,
    industry_influence_share_percent,
    market_influence_score,
    brand_risk_level,
    
    -- Market context
    industry_competitor_count,
    category_competitor_count
FROM strategic_business_intelligence
ORDER BY 
    market_influence_score DESC,
    followers_count DESC,
    industry_rank ASC,
    user_id;
```

## 🔗 Related LeetCode Questions

1. **#1667 - Fix Names in a Table** (Basic data manipulation)
2. **#1683 - Invalid Tweets** (Content filtering and validation)
3. **#1731 - The Number of Employees Which Report to Each Employee** (Counting relationships)
4. **#1757 - Recyclable and Low Fat Products** (Simple filtering and counting)
5. **#1978 - Employees Whose Manager Left the Company** (Relationship analysis)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY Aggregation**: Essential for counting grouped data
2. **COUNT Function**: Counts non-null values in specified column
3. **ORDER BY**: Ensures consistent, predictable output ordering
4. **Primary Key Understanding**: Each row represents one relationship

### 🚀 **Amazon Interview Tips**
1. **Explain aggregation concept**: "GROUP BY collects all followers per user, COUNT tallies them"
2. **Discuss COUNT variations**: "COUNT(column) vs COUNT(*) - both work here since no nulls expected"
3. **Address scalability**: "This query scales well with proper indexing on user_id"
4. **Consider edge cases**: "Users with no followers won't appear unless using LEFT JOIN"

### 🔧 **Common Patterns**
- Basic GROUP BY aggregation
- COUNT function for relationship counting
- ORDER BY for result consistency
- Simple relationship analysis

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting GROUP BY** (would return syntax error)
2. **Wrong COUNT usage** (COUNT(user_id) would count users, not followers)
3. **Missing ORDER BY** (results may be inconsistent across runs)
4. **Not handling missing users** (users with 0 followers won't appear)

### 🔍 **Performance Considerations**
- Index on user_id for efficient grouping
- INDEX(user_id, follower_id) for optimal query performance
- COUNT operations are generally fast with proper indexing
- Consider partitioning for very large follower datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding follower relationships improves user experience and recommendations
- **Operational Excellence**: Reliable social metrics enable data-driven product decisions
- **Innovation**: Advanced social analytics drive personalization and engagement features
- **Think Big**: Social network analysis scales to billions of relationships in Amazon's ecosystem

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find users who follow each other (mutual follows)**
2. **Calculate follower-to-following ratios**
3. **Find users with the most followers in specific time periods**
4. **Identify influencers based on follower quality metrics**

Remember: Social relationship analysis is fundamental to Amazon's social features, recommendation systems, customer networks, influencer programs, and community building across all platforms!