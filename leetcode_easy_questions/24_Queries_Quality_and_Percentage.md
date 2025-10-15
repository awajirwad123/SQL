# LeetCode Easy #1211: Queries Quality and Percentage

## 📋 Problem Statement

We define query **quality** as:
- The average of the ratio between query rating and its position.

We also define **poor query percentage** as:
- The percentage of all queries with rating less than 3.

Write a SQL query to find each `query_name`, the `quality` and `poor_query_percentage`.

Both `quality` and `poor_query_percentage` should be **rounded to 2 decimal places**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Queries Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| query_name  | varchar |
| result      | varchar |
| position    | int     |
| rating      | int     |
+-------------+---------+
```
- There is no primary key for this table, it may have duplicate rows.
- This table contains information collected from some queries on a database.
- The `position` column has a value from **1** to **500**.
- The `rating` column has a value from **1** to **5**. Query with `rating` less than 3 is a poor query.

## 📊 Sample Data

**Queries Table:**
| query_name | result            | position | rating |
|------------|-------------------|----------|--------|
| Dog        | Golden Retriever  | 1        | 5      |
| Dog        | German Shepherd   | 2        | 5      |
| Dog        | Mule              | 200      | 1      |
| Cat        | Shirazi           | 5        | 2      |
| Cat        | Siamese           | 3        | 3      |
| Cat        | Sphynx            | 7        | 4      |

**Expected Output:**
| query_name | quality | poor_query_percentage |
|------------|---------|----------------------|
| Dog        | 2.50    | 33.33                |
| Cat        | 1.66    | 33.33                |

**Explanation:** 
- **Dog queries:**
  - Quality = ((5/1) + (5/2) + (1/200)) / 3 = (5 + 2.5 + 0.005) / 3 = 2.50
  - Poor query percentage = 1/3 * 100 = 33.33% (1 rating < 3 out of 3 total)

- **Cat queries:**
  - Quality = ((2/5) + (3/3) + (4/7)) / 3 = (0.4 + 1 + 0.571) / 3 = 1.66
  - Poor query percentage = 1/3 * 100 = 33.33% (1 rating < 3 out of 3 total)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Calculate two metrics per query_name: quality and poor_query_percentage
- Quality = AVG(rating/position)
- Poor query percentage = COUNT(rating < 3) / COUNT(*) * 100
- Round both to 2 decimal places
- Group by query_name

### 2. **Key Insights**
- Use GROUP BY for query_name aggregation
- AVG function for quality calculation
- Conditional aggregation for poor query percentage
- ROUND function for decimal precision

### 3. **Interview Discussion Points**
- "This combines aggregation with conditional logic"
- "Need to calculate ratio averages and percentage metrics"
- "Important to handle division carefully for percentage calculation"

## 🔧 Step-by-Step Solution Logic

### Step 1: Calculate Quality per Query
```sql
-- AVG(rating * 1.0 / position)
-- Average of rating/position ratios for each query_name
```

### Step 2: Calculate Poor Query Percentage
```sql
-- (COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*)
-- Percentage of queries with rating < 3
```

### Step 3: Round Results
```sql
-- ROUND(quality, 2) and ROUND(percentage, 2)
-- Format to required decimal places
```

### Step 4: Group by Query Name
```sql
-- GROUP BY query_name
-- Aggregate metrics for each distinct query
```

## ✅ Optimized SQL Solution

**Solution 1: Basic Aggregation with ROUND**
```sql
SELECT 
    query_name,
    ROUND(AVG(rating * 1.0 / position), 2) as quality,
    ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name;
```

### Alternative Solutions

**Solution 2: Using SUM for Conditional Counting**
```sql
SELECT 
    query_name,
    ROUND(AVG(CAST(rating AS DECIMAL(10,5)) / position), 2) as quality,
    ROUND(SUM(CASE WHEN rating < 3 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) as poor_query_percentage
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name;
```

**Solution 3: With CTE for Clarity**
```sql
WITH query_metrics AS (
    SELECT 
        query_name,
        rating,
        position,
        rating * 1.0 / position as quality_score,
        CASE WHEN rating < 3 THEN 1 ELSE 0 END as is_poor_query
    FROM Queries
    WHERE query_name IS NOT NULL
)
SELECT 
    query_name,
    ROUND(AVG(quality_score), 2) as quality,
    ROUND(AVG(is_poor_query) * 100, 2) as poor_query_percentage
FROM query_metrics
GROUP BY query_name;
```

**Solution 4: Detailed Quality Analysis**
```sql
SELECT 
    query_name,
    COUNT(*) as total_queries,
    ROUND(AVG(rating * 1.0 / position), 2) as quality,
    ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as poor_query_percentage,
    COUNT(CASE WHEN rating < 3 THEN 1 END) as poor_queries_count,
    COUNT(CASE WHEN rating >= 4 THEN 1 END) as good_queries_count,
    ROUND(AVG(rating), 2) as avg_rating,
    ROUND(AVG(position), 2) as avg_position
FROM Queries
WHERE query_name IS NOT NULL
GROUP BY query_name;
```

**Solution 5: With Quality Categories**
```sql
WITH detailed_metrics AS (
    SELECT 
        query_name,
        COUNT(*) as total_queries,
        ROUND(AVG(rating * 1.0 / position), 2) as quality,
        ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as poor_query_percentage,
        ROUND(AVG(rating), 2) as avg_rating,
        MIN(rating) as min_rating,
        MAX(rating) as max_rating,
        MIN(position) as best_position,
        MAX(position) as worst_position
    FROM Queries
    WHERE query_name IS NOT NULL
    GROUP BY query_name
)
SELECT 
    *,
    CASE 
        WHEN quality >= 3.0 THEN 'Excellent'
        WHEN quality >= 2.0 THEN 'Good'
        WHEN quality >= 1.0 THEN 'Average'
        ELSE 'Poor'
    END as quality_category,
    CASE 
        WHEN poor_query_percentage <= 10 THEN 'Low Poor Rate'
        WHEN poor_query_percentage <= 25 THEN 'Moderate Poor Rate'
        WHEN poor_query_percentage <= 50 THEN 'High Poor Rate'
        ELSE 'Very High Poor Rate'
    END as poor_query_category
FROM detailed_metrics
ORDER BY quality DESC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Search Quality Analysis**
```sql
-- "Analyze search quality trends and patterns"

WITH daily_query_analysis AS (
    SELECT 
        query_name,
        DATE(created_at) as query_date,  -- Assuming timestamp column
        COUNT(*) as daily_queries,
        ROUND(AVG(rating * 1.0 / position), 2) as daily_quality,
        ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as daily_poor_percentage,
        AVG(rating) as avg_daily_rating,
        AVG(position) as avg_daily_position,
        COUNT(DISTINCT result) as unique_results_count
    FROM Queries
    WHERE query_name IS NOT NULL
    GROUP BY query_name, DATE(created_at)
),
trending_analysis AS (
    SELECT 
        query_name,
        AVG(daily_quality) as avg_quality,
        STDDEV(daily_quality) as quality_stddev,
        AVG(daily_poor_percentage) as avg_poor_percentage,
        COUNT(DISTINCT query_date) as active_days,
        SUM(daily_queries) as total_queries,
        MAX(daily_quality) as best_daily_quality,
        MIN(daily_quality) as worst_daily_quality,
        CASE 
            WHEN STDDEV(daily_quality) < 0.5 THEN 'Stable Quality'
            WHEN STDDEV(daily_quality) < 1.0 THEN 'Moderate Variation'
            ELSE 'High Variation'
        END as quality_stability
    FROM daily_query_analysis
    GROUP BY query_name
),
performance_categories AS (
    SELECT 
        ta.*,
        CASE 
            WHEN avg_quality >= 3.0 AND avg_poor_percentage <= 20 THEN 'High Performing'
            WHEN avg_quality >= 2.0 AND avg_poor_percentage <= 35 THEN 'Good Performing'
            WHEN avg_quality >= 1.0 AND avg_poor_percentage <= 50 THEN 'Average Performing'
            ELSE 'Needs Improvement'
        END as overall_performance,
        CASE 
            WHEN total_queries >= 1000 THEN 'High Volume'
            WHEN total_queries >= 100 THEN 'Medium Volume'
            ELSE 'Low Volume'
        END as query_volume_category,
        ROUND(total_queries * 1.0 / active_days, 2) as avg_daily_volume
    FROM trending_analysis ta
)
SELECT 
    query_name,
    total_queries,
    active_days,
    avg_daily_volume,
    ROUND(avg_quality, 2) as quality,
    ROUND(avg_poor_percentage, 2) as poor_query_percentage,
    ROUND(quality_stddev, 2) as quality_variation,
    overall_performance,
    quality_stability,
    query_volume_category,
    CASE 
        WHEN avg_quality > 2.5 AND avg_poor_percentage < 25 THEN 'Monitor and maintain'
        WHEN avg_quality < 1.5 OR avg_poor_percentage > 50 THEN 'Requires immediate attention'
        WHEN quality_stddev > 1.0 THEN 'Investigate quality inconsistency'
        ELSE 'Standard monitoring'
    END as recommended_action
FROM performance_categories
ORDER BY 
    CASE WHEN overall_performance = 'Needs Improvement' THEN 1 ELSE 2 END,
    avg_quality DESC,
    total_queries DESC;
```

#### 2. **Position Impact and Result Relevance Analysis**
```sql
-- "Analyze how search position affects quality and user satisfaction"

WITH position_analysis AS (
    SELECT 
        query_name,
        CASE 
            WHEN position <= 3 THEN 'Top 3'
            WHEN position <= 10 THEN 'Top 10'
            WHEN position <= 50 THEN 'Top 50'
            WHEN position <= 100 THEN 'Top 100'
            ELSE 'Beyond 100'
        END as position_range,
        COUNT(*) as queries_count,
        AVG(rating) as avg_rating,
        ROUND(AVG(rating * 1.0 / position), 2) as avg_quality_score,
        COUNT(CASE WHEN rating < 3 THEN 1 END) as poor_ratings,
        COUNT(CASE WHEN rating >= 4 THEN 1 END) as good_ratings,
        MIN(rating) as min_rating,
        MAX(rating) as max_rating
    FROM Queries
    WHERE query_name IS NOT NULL
    GROUP BY query_name, 
        CASE 
            WHEN position <= 3 THEN 'Top 3'
            WHEN position <= 10 THEN 'Top 10'
            WHEN position <= 50 THEN 'Top 50'
            WHEN position <= 100 THEN 'Top 100'
            ELSE 'Beyond 100'
        END
),
result_relevance AS (
    SELECT 
        query_name,
        result,
        COUNT(*) as result_frequency,
        AVG(rating) as avg_result_rating,
        AVG(position) as avg_result_position,
        ROUND(AVG(rating * 1.0 / position), 2) as result_quality_score,
        COUNT(CASE WHEN rating < 3 THEN 1 END) as poor_result_count
    FROM Queries
    WHERE query_name IS NOT NULL
    GROUP BY query_name, result
),
comprehensive_analysis AS (
    SELECT 
        pa.query_name,
        pa.position_range,
        pa.queries_count,
        pa.avg_rating,
        pa.avg_quality_score,
        ROUND(pa.poor_ratings * 100.0 / pa.queries_count, 2) as poor_percentage,
        pa.good_ratings,
        rr.result_count,
        rr.avg_result_quality
    FROM position_analysis pa
    LEFT JOIN (
        SELECT 
            query_name,
            COUNT(DISTINCT result) as result_count,
            AVG(result_quality_score) as avg_result_quality
        FROM result_relevance
        GROUP BY query_name
    ) rr ON pa.query_name = rr.query_name
)
SELECT 
    ca.query_name,
    ca.position_range,
    ca.queries_count,
    ca.avg_rating,
    ca.avg_quality_score,
    ca.poor_percentage,
    ca.result_count,
    ROUND(ca.avg_result_quality, 2) as avg_result_quality,
    CASE 
        WHEN ca.position_range = 'Top 3' AND ca.avg_rating >= 4 THEN 'Excellent Top Results'
        WHEN ca.position_range = 'Top 10' AND ca.avg_rating >= 3.5 THEN 'Good Visibility'
        WHEN ca.position_range IN ('Top 50', 'Top 100') AND ca.avg_rating < 3 THEN 'Poor Lower Results'
        WHEN ca.position_range = 'Beyond 100' THEN 'Deep Search Results'
        ELSE 'Standard Performance'
    END as position_performance_category,
    CASE 
        WHEN ca.poor_percentage > 50 AND ca.position_range IN ('Top 3', 'Top 10') THEN 'Critical: Poor top results'
        WHEN ca.poor_percentage > 30 AND ca.result_count < 5 THEN 'Limited relevant results'
        WHEN ca.avg_quality_score < 1.0 THEN 'Low overall quality'
        ELSE 'Acceptable performance'
    END as quality_concern_level
FROM comprehensive_analysis ca
ORDER BY ca.query_name, 
    CASE ca.position_range 
        WHEN 'Top 3' THEN 1 
        WHEN 'Top 10' THEN 2 
        WHEN 'Top 50' THEN 3 
        WHEN 'Top 100' THEN 4 
        ELSE 5 
    END;
```

#### 3. **Query Performance Optimization Recommendations**
```sql
-- "Provide actionable insights for search algorithm improvement"

WITH query_performance_metrics AS (
    SELECT 
        query_name,
        COUNT(*) as total_queries,
        ROUND(AVG(rating * 1.0 / position), 2) as current_quality,
        ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as poor_query_percentage,
        AVG(rating) as avg_rating,
        AVG(position) as avg_position,
        COUNT(DISTINCT result) as unique_results,
        
        -- Position-based metrics
        COUNT(CASE WHEN position <= 10 THEN 1 END) as top_10_queries,
        AVG(CASE WHEN position <= 10 THEN rating END) as top_10_avg_rating,
        COUNT(CASE WHEN position > 100 THEN 1 END) as deep_position_queries,
        AVG(CASE WHEN position > 100 THEN rating END) as deep_position_avg_rating,
        
        -- Rating distribution
        COUNT(CASE WHEN rating = 5 THEN 1 END) as excellent_ratings,
        COUNT(CASE WHEN rating = 4 THEN 1 END) as good_ratings,
        COUNT(CASE WHEN rating = 3 THEN 1 END) as average_ratings,
        COUNT(CASE WHEN rating = 2 THEN 1 END) as poor_ratings,
        COUNT(CASE WHEN rating = 1 THEN 1 END) as very_poor_ratings,
        
        -- Quality potential calculation
        AVG(CASE WHEN position <= 3 THEN rating * 1.0 / position END) as top_3_quality_potential,
        AVG(CASE WHEN rating >= 4 THEN rating * 1.0 / position END) as high_rating_quality_potential
    FROM Queries
    WHERE query_name IS NOT NULL
    GROUP BY query_name
),
optimization_analysis AS (
    SELECT 
        *,
        -- Calculate improvement opportunities
        CASE 
            WHEN deep_position_queries > 0 AND deep_position_avg_rating >= 3 
            THEN ROUND((deep_position_queries * deep_position_avg_rating / 10) - 
                      (deep_position_queries * deep_position_avg_rating / avg_position), 2)
            ELSE 0
        END as position_improvement_potential,
        
        CASE 
            WHEN poor_query_percentage > 30 THEN 'High Priority'
            WHEN poor_query_percentage > 15 THEN 'Medium Priority'
            ELSE 'Low Priority'
        END as optimization_priority,
        
        CASE 
            WHEN current_quality < 1.5 THEN 'Algorithm Overhaul Needed'
            WHEN current_quality < 2.5 THEN 'Significant Improvements Required'
            WHEN current_quality < 3.5 THEN 'Minor Optimizations Needed'
            ELSE 'Maintain Current Performance'
        END as optimization_level,
        
        ROUND(
            CASE 
                WHEN top_3_quality_potential IS NOT NULL 
                THEN top_3_quality_potential - current_quality 
                ELSE 0 
            END, 2
        ) as quality_improvement_potential
    FROM query_performance_metrics
),
recommendations AS (
    SELECT 
        query_name,
        total_queries,
        current_quality,
        poor_query_percentage,
        optimization_priority,
        optimization_level,
        quality_improvement_potential,
        
        -- Specific recommendations
        CASE 
            WHEN avg_position > 50 AND avg_rating >= 3.5 THEN 'Boost ranking algorithm for relevant results'
            WHEN poor_query_percentage > 40 THEN 'Filter or demote low-quality results'
            WHEN unique_results < 10 AND total_queries > 100 THEN 'Expand result diversity'
            WHEN top_10_avg_rating < 3.0 THEN 'Improve top result relevance'
            WHEN deep_position_queries > total_queries * 0.3 THEN 'Enhance result ranking'
            ELSE 'Monitor and maintain current performance'
        END as primary_recommendation,
        
        CASE 
            WHEN excellent_ratings + good_ratings < total_queries * 0.6 THEN 'Focus on result quality'
            WHEN very_poor_ratings + poor_ratings > total_queries * 0.3 THEN 'Remove poor quality results'
            WHEN quality_improvement_potential > 1.0 THEN 'Optimize high-potential queries'
            ELSE 'Incremental improvements'
        END as secondary_recommendation,
        
        -- Implementation suggestions
        CONCAT(
            CASE WHEN avg_position > 30 THEN 'Ranking optimization; ' ELSE '' END,
            CASE WHEN poor_query_percentage > 25 THEN 'Quality filtering; ' ELSE '' END,
            CASE WHEN unique_results < 15 THEN 'Result diversity; ' ELSE '' END,
            CASE WHEN quality_improvement_potential > 0.5 THEN 'Algorithm tuning' ELSE 'Standard monitoring' END
        ) as implementation_focus
    FROM optimization_analysis
)
SELECT 
    query_name,
    total_queries,
    current_quality,
    poor_query_percentage,
    optimization_priority,
    optimization_level,
    quality_improvement_potential,
    primary_recommendation,
    secondary_recommendation,
    implementation_focus,
    CASE 
        WHEN optimization_priority = 'High Priority' AND quality_improvement_potential > 1.0 THEN 'Immediate Action Required'
        WHEN optimization_priority = 'Medium Priority' THEN 'Schedule for Next Sprint'
        ELSE 'Monitor Quarterly'
    END as timeline_recommendation
FROM recommendations
ORDER BY 
    CASE optimization_priority 
        WHEN 'High Priority' THEN 1 
        WHEN 'Medium Priority' THEN 2 
        ELSE 3 
    END,
    quality_improvement_potential DESC,
    total_queries DESC;
```

#### 4. **Real-time Quality Monitoring Dashboard**
```sql
-- "Create metrics for real-time search quality monitoring"

WITH real_time_metrics AS (
    SELECT 
        query_name,
        DATE(created_at) as metric_date,
        HOUR(created_at) as metric_hour,
        COUNT(*) as hourly_queries,
        ROUND(AVG(rating * 1.0 / position), 2) as hourly_quality,
        ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 2) as hourly_poor_percentage,
        AVG(rating) as hourly_avg_rating,
        AVG(position) as hourly_avg_position,
        COUNT(DISTINCT result) as hourly_unique_results
    FROM Queries
    WHERE query_name IS NOT NULL
      AND created_at >= CURRENT_TIMESTAMP - INTERVAL 24 HOUR  -- Last 24 hours
    GROUP BY query_name, DATE(created_at), HOUR(created_at)
),
quality_alerts AS (
    SELECT 
        *,
        LAG(hourly_quality) OVER (PARTITION BY query_name ORDER BY metric_date, metric_hour) as prev_hour_quality,
        LAG(hourly_poor_percentage) OVER (PARTITION BY query_name ORDER BY metric_date, metric_hour) as prev_hour_poor_percentage,
        AVG(hourly_quality) OVER (PARTITION BY query_name ORDER BY metric_date, metric_hour ROWS BETWEEN 23 PRECEDING AND CURRENT ROW) as rolling_24h_quality,
        AVG(hourly_poor_percentage) OVER (PARTITION BY query_name ORDER BY metric_date, metric_hour ROWS BETWEEN 23 PRECEDING AND CURRENT ROW) as rolling_24h_poor_percentage
    FROM real_time_metrics
),
alert_conditions AS (
    SELECT 
        *,
        ABS(hourly_quality - prev_hour_quality) as quality_change,
        ABS(hourly_poor_percentage - prev_hour_poor_percentage) as poor_percentage_change,
        CASE 
            WHEN hourly_quality < 1.0 THEN 'CRITICAL: Very Low Quality'
            WHEN hourly_poor_percentage > 60 THEN 'CRITICAL: High Poor Rate'
            WHEN ABS(hourly_quality - prev_hour_quality) > 1.0 THEN 'WARNING: Quality Drop'
            WHEN ABS(hourly_poor_percentage - prev_hour_poor_percentage) > 25 THEN 'WARNING: Poor Rate Spike'
            WHEN hourly_queries < 5 AND rolling_24h_quality < 2.0 THEN 'INFO: Low Volume Poor Quality'
            ELSE 'OK'
        END as alert_level,
        CASE 
            WHEN hourly_quality > rolling_24h_quality * 1.2 THEN 'IMPROVEMENT'
            WHEN hourly_quality < rolling_24h_quality * 0.8 THEN 'DEGRADATION'
            ELSE 'STABLE'
        END as trend_status
    FROM quality_alerts
    WHERE prev_hour_quality IS NOT NULL
)
SELECT 
    query_name,
    CONCAT(metric_date, ' ', LPAD(metric_hour, 2, '0'), ':00') as time_period,
    hourly_queries,
    hourly_quality as current_quality,
    hourly_poor_percentage as current_poor_percentage,
    ROUND(rolling_24h_quality, 2) as rolling_24h_quality,
    ROUND(rolling_24h_poor_percentage, 2) as rolling_24h_poor_percentage,
    ROUND(quality_change, 2) as quality_change,
    ROUND(poor_percentage_change, 2) as poor_rate_change,
    alert_level,
    trend_status,
    CASE 
        WHEN alert_level LIKE 'CRITICAL%' THEN 'Immediate investigation required'
        WHEN alert_level LIKE 'WARNING%' THEN 'Monitor closely next hour'
        WHEN trend_status = 'IMPROVEMENT' THEN 'Positive trend - analyze what improved'
        WHEN trend_status = 'DEGRADATION' THEN 'Investigate potential issues'
        ELSE 'Continue monitoring'
    END as recommended_action
FROM alert_conditions
WHERE metric_date >= CURRENT_DATE - INTERVAL 1 DAY
ORDER BY 
    CASE alert_level 
        WHEN 'CRITICAL: Very Low Quality' THEN 1
        WHEN 'CRITICAL: High Poor Rate' THEN 2
        WHEN 'WARNING: Quality Drop' THEN 3
        WHEN 'WARNING: Poor Rate Spike' THEN 4
        ELSE 5 
    END,
    metric_date DESC, 
    metric_hour DESC,
    query_name;
```

#### 5. **Machine Learning Feature Engineering for Query Quality**
```sql
-- "Generate features for ML models to predict and improve query quality"

WITH query_features AS (
    SELECT 
        query_name,
        result,
        position,
        rating,
        
        -- Basic features
        LENGTH(query_name) as query_length,
        LENGTH(result) as result_length,
        CASE 
            WHEN query_name LIKE '% %' THEN 1 ELSE 0 
        END as is_multi_word,
        
        -- Position-based features
        CASE 
            WHEN position <= 5 THEN 1 ELSE 0 
        END as is_top_5,
        CASE 
            WHEN position <= 10 THEN 1 ELSE 0 
        END as is_top_10,
        LOG(position) as log_position,
        
        -- Historical features (window functions)
        AVG(rating) OVER (PARTITION BY query_name) as query_avg_rating,
        AVG(position) OVER (PARTITION BY query_name) as query_avg_position,
        COUNT(*) OVER (PARTITION BY query_name) as query_frequency,
        COUNT(*) OVER (PARTITION BY result) as result_frequency,
        
        -- Cross-features
        rating * 1.0 / position as quality_score,
        CASE WHEN rating >= 4 AND position <= 10 THEN 1 ELSE 0 END as high_quality_top_result,
        CASE WHEN rating < 3 AND position <= 5 THEN 1 ELSE 0 END as poor_quality_top_result
    FROM Queries
    WHERE query_name IS NOT NULL
),
aggregated_features AS (
    SELECT 
        query_name,
        
        -- Target variables
        ROUND(AVG(rating * 1.0 / position), 4) as quality_score,
        ROUND((COUNT(CASE WHEN rating < 3 THEN 1 END) * 100.0) / COUNT(*), 4) as poor_query_percentage,
        
        -- Basic aggregated features
        COUNT(*) as total_queries,
        AVG(query_length) as avg_query_length,
        AVG(result_length) as avg_result_length,
        AVG(position) as avg_position,
        AVG(rating) as avg_rating,
        
        -- Distribution features
        STDDEV(rating) as rating_stddev,
        STDDEV(position) as position_stddev,
        MIN(position) as min_position,
        MAX(position) as max_position,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY position) as median_position,
        
        -- Boolean aggregations
        SUM(is_multi_word) * 1.0 / COUNT(*) as multi_word_ratio,
        SUM(is_top_5) * 1.0 / COUNT(*) as top_5_ratio,
        SUM(is_top_10) * 1.0 / COUNT(*) as top_10_ratio,
        SUM(high_quality_top_result) * 1.0 / COUNT(*) as high_quality_top_ratio,
        SUM(poor_quality_top_result) * 1.0 / COUNT(*) as poor_quality_top_ratio,
        
        -- Advanced features
        COUNT(DISTINCT result) as unique_results,
        COUNT(DISTINCT result) * 1.0 / COUNT(*) as result_diversity_ratio,
        AVG(result_frequency) as avg_result_popularity,
        
        -- Rating distribution
        COUNT(CASE WHEN rating = 5 THEN 1 END) * 1.0 / COUNT(*) as rating_5_ratio,
        COUNT(CASE WHEN rating = 4 THEN 1 END) * 1.0 / COUNT(*) as rating_4_ratio,
        COUNT(CASE WHEN rating = 3 THEN 1 END) * 1.0 / COUNT(*) as rating_3_ratio,
        COUNT(CASE WHEN rating = 2 THEN 1 END) * 1.0 / COUNT(*) as rating_2_ratio,
        COUNT(CASE WHEN rating = 1 THEN 1 END) * 1.0 / COUNT(*) as rating_1_ratio
    FROM query_features
    GROUP BY query_name
),
ml_ready_features AS (
    SELECT 
        query_name,
        quality_score,
        poor_query_percentage,
        
        -- Normalized features (0-1 scale)
        ROUND((avg_query_length - 3.0) / 47.0, 4) as norm_query_length,  -- Assuming min=3, max=50
        ROUND((avg_result_length - 5.0) / 95.0, 4) as norm_result_length,  -- Assuming min=5, max=100
        ROUND(LOG(avg_position) / LOG(500), 4) as norm_log_position,  -- position max = 500
        ROUND((avg_rating - 1.0) / 4.0, 4) as norm_avg_rating,  -- rating 1-5
        
        -- Categorical encodings
        CASE 
            WHEN total_queries >= 100 THEN 3
            WHEN total_queries >= 20 THEN 2
            WHEN total_queries >= 5 THEN 1
            ELSE 0
        END as query_volume_category,
        
        CASE 
            WHEN top_10_ratio >= 0.8 THEN 3
            WHEN top_10_ratio >= 0.5 THEN 2
            WHEN top_10_ratio >= 0.2 THEN 1
            ELSE 0
        END as top_position_tendency,
        
        CASE 
            WHEN result_diversity_ratio >= 0.8 THEN 3
            WHEN result_diversity_ratio >= 0.5 THEN 2
            WHEN result_diversity_ratio >= 0.2 THEN 1
            ELSE 0
        END as result_diversity_level,
        
        -- Interaction features
        ROUND(top_10_ratio * norm_avg_rating, 4) as position_rating_interaction,
        ROUND(result_diversity_ratio * norm_avg_rating, 4) as diversity_quality_interaction,
        ROUND(multi_word_ratio * unique_results, 4) as complexity_results_interaction,
        
        -- Quality indicators
        high_quality_top_ratio,
        poor_quality_top_ratio,
        rating_5_ratio,
        rating_1_ratio,
        
        -- All original features for reference
        total_queries,
        avg_position,
        avg_rating,
        unique_results,
        result_diversity_ratio
    FROM aggregated_features
)
SELECT 
    query_name,
    -- Target variables
    quality_score,
    poor_query_percentage,
    
    -- ML Features
    norm_query_length,
    norm_result_length,
    norm_log_position,
    norm_avg_rating,
    query_volume_category,
    top_position_tendency,
    result_diversity_level,
    position_rating_interaction,
    diversity_quality_interaction,
    complexity_results_interaction,
    high_quality_top_ratio,
    poor_quality_top_ratio,
    rating_5_ratio,
    rating_1_ratio,
    
    -- Additional context
    total_queries,
    avg_position,
    avg_rating,
    unique_results,
    
    -- Model prediction categories (for training labels)
    CASE 
        WHEN quality_score >= 3.0 THEN 'high_quality'
        WHEN quality_score >= 2.0 THEN 'medium_quality'
        WHEN quality_score >= 1.0 THEN 'low_quality'
        ELSE 'very_low_quality'
    END as quality_label,
    
    CASE 
        WHEN poor_query_percentage <= 15 THEN 'low_poor_rate'
        WHEN poor_query_percentage <= 35 THEN 'medium_poor_rate'
        ELSE 'high_poor_rate'
    END as poor_rate_label
FROM ml_ready_features
ORDER BY quality_score DESC, poor_query_percentage ASC;
```

## 🔗 Related LeetCode Questions

1. **#550 - Game Play Analysis IV** (Conditional aggregation with fractions)
2. **#1045 - Customers Who Bought All Products** (GROUP BY with counting)
3. **#1070 - Product Sales Analysis III** (GROUP BY with calculations)
4. **#1204 - Last Person to Fit in Bus** (Running totals and aggregation)
5. **#1158 - Market Analysis I** (JOIN with aggregation and filtering)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY Aggregation**: Grouping data for calculation
2. **Conditional Aggregation**: Using CASE in COUNT/SUM
3. **Mathematical Operations**: Division, multiplication, percentages
4. **ROUND Function**: Formatting decimal precision

### 🚀 **Amazon Interview Tips**
1. **Clarify calculations**: "Quality is average of rating/position ratios, correct?"
2. **Handle edge cases**: "What about queries with no ratings or zero positions?"
3. **Discuss performance**: "GROUP BY with aggregation functions scales well with indexes"
4. **Business context**: "This helps optimize search algorithms and user experience"

### 🔧 **Common Patterns**
- GROUP BY with multiple aggregation functions
- Conditional counting with CASE statements
- Percentage calculations with proper type casting
- ROUND for consistent decimal formatting

### ⚠️ **Common Mistakes to Avoid**
1. **Integer division** (use 1.0 or CAST for proper decimals)
2. **Forgetting ROUND** (returning unformatted decimals)
3. **Wrong percentage calculation** (forgetting * 100)
4. **NULL handling** (not filtering NULL query_names)

### 🔍 **Performance Considerations**
- Index on query_name for efficient GROUP BY
- Consider partial aggregation for large datasets
- Monitor memory usage with large grouping operations
- Use appropriate data types for calculations

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Measuring search quality to improve user experience
- **Data-Driven Decisions**: Using metrics to guide search algorithm improvements
- **Operational Excellence**: Monitoring query performance for system optimization
- **Innovation**: Advanced analytics for search quality enhancement

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate quality trends over time periods**
2. **Find queries with improving vs declining quality**
3. **Rank queries by different quality metrics**
4. **Calculate position-weighted ratings differently**

Remember: Search quality metrics are fundamental to Amazon's e-commerce platform, driving product discovery, recommendation systems, advertising relevance, and overall customer satisfaction across all digital properties!