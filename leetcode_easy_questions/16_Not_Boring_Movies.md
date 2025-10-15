# LeetCode Easy #620: Not Boring Movies

## 📋 Problem Statement

Write a SQL query to report all movies with an **odd-numbered ID** and a **description that is not "boring"**.

Return the result table **ordered by rating in descending order**.

## 🗄️ Table Schema

**Cinema Table:**
```
+----------------+----------+
| Column Name    | Type     |
+----------------+----------+
| id             | int      |
| movie          | varchar  |
| description    | varchar  |
| rating         | float    |
+----------------+----------+
```
- `id` is the primary key for this table.
- Each row contains information about the name of a movie, its genre, and its rating.
- `rating` is a 2-decimal places float in the range [0, 10]

## 📊 Sample Data

**Cinema Table:**
| id | movie      | description | rating |
|----|------------|-------------|--------|
| 1  | War        | great 3D    | 8.9    |
| 2  | Science    | fiction     | 8.5    |
| 3  | irish      | boring      | 6.2    |
| 4  | Ice song   | Fantacy     | 8.6    |
| 5  | House card | Interesting | 9.1    |

**Expected Output:**
| id | movie      | description | rating |
|----|------------|-------------|--------|
| 5  | House card | Interesting | 9.1    |
| 1  | War        | great 3D    | 8.9    |

**Explanation:** 
- Movies with odd IDs: 1, 3, 5
- Movies that are not "boring": 1, 5 (ID 3 has description "boring")
- Result ordered by rating descending: 5 (9.1), then 1 (8.9)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Two filtering conditions: odd ID AND description != "boring"
- Simple WHERE clause with multiple conditions
- ORDER BY rating DESC for final sorting
- Straightforward filtering problem

### 2. **Key Insights**
- Use MOD function or % operator to check for odd numbers
- String comparison for filtering out "boring" description
- Combine conditions with AND operator
- Standard ORDER BY for sorting

### 3. **Interview Discussion Points**
- "This is a basic filtering problem with two conditions"
- "I need to check if ID is odd using modulo operation"
- "String filtering requires exact match for 'boring'"

## 🔧 Step-by-Step Solution Logic

### Step 1: Filter Odd IDs
```sql
-- Use MOD(id, 2) = 1 or id % 2 = 1 to check odd numbers
```

### Step 2: Filter Non-Boring Movies
```sql
-- Use description != 'boring' or description <> 'boring'
```

### Step 3: Combine and Sort
```sql
-- Combine conditions with AND
-- ORDER BY rating DESC for highest rating first
```

## ✅ Optimized SQL Solution

**Solution 1: Using MOD Function**
```sql
SELECT *
FROM Cinema
WHERE MOD(id, 2) = 1 
  AND description != 'boring'
ORDER BY rating DESC;
```

### Alternative Solutions

**Solution 2: Using % Operator**
```sql
SELECT *
FROM Cinema
WHERE id % 2 = 1 
  AND description <> 'boring'
ORDER BY rating DESC;
```

**Solution 3: Using NOT LIKE**
```sql
SELECT *
FROM Cinema
WHERE MOD(id, 2) = 1 
  AND description NOT LIKE 'boring'
ORDER BY rating DESC;
```

**Solution 4: Explicit Odd Check**
```sql
SELECT *
FROM Cinema
WHERE (id - 1) % 2 = 0  -- Alternative odd check
  AND description != 'boring'
ORDER BY rating DESC;
```

**Solution 5: With Case-Insensitive Check**
```sql
SELECT *
FROM Cinema
WHERE MOD(id, 2) = 1 
  AND LOWER(description) != 'boring'
ORDER BY rating DESC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Advanced Movie Analytics**
```sql
-- "Provide comprehensive movie analytics dashboard"

WITH movie_categories AS (
    SELECT 
        *,
        CASE 
            WHEN rating >= 9.0 THEN 'Excellent'
            WHEN rating >= 8.0 THEN 'Great'
            WHEN rating >= 7.0 THEN 'Good'
            WHEN rating >= 6.0 THEN 'Average'
            ELSE 'Poor'
        END as rating_category,
        CASE 
            WHEN MOD(id, 2) = 1 THEN 'Odd ID'
            ELSE 'Even ID'
        END as id_type,
        CASE 
            WHEN description = 'boring' THEN 'Boring'
            ELSE 'Interesting'
        END as interest_level
    FROM Cinema
),
analytics_summary AS (
    SELECT 
        COUNT(*) as total_movies,
        AVG(rating) as avg_rating,
        MAX(rating) as highest_rating,
        MIN(rating) as lowest_rating,
        COUNT(CASE WHEN MOD(id, 2) = 1 AND description != 'boring' THEN 1 END) as qualifying_movies,
        COUNT(CASE WHEN description = 'boring' THEN 1 END) as boring_movies,
        COUNT(CASE WHEN MOD(id, 2) = 1 THEN 1 END) as odd_id_movies
    FROM Cinema
)
SELECT 
    'Movie Analytics Dashboard' as report_type,
    total_movies,
    qualifying_movies,
    ROUND(qualifying_movies * 100.0 / total_movies, 2) as qualifying_percentage,
    ROUND(avg_rating, 2) as average_rating,
    highest_rating,
    lowest_rating,
    boring_movies,
    odd_id_movies
FROM analytics_summary;
```

#### 2. **Movie Recommendation System**
```sql
-- "Build a movie recommendation system based on criteria"

WITH movie_scoring AS (
    SELECT 
        *,
        CASE 
            WHEN rating >= 9.0 AND MOD(id, 2) = 1 AND description != 'boring' THEN 100
            WHEN rating >= 8.5 AND MOD(id, 2) = 1 AND description != 'boring' THEN 90
            WHEN rating >= 8.0 AND MOD(id, 2) = 1 AND description != 'boring' THEN 80
            WHEN rating >= 7.5 AND description != 'boring' THEN 60
            WHEN rating >= 7.0 THEN 40
            ELSE 20
        END as recommendation_score,
        CASE 
            WHEN MOD(id, 2) = 1 AND description != 'boring' THEN 'Highly Recommended'
            WHEN rating >= 8.0 AND description != 'boring' THEN 'Recommended'
            WHEN rating >= 7.0 THEN 'Worth Watching'
            WHEN description = 'boring' THEN 'Skip'
            ELSE 'Low Priority'
        END as recommendation_category
    FROM Cinema
),
top_recommendations AS (
    SELECT 
        *,
        ROW_NUMBER() OVER (ORDER BY recommendation_score DESC, rating DESC) as recommendation_rank
    FROM movie_scoring
)
SELECT 
    recommendation_rank,
    movie,
    description,
    rating,
    recommendation_score,
    recommendation_category,
    CASE 
        WHEN recommendation_rank <= 3 THEN 'Top Pick'
        WHEN recommendation_rank <= 5 THEN 'Great Choice'
        WHEN recommendation_rank <= 7 THEN 'Good Option'
        ELSE 'Consider if Time Allows'
    END as viewing_priority
FROM top_recommendations
ORDER BY recommendation_rank;
```

#### 3. **Content Quality Analysis**
```sql
-- "Analyze content quality patterns and trends"

WITH content_analysis AS (
    SELECT 
        description,
        COUNT(*) as movie_count,
        AVG(rating) as avg_rating,
        MAX(rating) as max_rating,
        MIN(rating) as min_rating,
        STDDEV(rating) as rating_variance
    FROM Cinema
    GROUP BY description
),
id_pattern_analysis AS (
    SELECT 
        CASE WHEN MOD(id, 2) = 1 THEN 'Odd' ELSE 'Even' END as id_pattern,
        COUNT(*) as count,
        AVG(rating) as avg_rating,
        COUNT(CASE WHEN description = 'boring' THEN 1 END) as boring_count
    FROM Cinema
    GROUP BY CASE WHEN MOD(id, 2) = 1 THEN 'Odd' ELSE 'Even' END
)
SELECT 
    'Content Quality Insights' as analysis_type,
    ca.description,
    ca.movie_count,
    ROUND(ca.avg_rating, 2) as avg_rating,
    ca.max_rating,
    ca.min_rating,
    ROUND(ca.rating_variance, 2) as rating_consistency,
    CASE 
        WHEN ca.avg_rating >= 8.5 THEN 'Premium Content'
        WHEN ca.avg_rating >= 7.5 THEN 'Quality Content'
        WHEN ca.avg_rating >= 6.5 THEN 'Standard Content'
        ELSE 'Below Average Content'
    END as content_tier
FROM content_analysis ca
ORDER BY ca.avg_rating DESC;
```

#### 4. **Performance and Scalability Optimization**
```sql
-- "Optimize for large movie databases"

-- Create indexes for efficient filtering
-- CREATE INDEX idx_cinema_id_rating ON Cinema(id, rating);
-- CREATE INDEX idx_cinema_description ON Cinema(description);

-- Optimized query with index hints
SELECT /*+ USE_INDEX(Cinema, idx_cinema_id_rating) */ *
FROM Cinema
WHERE MOD(id, 2) = 1 
  AND description != 'boring'
ORDER BY rating DESC;

-- For very large datasets, consider pre-filtering
WITH filtered_movies AS (
    SELECT *
    FROM Cinema
    WHERE description != 'boring'  -- Filter first for better performance
)
SELECT *
FROM filtered_movies
WHERE MOD(id, 2) = 1
ORDER BY rating DESC;

-- Query performance analysis
EXPLAIN SELECT *
FROM Cinema
WHERE MOD(id, 2) = 1 
  AND description != 'boring'
ORDER BY rating DESC;
```

#### 5. **Dynamic Filtering System**
```sql
-- "Create a flexible filtering system for different criteria"

-- Using parameters for dynamic filtering
WITH filter_criteria AS (
    SELECT 
        1 as check_odd_id,
        'boring' as excluded_description,
        7.0 as min_rating
),
dynamic_filter AS (
    SELECT 
        c.*,
        fc.check_odd_id,
        fc.excluded_description,
        fc.min_rating
    FROM Cinema c
    CROSS JOIN filter_criteria fc
    WHERE 
        (fc.check_odd_id = 0 OR MOD(c.id, 2) = 1)
        AND c.description != fc.excluded_description
        AND c.rating >= fc.min_rating
)
SELECT 
    id,
    movie,
    description,
    rating,
    'Meets All Criteria' as filter_status
FROM dynamic_filter
ORDER BY rating DESC;

-- Multi-criteria scoring system
WITH scoring_system AS (
    SELECT 
        *,
        (CASE WHEN MOD(id, 2) = 1 THEN 1 ELSE 0 END) * 25 as odd_id_points,
        (CASE WHEN description != 'boring' THEN 1 ELSE 0 END) * 25 as non_boring_points,
        LEAST(rating * 5, 50) as rating_points,  -- Cap at 50 points
        (CASE 
            WHEN rating >= 9.0 THEN 25
            WHEN rating >= 8.0 THEN 15
            WHEN rating >= 7.0 THEN 10
            ELSE 0
        END) as bonus_points
    FROM Cinema
)
SELECT 
    *,
    (odd_id_points + non_boring_points + rating_points + bonus_points) as total_score,
    CASE 
        WHEN (odd_id_points + non_boring_points + rating_points + bonus_points) >= 90 THEN 'Must Watch'
        WHEN (odd_id_points + non_boring_points + rating_points + bonus_points) >= 70 THEN 'Highly Recommended'
        WHEN (odd_id_points + non_boring_points + rating_points + bonus_points) >= 50 THEN 'Worth Watching'
        ELSE 'Skip'
    END as recommendation
FROM scoring_system
ORDER BY total_score DESC, rating DESC;
```

#### 6. **Business Intelligence Dashboard**
```sql
-- "Create comprehensive movie theater analytics"

WITH movie_metrics AS (
    SELECT 
        COUNT(*) as total_movies,
        COUNT(CASE WHEN MOD(id, 2) = 1 AND description != 'boring' THEN 1 END) as featured_movies,
        AVG(rating) as overall_avg_rating,
        AVG(CASE WHEN MOD(id, 2) = 1 AND description != 'boring' THEN rating END) as featured_avg_rating,
        MAX(rating) as highest_rating,
        MIN(rating) as lowest_rating
    FROM Cinema
),
category_breakdown AS (
    SELECT 
        description,
        COUNT(*) as count,
        AVG(rating) as avg_rating,
        COUNT(CASE WHEN MOD(id, 2) = 1 THEN 1 END) as odd_id_count
    FROM Cinema
    GROUP BY description
),
rating_distribution AS (
    SELECT 
        CASE 
            WHEN rating >= 9.0 THEN '9.0-10.0'
            WHEN rating >= 8.0 THEN '8.0-8.9'
            WHEN rating >= 7.0 THEN '7.0-7.9'
            WHEN rating >= 6.0 THEN '6.0-6.9'
            ELSE 'Below 6.0'
        END as rating_range,
        COUNT(*) as movie_count,
        COUNT(CASE WHEN MOD(id, 2) = 1 AND description != 'boring' THEN 1 END) as featured_in_range
    FROM Cinema
    GROUP BY 
        CASE 
            WHEN rating >= 9.0 THEN '9.0-10.0'
            WHEN rating >= 8.0 THEN '8.0-8.9'
            WHEN rating >= 7.0 THEN '7.0-7.9'
            WHEN rating >= 6.0 THEN '6.0-6.9'
            ELSE 'Below 6.0'
        END
)
SELECT 
    'Cinema Analytics Dashboard' as dashboard_section,
    CONCAT('Total Movies: ', mm.total_movies) as inventory_summary,
    CONCAT('Featured Movies: ', mm.featured_movies, ' (', 
           ROUND(mm.featured_movies * 100.0 / mm.total_movies, 1), '%)') as featured_analysis,
    CONCAT('Overall Rating: ', ROUND(mm.overall_avg_rating, 2)) as quality_metrics,
    CONCAT('Featured Rating: ', ROUND(mm.featured_avg_rating, 2)) as featured_quality,
    CONCAT('Rating Range: ', mm.lowest_rating, ' - ', mm.highest_rating) as rating_span
FROM movie_metrics mm;
```

## 🔗 Related LeetCode Questions

1. **#595 - Big Countries** (Multiple filtering conditions with OR)
2. **#584 - Find Customer Referee** (String filtering with NULL handling)
3. **#596 - Classes More Than 5 Students** (GROUP BY with HAVING)
4. **#1757 - Recyclable and Low Fat Products** (Multiple AND conditions)
5. **#1873 - Calculate Special Bonus** (Conditional logic with MOD)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **MOD/% Operator**: Checking for odd/even numbers
2. **String Filtering**: Exact string matching for exclusions
3. **Multiple Conditions**: Combining filters with AND/OR
4. **ORDER BY**: Sorting results by numerical columns

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should the comparison be case-sensitive?"
2. **Discuss performance**: "Indexing on filtered columns improves performance"
3. **Consider edge cases**: "What about NULL descriptions or invalid ratings?"
4. **Think business context**: "This helps curate quality content for users"

### 🔧 **Common Patterns**
- MOD(column, 2) = 1 for odd number checking
- String comparison with != or <> operators
- Multiple WHERE conditions with AND
- ORDER BY with DESC for highest-first sorting

### ⚠️ **Common Mistakes to Avoid**
1. **Using wrong modulo logic** (MOD(id, 2) = 0 for odd instead of = 1)
2. **Case sensitivity issues** (not considering UPPER/LOWER case variations)
3. **Missing ORDER BY** when specific sorting is required
4. **Performance issues** (not indexing frequently filtered columns)

### 🔍 **Performance Considerations**
- Index on id column for efficient modulo operations
- Index on description for string filtering
- Composite index on (id, description, rating) for optimal performance
- Consider query execution order for large datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Filtering quality content for better user experience
- **Operational Excellence**: Efficient content curation algorithms
- **Frugality**: Optimizing query performance for cost efficiency
- **Innovation**: Creative content recommendation systems

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find movies with even IDs and rating above 8.0**
2. **Count movies by description and rating category**
3. **Find the highest-rated movie in each description category**
4. **Calculate average rating excluding "boring" movies**

Remember: Content filtering and recommendation systems are core to Amazon Prime Video, content delivery optimization, and personalized user experiences!