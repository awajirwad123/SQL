# LeetCode Easy #1148: Article Views I

## 📋 Problem Statement

Write a SQL query to find all the **authors that viewed at least one of their own articles**.

Return the result table **sorted by id in ascending order**.

## 🗄️ Table Schema

**Views Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| article_id    | int     |
| author_id     | int     |
| viewer_id     | int     |
| view_date     | date    |
+---------------+---------+
```
- There is no primary key for this table, the table may have duplicate rows.
- Each row of this table indicates that some viewer viewed an article (written by some author) on some date.
- Note that equal `author_id` and `viewer_id` indicate the same person.

## 📊 Sample Data

**Views Table:**
| article_id | author_id | viewer_id | view_date  |
|------------|-----------|-----------|------------|
| 1          | 3         | 5         | 2019-08-01 |
| 1          | 3         | 6         | 2019-08-02 |
| 2          | 7         | 7         | 2019-08-01 |
| 2          | 7         | 6         | 2019-08-02 |
| 4          | 7         | 1         | 2019-07-22 |
| 3          | 4         | 4         | 2019-07-21 |
| 3          | 4         | 4         | 2019-07-21 |

**Expected Output:**
| id |
|----|
| 4  |
| 7  |

**Explanation:** 
- Author 4 viewed their own article 3 (viewer_id = author_id = 4)
- Author 7 viewed their own article 2 (viewer_id = author_id = 7)
- Author 3 did not view their own articles (viewers were 5 and 6, not 3)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find authors who viewed their own articles
- Self-viewing means author_id = viewer_id
- Need DISTINCT to avoid duplicate authors
- Sort results by author ID

### 2. **Key Insights**
- Simple WHERE clause with author_id = viewer_id
- Use DISTINCT to get unique author IDs
- No JOIN needed - all data in one table
- ORDER BY for required sorting

### 3. **Interview Discussion Points**
- "This is a basic filtering problem with self-reference"
- "I need to find records where author and viewer are the same person"
- "DISTINCT ensures each author appears only once in results"

## 🔧 Step-by-Step Solution Logic

### Step 1: Filter Self-Views
```sql
-- WHERE author_id = viewer_id
-- This identifies cases where authors viewed their own content
```

### Step 2: Get Unique Authors
```sql
-- SELECT DISTINCT author_id
-- Avoid duplicate entries for authors who self-viewed multiple times
```

### Step 3: Sort Results
```sql
-- ORDER BY author_id ASC
-- Return results in ascending order as required
```

## ✅ Optimized SQL Solution

**Solution 1: Basic DISTINCT with WHERE**
```sql
SELECT DISTINCT author_id as id
FROM Views
WHERE author_id = viewer_id
ORDER BY author_id;
```

### Alternative Solutions

**Solution 2: Using GROUP BY**
```sql
SELECT author_id as id
FROM Views
WHERE author_id = viewer_id
GROUP BY author_id
ORDER BY author_id;
```

**Solution 3: With Self-View Count**
```sql
SELECT 
    author_id as id,
    COUNT(*) as self_views_count
FROM Views
WHERE author_id = viewer_id
GROUP BY author_id
ORDER BY author_id;
```

**Solution 4: Using EXISTS**
```sql
SELECT DISTINCT author_id as id
FROM Views v1
WHERE EXISTS (
    SELECT 1 
    FROM Views v2 
    WHERE v2.author_id = v1.author_id 
    AND v2.author_id = v2.viewer_id
)
ORDER BY author_id;
```

**Solution 5: With Additional Metrics**
```sql
SELECT 
    author_id as id,
    COUNT(DISTINCT article_id) as self_viewed_articles,
    COUNT(*) as total_self_views,
    MIN(view_date) as first_self_view,
    MAX(view_date) as last_self_view
FROM Views
WHERE author_id = viewer_id
GROUP BY author_id
ORDER BY author_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Author Analytics**
```sql
-- "Analyze author behavior and engagement patterns"

WITH author_metrics AS (
    SELECT 
        author_id,
        COUNT(DISTINCT article_id) as articles_written,
        COUNT(DISTINCT CASE WHEN author_id = viewer_id THEN article_id END) as self_viewed_articles,
        COUNT(CASE WHEN author_id = viewer_id THEN 1 END) as total_self_views,
        COUNT(DISTINCT CASE WHEN author_id != viewer_id THEN viewer_id END) as unique_external_viewers,
        COUNT(CASE WHEN author_id != viewer_id THEN 1 END) as total_external_views,
        MIN(view_date) as first_view_date,
        MAX(view_date) as last_view_date
    FROM Views
    GROUP BY author_id
),
author_analysis AS (
    SELECT 
        *,
        ROUND(self_viewed_articles * 100.0 / articles_written, 2) as self_view_article_percentage,
        ROUND(total_self_views * 100.0 / (total_self_views + total_external_views), 2) as self_view_percentage,
        CASE 
            WHEN self_viewed_articles = articles_written THEN 'All Articles Self-Viewed'
            WHEN self_viewed_articles > 0 THEN 'Partial Self-Viewer'
            ELSE 'No Self-Views'
        END as self_viewing_behavior,
        CASE 
            WHEN unique_external_viewers = 0 THEN 'No External Audience'
            WHEN unique_external_viewers <= 2 THEN 'Limited Audience'
            WHEN unique_external_viewers <= 5 THEN 'Moderate Audience'
            ELSE 'Wide Audience'
        END as audience_reach
    FROM author_metrics
)
SELECT 
    author_id as id,
    articles_written,
    self_viewed_articles,
    total_self_views,
    unique_external_viewers,
    total_external_views,
    self_view_article_percentage,
    self_view_percentage,
    self_viewing_behavior,
    audience_reach,
    CASE 
        WHEN self_viewed_articles > 0 AND unique_external_viewers > 3 THEN 'Engaged Author'
        WHEN self_viewed_articles > 0 THEN 'Self-Conscious Author'
        WHEN unique_external_viewers > 5 THEN 'Popular Author'
        ELSE 'Regular Author'
    END as author_classification
FROM author_analysis
ORDER BY 
    CASE WHEN self_viewed_articles > 0 THEN 1 ELSE 2 END,
    total_external_views DESC,
    author_id;
```

#### 2. **Article Performance and Self-View Impact**
```sql
-- "Analyze how self-views relate to article performance"

WITH article_metrics AS (
    SELECT 
        article_id,
        author_id,
        COUNT(*) as total_views,
        COUNT(DISTINCT viewer_id) as unique_viewers,
        COUNT(CASE WHEN author_id = viewer_id THEN 1 END) as self_views,
        COUNT(CASE WHEN author_id != viewer_id THEN 1 END) as external_views,
        COUNT(DISTINCT CASE WHEN author_id != viewer_id THEN viewer_id END) as unique_external_viewers,
        MIN(view_date) as first_view_date,
        MAX(view_date) as last_view_date,
        DATEDIFF(MAX(view_date), MIN(view_date)) + 1 as active_days
    FROM Views
    GROUP BY article_id, author_id
),
performance_analysis AS (
    SELECT 
        *,
        CASE WHEN self_views > 0 THEN 'Self-Viewed' ELSE 'Not Self-Viewed' END as self_view_status,
        ROUND(external_views * 1.0 / NULLIF(active_days, 0), 2) as daily_external_engagement,
        CASE 
            WHEN external_views >= 10 THEN 'High Performance'
            WHEN external_views >= 5 THEN 'Medium Performance'
            WHEN external_views >= 1 THEN 'Low Performance'
            ELSE 'No External Engagement'
        END as performance_tier,
        CASE 
            WHEN self_views > 0 AND external_views = 0 THEN 'Self-Only Article'
            WHEN self_views > 0 AND external_views > 0 THEN 'Self-Promoted Article'
            WHEN self_views = 0 AND external_views > 0 THEN 'Organically Popular'
            ELSE 'No Engagement'
        END as engagement_pattern
    FROM article_metrics
),
correlation_analysis AS (
    SELECT 
        self_view_status,
        COUNT(*) as article_count,
        AVG(external_views) as avg_external_views,
        AVG(unique_external_viewers) as avg_unique_external_viewers,
        AVG(daily_external_engagement) as avg_daily_engagement,
        MAX(external_views) as max_external_views
    FROM performance_analysis
    GROUP BY self_view_status
)
SELECT 
    pa.article_id,
    pa.author_id,
    pa.self_view_status,
    pa.total_views,
    pa.external_views,
    pa.unique_external_viewers,
    pa.performance_tier,
    pa.engagement_pattern,
    ROUND(pa.daily_external_engagement, 2) as daily_engagement_rate,
    ca.avg_external_views as category_avg_external_views,
    CASE 
        WHEN pa.external_views > ca.avg_external_views * 1.5 THEN 'Above Category Average'
        WHEN pa.external_views > ca.avg_external_views * 0.5 THEN 'Near Category Average'
        ELSE 'Below Category Average'
    END as relative_performance
FROM performance_analysis pa
JOIN correlation_analysis ca ON pa.self_view_status = ca.self_view_status
ORDER BY pa.self_view_status, pa.external_views DESC;
```

#### 3. **Time-Based Self-Viewing Patterns**
```sql
-- "Analyze when authors view their own content"

WITH self_view_timing AS (
    SELECT 
        author_id,
        article_id,
        view_date,
        ROW_NUMBER() OVER (PARTITION BY author_id, article_id ORDER BY view_date) as self_view_sequence,
        LAG(view_date) OVER (PARTITION BY author_id, article_id ORDER BY view_date) as prev_self_view_date
    FROM Views
    WHERE author_id = viewer_id
),
first_views_analysis AS (
    SELECT 
        v.article_id,
        v.author_id,
        MIN(v.view_date) as article_first_view,
        MIN(CASE WHEN v.author_id = v.viewer_id THEN v.view_date END) as first_self_view,
        COUNT(DISTINCT CASE WHEN v.author_id != v.viewer_id THEN v.viewer_id END) as external_viewers_before_self_view
    FROM Views v
    GROUP BY v.article_id, v.author_id
),
timing_patterns AS (
    SELECT 
        svt.author_id,
        svt.article_id,
        svt.view_date as self_view_date,
        fva.article_first_view,
        fva.first_self_view,
        DATEDIFF(svt.view_date, fva.article_first_view) as days_after_first_view,
        DATEDIFF(svt.view_date, svt.prev_self_view_date) as days_since_last_self_view,
        svt.self_view_sequence,
        CASE 
            WHEN svt.view_date = fva.article_first_view THEN 'Immediate Self-View'
            WHEN DATEDIFF(svt.view_date, fva.article_first_view) <= 1 THEN 'Next Day Self-View'
            WHEN DATEDIFF(svt.view_date, fva.article_first_view) <= 7 THEN 'Weekly Self-View'
            ELSE 'Delayed Self-View'
        END as self_view_timing_category
    FROM self_view_timing svt
    JOIN first_views_analysis fva ON svt.article_id = fva.article_id AND svt.author_id = fva.author_id
),
author_timing_summary AS (
    SELECT 
        author_id,
        COUNT(DISTINCT article_id) as self_viewed_articles,
        AVG(days_after_first_view) as avg_days_to_self_view,
        COUNT(CASE WHEN self_view_timing_category = 'Immediate Self-View' THEN 1 END) as immediate_self_views,
        COUNT(CASE WHEN self_view_timing_category = 'Delayed Self-View' THEN 1 END) as delayed_self_views,
        MAX(self_view_sequence) as max_self_views_per_article
    FROM timing_patterns
    GROUP BY author_id
)
SELECT 
    ats.author_id as id,
    ats.self_viewed_articles,
    ROUND(ats.avg_days_to_self_view, 1) as avg_days_to_self_view,
    ats.immediate_self_views,
    ats.delayed_self_views,
    ats.max_self_views_per_article,
    ROUND(ats.immediate_self_views * 100.0 / ats.self_viewed_articles, 2) as immediate_self_view_rate,
    CASE 
        WHEN ats.avg_days_to_self_view <= 1 THEN 'Immediate Self-Viewer'
        WHEN ats.avg_days_to_self_view <= 7 THEN 'Regular Self-Viewer'
        ELSE 'Occasional Self-Viewer'
    END as self_viewing_pattern,
    CASE 
        WHEN ats.max_self_views_per_article > 3 THEN 'Frequent Re-viewer'
        WHEN ats.max_self_views_per_article > 1 THEN 'Occasional Re-viewer'
        ELSE 'Single Self-viewer'
    END as re_viewing_behavior
FROM author_timing_summary ats
ORDER BY ats.immediate_self_views DESC, ats.avg_days_to_self_view ASC;
```

#### 4. **Performance Optimization for Large Datasets**
```sql
-- "Optimize for large-scale view tracking data"

-- Create indexes for efficient self-view detection
-- CREATE INDEX idx_views_author_viewer ON Views(author_id, viewer_id);
-- CREATE INDEX idx_views_article_author ON Views(article_id, author_id);
-- CREATE INDEX idx_views_date ON Views(view_date);

-- Optimized basic query
EXPLAIN ANALYZE
SELECT DISTINCT author_id as id
FROM Views
WHERE author_id = viewer_id
ORDER BY author_id;

-- Batch processing approach for very large datasets
WITH self_view_batch AS (
    SELECT DISTINCT author_id
    FROM Views
    WHERE author_id = viewer_id
    AND view_date >= CURRENT_DATE - INTERVAL 30 DAY  -- Recent data only
)
SELECT author_id as id
FROM self_view_batch
ORDER BY author_id;

-- Materialized view approach for frequent queries
-- CREATE MATERIALIZED VIEW author_self_view_summary AS
-- SELECT 
--     author_id,
--     COUNT(DISTINCT article_id) as self_viewed_articles,
--     COUNT(*) as total_self_views,
--     MIN(view_date) as first_self_view,
--     MAX(view_date) as latest_self_view
-- FROM Views
-- WHERE author_id = viewer_id
-- GROUP BY author_id;

-- Usage of materialized view:
-- SELECT author_id as id FROM author_self_view_summary ORDER BY author_id;

-- Partitioned query for time-based analysis
SELECT DISTINCT author_id as id
FROM Views
WHERE author_id = viewer_id
  AND view_date >= '2019-07-01'  -- Partition by month/quarter
  AND view_date < '2019-08-01'
ORDER BY author_id;
```

#### 5. **Content Strategy Insights**
```sql
-- "Provide insights for content strategy based on self-viewing patterns"

WITH content_strategy_metrics AS (
    SELECT 
        v.author_id,
        COUNT(DISTINCT v.article_id) as total_articles,
        COUNT(DISTINCT CASE WHEN v.author_id = v.viewer_id THEN v.article_id END) as self_viewed_articles,
        COUNT(DISTINCT CASE WHEN v.author_id != v.viewer_id THEN v.viewer_id END) as unique_readers,
        COUNT(CASE WHEN v.author_id != v.viewer_id THEN 1 END) as external_views,
        COUNT(CASE WHEN v.author_id = v.viewer_id THEN 1 END) as self_views,
        AVG(article_stats.avg_external_views) as avg_external_views_per_article,
        MAX(article_stats.max_external_views) as best_performing_article_views
    FROM Views v
    JOIN (
        SELECT 
            article_id,
            author_id,
            COUNT(CASE WHEN author_id != viewer_id THEN 1 END) as avg_external_views,
            COUNT(CASE WHEN author_id != viewer_id THEN 1 END) as max_external_views
        FROM Views
        GROUP BY article_id, author_id
    ) article_stats ON v.article_id = article_stats.article_id AND v.author_id = article_stats.author_id
    GROUP BY v.author_id
),
strategy_analysis AS (
    SELECT 
        *,
        ROUND(self_viewed_articles * 100.0 / total_articles, 2) as self_view_coverage,
        ROUND(self_views * 100.0 / (self_views + external_views), 2) as self_view_ratio,
        ROUND(external_views * 1.0 / total_articles, 2) as avg_external_engagement,
        CASE 
            WHEN self_viewed_articles = 0 THEN 'Detached Author'
            WHEN self_viewed_articles = total_articles THEN 'Fully Self-Aware Author'
            WHEN self_viewed_articles > total_articles * 0.5 THEN 'Highly Self-Aware Author'
            ELSE 'Moderately Self-Aware Author'
        END as self_awareness_level,
        CASE 
            WHEN unique_readers >= 10 THEN 'Wide Reach Author'
            WHEN unique_readers >= 5 THEN 'Moderate Reach Author'
            WHEN unique_readers >= 1 THEN 'Limited Reach Author'
            ELSE 'No External Reach'
        END as audience_reach_level
    FROM content_strategy_metrics
),
recommendations AS (
    SELECT 
        author_id,
        self_awareness_level,
        audience_reach_level,
        CASE 
            WHEN self_view_coverage > 80 AND avg_external_engagement < 2 THEN 'Focus on external promotion'
            WHEN self_view_coverage < 20 AND avg_external_engagement > 5 THEN 'Monitor content quality'
            WHEN unique_readers < 3 THEN 'Improve content discoverability'
            WHEN self_view_ratio > 50 THEN 'Reduce self-viewing frequency'
            ELSE 'Continue current strategy'
        END as content_strategy_recommendation,
        CASE 
            WHEN best_performing_article_views > avg_external_views_per_article * 3 THEN 'Analyze top content for patterns'
            WHEN total_articles > 5 AND unique_readers < total_articles THEN 'Focus on reader retention'
            ELSE 'Maintain content consistency'
        END as growth_strategy_recommendation
    FROM strategy_analysis
)
SELECT 
    sa.author_id as id,
    sa.total_articles,
    sa.self_viewed_articles,
    sa.unique_readers,
    sa.self_view_coverage,
    sa.self_awareness_level,
    sa.audience_reach_level,
    r.content_strategy_recommendation,
    r.growth_strategy_recommendation,
    CASE 
        WHEN sa.self_viewed_articles > 0 THEN 'Include in self-viewer analysis'
        ELSE 'Standard author tracking'
    END as analysis_priority
FROM strategy_analysis sa
JOIN recommendations r ON sa.author_id = r.author_id
ORDER BY 
    CASE WHEN sa.self_viewed_articles > 0 THEN 1 ELSE 2 END,
    sa.unique_readers DESC,
    sa.author_id;
```

## 🔗 Related LeetCode Questions

1. **#196 - Delete Duplicate Emails** (Self-referencing operations)
2. **#181 - Employees Earning More Than Managers** (Self-comparison logic)
3. **#1141 - User Activity for the Past 30 Days I** (DISTINCT counting)
4. **#586 - Customer Placing the Largest Number of Orders** (GROUP BY with aggregation)
5. **#1050 - Actors and Directors Who Cooperated At Least Three Times** (Relationship analysis)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Referencing**: Comparing columns within same table
2. **DISTINCT**: Eliminating duplicate results
3. **Simple Filtering**: Using WHERE with equality conditions
4. **ORDER BY**: Sorting results as required

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I include the count of self-views or just the authors?"
2. **Discuss performance**: "Simple equality filter with index performs well"
3. **Consider edge cases**: "What about authors with no articles or views?"
4. **Think business context**: "This helps identify engaged content creators"

### 🔧 **Common Patterns**
- Self-referencing with column equality (author_id = viewer_id)
- DISTINCT for unique results
- Basic filtering with WHERE clause
- ORDER BY for result organization

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting DISTINCT** (returning duplicate authors)
2. **Wrong column comparison** (using != instead of =)
3. **Missing ORDER BY** when specific sorting is required
4. **Performance issues** (not indexing compared columns)

### 🔍 **Performance Considerations**
- Index on (author_id, viewer_id) for efficient equality checks
- DISTINCT vs GROUP BY performance considerations
- Consider covering indexes for larger datasets
- Query execution plan analysis for optimization

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding content creator engagement
- **Data-Driven Decisions**: Using author behavior for platform improvements
- **Operational Excellence**: Efficient content analytics and reporting
- **Innovation**: Creative approaches to content engagement analysis

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find authors who never viewed their own articles**
2. **Count how many times each author viewed their own content**
3. **Find the most self-viewed article**
4. **Calculate the percentage of authors who self-view**

Remember: Content creator behavior analysis is crucial for Amazon's content platforms, author engagement strategies, recommendation systems, and understanding creator-user dynamics across digital properties!