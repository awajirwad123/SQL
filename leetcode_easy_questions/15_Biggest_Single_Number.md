# LeetCode Easy #619: Biggest Single Number

## 📋 Problem Statement

A **single number** is a number that appeared only once in the `MyNumbers` table.

Write a SQL query to report the largest **single number**. If there is no **single number**, return `null`.

## 🗄️ Table Schema

**MyNumbers Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| num         | int  |
+-------------+------+
```
- There is no primary key for this table. It may contain duplicates.
- Each row of this table contains an integer.

## 📊 Sample Data

**Example 1:**

**MyNumbers Table:**
| num |
|-----|
| 8   |
| 8   |
| 3   |
| 3   |
| 1   |
| 4   |
| 5   |
| 6   |

**Expected Output:**
| num |
|-----|
| 6   |

**Explanation:** The single numbers are 1, 4, 5, and 6. Since 6 is the largest single number, we return 6.

**Example 2:**

**MyNumbers Table:**
| num |
|-----|
| 8   |
| 8   |
| 7   |
| 7   |
| 3   |
| 3   |
| 3   |

**Expected Output:**
| num |
|-----|
| null|

**Explanation:** There are no single numbers in the input table so we return null.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find numbers that appear exactly once (single numbers)
- Among single numbers, find the maximum
- Return NULL if no single numbers exist
- Need GROUP BY with HAVING to filter single occurrences

### 2. **Key Insights**
- GROUP BY num with COUNT(*) = 1 identifies single numbers
- Use MAX() to find the largest among single numbers
- Handle the case where no single numbers exist (returns NULL automatically)

### 3. **Interview Discussion Points**
- "First I need to identify which numbers appear exactly once"
- "Then find the maximum among those single numbers"
- "SQL MAX() naturally returns NULL if no rows qualify"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Single Numbers
```sql
-- GROUP BY num and use HAVING COUNT(*) = 1
-- This gives us only numbers that appear exactly once
```

### Step 2: Find Maximum Single Number
```sql
-- Apply MAX() to the single numbers
-- MAX() returns NULL if no rows exist
```

### Step 3: Handle Edge Cases
```sql
-- No additional NULL handling needed
-- MAX() inherently returns NULL for empty result sets
```

## ✅ Optimized SQL Solution

**Solution 1: Subquery Approach**
```sql
SELECT MAX(num) as num
FROM (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
) as single_numbers;
```

### Alternative Solutions

**Solution 2: Direct MAX with GROUP BY**
```sql
SELECT 
    CASE 
        WHEN COUNT(*) > 0 THEN MAX(num)
        ELSE NULL
    END as num
FROM (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
) as single_numbers;
```

**Solution 3: Using CTE**
```sql
WITH single_numbers AS (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
)
SELECT MAX(num) as num
FROM single_numbers;
```

**Solution 4: Window Function Approach**
```sql
WITH number_counts AS (
    SELECT 
        num,
        COUNT(*) OVER (PARTITION BY num) as cnt
    FROM MyNumbers
),
single_numbers AS (
    SELECT DISTINCT num
    FROM number_counts
    WHERE cnt = 1
)
SELECT MAX(num) as num
FROM single_numbers;
```

**Solution 5: Self-Join Approach (Less Efficient)**
```sql
SELECT MAX(m1.num) as num
FROM MyNumbers m1
WHERE m1.num NOT IN (
    SELECT m2.num
    FROM MyNumbers m2
    GROUP BY m2.num
    HAVING COUNT(*) > 1
);
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Number Analysis**
```sql
-- "Provide complete analysis of number frequencies"

WITH number_analysis AS (
    SELECT 
        num,
        COUNT(*) as frequency,
        CASE 
            WHEN COUNT(*) = 1 THEN 'Single'
            WHEN COUNT(*) = 2 THEN 'Double'
            WHEN COUNT(*) <= 5 THEN 'Multiple'
            ELSE 'Frequent'
        END as category
    FROM MyNumbers
    GROUP BY num
),
summary_stats AS (
    SELECT 
        COUNT(*) as unique_numbers,
        SUM(CASE WHEN frequency = 1 THEN 1 ELSE 0 END) as single_count,
        MAX(CASE WHEN frequency = 1 THEN num ELSE NULL END) as biggest_single,
        MIN(CASE WHEN frequency = 1 THEN num ELSE NULL END) as smallest_single,
        AVG(CASE WHEN frequency = 1 THEN num ELSE NULL END) as avg_single,
        MAX(frequency) as max_frequency,
        AVG(frequency) as avg_frequency
    FROM number_analysis
)
SELECT 
    'Number Analysis Summary' as analysis_type,
    unique_numbers,
    single_count,
    biggest_single,
    smallest_single,
    ROUND(avg_single, 2) as avg_single,
    max_frequency,
    ROUND(avg_frequency, 2) as avg_frequency,
    ROUND(single_count * 100.0 / unique_numbers, 2) as single_percentage
FROM summary_stats;
```

#### 2. **Top K Single Numbers**
```sql
-- "Find the top 3 largest single numbers"

WITH single_numbers AS (
    SELECT num
    FROM MyNumbers
    GROUP BY num
    HAVING COUNT(*) = 1
),
ranked_singles AS (
    SELECT 
        num,
        ROW_NUMBER() OVER (ORDER BY num DESC) as rank_desc
    FROM single_numbers
)
SELECT 
    rank_desc as rank_position,
    num
FROM ranked_singles
WHERE rank_desc <= 3
ORDER BY rank_desc;
```

#### 3. **Statistical Distribution Analysis**
```sql
-- "Analyze the distribution of number frequencies"

WITH frequency_distribution AS (
    SELECT 
        COUNT(*) as frequency,
        COUNT(DISTINCT num) as numbers_with_this_frequency
    FROM MyNumbers
    GROUP BY num
),
distribution_summary AS (
    SELECT 
        frequency,
        numbers_with_this_frequency,
        numbers_with_this_frequency * 100.0 / SUM(numbers_with_this_frequency) OVER() as percentage
    FROM frequency_distribution
)
SELECT 
    frequency,
    numbers_with_this_frequency,
    ROUND(percentage, 2) as percentage,
    CASE 
        WHEN frequency = 1 THEN 'Single Occurrence'
        WHEN frequency = 2 THEN 'Double Occurrence'
        WHEN frequency <= 5 THEN 'Multiple Occurrence'
        ELSE 'High Frequency'
    END as category
FROM distribution_summary
ORDER BY frequency;
```

#### 4. **Data Quality and Outlier Detection**
```sql
-- "Detect outliers and analyze data quality"

WITH number_stats AS (
    SELECT 
        num,
        COUNT(*) as frequency
    FROM MyNumbers
    GROUP BY num
),
basic_stats AS (
    SELECT 
        COUNT(*) as total_unique_numbers,
        MIN(num) as min_number,
        MAX(num) as max_number,
        AVG(num) as avg_number,
        STDDEV(num) as stddev_number
    FROM number_stats
),
outlier_analysis AS (
    SELECT 
        ns.num,
        ns.frequency,
        ABS(ns.num - bs.avg_number) / bs.stddev_number as z_score,
        CASE 
            WHEN ABS(ns.num - bs.avg_number) / bs.stddev_number > 2 THEN 'Outlier'
            WHEN ABS(ns.num - bs.avg_number) / bs.stddev_number > 1.5 THEN 'Potential Outlier'
            ELSE 'Normal'
        END as outlier_status
    FROM number_stats ns
    CROSS JOIN basic_stats bs
    WHERE ns.frequency = 1  -- Only single numbers
)
SELECT 
    num,
    frequency,
    ROUND(z_score, 2) as z_score,
    outlier_status,
    CASE 
        WHEN num = (SELECT MAX(num) FROM outlier_analysis WHERE frequency = 1) THEN 'Biggest Single'
        WHEN outlier_status = 'Outlier' THEN 'Statistical Outlier'
        ELSE 'Regular Single'
    END as classification
FROM outlier_analysis
ORDER BY num DESC;
```

#### 5. **Time-Series Analysis (if timestamps available)**
```sql
-- "Analyze single numbers over time periods"

-- Assuming we have a timestamp column
WITH time_periods AS (
    SELECT 
        DATE_FORMAT(created_at, '%Y-%m') as month_year,
        num
    FROM MyNumbers_with_timestamps
),
monthly_singles AS (
    SELECT 
        month_year,
        num,
        COUNT(*) as monthly_frequency
    FROM time_periods
    GROUP BY month_year, num
    HAVING COUNT(*) = 1
),
monthly_biggest AS (
    SELECT 
        month_year,
        MAX(num) as biggest_single_of_month,
        COUNT(*) as single_numbers_count
    FROM monthly_singles
    GROUP BY month_year
)
SELECT 
    month_year,
    biggest_single_of_month,
    single_numbers_count,
    LAG(biggest_single_of_month) OVER (ORDER BY month_year) as previous_month_biggest,
    biggest_single_of_month - LAG(biggest_single_of_month) OVER (ORDER BY month_year) as month_over_month_change
FROM monthly_biggest
ORDER BY month_year DESC;
```

#### 6. **Performance Optimization and Scalability**
```sql
-- "Optimize for large datasets"

-- Create index for better GROUP BY performance
-- CREATE INDEX idx_mynumbers_num ON MyNumbers(num);

-- Optimized query with early filtering
WITH RECURSIVE number_frequency AS (
    SELECT 
        num,
        COUNT(*) as freq
    FROM MyNumbers
    GROUP BY num
),
single_only AS (
    SELECT num
    FROM number_frequency
    WHERE freq = 1
)
SELECT MAX(num) as num
FROM single_only;

-- Alternative using window functions for very large datasets
WITH number_counts AS (
    SELECT 
        num,
        COUNT(*) OVER (PARTITION BY num) as cnt,
        ROW_NUMBER() OVER (PARTITION BY num ORDER BY num) as rn
    FROM MyNumbers
)
SELECT MAX(num) as num
FROM number_counts
WHERE cnt = 1 AND rn = 1;  -- Only take one row per number to avoid duplicates
```

#### 7. **Business Intelligence Dashboard**
```sql
-- "Create a comprehensive dashboard query"

WITH comprehensive_analysis AS (
    SELECT 
        num,
        COUNT(*) as frequency
    FROM MyNumbers
    GROUP BY num
),
dashboard_metrics AS (
    SELECT 
        COUNT(*) as total_unique_numbers,
        SUM(frequency) as total_records,
        COUNT(CASE WHEN frequency = 1 THEN 1 END) as single_numbers,
        COUNT(CASE WHEN frequency > 1 THEN 1 END) as duplicate_numbers,
        MAX(CASE WHEN frequency = 1 THEN num END) as biggest_single,
        MIN(CASE WHEN frequency = 1 THEN num END) as smallest_single,
        MAX(frequency) as highest_frequency,
        MIN(num) as overall_min,
        MAX(num) as overall_max,
        AVG(num) as overall_average
    FROM comprehensive_analysis
)
SELECT 
    'Dataset Overview' as metric_category,
    CONCAT(total_records, ' total records') as total_data,
    CONCAT(total_unique_numbers, ' unique numbers') as unique_data,
    CONCAT(single_numbers, ' single numbers (', 
           ROUND(single_numbers * 100.0 / total_unique_numbers, 1), '%)') as single_analysis,
    CONCAT('Range: ', overall_min, ' to ', overall_max) as data_range,
    CONCAT('Biggest Single: ', COALESCE(CAST(biggest_single AS CHAR), 'None')) as result,
    CONCAT('Data Quality: ', 
           CASE 
               WHEN single_numbers > duplicate_numbers THEN 'Good (more singles)'
               WHEN single_numbers = duplicate_numbers THEN 'Balanced'
               ELSE 'High Duplication'
           END) as quality_assessment
FROM dashboard_metrics;
```

## 🔗 Related LeetCode Questions

1. **#177 - Nth Highest Salary** (Finding Nth largest with NULL handling)
2. **#178 - Rank Scores** (Ranking and finding top values)
3. **#184 - Department Highest Salary** (GROUP BY with MAX)
4. **#626 - Exchange Seats** (Conditional logic with CASE)
5. **#1112 - Highest Grade For Each Student** (MAX with GROUP BY)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY + HAVING**: Filtering aggregated results
2. **MAX with NULL handling**: Automatic NULL return for empty sets
3. **Subquery patterns**: Breaking complex logic into steps
4. **Frequency analysis**: Counting occurrences for filtering

### 🚀 **Amazon Interview Tips**
1. **Explain the approach**: "First filter to single numbers, then find maximum"
2. **Discuss edge cases**: "MAX() naturally handles the empty result case"
3. **Consider performance**: "GROUP BY needs proper indexing for large datasets"
4. **Think business context**: "Finding unique values is common in data deduplication"

### 🔧 **Common Patterns**
- GROUP BY + HAVING for frequency filtering
- Subqueries for complex filtering logic
- MAX() with automatic NULL handling
- CTE for readable multi-step logic

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting HAVING clause** (using WHERE instead of HAVING for aggregates)
2. **Overcomplicating NULL handling** (MAX() handles it automatically)
3. **Performance issues** (not indexing the grouped column)
4. **Incorrect frequency logic** (using != 1 instead of = 1)

### 🔍 **Performance Considerations**
- Index on num column for efficient GROUP BY
- Consider materialized views for frequently accessed statistics
- Window functions may be more efficient for very large datasets
- Monitor execution plans for optimization opportunities

### 🎯 **Amazon Leadership Principles Applied**
- **Data-Driven Decisions**: Using statistical analysis for insights
- **Customer Obsession**: Finding unique patterns in customer behavior
- **Operational Excellence**: Efficient data processing and analysis
- **Innovation**: Creative approaches to data uniqueness problems

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find the second largest single number**
2. **Find all numbers that appear exactly twice**
3. **Calculate the sum of all single numbers**
4. **Find the median of single numbers**

Remember: Identifying unique patterns and outliers is fundamental to Amazon's recommendation systems, fraud detection, inventory management, and customer behavior analysis!