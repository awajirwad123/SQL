# LeetCode Easy #603: Consecutive Available Seats

## 📋 Problem Statement

Write a SQL query to report all the **consecutive available seats** in the cinema.

Return the result table **ordered by seat_id in ascending order**.

The test cases are generated so that **more than two seats are consecutively available**.

## 🗄️ Table Schema

**Cinema Table:**
```
+---------+------+
| Column Name | Type |
+---------+------+
| seat_id | int  |
| free    | bool |
+---------+------+
```
- `seat_id` is an auto-increment primary key column for this table.
- Each row of this table indicates whether the i-th seat is free or not. 1 means free while 0 means occupied.

## 📊 Sample Data

**Cinema Table:**
| seat_id | free |
|---------|------|
| 1       | 1    |
| 2       | 0    |
| 3       | 1    |
| 4       | 1    |
| 5       | 1    |

**Expected Output:**
| seat_id |
|---------|
| 3       |
| 4       |
| 5       |

**Explanation:** Only seats 3, 4, and 5 are consecutively available.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find seats that are free (free = 1)
- Seats must be consecutive (adjacent seat_ids)
- Need to identify groups of consecutive available seats
- Return all seats in consecutive groups

### 2. **Key Insights**
- Self-join approach: Join table with itself on consecutive seat_ids
- Check if current seat and next seat are both free
- Alternative: Use window functions to check adjacent seats
- Need to handle both directions of consecutiveness

### 3. **Interview Discussion Points**
- "I need to find seats where both current and adjacent seats are free"
- "Self-join on seat_id and seat_id+1 can identify consecutive pairs"
- "I should check both previous and next seats for completeness"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Adjacent Free Seats
```sql
-- Join table with itself to check consecutive seats
-- Check if both current and next seat are free
```

### Step 2: Handle Bidirectional Consecutiveness
```sql
-- A seat is part of consecutive group if:
-- (current seat is free AND next seat is free) OR
-- (current seat is free AND previous seat is free)
```

### Step 3: Return All Consecutive Seats
```sql
-- Use DISTINCT to avoid duplicates
-- Order by seat_id as required
```

## ✅ Optimized SQL Solution

**Solution 1: Self-Join Approach**
```sql
SELECT DISTINCT c1.seat_id
FROM Cinema c1
JOIN Cinema c2 ON ABS(c1.seat_id - c2.seat_id) = 1
WHERE c1.free = 1 AND c2.free = 1
ORDER BY c1.seat_id;
```

### Alternative Solutions

**Solution 2: Using OR with Self-Join**
```sql
SELECT DISTINCT c1.seat_id
FROM Cinema c1
JOIN Cinema c2 ON (c1.seat_id = c2.seat_id + 1 OR c1.seat_id = c2.seat_id - 1)
WHERE c1.free = 1 AND c2.free = 1
ORDER BY c1.seat_id;
```

**Solution 3: Window Functions Approach**
```sql
SELECT seat_id
FROM (
    SELECT 
        seat_id,
        free,
        LAG(free) OVER (ORDER BY seat_id) as prev_free,
        LEAD(free) OVER (ORDER BY seat_id) as next_free
    FROM Cinema
) t
WHERE free = 1 
  AND (prev_free = 1 OR next_free = 1)
ORDER BY seat_id;
```

**Solution 4: Exists Approach**
```sql
SELECT seat_id
FROM Cinema c1
WHERE c1.free = 1
  AND (
    EXISTS (SELECT 1 FROM Cinema c2 WHERE c2.seat_id = c1.seat_id + 1 AND c2.free = 1)
    OR
    EXISTS (SELECT 1 FROM Cinema c2 WHERE c2.seat_id = c1.seat_id - 1 AND c2.free = 1)
  )
ORDER BY seat_id;
```

**Solution 5: Union Approach**
```sql
SELECT DISTINCT seat_id
FROM (
    SELECT c1.seat_id
    FROM Cinema c1
    JOIN Cinema c2 ON c1.seat_id = c2.seat_id - 1
    WHERE c1.free = 1 AND c2.free = 1
    
    UNION
    
    SELECT c1.seat_id
    FROM Cinema c1
    JOIN Cinema c2 ON c1.seat_id = c2.seat_id + 1
    WHERE c1.free = 1 AND c2.free = 1
) consecutive_seats
ORDER BY seat_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Find Consecutive Groups with Size**
```sql
-- "Find all consecutive groups and their sizes"

WITH consecutive_marked AS (
    SELECT 
        seat_id,
        free,
        seat_id - ROW_NUMBER() OVER (ORDER BY seat_id) as group_id
    FROM Cinema
    WHERE free = 1
),
group_sizes AS (
    SELECT 
        group_id,
        COUNT(*) as group_size,
        MIN(seat_id) as start_seat,
        MAX(seat_id) as end_seat
    FROM consecutive_marked
    GROUP BY group_id
    HAVING COUNT(*) >= 2  -- Only groups with 2+ consecutive seats
)
SELECT 
    start_seat,
    end_seat,
    group_size,
    CONCAT(start_seat, '-', end_seat) as seat_range
FROM group_sizes
ORDER BY start_seat;
```

#### 2. **Seat Availability Analytics**
```sql
-- "Provide comprehensive seat availability analysis"

WITH seat_analysis AS (
    SELECT 
        COUNT(*) as total_seats,
        SUM(free) as available_seats,
        COUNT(*) - SUM(free) as occupied_seats,
        ROUND(SUM(free) * 100.0 / COUNT(*), 2) as availability_percentage
    FROM Cinema
),
consecutive_groups AS (
    SELECT 
        seat_id - ROW_NUMBER() OVER (ORDER BY seat_id) as group_id,
        COUNT(*) as consecutive_count
    FROM Cinema
    WHERE free = 1
    GROUP BY seat_id - ROW_NUMBER() OVER (ORDER BY seat_id)
    HAVING COUNT(*) >= 2
)
SELECT 
    sa.*,
    COUNT(cg.group_id) as consecutive_groups,
    MAX(cg.consecutive_count) as largest_consecutive_block,
    AVG(cg.consecutive_count) as avg_consecutive_block_size
FROM seat_analysis sa
CROSS JOIN consecutive_groups cg
GROUP BY sa.total_seats, sa.available_seats, sa.occupied_seats, sa.availability_percentage;
```

#### 3. **Optimal Seating Recommendations**
```sql
-- "Recommend best seating options for different group sizes"

WITH consecutive_blocks AS (
    SELECT 
        seat_id - ROW_NUMBER() OVER (ORDER BY seat_id) as block_id,
        MIN(seat_id) as start_seat,
        MAX(seat_id) as end_seat,
        COUNT(*) as block_size
    FROM Cinema
    WHERE free = 1
    GROUP BY seat_id - ROW_NUMBER() OVER (ORDER BY seat_id)
    HAVING COUNT(*) >= 2
)
SELECT 
    2 as requested_group_size,
    start_seat,
    start_seat + 1 as end_seat,
    'Perfect Fit' as recommendation_type
FROM consecutive_blocks
WHERE block_size >= 2

UNION ALL

SELECT 
    3 as requested_group_size,
    start_seat,
    start_seat + 2 as end_seat,
    CASE 
        WHEN block_size = 3 THEN 'Perfect Fit'
        WHEN block_size > 3 THEN 'Room to Spare'
    END as recommendation_type
FROM consecutive_blocks
WHERE block_size >= 3

UNION ALL

SELECT 
    4 as requested_group_size,
    start_seat,
    start_seat + 3 as end_seat,
    CASE 
        WHEN block_size = 4 THEN 'Perfect Fit'
        WHEN block_size > 4 THEN 'Room to Spare'
    END as recommendation_type
FROM consecutive_blocks
WHERE block_size >= 4

ORDER BY requested_group_size, start_seat;
```

#### 4. **Time-Series Seat Utilization**
```sql
-- "Track seat utilization patterns over time"

-- Assuming we have a timestamp column
WITH hourly_utilization AS (
    SELECT 
        DATE_FORMAT(booking_time, '%H:00') as hour_slot,
        COUNT(*) as total_seats,
        SUM(CASE WHEN free = 0 THEN 1 ELSE 0 END) as occupied_seats,
        SUM(CASE WHEN free = 1 THEN 1 ELSE 0 END) as available_seats,
        ROUND(SUM(CASE WHEN free = 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) as occupancy_rate
    FROM Cinema_historical
    WHERE DATE(booking_time) = CURRENT_DATE
    GROUP BY DATE_FORMAT(booking_time, '%H:00')
)
SELECT 
    hour_slot,
    occupancy_rate,
    CASE 
        WHEN occupancy_rate >= 90 THEN 'Peak Time'
        WHEN occupancy_rate >= 70 THEN 'High Demand'
        WHEN occupancy_rate >= 50 THEN 'Moderate Demand'
        ELSE 'Low Demand'
    END as demand_level,
    available_seats
FROM hourly_utilization
ORDER BY hour_slot;
```

#### 5. **Premium Seat Analysis**
```sql
-- "Analyze consecutive availability for premium seats"

-- Assuming seats have a tier (Premium, Standard, Economy)
WITH premium_consecutive AS (
    SELECT 
        c1.seat_id,
        c1.seat_tier,
        CASE 
            WHEN c1.seat_tier = 'Premium' THEN 'Premium'
            WHEN c1.seat_tier = 'Standard' THEN 'Standard'
            ELSE 'Economy'
        END as tier_category
    FROM Cinema c1
    JOIN Cinema c2 ON ABS(c1.seat_id - c2.seat_id) = 1
    WHERE c1.free = 1 AND c2.free = 1
)
SELECT 
    tier_category,
    COUNT(*) as consecutive_seats_available,
    COUNT(*) * 100.0 / (
        SELECT COUNT(*) 
        FROM Cinema 
        WHERE seat_tier = pc.tier_category
    ) as percentage_of_tier_available
FROM premium_consecutive pc
GROUP BY tier_category
ORDER BY 
    CASE tier_category 
        WHEN 'Premium' THEN 1 
        WHEN 'Standard' THEN 2 
        ELSE 3 
    END;
```

#### 6. **Seat Assignment Optimization**
```sql
-- "Optimize seat assignments to maximize consecutive availability"

WITH available_blocks AS (
    SELECT 
        seat_id - ROW_NUMBER() OVER (ORDER BY seat_id) as block_id,
        MIN(seat_id) as start_seat,
        MAX(seat_id) as end_seat,
        COUNT(*) as block_size,
        GROUP_CONCAT(seat_id ORDER BY seat_id) as seats_in_block
    FROM Cinema
    WHERE free = 1
    GROUP BY seat_id - ROW_NUMBER() OVER (ORDER BY seat_id)
    HAVING COUNT(*) >= 2
),
fragmentation_analysis AS (
    SELECT 
        COUNT(*) as total_blocks,
        AVG(block_size) as avg_block_size,
        MAX(block_size) as max_block_size,
        MIN(block_size) as min_block_size,
        SUM(block_size) as total_consecutive_seats
    FROM available_blocks
)
SELECT 
    'Seat Fragmentation Analysis' as analysis_type,
    total_blocks,
    total_consecutive_seats,
    avg_block_size,
    max_block_size,
    CASE 
        WHEN total_blocks <= 3 THEN 'Low Fragmentation'
        WHEN total_blocks <= 6 THEN 'Moderate Fragmentation'
        ELSE 'High Fragmentation'
    END as fragmentation_level
FROM fragmentation_analysis;
```

## 🔗 Related LeetCode Questions

1. **#197 - Rising Temperature** (Self-join with consecutive dates)
2. **#180 - Consecutive Numbers** (Finding consecutive sequences)
3. **#626 - Exchange Seats** (Seat manipulation)
4. **#1045 - Customers Who Bought All Products** (Group analysis)
5. **#1532 - The Most Recent Three Orders** (Consecutive ordering)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Join**: Joining table with itself for consecutive checks
2. **Consecutive Logic**: Using ABS() or explicit +1/-1 conditions
3. **Window Functions**: LAG/LEAD for checking adjacent rows
4. **Group Analysis**: Identifying consecutive sequences

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I return individual seats or groups?"
2. **Discuss approaches**: "I can use self-join or window functions"
3. **Consider performance**: "Self-join vs window functions have different performance characteristics"
4. **Think business value**: "This helps optimize cinema seating and revenue"

### 🔧 **Common Patterns**
- Self-join with ABS(id1 - id2) = 1 for consecutive check
- Window functions LAG/LEAD for adjacent row analysis
- ROW_NUMBER() for identifying consecutive groups
- EXISTS for checking adjacent records

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting DISTINCT** when using self-join (may create duplicates)
2. **Only checking one direction** (next seat but not previous)
3. **Not handling edge cases** (first/last seats)
4. **Incorrect consecutive logic** (using > 1 instead of = 1)

### 🔍 **Performance Considerations**
- Self-join can be expensive on large datasets
- Index on seat_id for efficient joins
- Window functions may be more efficient for consecutive analysis
- Consider partitioning for very large cinema chains

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Ensuring customers can sit together
- **Operational Excellence**: Optimizing seat allocation algorithms
- **Frugality**: Maximizing venue utilization and revenue
- **Innovation**: Advanced analytics for dynamic pricing

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find the longest consecutive block of available seats**
2. **Find seats where you can sit with exactly 2 friends (3 consecutive)**
3. **Calculate seat utilization rate for different sections**
4. **Find optimal seating for groups of different sizes**

Remember: Seat management and optimization is crucial for Amazon's logistics planning, warehouse layout optimization, and resource allocation strategies!