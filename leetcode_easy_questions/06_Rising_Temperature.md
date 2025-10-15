# LeetCode Easy #197: Rising Temperature

## 📋 Problem Statement

Write a SQL query to find all dates' `Id` with higher temperature compared to its previous dates (yesterday).

Return the result table in **any order**.

## 🗄️ Table Schema

**Weather Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| recordDate    | date    |
| temperature   | int     |
+---------------+---------+
```
- `id` is the primary key column for this table.
- This table contains information about the temperature on a certain day.

## 📊 Sample Data

**Weather Table:**
| id | recordDate | temperature |
|----|------------|-------------|
| 1  | 2015-01-01 | 10          |
| 2  | 2015-01-02 | 25          |
| 3  | 2015-01-03 | 20          |
| 4  | 2015-01-04 | 30          |

**Expected Output:**
| id |
|----|
| 2  |
| 4  |

**Explanation:** 
- On 2015-01-02, the temperature was higher than the previous day (25 > 10).
- On 2015-01-04, the temperature was higher than the previous day (30 > 20).

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to compare each day's temperature with the previous day
- This requires self-join on consecutive dates
- Previous day = current date - 1 day
- Return IDs where current temperature > previous temperature

### 2. **Key Insights**
- Self-join the Weather table with itself
- Join condition: w1.recordDate = w2.recordDate + 1 day
- Compare temperatures: w1.temperature > w2.temperature
- Handle date arithmetic correctly

### 3. **Interview Discussion Points**
- "I need to compare each day with its previous day"
- "Self-join on consecutive dates using date arithmetic"
- "Need to handle cases where previous day data might be missing"

## 🔧 Step-by-Step Solution Logic

### Step 1: Set Up Self-Join
```sql
-- Join Weather table with itself
-- w1 represents current day, w2 represents previous day
```

### Step 2: Define Date Relationship
```sql
-- Join condition: current_date = previous_date + 1
-- Handle different database date arithmetic
```

### Step 3: Compare Temperatures
```sql
-- Filter where current temperature > previous temperature
```

## ✅ Optimized SQL Solution

**Solution 1: Using DATE_ADD/DATE_SUB (MySQL)**
```sql
SELECT w1.id
FROM Weather w1
INNER JOIN Weather w2 
ON w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
WHERE w1.temperature > w2.temperature;
```

### Alternative Solutions

**Solution 2: Using DATEADD (SQL Server)**
```sql
SELECT w1.id
FROM Weather w1
INNER JOIN Weather w2 
ON w1.recordDate = DATEADD(day, 1, w2.recordDate)
WHERE w1.temperature > w2.temperature;
```

**Solution 3: Using Date Arithmetic (PostgreSQL)**
```sql
SELECT w1.id
FROM Weather w1
INNER JOIN Weather w2 
ON w1.recordDate = w2.recordDate + INTERVAL '1 day'
WHERE w1.temperature > w2.temperature;
```

**Solution 4: Using Window Functions (More Advanced)**
```sql
SELECT id
FROM (
    SELECT id,
           temperature,
           LAG(temperature) OVER (ORDER BY recordDate) as prev_temperature
    FROM Weather
) t
WHERE temperature > prev_temperature;
```

**Solution 5: Using DATEDIFF (Generic Approach)**
```sql
SELECT w1.id
FROM Weather w1
INNER JOIN Weather w2 
ON DATEDIFF(w1.recordDate, w2.recordDate) = 1
WHERE w1.temperature > w2.temperature;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Handle Missing Dates**
```sql
-- "What if some dates are missing in the dataset?"

-- Window function approach handles gaps better
WITH WeatherWithPrevious AS (
    SELECT id,
           recordDate,
           temperature,
           LAG(temperature) OVER (ORDER BY recordDate) as prev_temperature,
           LAG(recordDate) OVER (ORDER BY recordDate) as prev_date
    FROM Weather
)
SELECT id
FROM WeatherWithPrevious
WHERE temperature > prev_temperature
  AND DATEDIFF(recordDate, prev_date) = 1; -- Ensure consecutive days
```

#### 2. **Multiple Day Comparisons**
```sql
-- "Find days with temperature higher than both previous and next day"

SELECT w2.id
FROM Weather w1
INNER JOIN Weather w2 ON w2.recordDate = DATE_ADD(w1.recordDate, INTERVAL 1 DAY)
INNER JOIN Weather w3 ON w3.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
WHERE w2.temperature > w1.temperature 
  AND w2.temperature > w3.temperature;
```

#### 3. **Temperature Trend Analysis**
```sql
-- "Find consecutive days of rising temperature"

WITH TemperatureComparison AS (
    SELECT id,
           recordDate,
           temperature,
           LAG(temperature) OVER (ORDER BY recordDate) as prev_temp,
           CASE 
               WHEN temperature > LAG(temperature) OVER (ORDER BY recordDate) 
               THEN 1 ELSE 0 
           END as is_rising
    FROM Weather
),
ConsecutiveRising AS (
    SELECT *,
           SUM(CASE WHEN is_rising = 0 THEN 1 ELSE 0 END) 
           OVER (ORDER BY recordDate) as group_id
    FROM TemperatureComparison
)
SELECT GROUP_CONCAT(id) as consecutive_rising_days,
       COUNT(*) as streak_length
FROM ConsecutiveRising
WHERE is_rising = 1
GROUP BY group_id
HAVING COUNT(*) >= 2;  -- At least 2 consecutive rising days
```

#### 4. **Performance Optimization**
```sql
-- "How would you optimize this for millions of weather records?"

-- Add composite index on recordDate and temperature
CREATE INDEX idx_weather_date_temp ON Weather(recordDate, temperature);

-- The query remains efficient with proper indexing
SELECT w1.id
FROM Weather w1
INNER JOIN Weather w2 
ON w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
WHERE w1.temperature > w2.temperature;
```

#### 5. **Statistical Analysis**
```sql
-- "Provide temperature trend statistics"

WITH DailyComparison AS (
    SELECT w1.id,
           w1.recordDate,
           w1.temperature,
           w2.temperature as prev_temperature,
           w1.temperature - w2.temperature as temp_change
    FROM Weather w1
    LEFT JOIN Weather w2 
    ON w1.recordDate = DATE_ADD(w2.recordDate, INTERVAL 1 DAY)
)
SELECT 
    COUNT(*) as total_days_with_previous,
    SUM(CASE WHEN temp_change > 0 THEN 1 ELSE 0 END) as rising_days,
    SUM(CASE WHEN temp_change < 0 THEN 1 ELSE 0 END) as falling_days,
    SUM(CASE WHEN temp_change = 0 THEN 1 ELSE 0 END) as stable_days,
    AVG(temp_change) as avg_daily_change,
    MAX(temp_change) as max_daily_increase,
    MIN(temp_change) as max_daily_decrease
FROM DailyComparison
WHERE prev_temperature IS NOT NULL;
```

## 🔗 Related LeetCode Questions

1. **#180 - Consecutive Numbers** (Consecutive sequence analysis)
2. **#181 - Employees Earning More Than Their Managers** (Self-join comparisons)
3. **#626 - Exchange Seats** (Date/sequence manipulation)
4. **#1204 - Last Person to Fit in the Elevator** (Cumulative analysis)
5. **#1341 - Movie Rating** (Time-based analysis)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Join**: Joining table with itself for comparisons
2. **Date Arithmetic**: Adding/subtracting days from dates
3. **Window Functions**: Alternative approach using LAG/LEAD
4. **Consecutive Records**: Handling sequential data analysis

### 🚀 **Amazon Interview Tips**
1. **Clarify date handling**: "Should I handle weekends or missing dates differently?"
2. **Discuss performance**: "For large datasets, I'd index on recordDate"
3. **Consider edge cases**: "What about the first record with no previous day?"
4. **Alternative approaches**: "Window functions can be more readable and handle gaps better"

### 🔧 **Common Patterns**
- Self-join with date arithmetic for consecutive day analysis
- Window functions (LAG/LEAD) for time series comparisons
- Handle missing dates appropriately
- Index date columns for performance

### ⚠️ **Common Mistakes to Avoid**
1. Wrong date arithmetic syntax for different databases
2. Not handling missing dates or gaps in data
3. Forgetting to index date columns for performance
4. Incorrect join conditions leading to wrong results

### 🎯 **Amazon Leadership Principles Applied**
- **Dive Deep**: Understand time series data patterns
- **Customer Obsession**: Accurate weather data affects customer decisions
- **Operational Excellence**: Handle edge cases and missing data gracefully
- **Think Big**: Design solutions that scale to global weather data

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find days with temperature drop of more than 5 degrees**
2. **Find the longest streak of rising temperatures**
3. **Calculate moving average temperature over 7 days**
4. **Find days with temperature higher than same day last week**

Remember: Time series analysis is crucial for Amazon's demand forecasting and logistics optimization!