# LeetCode Easy #1050: Actors and Directors Who Cooperated At Least Three Times

## 📋 Problem Statement

Write a SQL query to find all **actor_id** and **director_id** pairs that cooperated at least **three times**.

Return the result table in **any order**.

## 🗄️ Table Schema

**ActorDirector Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| actor_id    | int     |
| director_id | int     |
| timestamp   | int     |
+-------------+---------+
```
- `timestamp` is the primary key column for this table.
- Each row of this table indicates that the actor with `actor_id` cooperated with the director with `director_id` at the time `timestamp`.

## 📊 Sample Data

**ActorDirector Table:**
| actor_id | director_id | timestamp |
|----------|-------------|-----------|
| 1        | 1           | 0         |
| 1        | 1           | 1         |
| 1        | 1           | 2         |
| 1        | 2           | 3         |
| 1        | 2           | 4         |
| 2        | 1           | 5         |
| 2        | 1           | 6         |

**Expected Output:**
| actor_id | director_id |
|----------|-------------|
| 1        | 1           |

**Explanation:** 
- Actor 1 and Director 1 cooperated 3 times (timestamps 0, 1, 2)
- Actor 1 and Director 2 cooperated 2 times (timestamps 3, 4) - not enough
- Actor 2 and Director 1 cooperated 2 times (timestamps 5, 6) - not enough

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to count collaborations per actor-director pair
- Filter pairs with 3 or more collaborations
- GROUP BY actor_id and director_id with COUNT
- Use HAVING for filtering aggregated results

### 2. **Key Insights**
- GROUP BY creates unique actor-director combinations
- COUNT(*) counts collaborations for each pair
- HAVING filters groups (not individual rows)
- Simple aggregation problem with threshold filtering

### 3. **Interview Discussion Points**
- "This is a GROUP BY problem with COUNT and HAVING"
- "I need to group by both actor_id and director_id"
- "HAVING COUNT(*) >= 3 filters successful partnerships"

## 🔧 Step-by-Step Solution Logic

### Step 1: Group by Actor-Director Pairs
```sql
-- GROUP BY actor_id, director_id creates unique pairs
-- Each group contains all collaborations for that pair
```

### Step 2: Count Collaborations
```sql
-- COUNT(*) counts rows in each group
-- This gives total collaborations per pair
```

### Step 3: Filter Successful Partnerships
```sql
-- HAVING COUNT(*) >= 3 keeps only prolific partnerships
-- Return actor_id and director_id for successful pairs
```

## ✅ Optimized SQL Solution

**Solution 1: Basic GROUP BY with HAVING**
```sql
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;
```

### Alternative Solutions

**Solution 2: With Collaboration Count**
```sql
SELECT 
    actor_id, 
    director_id,
    COUNT(*) as collaboration_count
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;
```

**Solution 3: Using Window Functions**
```sql
WITH collaboration_counts AS (
    SELECT 
        actor_id, 
        director_id,
        COUNT(*) OVER (PARTITION BY actor_id, director_id) as collab_count
    FROM ActorDirector
)
SELECT DISTINCT actor_id, director_id
FROM collaboration_counts
WHERE collab_count >= 3;
```

**Solution 4: Subquery Approach**
```sql
SELECT actor_id, director_id
FROM (
    SELECT 
        actor_id, 
        director_id, 
        COUNT(*) as collaborations
    FROM ActorDirector
    GROUP BY actor_id, director_id
) as partnership_stats
WHERE collaborations >= 3;
```

**Solution 5: With Ranking by Collaboration Frequency**
```sql
SELECT 
    actor_id, 
    director_id,
    collaboration_count,
    RANK() OVER (ORDER BY collaboration_count DESC) as partnership_rank
FROM (
    SELECT 
        actor_id, 
        director_id,
        COUNT(*) as collaboration_count
    FROM ActorDirector
    GROUP BY actor_id, director_id
    HAVING COUNT(*) >= 3
) as successful_partnerships
ORDER BY collaboration_count DESC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Partnership Analytics**
```sql
-- "Provide detailed analytics on actor-director partnerships"

WITH partnership_analysis AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as total_collaborations,
        MIN(timestamp) as first_collaboration,
        MAX(timestamp) as latest_collaboration,
        MAX(timestamp) - MIN(timestamp) as partnership_duration
    FROM ActorDirector
    GROUP BY actor_id, director_id
),
partnership_categories AS (
    SELECT 
        *,
        CASE 
            WHEN total_collaborations >= 10 THEN 'Frequent Partners'
            WHEN total_collaborations >= 5 THEN 'Regular Partners'
            WHEN total_collaborations >= 3 THEN 'Established Partners'
            WHEN total_collaborations = 2 THEN 'Occasional Partners'
            ELSE 'One-time Collaboration'
        END as partnership_level,
        CASE 
            WHEN partnership_duration = 0 THEN 'Same Period'
            WHEN partnership_duration <= 5 THEN 'Short Term'
            WHEN partnership_duration <= 20 THEN 'Medium Term'
            ELSE 'Long Term'
        END as collaboration_span
    FROM partnership_analysis
)
SELECT 
    actor_id,
    director_id,
    total_collaborations,
    partnership_level,
    collaboration_span,
    first_collaboration,
    latest_collaboration,
    partnership_duration
FROM partnership_categories
WHERE total_collaborations >= 3
ORDER BY total_collaborations DESC, partnership_duration DESC;
```

#### 2. **Network Analysis and Relationship Strength**
```sql
-- "Analyze the network of collaborations and relationship strength"

WITH actor_stats AS (
    SELECT 
        actor_id,
        COUNT(DISTINCT director_id) as directors_worked_with,
        COUNT(*) as total_projects,
        AVG(COUNT(*)) OVER (PARTITION BY actor_id) as avg_collaborations_per_director
    FROM ActorDirector
    GROUP BY actor_id, director_id
),
director_stats AS (
    SELECT 
        director_id,
        COUNT(DISTINCT actor_id) as actors_worked_with,
        COUNT(*) as total_projects,
        AVG(COUNT(*)) OVER (PARTITION BY director_id) as avg_collaborations_per_actor
    FROM ActorDirector
    GROUP BY director_id, actor_id
),
successful_partnerships AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as collaborations
    FROM ActorDirector
    GROUP BY actor_id, director_id
    HAVING COUNT(*) >= 3
),
network_metrics AS (
    SELECT 
        sp.actor_id,
        sp.director_id,
        sp.collaborations,
        aas.directors_worked_with,
        aas.total_projects as actor_total_projects,
        dds.actors_worked_with,
        dds.total_projects as director_total_projects,
        ROUND(sp.collaborations * 100.0 / aas.total_projects, 2) as actor_partnership_percentage,
        ROUND(sp.collaborations * 100.0 / dds.total_projects, 2) as director_partnership_percentage
    FROM successful_partnerships sp
    JOIN (SELECT DISTINCT actor_id, directors_worked_with, total_projects FROM actor_stats) aas 
        ON sp.actor_id = aas.actor_id
    JOIN (SELECT DISTINCT director_id, actors_worked_with, total_projects FROM director_stats) dds 
        ON sp.director_id = dds.director_id
)
SELECT 
    actor_id,
    director_id,
    collaborations,
    actor_partnership_percentage,
    director_partnership_percentage,
    CASE 
        WHEN actor_partnership_percentage > 50 AND director_partnership_percentage > 50 THEN 'Exclusive Partnership'
        WHEN actor_partnership_percentage > 30 OR director_partnership_percentage > 30 THEN 'Primary Partnership'
        ELSE 'Strong Partnership'
    END as partnership_strength
FROM network_metrics
ORDER BY collaborations DESC;
```

#### 3. **Time-Based Collaboration Patterns**
```sql
-- "Analyze collaboration patterns over time"

WITH time_analysis AS (
    SELECT 
        actor_id,
        director_id,
        timestamp,
        LAG(timestamp) OVER (PARTITION BY actor_id, director_id ORDER BY timestamp) as prev_timestamp,
        LEAD(timestamp) OVER (PARTITION BY actor_id, director_id ORDER BY timestamp) as next_timestamp
    FROM ActorDirector
),
collaboration_intervals AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as total_collaborations,
        AVG(CASE 
            WHEN prev_timestamp IS NOT NULL 
            THEN timestamp - prev_timestamp 
            ELSE NULL 
        END) as avg_time_between_projects,
        MIN(timestamp) as career_start,
        MAX(timestamp) as career_end,
        MAX(timestamp) - MIN(timestamp) as collaboration_timespan
    FROM time_analysis
    GROUP BY actor_id, director_id
    HAVING COUNT(*) >= 3
),
collaboration_patterns AS (
    SELECT 
        *,
        CASE 
            WHEN avg_time_between_projects <= 1 THEN 'Rapid Succession'
            WHEN avg_time_between_projects <= 3 THEN 'Regular Collaboration'
            WHEN avg_time_between_projects <= 10 THEN 'Periodic Collaboration'
            ELSE 'Sporadic Collaboration'
        END as collaboration_pattern,
        ROUND(total_collaborations * 1.0 / NULLIF(collaboration_timespan, 0), 2) as collaboration_intensity
    FROM collaboration_intervals
)
SELECT 
    actor_id,
    director_id,
    total_collaborations,
    collaboration_pattern,
    ROUND(avg_time_between_projects, 2) as avg_gap_between_projects,
    collaboration_intensity,
    career_start,
    career_end,
    collaboration_timespan
FROM collaboration_patterns
ORDER BY collaboration_intensity DESC, total_collaborations DESC;
```

#### 4. **Performance Optimization for Large Datasets**
```sql
-- "Optimize the query for millions of collaboration records"

-- Create indexes for efficient GROUP BY
-- CREATE INDEX idx_actor_director ON ActorDirector(actor_id, director_id);
-- CREATE INDEX idx_timestamp ON ActorDirector(timestamp);

-- Optimized query with pre-filtering
WITH recent_collaborations AS (
    SELECT actor_id, director_id
    FROM ActorDirector
    WHERE timestamp >= (SELECT MAX(timestamp) - 1000 FROM ActorDirector)  -- Recent collaborations only
),
collaboration_counts AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as collaboration_count
    FROM ActorDirector
    GROUP BY actor_id, director_id
)
SELECT 
    actor_id,
    director_id,
    collaboration_count
FROM collaboration_counts
WHERE collaboration_count >= 3
ORDER BY collaboration_count DESC;

-- Query performance analysis
EXPLAIN ANALYZE
SELECT actor_id, director_id
FROM ActorDirector
GROUP BY actor_id, director_id
HAVING COUNT(*) >= 3;

-- Alternative approach using materialized aggregation
-- CREATE TABLE partnership_summary AS
-- SELECT 
--     actor_id,
--     director_id,
--     COUNT(*) as collaboration_count,
--     MIN(timestamp) as first_collab,
--     MAX(timestamp) as last_collab
-- FROM ActorDirector
-- GROUP BY actor_id, director_id;

-- Then simple query:
-- SELECT actor_id, director_id 
-- FROM partnership_summary 
-- WHERE collaboration_count >= 3;
```

#### 5. **Business Intelligence Dashboard**
```sql
-- "Create a comprehensive entertainment industry dashboard"

WITH industry_metrics AS (
    SELECT 
        COUNT(DISTINCT actor_id) as total_actors,
        COUNT(DISTINCT director_id) as total_directors,
        COUNT(*) as total_collaborations,
        COUNT(DISTINCT CONCAT(actor_id, '-', director_id)) as unique_partnerships,
        AVG(COUNT(*)) OVER (PARTITION BY actor_id, director_id) as avg_collaborations_per_partnership
    FROM ActorDirector
    GROUP BY actor_id, director_id
),
successful_partnerships AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as collaborations
    FROM ActorDirector
    GROUP BY actor_id, director_id
    HAVING COUNT(*) >= 3
),
partnership_distribution AS (
    SELECT 
        collaboration_tier,
        COUNT(*) as partnership_count,
        COUNT(*) * 100.0 / SUM(COUNT(*)) OVER() as percentage
    FROM (
        SELECT 
            CASE 
                WHEN COUNT(*) >= 10 THEN '10+ Collaborations'
                WHEN COUNT(*) >= 5 THEN '5-9 Collaborations'
                WHEN COUNT(*) >= 3 THEN '3-4 Collaborations'
                WHEN COUNT(*) = 2 THEN '2 Collaborations'
                ELSE '1 Collaboration'
            END as collaboration_tier
        FROM ActorDirector
        GROUP BY actor_id, director_id
    ) tier_counts
    GROUP BY collaboration_tier
),
top_performers AS (
    SELECT 
        'Most Collaborative Actor' as metric,
        CAST(actor_id AS CHAR) as id,
        COUNT(DISTINCT director_id) as unique_partners,
        COUNT(*) as total_projects
    FROM ActorDirector
    GROUP BY actor_id
    ORDER BY COUNT(DISTINCT director_id) DESC, COUNT(*) DESC
    LIMIT 1
    
    UNION ALL
    
    SELECT 
        'Most Collaborative Director' as metric,
        CAST(director_id AS CHAR) as id,
        COUNT(DISTINCT actor_id) as unique_partners,
        COUNT(*) as total_projects
    FROM ActorDirector
    GROUP BY director_id
    ORDER BY COUNT(DISTINCT actor_id) DESC, COUNT(*) DESC
    LIMIT 1
)
SELECT 
    'Entertainment Industry Dashboard' as dashboard_section,
    im.total_actors,
    im.total_directors,
    im.total_collaborations,
    im.unique_partnerships,
    COUNT(sp.actor_id) as successful_partnerships,
    ROUND(COUNT(sp.actor_id) * 100.0 / im.unique_partnerships, 2) as success_rate_percentage
FROM industry_metrics im
CROSS JOIN successful_partnerships sp
GROUP BY im.total_actors, im.total_directors, im.total_collaborations, im.unique_partnerships;
```

#### 6. **Advanced Analytics and Predictions**
```sql
-- "Predict future successful partnerships based on current trends"

WITH partnership_trajectories AS (
    SELECT 
        actor_id,
        director_id,
        COUNT(*) as current_collaborations,
        MAX(timestamp) as last_collaboration,
        MIN(timestamp) as first_collaboration,
        (MAX(timestamp) - MIN(timestamp)) / NULLIF(COUNT(*) - 1, 0) as avg_project_interval
    FROM ActorDirector
    GROUP BY actor_id, director_id
),
emerging_partnerships AS (
    SELECT 
        *,
        CASE 
            WHEN current_collaborations = 2 AND (
                SELECT MAX(timestamp) FROM ActorDirector
            ) - last_collaboration <= avg_project_interval * 2 THEN 'Likely to reach 3+'
            WHEN current_collaborations = 2 THEN 'Possible future partnership'
            WHEN current_collaborations >= 3 THEN 'Established partnership'
            ELSE 'One-time collaboration'
        END as partnership_prediction,
        CASE 
            WHEN avg_project_interval <= 2 THEN 'High frequency'
            WHEN avg_project_interval <= 5 THEN 'Medium frequency'
            ELSE 'Low frequency'
        END as collaboration_frequency
    FROM partnership_trajectories
)
SELECT 
    actor_id,
    director_id,
    current_collaborations,
    partnership_prediction,
    collaboration_frequency,
    ROUND(avg_project_interval, 2) as avg_gap_between_projects,
    last_collaboration,
    CASE 
        WHEN partnership_prediction = 'Likely to reach 3+' THEN 'Monitor for milestone'
        WHEN current_collaborations >= 3 THEN 'Include in analysis'
        ELSE 'Track for future potential'
    END as recommendation
FROM emerging_partnerships
WHERE current_collaborations >= 2
ORDER BY 
    CASE partnership_prediction 
        WHEN 'Established partnership' THEN 1
        WHEN 'Likely to reach 3+' THEN 2
        ELSE 3
    END,
    current_collaborations DESC;
```

## 🔗 Related LeetCode Questions

1. **#586 - Customer Placing the Largest Number of Orders** (GROUP BY with aggregation)
2. **#596 - Classes More Than 5 Students** (GROUP BY with HAVING)
3. **#1141 - User Activity for the Past 30 Days I** (GROUP BY with time filtering)
4. **#1179 - Reformat Department Table** (GROUP BY with pivoting)
5. **#1393 - Capital Gain/Loss** (GROUP BY with calculations)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY Multiple Columns**: Creating unique combinations
2. **COUNT with HAVING**: Filtering aggregated results
3. **Aggregation Functions**: Counting occurrences and relationships
4. **Threshold Filtering**: Using HAVING for group-level conditions

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I return the count or just the pairs?"
2. **Discuss performance**: "GROUP BY needs proper indexing on large datasets"
3. **Consider extensions**: "How would you handle time-based analysis?"
4. **Think business context**: "This helps identify successful creative partnerships"

### 🔧 **Common Patterns**
- GROUP BY multiple columns for unique combinations
- COUNT(*) for counting occurrences in groups
- HAVING for filtering aggregated results
- Window functions for advanced analytics

### ⚠️ **Common Mistakes to Avoid**
1. **Using WHERE instead of HAVING** for aggregate filtering
2. **Forgetting to GROUP BY all required columns**
3. **Performance issues** without proper indexing
4. **Not considering NULL values** in grouping columns

### 🔍 **Performance Considerations**
- Composite index on (actor_id, director_id) for efficient grouping
- Consider materialized views for frequently accessed aggregations
- Partitioning for very large datasets
- Query execution plan analysis for optimization

### 🎯 **Amazon Leadership Principles Applied**
- **Data-Driven Decisions**: Using collaboration data for content strategy
- **Customer Obsession**: Understanding successful content partnerships
- **Operational Excellence**: Efficient partnership analysis systems
- **Innovation**: Advanced analytics for predictive insights

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find pairs that collaborated exactly twice**
2. **Find the most collaborative actor-director pair**
3. **Calculate average time between collaborations**
4. **Identify partnerships that formed recently**

Remember: Relationship analysis and partnership patterns are essential for Amazon's content strategy, vendor partnerships, marketplace seller analysis, and collaboration network optimization!