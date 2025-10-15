# LeetCode Easy #1141: User Activity for the Past 30 Days I

## 📋 Problem Statement

Write a SQL query to find the **daily active user count** for a period of **30 days ending 2019-07-27** inclusive.

A user was active on someday if they made at least one activity on that day.

Return the result table in **any order**.

## 🗄️ Table Schema

**Activity Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| session_id    | int     |
| activity_date | date    |
| activity_type | enum    |
+---------------+---------+
```
- There is no primary key for this table, it may have duplicate rows.
- The `activity_type` column is an ENUM of type ('open_session', 'end_session', 'scroll_down', 'send_message').
- The table shows the user activities for a social media website.
- Each row is a record of one user activity in one session on one day.

## 📊 Sample Data

**Activity Table:**
| user_id | session_id | activity_date | activity_type |
|---------|------------|---------------|---------------|
| 1       | 1          | 2019-07-20    | open_session  |
| 1       | 1          | 2019-07-20    | scroll_down   |
| 1       | 1          | 2019-07-20    | end_session   |
| 2       | 4          | 2019-07-20    | open_session  |
| 2       | 4          | 2019-07-21    | send_message  |
| 2       | 4          | 2019-07-21    | end_session   |
| 3       | 2          | 2019-07-21    | open_session  |
| 3       | 2          | 2019-07-21    | send_message  |
| 3       | 2          | 2019-07-21    | end_session   |
| 4       | 3          | 2019-06-25    | open_session  |
| 4       | 3          | 2019-06-25    | end_session   |

**Expected Output:**
| day        | active_users |
|------------|--------------|
| 2019-07-20 | 2            |
| 2019-07-21 | 2            |

**Explanation:** 
- 2019-07-20: Users 1 and 2 were active
- 2019-07-21: Users 2 and 3 were active
- 2019-06-25 is outside the 30-day window (before 2019-06-28)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Calculate 30-day window: 2019-07-27 going back 30 days to 2019-06-28
- Count unique users per day within this window
- GROUP BY date and COUNT DISTINCT users
- Filter dates within the specified range

### 2. **Key Insights**
- Date range: 2019-06-28 to 2019-07-27 (inclusive)
- Use DISTINCT to count unique users per day
- GROUP BY activity_date for daily aggregation
- Date arithmetic: DATEADD or direct date comparison

### 3. **Interview Discussion Points**
- "This is a time-based aggregation with user deduplication"
- "I need to calculate the 30-day window ending on 2019-07-27"
- "COUNT DISTINCT ensures users are counted once per day"

## 🔧 Step-by-Step Solution Logic

### Step 1: Calculate Date Range
```sql
-- 30 days ending 2019-07-27 means: 2019-06-28 to 2019-07-27
-- Inclusive range with proper date filtering
```

### Step 2: Filter Activities in Range
```sql
-- WHERE activity_date BETWEEN start_date AND end_date
-- Only include activities within the 30-day window
```

### Step 3: Count Unique Users per Day
```sql
-- GROUP BY activity_date
-- COUNT DISTINCT user_id for each date
```

## ✅ Optimized SQL Solution

**Solution 1: Direct Date Range**
```sql
SELECT 
    activity_date as day,
    COUNT(DISTINCT user_id) as active_users
FROM Activity
WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
GROUP BY activity_date;
```

### Alternative Solutions

**Solution 2: Using DATEDIFF**
```sql
SELECT 
    activity_date as day,
    COUNT(DISTINCT user_id) as active_users
FROM Activity
WHERE DATEDIFF('2019-07-27', activity_date) BETWEEN 0 AND 29
GROUP BY activity_date;
```

**Solution 3: Using DATE_SUB**
```sql
SELECT 
    activity_date as day,
    COUNT(DISTINCT user_id) as active_users
FROM Activity
WHERE activity_date BETWEEN DATE_SUB('2019-07-27', INTERVAL 29 DAY) AND '2019-07-27'
GROUP BY activity_date;
```

**Solution 4: With Additional Metrics**
```sql
SELECT 
    activity_date as day,
    COUNT(DISTINCT user_id) as active_users,
    COUNT(*) as total_activities,
    COUNT(DISTINCT session_id) as unique_sessions,
    ROUND(COUNT(*) / COUNT(DISTINCT user_id), 2) as avg_activities_per_user
FROM Activity
WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
GROUP BY activity_date
ORDER BY activity_date;
```

**Solution 5: Using Window Functions for Trends**
```sql
WITH daily_stats AS (
    SELECT 
        activity_date as day,
        COUNT(DISTINCT user_id) as active_users
    FROM Activity
    WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
    GROUP BY activity_date
)
SELECT 
    day,
    active_users,
    LAG(active_users) OVER (ORDER BY day) as prev_day_users,
    active_users - LAG(active_users) OVER (ORDER BY day) as day_over_day_change,
    AVG(active_users) OVER (ORDER BY day ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as weekly_avg
FROM daily_stats
ORDER BY day;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive User Engagement Analysis**
```sql
-- "Provide detailed user engagement metrics for the 30-day period"

WITH date_range AS (
    SELECT DATE_SUB('2019-07-27', INTERVAL 29 DAY) as start_date, 
           '2019-07-27' as end_date
),
daily_activity_metrics AS (
    SELECT 
        activity_date,
        COUNT(DISTINCT user_id) as daily_active_users,
        COUNT(DISTINCT session_id) as unique_sessions,
        COUNT(*) as total_activities,
        COUNT(DISTINCT CASE WHEN activity_type = 'open_session' THEN user_id END) as users_opening_sessions,
        COUNT(DISTINCT CASE WHEN activity_type = 'send_message' THEN user_id END) as users_sending_messages,
        COUNT(DISTINCT CASE WHEN activity_type = 'scroll_down' THEN user_id END) as users_scrolling,
        COUNT(CASE WHEN activity_type = 'send_message' THEN 1 END) as total_messages_sent,
        COUNT(CASE WHEN activity_type = 'scroll_down' THEN 1 END) as total_scrolls
    FROM Activity
    CROSS JOIN date_range dr
    WHERE activity_date BETWEEN dr.start_date AND dr.end_date
    GROUP BY activity_date
),
engagement_analysis AS (
    SELECT 
        *,
        ROUND(total_activities / daily_active_users, 2) as avg_activities_per_user,
        ROUND(unique_sessions / daily_active_users, 2) as avg_sessions_per_user,
        ROUND(users_sending_messages * 100.0 / daily_active_users, 2) as message_engagement_rate,
        ROUND(users_scrolling * 100.0 / daily_active_users, 2) as scroll_engagement_rate,
        CASE 
            WHEN DAYOFWEEK(activity_date) IN (1, 7) THEN 'Weekend'
            ELSE 'Weekday'
        END as day_type
    FROM daily_activity_metrics
)
SELECT 
    activity_date as day,
    daily_active_users as active_users,
    unique_sessions,
    total_activities,
    avg_activities_per_user,
    avg_sessions_per_user,
    message_engagement_rate,
    scroll_engagement_rate,
    day_type,
    CASE 
        WHEN daily_active_users >= 3 THEN 'High Activity'
        WHEN daily_active_users >= 2 THEN 'Medium Activity'
        ELSE 'Low Activity'
    END as activity_level
FROM engagement_analysis
ORDER BY activity_date;
```

#### 2. **User Retention and Cohort Analysis**
```sql
-- "Analyze user retention patterns within the 30-day period"

WITH date_range AS (
    SELECT DATE_SUB('2019-07-27', INTERVAL 29 DAY) as start_date, 
           '2019-07-27' as end_date
),
user_activity_summary AS (
    SELECT 
        user_id,
        MIN(activity_date) as first_activity_date,
        MAX(activity_date) as last_activity_date,
        COUNT(DISTINCT activity_date) as active_days,
        COUNT(*) as total_activities,
        GROUP_CONCAT(DISTINCT activity_type) as activity_types_used
    FROM Activity
    CROSS JOIN date_range dr
    WHERE activity_date BETWEEN dr.start_date AND dr.end_date
    GROUP BY user_id
),
user_segments AS (
    SELECT 
        *,
        DATEDIFF(last_activity_date, first_activity_date) + 1 as user_span_days,
        CASE 
            WHEN active_days = 1 THEN 'Single Day User'
            WHEN active_days <= 3 THEN 'Casual User'
            WHEN active_days <= 7 THEN 'Regular User'
            WHEN active_days <= 15 THEN 'Frequent User'
            ELSE 'Power User'
        END as user_segment,
        CASE 
            WHEN first_activity_date = (SELECT start_date FROM date_range) THEN 'Period Starter'
            WHEN last_activity_date = (SELECT end_date FROM date_range) THEN 'Recent Active'
            ELSE 'Mid-Period User'
        END as engagement_timing
    FROM user_activity_summary
),
retention_analysis AS (
    SELECT 
        user_segment,
        COUNT(*) as user_count,
        AVG(active_days) as avg_active_days,
        AVG(total_activities) as avg_total_activities,
        AVG(user_span_days) as avg_span_days,
        ROUND(AVG(active_days) / AVG(user_span_days), 2) as retention_rate
    FROM user_segments
    GROUP BY user_segment
)
SELECT 
    user_segment,
    user_count,
    ROUND(avg_active_days, 1) as avg_days_active,
    ROUND(avg_total_activities, 1) as avg_activities,
    ROUND(avg_span_days, 1) as avg_period_span,
    retention_rate,
    ROUND(user_count * 100.0 / SUM(user_count) OVER(), 2) as segment_percentage
FROM retention_analysis
ORDER BY user_count DESC;
```

#### 3. **Time-Based Activity Patterns**
```sql
-- "Identify activity patterns and trends over the 30-day period"

WITH date_range AS (
    SELECT DATE_SUB('2019-07-27', INTERVAL 29 DAY) as start_date, 
           '2019-07-27' as end_date
),
daily_trends AS (
    SELECT 
        activity_date,
        COUNT(DISTINCT user_id) as daily_active_users,
        COUNT(*) as daily_activities,
        DAYOFWEEK(activity_date) as day_of_week,
        CASE 
            WHEN DAYOFWEEK(activity_date) IN (1, 7) THEN 'Weekend'
            ELSE 'Weekday'
        END as day_type,
        DATEDIFF(activity_date, (SELECT start_date FROM date_range)) + 1 as day_number
    FROM Activity
    CROSS JOIN date_range dr
    WHERE activity_date BETWEEN dr.start_date AND dr.end_date
    GROUP BY activity_date
),
weekly_analysis AS (
    SELECT 
        FLOOR((day_number - 1) / 7) + 1 as week_number,
        day_type,
        AVG(daily_active_users) as avg_daily_users,
        AVG(daily_activities) as avg_daily_activities,
        MIN(daily_active_users) as min_daily_users,
        MAX(daily_active_users) as max_daily_users
    FROM daily_trends
    GROUP BY FLOOR((day_number - 1) / 7) + 1, day_type
),
trend_analysis AS (
    SELECT 
        activity_date,
        daily_active_users,
        day_type,
        week_number,
        LAG(daily_active_users) OVER (ORDER BY activity_date) as prev_day_users,
        AVG(daily_active_users) OVER (ORDER BY activity_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) as weekly_moving_avg,
        ROW_NUMBER() OVER (ORDER BY daily_active_users DESC) as activity_rank
    FROM daily_trends dt
    JOIN weekly_analysis wa ON FLOOR((dt.day_number - 1) / 7) + 1 = wa.week_number AND dt.day_type = wa.day_type
)
SELECT 
    activity_date as day,
    daily_active_users as active_users,
    day_type,
    ROUND(weekly_moving_avg, 1) as seven_day_avg,
    daily_active_users - prev_day_users as day_over_day_change,
    CASE 
        WHEN activity_rank <= 3 THEN 'Peak Day'
        WHEN activity_rank >= 28 THEN 'Low Activity Day'
        ELSE 'Normal Day'
    END as activity_classification,
    CASE 
        WHEN daily_active_users > weekly_moving_avg * 1.2 THEN 'Above Trend'
        WHEN daily_active_users < weekly_moving_avg * 0.8 THEN 'Below Trend'
        ELSE 'On Trend'
    END as trend_status
FROM trend_analysis
ORDER BY activity_date;
```

#### 4. **Performance Optimization for Large Datasets**
```sql
-- "Optimize for large-scale activity data"

-- Create indexes for efficient date-based queries
-- CREATE INDEX idx_activity_date_user ON Activity(activity_date, user_id);
-- CREATE INDEX idx_activity_user_date ON Activity(user_id, activity_date);

-- Partitioned approach for better performance
WITH date_boundaries AS (
    SELECT 
        '2019-06-28' as start_date,
        '2019-07-27' as end_date
)
SELECT 
    activity_date as day,
    COUNT(DISTINCT user_id) as active_users
FROM Activity
WHERE activity_date >= '2019-06-28' 
  AND activity_date <= '2019-07-27'
GROUP BY activity_date
ORDER BY activity_date;

-- Optimized version using pre-aggregated daily summaries
-- CREATE TABLE daily_user_activity_summary AS
-- SELECT 
--     activity_date,
--     COUNT(DISTINCT user_id) as daily_active_users,
--     COUNT(*) as daily_activities,
--     COUNT(DISTINCT session_id) as daily_sessions
-- FROM Activity
-- GROUP BY activity_date;

-- Query using pre-aggregated table:
-- SELECT 
--     activity_date as day,
--     daily_active_users as active_users
-- FROM daily_user_activity_summary
-- WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
-- ORDER BY activity_date;

-- For real-time scenarios, consider rolling window approach
WITH rolling_window AS (
    SELECT 
        activity_date,
        user_id,
        ROW_NUMBER() OVER (PARTITION BY activity_date, user_id ORDER BY session_id) as daily_rank
    FROM Activity
    WHERE activity_date >= CURRENT_DATE - INTERVAL 30 DAY
)
SELECT 
    activity_date as day,
    COUNT(*) as active_users
FROM rolling_window
WHERE daily_rank = 1  -- Count each user once per day
GROUP BY activity_date
ORDER BY activity_date DESC
LIMIT 30;
```

#### 5. **Business Intelligence Dashboard**
```sql
-- "Create comprehensive activity dashboard metrics"

WITH period_summary AS (
    SELECT 
        COUNT(DISTINCT user_id) as total_unique_users,
        COUNT(DISTINCT activity_date) as days_with_activity,
        COUNT(*) as total_activities,
        COUNT(DISTINCT session_id) as total_sessions,
        MIN(activity_date) as period_start,
        MAX(activity_date) as period_end
    FROM Activity
    WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
),
daily_metrics AS (
    SELECT 
        activity_date,
        COUNT(DISTINCT user_id) as daily_active_users,
        COUNT(*) as daily_activities,
        COUNT(DISTINCT session_id) as daily_sessions
    FROM Activity
    WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
    GROUP BY activity_date
),
user_behavior AS (
    SELECT 
        user_id,
        COUNT(DISTINCT activity_date) as active_days,
        COUNT(*) as total_user_activities
    FROM Activity
    WHERE activity_date BETWEEN '2019-06-28' AND '2019-07-27'
    GROUP BY user_id
),
engagement_metrics AS (
    SELECT 
        AVG(daily_active_users) as avg_daily_active_users,
        MAX(daily_active_users) as peak_daily_users,
        MIN(daily_active_users) as min_daily_users,
        STDDEV(daily_active_users) as dau_volatility,
        AVG(daily_activities) as avg_daily_activities,
        AVG(daily_sessions) as avg_daily_sessions
    FROM daily_metrics
),
user_engagement_distribution AS (
    SELECT 
        CASE 
            WHEN active_days = 1 THEN '1 Day'
            WHEN active_days <= 3 THEN '2-3 Days'
            WHEN active_days <= 7 THEN '4-7 Days'
            WHEN active_days <= 15 THEN '8-15 Days'
            ELSE '16+ Days'
        END as engagement_bucket,
        COUNT(*) as user_count,
        AVG(total_user_activities) as avg_activities_per_user
    FROM user_behavior
    GROUP BY 
        CASE 
            WHEN active_days = 1 THEN '1 Day'
            WHEN active_days <= 3 THEN '2-3 Days'
            WHEN active_days <= 7 THEN '4-7 Days'
            WHEN active_days <= 15 THEN '8-15 Days'
            ELSE '16+ Days'
        END
)
SELECT 
    'User Activity Dashboard - 30 Days Ending 2019-07-27' as report_title,
    ps.total_unique_users,
    ps.days_with_activity,
    ROUND(em.avg_daily_active_users, 1) as avg_daily_active_users,
    em.peak_daily_users,
    em.min_daily_users,
    ROUND(em.dau_volatility, 2) as daily_user_volatility,
    ROUND(ps.total_activities / ps.total_unique_users, 1) as avg_activities_per_user,
    ROUND(em.avg_daily_sessions / em.avg_daily_active_users, 2) as avg_sessions_per_active_user,
    (SELECT engagement_bucket FROM user_engagement_distribution ORDER BY user_count DESC LIMIT 1) as most_common_engagement_pattern,
    ROUND(ps.total_unique_users / ps.days_with_activity, 1) as avg_unique_users_per_active_day
FROM period_summary ps
CROSS JOIN engagement_metrics em;
```

## 🔗 Related LeetCode Questions

1. **#1084 - Sales Analysis III** (Date range filtering with conditions)
2. **#1075 - Project Employees I** (GROUP BY with aggregation)
3. **#586 - Customer Placing the Largest Number of Orders** (COUNT with GROUP BY)
4. **#1393 - Capital Gain/Loss** (Time-based analysis)
5. **#1211 - Queries Quality and Percentage** (Percentage calculations with GROUP BY)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Date Range Calculations**: Computing 30-day windows with proper boundaries
2. **DISTINCT Counting**: Avoiding duplicate user counts per day
3. **Time-Based Aggregation**: GROUP BY date for daily metrics
4. **Window Functions**: Analyzing trends and moving averages

### 🚀 **Amazon Interview Tips**
1. **Clarify date boundaries**: "Is the 30-day period inclusive or exclusive?"
2. **Discuss performance**: "Date range indexes are crucial for large activity tables"
3. **Consider time zones**: "Should I account for different time zones?"
4. **Think scalability**: "How would this work with millions of daily activities?"

### 🔧 **Common Patterns**
- Date range filtering with BETWEEN or comparison operators
- COUNT DISTINCT for unique user counting
- GROUP BY date for daily aggregation
- Window functions for trend analysis

### ⚠️ **Common Mistakes to Avoid**
1. **Incorrect date arithmetic** (off-by-one errors in 30-day calculation)
2. **Missing DISTINCT** (counting users multiple times per day)
3. **Performance issues** (not indexing date columns)
4. **Time zone confusion** (assuming all dates are in same timezone)

### 🔍 **Performance Considerations**
- Index on (activity_date, user_id) for efficient GROUP BY
- Consider partitioning large activity tables by date
- Pre-aggregate daily summaries for frequently accessed metrics
- Use appropriate date comparison operators for index utilization

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding user engagement patterns
- **Data-Driven Decisions**: Using activity metrics for product improvements
- **Operational Excellence**: Efficient monitoring of user engagement
- **Innovation**: Advanced analytics for user behavior insights

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find weekly active user count for the same period**
2. **Calculate user retention rate from first to last day**
3. **Identify the most active day of the week**
4. **Find users who were active every day in the period**

Remember: User activity analysis and engagement metrics are fundamental to Amazon's product analytics, recommendation systems, user retention strategies, and business intelligence across all digital products!