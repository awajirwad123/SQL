# LeetCode Easy #511: Game Play Analysis I

## 📋 Problem Statement

Write a SQL query to report the **first login date** for each player.

Return the result table in **any order**.

## 🗄️ Table Schema

**Activity Table:**
```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| player_id    | int     |
| device_id    | int     |
| event_date   | date    |
| games_played | int     |
+--------------+---------+
```
- `(player_id, event_date)` is the primary key of this table.
- This table shows the activity of players of some games.
- Each row shows a player who logged in and played a number of games on a certain day using some device.

## 📊 Sample Data

**Activity Table:**
| player_id | device_id | event_date | games_played |
|-----------|-----------|------------|--------------|
| 1         | 2         | 2016-03-01 | 5            |
| 1         | 2         | 2016-03-02 | 6            |
| 2         | 3         | 2017-06-25 | 1            |
| 3         | 1         | 2016-03-02 | 0            |
| 3         | 4         | 2018-07-03 | 5            |

**Expected Output:**
| player_id | first_login |
|-----------|-------------|
| 1         | 2016-03-01  |
| 2         | 2017-06-25  |
| 3         | 2016-03-02  |

**Explanation:** Each player's first login date is shown.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find the earliest (minimum) date for each player
- This is a classic GROUP BY with aggregation problem
- Use MIN() function to find the earliest date
- Group by player_id to get result per player

### 2. **Key Insights**
- GROUP BY player_id to analyze each player separately
- MIN(event_date) gives the first login date
- Simple aggregation query - no complex joins needed
- Result should have one row per player

### 3. **Interview Discussion Points**
- "I need to find the minimum date for each player"
- "GROUP BY player_id and use MIN() aggregation"
- "This is straightforward aggregation without complex logic"

## 🔧 Step-by-Step Solution Logic

### Step 1: Group by Player
```sql
-- Group all activity records by player_id
GROUP BY player_id
```

### Step 2: Find Minimum Date
```sql
-- Use MIN() to find earliest event_date for each player
MIN(event_date) as first_login
```

### Step 3: Select Required Columns
```sql
-- Return player_id and their first login date
```

## ✅ Optimized SQL Solution

```sql
SELECT 
    player_id,
    MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;
```

### Alternative Solutions

**Using Window Function (More Complex but Shows Advanced Knowledge):**
```sql
SELECT DISTINCT
    player_id,
    FIRST_VALUE(event_date) OVER (
        PARTITION BY player_id 
        ORDER BY event_date
    ) AS first_login
FROM Activity;
```

**Using ROW_NUMBER() to Rank by Date:**
```sql
SELECT 
    player_id,
    event_date AS first_login
FROM (
    SELECT 
        player_id,
        event_date,
        ROW_NUMBER() OVER (
            PARTITION BY player_id 
            ORDER BY event_date
        ) AS rn
    FROM Activity
) ranked
WHERE rn = 1;
```

**Including Additional First-Day Information:**
```sql
SELECT 
    player_id,
    MIN(event_date) AS first_login,
    MIN(device_id) AS first_device,  -- Device used on first login
    MIN(games_played) AS games_on_first_day
FROM Activity
WHERE event_date = (
    SELECT MIN(event_date) 
    FROM Activity a2 
    WHERE a2.player_id = Activity.player_id
)
GROUP BY player_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Include Device Information**
```sql
-- "Also return the device used on the first login"

SELECT a.player_id,
       a.event_date AS first_login,
       a.device_id AS first_device,
       a.games_played AS games_on_first_day
FROM Activity a
INNER JOIN (
    SELECT player_id, MIN(event_date) AS first_date
    FROM Activity
    GROUP BY player_id
) first_login ON a.player_id = first_login.player_id 
                AND a.event_date = first_login.first_date;
```

#### 2. **Player Retention Analysis**
```sql
-- "Calculate how many days after first login each player returned"

WITH FirstLogin AS (
    SELECT player_id, MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id
)
SELECT 
    a.player_id,
    fl.first_login,
    a.event_date AS return_date,
    DATEDIFF(a.event_date, fl.first_login) AS days_since_first_login
FROM Activity a
INNER JOIN FirstLogin fl ON a.player_id = fl.player_id
WHERE a.event_date > fl.first_login
ORDER BY a.player_id, a.event_date;
```

#### 3. **Player Segmentation by First Login**
```sql
-- "Segment players by their first login period"

WITH FirstLogin AS (
    SELECT player_id, MIN(event_date) AS first_login
    FROM Activity
    GROUP BY player_id
)
SELECT 
    CASE 
        WHEN YEAR(first_login) = 2016 THEN 'Early Adopters'
        WHEN YEAR(first_login) = 2017 THEN 'Growth Phase'
        WHEN YEAR(first_login) >= 2018 THEN 'Recent Players'
    END AS player_segment,
    COUNT(*) AS player_count,
    MIN(first_login) AS earliest_in_segment,
    MAX(first_login) AS latest_in_segment
FROM FirstLogin
GROUP BY 
    CASE 
        WHEN YEAR(first_login) = 2016 THEN 'Early Adopters'
        WHEN YEAR(first_login) = 2017 THEN 'Growth Phase'
        WHEN YEAR(first_login) >= 2018 THEN 'Recent Players'
    END;
```

#### 4. **Performance Optimization**
```sql
-- "How would you optimize this for millions of player records?"

-- Add composite index for efficient grouping
CREATE INDEX idx_activity_player_date ON Activity(player_id, event_date);

-- The basic query remains efficient with proper indexing
SELECT 
    player_id,
    MIN(event_date) AS first_login
FROM Activity
GROUP BY player_id;

-- For very large datasets, consider partitioning by date
```

#### 5. **Advanced Analytics**
```sql
-- "Provide comprehensive first-day analysis"

WITH FirstDayStats AS (
    SELECT 
        player_id,
        MIN(event_date) AS first_login,
        SUM(CASE WHEN event_date = MIN(event_date) OVER (PARTITION BY player_id) 
                 THEN games_played ELSE 0 END) AS first_day_games,
        COUNT(DISTINCT CASE WHEN event_date = MIN(event_date) OVER (PARTITION BY player_id) 
                           THEN device_id END) AS devices_used_first_day
    FROM Activity
    GROUP BY player_id
)
SELECT 
    DATE(first_login) AS login_date,
    COUNT(*) AS new_players,
    AVG(first_day_games) AS avg_games_first_day,
    SUM(CASE WHEN first_day_games > 0 THEN 1 ELSE 0 END) AS active_players,
    SUM(CASE WHEN devices_used_first_day > 1 THEN 1 ELSE 0 END) AS multi_device_players
FROM FirstDayStats
GROUP BY DATE(first_login)
ORDER BY login_date;
```

## 🔗 Related LeetCode Questions

1. **#512 - Game Play Analysis II** (First login device analysis)
2. **#534 - Game Play Analysis III** (Cumulative games played)
3. **#550 - Game Play Analysis IV** (Player retention rate)
4. **#569 - Median Employee Salary** (GROUP BY with aggregation)
5. **#585 - Investments in 2016** (Complex aggregation with conditions)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **GROUP BY**: Essential for per-entity analysis
2. **MIN() Aggregation**: Finding earliest/smallest values
3. **Basic Analytics**: Foundation for more complex player analysis
4. **Primary Key Understanding**: Composite key (player_id, event_date)

### 🚀 **Amazon Interview Tips**
1. **Think beyond the basic query**: "What other insights could be valuable?"
2. **Consider performance**: "For large datasets, indexing on (player_id, event_date) is crucial"
3. **Business context**: "First login date helps with cohort analysis and retention"
4. **Data quality**: "What if a player has duplicate entries for the same date?"

### 🔧 **Common Patterns**
- GROUP BY + MIN/MAX for finding earliest/latest records
- Composite indexes on grouping and ordering columns
- Use results as foundation for more complex analytics
- Consider window functions for advanced requirements

### ⚠️ **Common Mistakes to Avoid**
1. Forgetting GROUP BY when using aggregation functions
2. Not considering composite primary keys properly
3. Missing indexes on frequently grouped columns
4. Not thinking about business applications of the results

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding player behavior improves game experience
- **Dive Deep**: First login is foundation for player lifecycle analysis
- **Data-Driven Decisions**: Enable product teams to understand user acquisition
- **Think Big**: Design queries that scale to millions of players

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find the last login date for each player**
2. **Find players who played games on their first day**
3. **Calculate days between first and second login**
4. **Find the most popular first-login device**

Remember: Player analytics is crucial for Amazon's gaming services and helps optimize user acquisition and retention strategies!