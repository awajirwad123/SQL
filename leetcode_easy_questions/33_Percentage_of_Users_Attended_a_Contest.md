# LeetCode Easy #1633: Percentage of Users Attended a Contest

## 📋 Problem Statement

Write a SQL query to find the **percentage** of the users registered in each contest rounded to **two decimals**.

Return the result table ordered by **percentage in descending order**. In case of a tie, order it by **contest_id in ascending order**.

The query result format is in the following example.

## 🗄️ Table Schema

**Users Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| user_id     | int     |
| user_name   | varchar |
+-------------+---------+
```
- user_id is the primary key for this table.
- user_name is the name of the user.

**Register Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| contest_id  | int     |
| user_id     | int     |
+-------------+---------+
```
- (contest_id, user_id) is the primary key for this table.
- Each row of this table indicates that the user with user_id registered for the contest with contest_id.

## 📊 Sample Data

**Users Table:**
| user_id | user_name |
|---------|-----------|
| 6       | Alice     |
| 2       | Bob       |
| 7       | Alex      |

**Register Table:**
| contest_id | user_id |
|------------|---------|
| 215        | 6       |
| 209        | 2       |
| 209        | 7       |
| 215        | 7       |
| 208        | 2       |
| 210        | 6       |
| 208        | 6       |
| 209        | 6       |

**Expected Output:**
| contest_id | percentage |
|------------|------------|
| 208        | 66.67      |
| 209        | 100.00     |
| 210        | 33.33      |
| 215        | 66.67      |

**Explanation:**
- **Total Users**: 3 (Alice, Bob, Alex)
- **Contest 208**: 2 users registered (Bob, Alice) → 2/3 = 66.67%
- **Contest 209**: 3 users registered (Bob, Alex, Alice) → 3/3 = 100.00%
- **Contest 210**: 1 user registered (Alice) → 1/3 = 33.33%
- **Contest 215**: 2 users registered (Alice, Alex) → 2/3 = 66.67%

**Ordering Logic**:
1. Primary: percentage DESC (209: 100.00%, then 208 & 215: 66.67%, then 210: 33.33%)
2. Secondary: contest_id ASC (for ties, 208 < 215)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Calculate percentage of users registered for each contest
- Need total count of users as denominator
- Need count of registered users per contest as numerator
- Round to 2 decimal places and sort by percentage DESC, contest_id ASC

### 2. **Key Insights**
- Total users = COUNT(*) FROM Users
- Users per contest = COUNT(user_id) FROM Register GROUP BY contest_id
- Percentage = (users_per_contest / total_users) * 100
- Use ROUND() for 2 decimal places

### 3. **Interview Discussion Points**
- "Need to calculate percentage as (registered/total) * 100"
- "Must round to exactly 2 decimal places"
- "Tie-breaking by contest_id ascending is important"

## 🔧 Step-by-Step Solution Logic

### Step 1: Count Total Users
```sql
-- Get total number of users in system
SELECT COUNT(*) FROM Users
```

### Step 2: Count Users Per Contest
```sql
-- Group registrations by contest
SELECT contest_id, COUNT(user_id) as registered_users
FROM Register
GROUP BY contest_id
```

### Step 3: Calculate Percentage
```sql
-- Divide registered by total, multiply by 100
(registered_users * 100.0 / total_users) as percentage
```

### Step 4: Round and Sort
```sql
-- Round to 2 decimals, sort by percentage DESC, contest_id ASC
ROUND(percentage, 2)
ORDER BY percentage DESC, contest_id ASC
```

## ✅ Optimized SQL Solution

**Solution 1: Using Cross Join with Subquery**
```sql
SELECT 
    contest_id,
    ROUND(COUNT(user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) as percentage
FROM Register
GROUP BY contest_id
ORDER BY percentage DESC, contest_id ASC;
```

### Alternative Solutions

**Solution 2: Using Cross Join**
```sql
SELECT 
    r.contest_id,
    ROUND(COUNT(r.user_id) * 100.0 / u.total_users, 2) as percentage
FROM Register r
CROSS JOIN (SELECT COUNT(*) as total_users FROM Users) u
GROUP BY r.contest_id, u.total_users
ORDER BY percentage DESC, r.contest_id ASC;
```

**Solution 3: Using Window Function**
```sql
WITH contest_stats AS (
    SELECT 
        contest_id,
        COUNT(user_id) as registered_users
    FROM Register
    GROUP BY contest_id
),
user_total AS (
    SELECT COUNT(*) as total_users FROM Users
)
SELECT 
    cs.contest_id,
    ROUND(cs.registered_users * 100.0 / ut.total_users, 2) as percentage
FROM contest_stats cs
CROSS JOIN user_total ut
ORDER BY percentage DESC, cs.contest_id ASC;
```

**Solution 4: Using Common Table Expression (CTE)**
```sql
WITH contest_participation AS (
    SELECT 
        contest_id,
        COUNT(user_id) as participants,
        (SELECT COUNT(*) FROM Users) as total_users
    FROM Register
    GROUP BY contest_id
)
SELECT 
    contest_id,
    ROUND(participants * 100.0 / total_users, 2) as percentage
FROM contest_participation
ORDER BY percentage DESC, contest_id ASC;
```

**Solution 5: Comprehensive Analysis with Detailed Metrics**
```sql
WITH user_metrics AS (
    SELECT COUNT(*) as total_users FROM Users
),
contest_analytics AS (
    SELECT 
        r.contest_id,
        COUNT(r.user_id) as registered_users,
        COUNT(DISTINCT r.user_id) as unique_registered_users,
        
        -- User engagement analysis
        GROUP_CONCAT(DISTINCT u.user_name ORDER BY u.user_name) as participant_names,
        
        -- Contest popularity metrics
        RANK() OVER (ORDER BY COUNT(r.user_id) DESC) as popularity_rank,
        DENSE_RANK() OVER (ORDER BY COUNT(r.user_id) DESC) as popularity_dense_rank,
        
        -- Percentile ranking
        PERCENT_RANK() OVER (ORDER BY COUNT(r.user_id)) as popularity_percentile
    FROM Register r
    JOIN Users u ON r.user_id = u.user_id
    GROUP BY r.contest_id
)
SELECT 
    ca.contest_id,
    ca.registered_users,
    ca.unique_registered_users,
    ROUND(ca.registered_users * 100.0 / um.total_users, 2) as percentage,
    ca.participant_names,
    ca.popularity_rank,
    
    -- Additional analytics
    CASE 
        WHEN ca.registered_users = um.total_users THEN 'Full Participation'
        WHEN ca.registered_users >= um.total_users * 0.8 THEN 'High Participation'
        WHEN ca.registered_users >= um.total_users * 0.5 THEN 'Medium Participation'
        WHEN ca.registered_users >= um.total_users * 0.2 THEN 'Low Participation'
        ELSE 'Very Low Participation'
    END as participation_category,
    
    CASE 
        WHEN ca.popularity_percentile >= 0.8 THEN 'Top Tier Contest'
        WHEN ca.popularity_percentile >= 0.6 THEN 'Popular Contest'
        WHEN ca.popularity_percentile >= 0.4 THEN 'Average Contest'
        WHEN ca.popularity_percentile >= 0.2 THEN 'Below Average Contest'
        ELSE 'Low Interest Contest'
    END as contest_tier,
    
    um.total_users as total_system_users
FROM contest_analytics ca
CROSS JOIN user_metrics um
ORDER BY percentage DESC, ca.contest_id ASC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Contest Engagement Analytics and User Behavior System**
```sql
-- "Build comprehensive contest engagement analytics to optimize user participation and contest strategy"

WITH user_participation_analysis AS (
    SELECT 
        u.user_id,
        u.user_name,
        COUNT(r.contest_id) as contests_joined,
        GROUP_CONCAT(r.contest_id ORDER BY r.contest_id) as contest_list,
        
        -- User engagement classification
        CASE 
            WHEN COUNT(r.contest_id) >= 4 THEN 'Super Active'
            WHEN COUNT(r.contest_id) >= 3 THEN 'Very Active' 
            WHEN COUNT(r.contest_id) >= 2 THEN 'Active'
            WHEN COUNT(r.contest_id) = 1 THEN 'Casual'
            ELSE 'Inactive'
        END as user_engagement_level,
        
        -- Participation rate
        ROUND(COUNT(r.contest_id) * 100.0 / (SELECT COUNT(DISTINCT contest_id) FROM Register), 2) as user_participation_rate
    FROM Users u
    LEFT JOIN Register r ON u.user_id = r.user_id
    GROUP BY u.user_id, u.user_name
),
contest_performance_metrics AS (
    SELECT 
        r.contest_id,
        COUNT(r.user_id) as participants,
        (SELECT COUNT(*) FROM Users) as total_users,
        ROUND(COUNT(r.user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) as participation_percentage,
        
        -- Contest attractiveness metrics
        COUNT(CASE WHEN upa.user_engagement_level = 'Super Active' THEN 1 END) as super_active_participants,
        COUNT(CASE WHEN upa.user_engagement_level = 'Very Active' THEN 1 END) as very_active_participants,
        COUNT(CASE WHEN upa.user_engagement_level = 'Active' THEN 1 END) as active_participants,
        COUNT(CASE WHEN upa.user_engagement_level = 'Casual' THEN 1 END) as casual_participants,
        
        -- Engagement quality score
        (COUNT(CASE WHEN upa.user_engagement_level = 'Super Active' THEN 1 END) * 4 +
         COUNT(CASE WHEN upa.user_engagement_level = 'Very Active' THEN 1 END) * 3 +
         COUNT(CASE WHEN upa.user_engagement_level = 'Active' THEN 1 END) * 2 +
         COUNT(CASE WHEN upa.user_engagement_level = 'Casual' THEN 1 END) * 1) as engagement_quality_score,
        
        -- Contest appeal analysis
        CASE 
            WHEN COUNT(r.user_id) = (SELECT COUNT(*) FROM Users) THEN 'Universal Appeal'
            WHEN COUNT(r.user_id) >= (SELECT COUNT(*) FROM Users) * 0.8 THEN 'Broad Appeal'
            WHEN COUNT(r.user_id) >= (SELECT COUNT(*) FROM Users) * 0.5 THEN 'Moderate Appeal'
            WHEN COUNT(r.user_id) >= (SELECT COUNT(*) FROM Users) * 0.2 THEN 'Niche Appeal'
            ELSE 'Limited Appeal'
        END as contest_appeal_category
    FROM Register r
    JOIN user_participation_analysis upa ON r.user_id = upa.user_id
    GROUP BY r.contest_id
),
cross_contest_analysis AS (
    SELECT 
        cpm.*,
        
        -- Competitive benchmarking
        AVG(participation_percentage) OVER() as avg_participation_rate,
        MAX(participation_percentage) OVER() as best_participation_rate,
        MIN(participation_percentage) OVER() as worst_participation_rate,
        
        -- Performance vs average
        CASE 
            WHEN participation_percentage > AVG(participation_percentage) OVER() * 1.5 
            THEN 'Far Above Average'
            WHEN participation_percentage > AVG(participation_percentage) OVER() * 1.2 
            THEN 'Above Average'
            WHEN participation_percentage > AVG(participation_percentage) OVER() * 0.8 
            THEN 'Near Average'
            WHEN participation_percentage > AVG(participation_percentage) OVER() * 0.5 
            THEN 'Below Average'
            ELSE 'Far Below Average'
        END as performance_vs_average,
        
        -- Market share analysis
        ROUND(participants * 100.0 / SUM(participants) OVER(), 2) as market_share_of_participants,
        
        -- Strategic positioning
        CASE 
            WHEN participation_percentage >= 90 THEN 'Flagship Contest - Maximum Engagement'
            WHEN participation_percentage >= 70 THEN 'Premium Contest - High Engagement'
            WHEN participation_percentage >= 50 THEN 'Core Contest - Standard Engagement'
            WHEN participation_percentage >= 30 THEN 'Growth Contest - Development Opportunity'
            ELSE 'Challenge Contest - Needs Improvement'
        END as strategic_positioning,
        
        -- Optimization recommendations
        CASE 
            WHEN super_active_participants = 0 AND very_active_participants = 0 
            THEN 'Improve: Target engaged users, enhance contest visibility'
            WHEN casual_participants > active_participants + very_active_participants 
            THEN 'Optimize: Convert casual users to more engaged participation'
            WHEN participation_percentage < 50 AND contest_appeal_category = 'Limited Appeal' 
            THEN 'Redesign: Fundamental contest structure needs review'
            WHEN participation_percentage >= 80 
            THEN 'Maintain: Successful format, consider replication'
            ELSE 'Enhance: Incremental improvements to boost participation'
        END as optimization_strategy
    FROM contest_performance_metrics cpm
),
user_behavior_insights AS (
    SELECT 
        cca.*,
        
        -- User acquisition potential
        total_users - participants as non_participants,
        ROUND((total_users - participants) * 100.0 / total_users, 2) as non_participation_rate,
        
        -- Growth opportunity scoring
        CASE 
            WHEN non_participation_rate >= 60 AND contest_appeal_category IN ('Niche Appeal', 'Limited Appeal')
            THEN 'High Growth Potential - Untapped Audience'
            WHEN non_participation_rate >= 40 AND performance_vs_average IN ('Above Average', 'Near Average')
            THEN 'Medium Growth Potential - Proven Format'
            WHEN non_participation_rate >= 20 AND strategic_positioning LIKE 'Premium%'
            THEN 'Low Growth Potential - Mature Contest'
            ELSE 'Minimal Growth Potential - Saturated or Poor Performance'
        END as growth_opportunity,
        
        -- Resource allocation priority
        CASE 
            WHEN strategic_positioning = 'Flagship Contest - Maximum Engagement' 
            THEN 'Maintain Excellence - High Resource Priority'
            WHEN strategic_positioning = 'Premium Contest - High Engagement' 
            THEN 'Sustain Performance - Medium-High Resource Priority'
            WHEN growth_opportunity = 'High Growth Potential - Untapped Audience' 
            THEN 'Investment Opportunity - High Resource Priority'
            WHEN strategic_positioning = 'Growth Contest - Development Opportunity' 
            THEN 'Development Focus - Medium Resource Priority'
            WHEN strategic_positioning = 'Challenge Contest - Needs Improvement' 
            THEN 'Improvement Required - Low-Medium Resource Priority'
            ELSE 'Standard Maintenance - Low Resource Priority'
        END as resource_priority,
        
        -- Success metrics and KPIs
        CASE 
            WHEN participation_percentage >= 80 
            THEN 'KPI: Maintain 80%+ participation, increase engagement quality'
            WHEN participation_percentage >= 50 
            THEN 'KPI: Achieve 70%+ participation, improve user experience'
            WHEN participation_percentage >= 30 
            THEN 'KPI: Reach 50%+ participation, enhance contest appeal'
            ELSE 'KPI: Achieve 30%+ participation, fundamental improvements'
        END as success_kpis
    FROM cross_contest_analysis cca
),
organizational_portfolio_analysis AS (
    SELECT 
        COUNT(*) as total_contests,
        SUM(participants) as total_participation_instances,
        AVG(participation_percentage) as portfolio_avg_participation,
        MAX(participation_percentage) as best_contest_performance,
        MIN(participation_percentage) as worst_contest_performance,
        
        -- Portfolio distribution
        COUNT(CASE WHEN strategic_positioning = 'Flagship Contest - Maximum Engagement' THEN 1 END) as flagship_contests,
        COUNT(CASE WHEN strategic_positioning = 'Premium Contest - High Engagement' THEN 1 END) as premium_contests,
        COUNT(CASE WHEN strategic_positioning = 'Core Contest - Standard Engagement' THEN 1 END) as core_contests,
        COUNT(CASE WHEN strategic_positioning = 'Growth Contest - Development Opportunity' THEN 1 END) as growth_contests,
        COUNT(CASE WHEN strategic_positioning = 'Challenge Contest - Needs Improvement' THEN 1 END) as challenge_contests,
        
        -- Resource allocation summary
        COUNT(CASE WHEN resource_priority LIKE '%High%' THEN 1 END) as high_priority_contests,
        COUNT(CASE WHEN resource_priority LIKE '%Medium%' THEN 1 END) as medium_priority_contests,
        COUNT(CASE WHEN resource_priority LIKE '%Low%' THEN 1 END) as low_priority_contests,
        
        -- Growth opportunity summary
        COUNT(CASE WHEN growth_opportunity LIKE 'High Growth%' THEN 1 END) as high_growth_opportunities,
        COUNT(CASE WHEN growth_opportunity LIKE 'Medium Growth%' THEN 1 END) as medium_growth_opportunities
    FROM user_behavior_insights
)
SELECT 
    ubi.contest_id,
    ubi.participation_percentage as percentage,
    ubi.participants,
    ubi.contest_appeal_category,
    ubi.strategic_positioning,
    ubi.performance_vs_average,
    ubi.growth_opportunity,
    ubi.optimization_strategy,
    ubi.resource_priority,
    ubi.success_kpis,
    ubi.engagement_quality_score,
    
    -- Portfolio context
    opa.portfolio_avg_participation,
    CASE 
        WHEN ubi.participation_percentage > opa.portfolio_avg_participation 
        THEN 'Above Portfolio Average'
        ELSE 'Below Portfolio Average'
    END as portfolio_comparison,
    
    -- Market share and competitive position
    ubi.market_share_of_participants,
    CASE 
        WHEN ubi.market_share_of_participants >= 30 THEN 'Dominant Contest'
        WHEN ubi.market_share_of_participants >= 20 THEN 'Leading Contest'
        WHEN ubi.market_share_of_participants >= 15 THEN 'Competitive Contest'
        ELSE 'Niche Contest'
    END as market_position
FROM user_behavior_insights ubi
CROSS JOIN organizational_portfolio_analysis opa
ORDER BY ubi.participation_percentage DESC, ubi.contest_id ASC;
```

#### 2. **Real-time Contest Optimization and Dynamic Participation System**
```sql
-- "Create real-time contest optimization system with dynamic participation tracking and intervention capabilities"

WITH real_time_contest_metrics AS (
    SELECT 
        r.contest_id,
        COUNT(r.user_id) as current_participants,
        (SELECT COUNT(*) FROM Users) as total_users,
        ROUND(COUNT(r.user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) as current_participation_rate,
        
        -- Simulate real-time registration timestamps
        COUNT(CASE WHEN r.user_id % 6 = 0 THEN 1 END) as recent_registrations,  -- Last hour
        COUNT(CASE WHEN r.user_id % 4 = 0 THEN 1 END) as very_recent_registrations,  -- Last 15 minutes
        
        -- User engagement momentum
        AVG(CASE 
            WHEN r.user_id % 3 = 0 THEN 90  -- High engagement score
            WHEN r.user_id % 3 = 1 THEN 60  -- Medium engagement score
            ELSE 30  -- Low engagement score
        END) as avg_user_engagement,
        
        -- Registration velocity simulation
        CASE 
            WHEN COUNT(r.user_id) >= 3 THEN 'High Velocity'
            WHEN COUNT(r.user_id) >= 2 THEN 'Medium Velocity'
            WHEN COUNT(r.user_id) >= 1 THEN 'Low Velocity'
            ELSE 'No Velocity'
        END as registration_velocity,
        
        -- Contest lifecycle stage simulation
        CASE 
            WHEN r.contest_id % 4 = 0 THEN 'Just Launched'
            WHEN r.contest_id % 4 = 1 THEN 'Early Registration'
            WHEN r.contest_id % 4 = 2 THEN 'Peak Registration'
            ELSE 'Final Registration'
        END as contest_lifecycle_stage
    FROM Register r
    GROUP BY r.contest_id
),
dynamic_intervention_analysis AS (
    SELECT 
        rtcm.*,
        
        -- Real-time performance assessment
        CASE 
            WHEN current_participation_rate >= 80 THEN 'Exceeding Expectations'
            WHEN current_participation_rate >= 60 THEN 'Meeting Expectations'
            WHEN current_participation_rate >= 40 THEN 'Below Expectations'
            WHEN current_participation_rate >= 20 THEN 'Significantly Below Expectations'
            ELSE 'Critical Performance Issues'
        END as real_time_performance_status,
        
        -- Immediate intervention triggers
        CASE 
            WHEN current_participation_rate < 30 AND contest_lifecycle_stage = 'Final Registration'
            THEN 'URGENT: Last-chance promotion needed'
            WHEN current_participation_rate < 50 AND contest_lifecycle_stage = 'Peak Registration'
            THEN 'HIGH: Boost promotion during peak period'
            WHEN registration_velocity = 'No Velocity' AND contest_lifecycle_stage != 'Just Launched'
            THEN 'MEDIUM: Reactivate interest campaigns'
            WHEN recent_registrations = 0 AND contest_lifecycle_stage = 'Peak Registration'
            THEN 'MEDIUM: Address registration stagnation'
            ELSE 'LOW: Standard monitoring'
        END as intervention_urgency,
        
        -- Dynamic optimization strategies
        CASE 
            WHEN avg_user_engagement >= 80 AND current_participation_rate < 70
            THEN 'High Engagement, Low Participation - Increase Visibility'
            WHEN avg_user_engagement < 50 AND current_participation_rate >= 60
            THEN 'Low Engagement, High Participation - Improve Experience'
            WHEN registration_velocity = 'High Velocity' AND contest_lifecycle_stage = 'Early Registration'
            THEN 'Momentum Building - Amplify Successful Elements'
            WHEN registration_velocity = 'Low Velocity' AND contest_lifecycle_stage = 'Peak Registration'
            THEN 'Momentum Lost - Investigate Barriers'
            ELSE 'Standard Optimization - Continuous Improvement'
        END as dynamic_strategy,
        
        -- Predictive participation modeling
        CASE 
            WHEN contest_lifecycle_stage = 'Just Launched' AND registration_velocity = 'High Velocity'
            THEN current_participation_rate * 2.5  -- High growth expected
            WHEN contest_lifecycle_stage = 'Early Registration' AND registration_velocity = 'Medium Velocity'
            THEN current_participation_rate * 1.8  -- Moderate growth expected
            WHEN contest_lifecycle_stage = 'Peak Registration'
            THEN current_participation_rate * 1.2  -- Peak phase boost
            WHEN contest_lifecycle_stage = 'Final Registration'
            THEN current_participation_rate * 1.05  -- Minimal additional growth
            ELSE current_participation_rate  -- No change expected
        END as predicted_final_participation_rate,
        
        -- Resource allocation for real-time optimization
        CASE 
            WHEN intervention_urgency = 'URGENT: Last-chance promotion needed'
            THEN 'Emergency Budget: $500-1000 immediate promotion'
            WHEN intervention_urgency = 'HIGH: Boost promotion during peak period'
            THEN 'High Priority Budget: $200-500 targeted campaigns'
            WHEN intervention_urgency = 'MEDIUM: Reactivate interest campaigns'
            THEN 'Medium Budget: $100-200 reactivation efforts'
            WHEN intervention_urgency = 'MEDIUM: Address registration stagnation'
            THEN 'Medium Budget: $100-200 engagement boost'
            ELSE 'Standard Budget: $25-50 routine optimization'
        END as real_time_budget_allocation
    FROM real_time_contest_metrics rtcm
),
automated_response_system AS (
    SELECT 
        dia.*,
        
        -- Automated marketing triggers
        CASE 
            WHEN intervention_urgency LIKE 'URGENT%'
            THEN 'Auto-trigger: Flash promotion email + social media blitz + push notifications'
            WHEN intervention_urgency LIKE 'HIGH%'
            THEN 'Auto-trigger: Enhanced email campaign + social media boost'
            WHEN intervention_urgency LIKE 'MEDIUM%' AND dynamic_strategy LIKE '%Visibility%'
            THEN 'Auto-trigger: Visibility campaign + influencer outreach'
            WHEN intervention_urgency LIKE 'MEDIUM%' AND dynamic_strategy LIKE '%Experience%'
            THEN 'Auto-trigger: User experience survey + quick improvements'
            ELSE 'Scheduled: Regular promotional activities'
        END as automated_marketing_response,
        
        -- Content personalization triggers
        CASE 
            WHEN avg_user_engagement >= 80
            THEN 'Personalize: Premium content + exclusive features'
            WHEN avg_user_engagement >= 60
            THEN 'Personalize: Engaging content + social elements'
            WHEN avg_user_engagement >= 40
            THEN 'Personalize: Simplified content + clear benefits'
            ELSE 'Personalize: Basic information + strong incentives'
        END as content_personalization_strategy,
        
        -- Channel optimization
        CASE 
            WHEN recent_registrations >= 2 AND very_recent_registrations >= 1
            THEN 'Multi-channel: Email + SMS + Push + Social (high momentum)'
            WHEN recent_registrations >= 1
            THEN 'Dual-channel: Email + Push notifications (moderate momentum)'
            WHEN registration_velocity = 'No Velocity'
            THEN 'Broad-channel: Email + Social + Display ads (recovery mode)'
            ELSE 'Standard-channel: Email + periodic social media'
        END as channel_optimization,
        
        -- Success probability and ROI prediction
        CASE 
            WHEN predicted_final_participation_rate >= 80 THEN 85  -- High success probability
            WHEN predicted_final_participation_rate >= 60 THEN 70  -- Good success probability
            WHEN predicted_final_participation_rate >= 40 THEN 55  -- Moderate success probability
            WHEN predicted_final_participation_rate >= 20 THEN 35  -- Low success probability
            ELSE 15  -- Very low success probability
        END as optimization_success_probability,
        
        -- Real-time KPI tracking
        CASE 
            WHEN current_participation_rate >= 70
            THEN 'Track: Maintain momentum, monitor satisfaction, prepare scaling'
            WHEN current_participation_rate >= 50
            THEN 'Track: Boost participation +15%, improve engagement metrics'
            WHEN current_participation_rate >= 30
            THEN 'Track: Achieve 50%+ participation, reduce barriers'
            ELSE 'Track: Emergency improvement, fundamental changes needed'
        END as real_time_kpis
    FROM dynamic_intervention_analysis dia
),
competitive_intelligence AS (
    SELECT 
        ars.*,
        
        -- Cross-contest competitive analysis
        RANK() OVER (ORDER BY current_participation_rate DESC) as participation_rank,
        COUNT(*) OVER() as total_contests,
        
        -- Market position assessment
        CASE 
            WHEN RANK() OVER (ORDER BY current_participation_rate DESC) = 1
            THEN 'Market Leader - Best Performing Contest'
            WHEN RANK() OVER (ORDER BY current_participation_rate DESC) <= COUNT(*) OVER() * 0.25
            THEN 'Top Performer - Upper Quartile'
            WHEN RANK() OVER (ORDER BY current_participation_rate DESC) <= COUNT(*) OVER() * 0.5
            THEN 'Above Average - Upper Half'
            WHEN RANK() OVER (ORDER BY current_participation_rate DESC) <= COUNT(*) OVER() * 0.75
            THEN 'Below Average - Lower Half'
            ELSE 'Poor Performer - Bottom Quartile'
        END as market_position,
        
        -- Competitive pressure assessment
        AVG(current_participation_rate) OVER() as market_avg_participation,
        MAX(current_participation_rate) OVER() as best_competitor_rate,
        MIN(current_participation_rate) OVER() as worst_competitor_rate,
        
        CASE 
            WHEN current_participation_rate >= AVG(current_participation_rate) OVER() * 1.5
            THEN 'Dominant Position - Far Above Market'
            WHEN current_participation_rate >= AVG(current_participation_rate) OVER() * 1.2
            THEN 'Strong Position - Above Market Average'
            WHEN current_participation_rate >= AVG(current_participation_rate) OVER() * 0.8
            THEN 'Competitive Position - Near Market Average'
            WHEN current_participation_rate >= AVG(current_participation_rate) OVER() * 0.5
            THEN 'Weak Position - Below Market Average'
            ELSE 'Critical Position - Far Below Market'
        END as competitive_strength
    FROM automated_response_system ars
)
SELECT 
    contest_id,
    current_participation_rate as percentage,
    current_participants as participants,
    contest_lifecycle_stage,
    registration_velocity,
    real_time_performance_status,
    intervention_urgency,
    dynamic_strategy,
    predicted_final_participation_rate,
    automated_marketing_response,
    channel_optimization,
    real_time_budget_allocation,
    optimization_success_probability,
    market_position,
    competitive_strength,
    real_time_kpis,
    
    -- Context metrics
    market_avg_participation,
    participation_rank,
    total_contests
FROM competitive_intelligence
ORDER BY current_participation_rate DESC, contest_id ASC;
```

#### 3. **Contest Portfolio Strategy and User Lifetime Value Optimization**
```sql
-- "Develop comprehensive contest portfolio strategy with user lifetime value optimization and strategic planning"

WITH user_lifetime_participation AS (
    SELECT 
        u.user_id,
        u.user_name,
        COUNT(r.contest_id) as lifetime_contests,
        GROUP_CONCAT(DISTINCT r.contest_id ORDER BY r.contest_id) as contest_history,
        
        -- User value classification
        CASE 
            WHEN COUNT(r.contest_id) >= 4 THEN 'VIP User'
            WHEN COUNT(r.contest_id) >= 3 THEN 'Loyal User'
            WHEN COUNT(r.contest_id) >= 2 THEN 'Regular User'
            WHEN COUNT(r.contest_id) = 1 THEN 'Occasional User'
            ELSE 'Non-Participant'
        END as user_value_tier,
        
        -- Engagement consistency
        CASE 
            WHEN COUNT(r.contest_id) = (SELECT COUNT(DISTINCT contest_id) FROM Register)
            THEN 'Perfect Consistency'
            WHEN COUNT(r.contest_id) >= (SELECT COUNT(DISTINCT contest_id) FROM Register) * 0.8
            THEN 'High Consistency'
            WHEN COUNT(r.contest_id) >= (SELECT COUNT(DISTINCT contest_id) FROM Register) * 0.5
            THEN 'Medium Consistency'
            WHEN COUNT(r.contest_id) >= (SELECT COUNT(DISTINCT contest_id) FROM Register) * 0.25
            THEN 'Low Consistency'
            ELSE 'Minimal Consistency'
        END as engagement_consistency,
        
        -- User influence potential (simulated)
        CASE 
            WHEN u.user_id % 5 = 0 THEN 'High Influence'
            WHEN u.user_id % 5 = 1 THEN 'Medium Influence'
            ELSE 'Standard Influence'
        END as social_influence_level
    FROM Users u
    LEFT JOIN Register r ON u.user_id = r.user_id
    GROUP BY u.user_id, u.user_name
),
contest_portfolio_analysis AS (
    SELECT 
        r.contest_id,
        COUNT(r.user_id) as participants,
        (SELECT COUNT(*) FROM Users) as total_users,
        ROUND(COUNT(r.user_id) * 100.0 / (SELECT COUNT(*) FROM Users), 2) as participation_rate,
        
        -- Value-based participation analysis
        COUNT(CASE WHEN ulp.user_value_tier = 'VIP User' THEN 1 END) as vip_participants,
        COUNT(CASE WHEN ulp.user_value_tier = 'Loyal User' THEN 1 END) as loyal_participants,
        COUNT(CASE WHEN ulp.user_value_tier = 'Regular User' THEN 1 END) as regular_participants,
        COUNT(CASE WHEN ulp.user_value_tier = 'Occasional User' THEN 1 END) as occasional_participants,
        
        -- Influence-based participation
        COUNT(CASE WHEN ulp.social_influence_level = 'High Influence' THEN 1 END) as high_influence_participants,
        COUNT(CASE WHEN ulp.social_influence_level = 'Medium Influence' THEN 1 END) as medium_influence_participants,
        
        -- Engagement quality metrics
        AVG(ulp.lifetime_contests) as avg_participant_lifetime_value,
        COUNT(CASE WHEN ulp.engagement_consistency IN ('Perfect Consistency', 'High Consistency') THEN 1 END) as consistent_participants,
        
        -- Contest value scoring
        (COUNT(CASE WHEN ulp.user_value_tier = 'VIP User' THEN 1 END) * 10 +
         COUNT(CASE WHEN ulp.user_value_tier = 'Loyal User' THEN 1 END) * 7 +
         COUNT(CASE WHEN ulp.user_value_tier = 'Regular User' THEN 1 END) * 4 +
         COUNT(CASE WHEN ulp.user_value_tier = 'Occasional User' THEN 1 END) * 2) as contest_value_score
    FROM Register r
    JOIN user_lifetime_participation ulp ON r.user_id = ulp.user_id
    GROUP BY r.contest_id
),
strategic_portfolio_positioning AS (
    SELECT 
        cpa.*,
        
        -- Contest strategic classification
        CASE 
            WHEN participation_rate >= 80 AND vip_participants >= 2
            THEN 'Flagship Premium Contest'
            WHEN participation_rate >= 70 AND contest_value_score >= 25
            THEN 'High-Value Strategic Contest'
            WHEN participation_rate >= 50 AND consistent_participants >= 2
            THEN 'Core Engagement Contest'
            WHEN vip_participants + loyal_participants >= participants * 0.7
            THEN 'Loyalty-Focused Contest'
            WHEN high_influence_participants >= 1 AND participation_rate >= 40
            THEN 'Influence-Driven Contest'
            WHEN occasional_participants >= participants * 0.6
            THEN 'Acquisition-Focused Contest'
            ELSE 'Development-Stage Contest'
        END as strategic_classification,
        
        -- Portfolio role assignment
        CASE 
            WHEN participation_rate = (SELECT MAX(participation_rate) FROM contest_portfolio_analysis)
            THEN 'Portfolio Leader - Benchmark Contest'
            WHEN contest_value_score = (SELECT MAX(contest_value_score) FROM contest_portfolio_analysis)
            THEN 'Value Driver - Highest ROI Contest'
            WHEN vip_participants = (SELECT MAX(vip_participants) FROM contest_portfolio_analysis)
            THEN 'VIP Magnet - Premium User Attractor'
            WHEN high_influence_participants = (SELECT MAX(high_influence_participants) FROM contest_portfolio_analysis)
            THEN 'Influence Hub - Network Effect Generator'
            WHEN occasional_participants = (SELECT MAX(occasional_participants) FROM contest_portfolio_analysis)
            THEN 'Gateway Contest - User Acquisition Engine'
            ELSE 'Supporting Contest - Portfolio Complement'
        END as portfolio_role,
        
        -- Investment priority matrix
        CASE 
            WHEN strategic_classification = 'Flagship Premium Contest'
            THEN 'Maximum Investment - Protect and Enhance'
            WHEN strategic_classification = 'High-Value Strategic Contest'
            THEN 'High Investment - Scale and Optimize'
            WHEN strategic_classification = 'Core Engagement Contest'
            THEN 'Standard Investment - Maintain and Improve'
            WHEN strategic_classification = 'Loyalty-Focused Contest'
            THEN 'Targeted Investment - Deepen Relationships'
            WHEN strategic_classification = 'Influence-Driven Contest'
            THEN 'Strategic Investment - Amplify Network Effects'
            WHEN strategic_classification = 'Acquisition-Focused Contest'
            THEN 'Growth Investment - Convert and Retain'
            ELSE 'Development Investment - Build Foundation'
        END as investment_strategy,
        
        -- Competitive differentiation
        CASE 
            WHEN vip_participants >= 2 AND high_influence_participants >= 1
            THEN 'Premium Differentiation - Exclusive High-Value Experience'
            WHEN consistent_participants >= 3
            THEN 'Reliability Differentiation - Dependable Engagement Platform'
            WHEN contest_value_score >= 30
            THEN 'Quality Differentiation - Superior Value Proposition'
            WHEN participation_rate >= 80
            THEN 'Popularity Differentiation - Mass Appeal Champion'
            ELSE 'Niche Differentiation - Specialized Focus'
        END as competitive_differentiation
    FROM contest_portfolio_analysis cpa
),
optimization_roadmap AS (
    SELECT 
        spp.*,
        
        -- Short-term optimization (30-90 days)
        CASE 
            WHEN strategic_classification = 'Flagship Premium Contest'
            THEN 'Short-term: VIP experience enhancement + exclusive features'
            WHEN strategic_classification = 'High-Value Strategic Contest'
            THEN 'Short-term: Advanced engagement tools + personalization'
            WHEN strategic_classification = 'Acquisition-Focused Contest'
            THEN 'Short-term: Onboarding optimization + new user incentives'
            WHEN occasional_participants >= participants * 0.5
            THEN 'Short-term: Conversion campaigns + engagement boosters'
            ELSE 'Short-term: User experience improvements + basic optimizations'
        END as short_term_roadmap,
        
        -- Medium-term strategy (3-6 months)
        CASE 
            WHEN portfolio_role LIKE '%Leader%' OR portfolio_role LIKE '%Driver%'
            THEN 'Medium-term: Innovation leadership + market expansion'
            WHEN strategic_classification LIKE '%Strategic%'
            THEN 'Medium-term: Feature enhancement + user base expansion'
            WHEN competitive_differentiation LIKE 'Premium%'
            THEN 'Medium-term: Premium service tier + advanced features'
            WHEN participation_rate < 50
            THEN 'Medium-term: Fundamental improvements + positioning refresh'
            ELSE 'Medium-term: Incremental enhancements + efficiency gains'
        END as medium_term_strategy,
        
        -- Long-term vision (6-12 months)
        CASE 
            WHEN strategic_classification = 'Flagship Premium Contest'
            THEN 'Long-term: Industry benchmark + ecosystem expansion'
            WHEN portfolio_role LIKE '%Leader%'
            THEN 'Long-term: Market leadership + platform development'
            WHEN contest_value_score >= 25
            THEN 'Long-term: Value maximization + strategic partnerships'
            WHEN high_influence_participants >= 1
            THEN 'Long-term: Network effect optimization + viral growth'
            ELSE 'Long-term: Sustainable growth + market positioning'
        END as long_term_vision,
        
        -- Resource allocation recommendation
        CASE 
            WHEN investment_strategy = 'Maximum Investment - Protect and Enhance'
            THEN '35-50% of total contest budget'
            WHEN investment_strategy = 'High Investment - Scale and Optimize'
            THEN '20-35% of total contest budget'
            WHEN investment_strategy = 'Standard Investment - Maintain and Improve'
            THEN '10-20% of total contest budget'
            WHEN investment_strategy = 'Targeted Investment - Deepen Relationships'
            THEN '8-15% of total contest budget'
            WHEN investment_strategy = 'Strategic Investment - Amplify Network Effects'
            THEN '5-12% of total contest budget'
            WHEN investment_strategy = 'Growth Investment - Convert and Retain'
            THEN '5-10% of total contest budget'
            ELSE '2-5% of total contest budget'
        END as budget_allocation_recommendation,
        
        -- Success metrics framework
        CASE 
            WHEN strategic_classification = 'Flagship Premium Contest'
            THEN 'Metrics: 90%+ participation, VIP satisfaction >95%, innovation index'
            WHEN strategic_classification = 'High-Value Strategic Contest'
            THEN 'Metrics: 80%+ participation, user lifetime value +20%, retention >90%'
            WHEN strategic_classification = 'Core Engagement Contest'
            THEN 'Metrics: 60%+ participation, consistent user growth, engagement depth'
            WHEN strategic_classification = 'Acquisition-Focused Contest'
            THEN 'Metrics: New user acquisition +25%, conversion rate >40%, onboarding success'
            ELSE 'Metrics: 40%+ participation, user satisfaction >80%, growth trajectory'
        END as success_metrics_framework
    FROM strategic_portfolio_positioning spp
)
SELECT 
    contest_id,
    participation_rate as percentage,
    participants,
    strategic_classification,
    portfolio_role,
    competitive_differentiation,
    investment_strategy,
    short_term_roadmap,
    medium_term_strategy,
    long_term_vision,
    budget_allocation_recommendation,
    success_metrics_framework,
    
    -- Key performance indicators
    contest_value_score,
    vip_participants,
    high_influence_participants,
    consistent_participants,
    avg_participant_lifetime_value
FROM optimization_roadmap
ORDER BY participation_rate DESC, contest_id ASC;
```

## 🔗 Related LeetCode Questions

1. **#1251 - Average Selling Price** (Percentage calculations with aggregation)
2. **#1661 - Average Time of Process per Machine** (Average calculations and rounding)
3. **#1484 - Group Sold Products By The Date** (GROUP BY with counting)
4. **#1280 - Students and Examinations** (Cross tabulation with percentages)
5. **#1795 - Rearrange Products Table** (Data aggregation and transformation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Percentage Calculation**: (Count/Total) * 100 with proper decimal handling
2. **Cross Join or Subquery**: Getting total count for percentage denominator
3. **ROUND Function**: Ensuring exactly 2 decimal places
4. **Dual Sorting**: ORDER BY percentage DESC, contest_id ASC

### 🚀 **Amazon Interview Tips**
1. **Explain percentage formula**: "Divide registered users by total users, multiply by 100"
2. **Discuss rounding precision**: "ROUND(value, 2) ensures exactly 2 decimal places"
3. **Address tie-breaking**: "Secondary sort by contest_id ASC for deterministic results"
4. **Consider performance**: "Subquery vs JOIN for getting total count"

### 🔧 **Common Patterns**
- Percentage = (part/whole) * 100
- ROUND(value, 2) for 2 decimal places
- Cross join or subquery for getting totals
- Multiple column sorting for tie-breaking

### ⚠️ **Common Mistakes to Avoid**
1. **Integer division** (use 100.0 not 100 to force decimal arithmetic)
2. **Wrong rounding** (not specifying decimal places or wrong precision)
3. **Missing tie-breaking** (not sorting by contest_id for ties)
4. **Performance issues** (inefficient total count calculation)

### 🔍 **Performance Considerations**
- Subquery in SELECT can be less efficient than Cross Join
- Consider materializing total count if used multiple times
- INDEX on join columns improves performance
- ROUND function has minimal performance impact

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding user engagement patterns improves platform experience
- **Data-Driven Decisions**: Percentage analysis guides contest optimization strategies
- **Operational Excellence**: Systematic measurement enables continuous improvement
- **Innovation**: Advanced analytics drive competitive differentiation

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate percentage of users who joined multiple contests**
2. **Find contests with participation rates above platform average**
3. **Calculate participation rate trends over time periods**
4. **Analyze user engagement levels by contest characteristics**

Remember: Participation percentage analysis is vital for Amazon's engagement platforms, marketing effectiveness measurement, user behavior analytics, and strategic decision-making across all customer-facing contest and event systems!