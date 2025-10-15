# LeetCode Easy #1741: Find Total Time Spent by Each Employee

## 📋 Problem Statement

Write a SQL query to calculate the **total time** in minutes spent by each employee on each day at the office. Note that within one day, an employee can enter and leave more than once. The time spent in the office for a single entry is **out_time - in_time**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Employees Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| emp_id      | int  |
| event_day   | date |
| in_time     | int  |
| out_time    | int  |
+-------------+------+
```
- (emp_id, event_day, in_time) is the primary key of this table.
- The table shows the employees' entries and exits in an office.
- event_day is the day at which this event happened, in_time is the minute at which the employee entered the office, and out_time is the minute at which they left the office.
- in_time and out_time are between 1 and 1440.
- It is guaranteed that no two events on the same day intersect in time, and in_time < out_time.

## 📊 Sample Data

**Employees Table:**
| emp_id | event_day  | in_time | out_time |
|--------|------------|---------|----------|
| 1      | 2020-11-28 | 4       | 32       |
| 1      | 2020-11-28 | 55      | 200      |
| 1      | 2020-12-03 | 1       | 42       |
| 2      | 2020-11-28 | 3       | 33       |
| 2      | 2020-12-09 | 47      | 74       |

**Expected Output:**
| day        | emp_id | total_time |
|------------|--------|------------|
| 2020-11-28 | 1      | 173        |
| 2020-11-28 | 2      | 30         |
| 2020-12-03 | 1      | 41         |
| 2020-12-09 | 2      | 27         |

**Explanation:**
- Employee 1 on 2020-11-28: (32-4) + (200-55) = 28 + 145 = 173 minutes
- Employee 2 on 2020-11-28: (33-3) = 30 minutes
- Employee 1 on 2020-12-03: (42-1) = 41 minutes
- Employee 2 on 2020-12-09: (74-47) = 27 minutes

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Calculate time spent for each employee entry/exit session
- Sum all time periods for each employee on each day
- Handle multiple entries/exits per day per employee
- Time calculation: out_time - in_time

### 2. **Key Insights**
- GROUP BY employee and day to aggregate multiple sessions
- SUM(out_time - in_time) calculates total daily time
- Each row represents one work session
- Need to aggregate across all sessions per employee per day

### 3. **Interview Discussion Points**
- "Each row is one work session, need to sum all sessions per employee per day"
- "out_time - in_time gives session duration, SUM aggregates daily total"
- "GROUP BY emp_id and event_day handles multiple entries per day"

## 🔧 Step-by-Step Solution Logic

### Step 1: Calculate Session Duration
```sql
-- Calculate time spent in each session
out_time - in_time as session_duration
```

### Step 2: Group by Employee and Day
```sql
-- Group to aggregate multiple sessions per employee per day
GROUP BY emp_id, event_day
```

### Step 3: Sum All Sessions per Day
```sql
-- Sum all session durations for each employee per day
SUM(out_time - in_time) as total_time
```

## ✅ Optimized SQL Solution

**Solution 1: Basic Aggregation**
```sql
SELECT 
    event_day as day,
    emp_id,
    SUM(out_time - in_time) as total_time
FROM Employees
GROUP BY emp_id, event_day;
```

### Alternative Solutions

**Solution 2: With Detailed Session Information**
```sql
SELECT 
    event_day as day,
    emp_id,
    COUNT(*) as sessions_count,
    SUM(out_time - in_time) as total_time,
    AVG(out_time - in_time) as avg_session_time
FROM Employees
GROUP BY emp_id, event_day
ORDER BY event_day, emp_id;
```

**Solution 3: Using Subquery for Clarity**
```sql
SELECT 
    day,
    emp_id,
    SUM(session_time) as total_time
FROM (
    SELECT 
        event_day as day,
        emp_id,
        out_time - in_time as session_time
    FROM Employees
) session_data
GROUP BY emp_id, day;
```

**Solution 4: With Time Range Analysis**
```sql
SELECT 
    event_day as day,
    emp_id,
    SUM(out_time - in_time) as total_time,
    MIN(in_time) as first_entry,
    MAX(out_time) as last_exit,
    MAX(out_time) - MIN(in_time) as office_span
FROM Employees
GROUP BY emp_id, event_day
ORDER BY event_day, emp_id;
```

**Solution 5: Comprehensive Workforce Analytics System**
```sql
WITH employee_session_analysis AS (
    SELECT 
        emp_id,
        event_day,
        in_time,
        out_time,
        out_time - in_time as session_duration,
        
        -- Session timing classification
        CASE 
            WHEN in_time <= 60 THEN 'Early Start'  -- Before 1:00 AM (60 minutes)
            WHEN in_time <= 480 THEN 'Morning Start'  -- Before 8:00 AM (480 minutes)
            WHEN in_time <= 720 THEN 'Standard Start'  -- Before 12:00 PM (720 minutes)
            WHEN in_time <= 960 THEN 'Afternoon Start'  -- Before 4:00 PM (960 minutes)
            ELSE 'Late Start'  -- After 4:00 PM
        END as entry_time_category,
        
        -- Session length classification
        CASE 
            WHEN out_time - in_time <= 30 THEN 'Short Session'  -- <= 30 minutes
            WHEN out_time - in_time <= 120 THEN 'Medium Session'  -- <= 2 hours
            WHEN out_time - in_time <= 480 THEN 'Long Session'  -- <= 8 hours
            ELSE 'Extended Session'  -- > 8 hours
        END as session_length_category,
        
        -- Work pattern simulation
        CASE 
            WHEN emp_id % 4 = 0 THEN 'Flexible Schedule'
            WHEN emp_id % 4 = 1 THEN 'Standard Schedule'
            WHEN emp_id % 4 = 2 THEN 'Shift Worker'
            ELSE 'Remote/Hybrid Worker'
        END as work_pattern,
        
        -- Day of week analysis (simulated)
        CASE 
            WHEN DAYOFWEEK(event_day) = 1 THEN 'Sunday'
            WHEN DAYOFWEEK(event_day) = 2 THEN 'Monday'
            WHEN DAYOFWEEK(event_day) = 3 THEN 'Tuesday'
            WHEN DAYOFWEEK(event_day) = 4 THEN 'Wednesday'
            WHEN DAYOFWEEK(event_day) = 5 THEN 'Thursday'
            WHEN DAYOFWEEK(event_day) = 6 THEN 'Friday'
            ELSE 'Saturday'
        END as day_of_week,
        
        -- Productivity indicators (simulated)
        CASE 
            WHEN out_time - in_time >= 240 AND in_time <= 480 THEN 'High Productivity Window'
            WHEN out_time - in_time >= 120 AND in_time <= 720 THEN 'Good Productivity Window'
            WHEN out_time - in_time >= 60 THEN 'Standard Productivity Window'
            ELSE 'Short Productivity Window'
        END as productivity_window
    FROM Employees
),
daily_workforce_analytics AS (
    SELECT 
        emp_id,
        event_day,
        work_pattern,
        day_of_week,
        
        -- Core time metrics
        COUNT(*) as total_sessions,
        SUM(session_duration) as total_time,
        AVG(session_duration) as avg_session_duration,
        MIN(session_duration) as shortest_session,
        MAX(session_duration) as longest_session,
        STDDEV(session_duration) as session_variability,
        
        -- Time range analysis
        MIN(in_time) as earliest_entry,
        MAX(out_time) as latest_exit,
        MAX(out_time) - MIN(in_time) as office_time_span,
        
        -- Session pattern analysis
        COUNT(CASE WHEN session_length_category = 'Short Session' THEN 1 END) as short_sessions,
        COUNT(CASE WHEN session_length_category = 'Medium Session' THEN 1 END) as medium_sessions,
        COUNT(CASE WHEN session_length_category = 'Long Session' THEN 1 END) as long_sessions,
        COUNT(CASE WHEN session_length_category = 'Extended Session' THEN 1 END) as extended_sessions,
        
        -- Entry time patterns
        COUNT(CASE WHEN entry_time_category = 'Early Start' THEN 1 END) as early_starts,
        COUNT(CASE WHEN entry_time_category = 'Morning Start' THEN 1 END) as morning_starts,
        COUNT(CASE WHEN entry_time_category = 'Standard Start' THEN 1 END) as standard_starts,
        COUNT(CASE WHEN entry_time_category = 'Afternoon Start' THEN 1 END) as afternoon_starts,
        COUNT(CASE WHEN entry_time_category = 'Late Start' THEN 1 END) as late_starts,
        
        -- Productivity assessment
        COUNT(CASE WHEN productivity_window = 'High Productivity Window' THEN 1 END) as high_productivity_sessions,
        SUM(CASE WHEN productivity_window = 'High Productivity Window' THEN session_duration ELSE 0 END) as high_productivity_time,
        
        -- Work efficiency indicators
        CASE 
            WHEN SUM(session_duration) >= 480 AND COUNT(*) <= 2 THEN 'Efficient Full-Day Worker'
            WHEN SUM(session_duration) >= 240 AND COUNT(*) <= 3 THEN 'Efficient Part-Time Worker'
            WHEN COUNT(*) >= 4 AND AVG(session_duration) >= 60 THEN 'Flexible Multi-Session Worker'
            WHEN COUNT(*) >= 5 THEN 'Highly Fragmented Schedule'
            ELSE 'Standard Work Pattern'
        END as work_efficiency_pattern,
        
        -- Schedule consistency
        CASE 
            WHEN STDDEV(session_duration) <= 30 THEN 'Highly Consistent'
            WHEN STDDEV(session_duration) <= 60 THEN 'Moderately Consistent'
            WHEN STDDEV(session_duration) <= 120 THEN 'Somewhat Variable'
            ELSE 'Highly Variable'
        END as schedule_consistency
    FROM employee_session_analysis
    GROUP BY emp_id, event_day, work_pattern, day_of_week
),
workforce_performance_insights AS (
    SELECT 
        dwa.*,
        
        -- Performance classification
        CASE 
            WHEN total_time >= 480 AND work_efficiency_pattern = 'Efficient Full-Day Worker'
            THEN 'High Performer - Full Commitment'
            WHEN total_time >= 240 AND high_productivity_sessions >= 2
            THEN 'High Performer - Quality Focus'
            WHEN total_time >= 360 AND schedule_consistency = 'Highly Consistent'
            THEN 'Reliable Performer - Consistent Delivery'
            WHEN total_sessions >= 4 AND avg_session_duration >= 90
            THEN 'Flexible Performer - Adaptive Schedule'
            ELSE 'Standard Performer - Meeting Expectations'
        END as performance_classification,
        
        -- Work-life balance assessment
        CASE 
            WHEN latest_exit <= 1080 AND total_time >= 420 THEN 'Excellent Work-Life Balance'  -- End by 6 PM, 7+ hours
            WHEN latest_exit <= 1200 AND total_time >= 360 THEN 'Good Work-Life Balance'      -- End by 8 PM, 6+ hours
            WHEN latest_exit <= 1320 AND total_time >= 480 THEN 'Standard Work-Life Balance'  -- End by 10 PM, 8+ hours
            WHEN latest_exit > 1320 OR total_time > 600 THEN 'Work-Life Balance Concerns'     -- Late nights or long hours
            ELSE 'Balanced Approach'
        END as work_life_balance,
        
        -- Collaboration and availability insights
        CASE 
            WHEN office_time_span >= 600 AND total_sessions <= 2
            THEN 'High Availability - Long Office Presence'
            WHEN total_sessions >= 4 AND office_time_span <= 480
            THEN 'Focused Availability - Multiple Short Sessions'
            WHEN morning_starts >= 1 AND afternoon_starts >= 1
            THEN 'Extended Availability - Morning and Afternoon Presence'
            WHEN standard_starts = total_sessions
            THEN 'Standard Availability - Consistent Schedule'
            ELSE 'Variable Availability - Flexible Presence'
        END as collaboration_availability,
        
        -- Schedule optimization recommendations
        CASE 
            WHEN total_sessions >= 5 AND avg_session_duration < 60
            THEN 'Optimization: Consolidate short sessions + reduce fragmentation + improve focus blocks'
            WHEN schedule_consistency = 'Highly Variable' AND total_time >= 360
            THEN 'Optimization: Establish routine + consistent start times + predictable schedule'
            WHEN work_efficiency_pattern = 'Highly Fragmented Schedule'
            THEN 'Optimization: Block scheduling + minimize interruptions + structured work periods'
            WHEN high_productivity_sessions = 0 AND total_time >= 240
            THEN 'Optimization: Identify peak hours + align complex work + optimize energy cycles'
            ELSE 'Optimization: Maintain current approach + minor efficiency improvements'
        END as schedule_optimization,
        
        -- Manager insights and recommendations
        CASE 
            WHEN performance_classification LIKE 'High Performer%' AND work_life_balance = 'Excellent Work-Life Balance'
            THEN 'Management: Exemplary employee + potential mentor + leadership development candidate'
            WHEN work_efficiency_pattern = 'Flexible Multi-Session Worker' AND total_time >= 360
            THEN 'Management: Support flexible schedule + optimize for productivity + leverage adaptability'
            WHEN work_life_balance = 'Work-Life Balance Concerns'
            THEN 'Management: Wellness check + workload assessment + support resources + boundary setting'
            WHEN schedule_consistency = 'Highly Variable' AND total_sessions >= 4
            THEN 'Management: Schedule discussion + expectation alignment + workflow optimization'
            ELSE 'Management: Standard support + periodic check-ins + performance recognition'
        END as management_recommendations
    FROM daily_workforce_analytics dwa
)
SELECT 
    event_day as day,
    emp_id,
    total_time,
    work_pattern,
    day_of_week,
    performance_classification,
    work_life_balance,
    collaboration_availability,
    schedule_optimization,
    management_recommendations,
    
    -- Session details
    total_sessions,
    avg_session_duration,
    work_efficiency_pattern,
    schedule_consistency,
    
    -- Time analysis
    earliest_entry,
    latest_exit,
    office_time_span,
    
    -- Productivity metrics
    high_productivity_sessions,
    high_productivity_time,
    
    -- Session distribution
    short_sessions,
    medium_sessions,
    long_sessions,
    extended_sessions
FROM workforce_performance_insights
ORDER BY day, emp_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Workforce Optimization and Productivity Analytics Platform**
```sql
-- "Build comprehensive workforce optimization system for Amazon's diverse work environments and schedules"

WITH amazon_workforce_time_analysis AS (
    SELECT 
        emp_id,
        event_day,
        in_time,
        out_time,
        out_time - in_time as session_duration,
        
        -- Amazon business unit simulation
        CASE 
            WHEN emp_id % 8 = 0 THEN 'AWS'
            WHEN emp_id % 8 = 1 THEN 'Fulfillment Centers'
            WHEN emp_id % 8 = 2 THEN 'Prime Video'
            WHEN emp_id % 8 = 3 THEN 'Alexa'
            WHEN emp_id % 8 = 4 THEN 'Retail'
            WHEN emp_id % 8 = 5 THEN 'Logistics'
            WHEN emp_id % 8 = 6 THEN 'Advertising'
            ELSE 'Corporate'
        END as business_unit,
        
        -- Work location simulation
        CASE 
            WHEN emp_id % 5 = 0 THEN 'Seattle HQ'
            WHEN emp_id % 5 = 1 THEN 'Fulfillment Center'
            WHEN emp_id % 5 = 2 THEN 'Remote'
            WHEN emp_id % 5 = 3 THEN 'Regional Office'
            ELSE 'Customer Service Center'
        END as work_location,
        
        -- Amazon role level simulation
        CASE 
            WHEN emp_id % 6 = 0 THEN 'L8-L10 Principal/Director'
            WHEN emp_id % 6 = 1 THEN 'L7 Senior Manager'
            WHEN emp_id % 6 = 2 THEN 'L6 Manager'
            WHEN emp_id % 6 = 3 THEN 'L5 Senior Individual Contributor'
            WHEN emp_id % 6 = 4 THEN 'L4 Individual Contributor'
            ELSE 'L1-L3 Associate/Support'
        END as amazon_level,
        
        -- Shift type based on timing
        CASE 
            WHEN in_time <= 360 THEN 'Night Shift'     -- Before 6 AM
            WHEN in_time <= 600 THEN 'Early Shift'     -- 6 AM - 10 AM
            WHEN in_time <= 840 THEN 'Day Shift'       -- 10 AM - 2 PM
            WHEN in_time <= 1080 THEN 'Afternoon Shift' -- 2 PM - 6 PM
            ELSE 'Evening Shift'                        -- After 6 PM
        END as shift_type,
        
        -- Operational impact based on business unit
        CASE 
            WHEN emp_id % 8 = 1 AND in_time <= 480 THEN 'Critical Operations - Early Fulfillment'
            WHEN emp_id % 8 = 5 AND out_time - in_time >= 480 THEN 'Critical Operations - Extended Logistics'
            WHEN emp_id % 8 = 0 AND out_time - in_time >= 360 THEN 'High Impact - AWS Infrastructure'
            WHEN emp_id % 8 = 4 AND in_time BETWEEN 480 AND 1080 THEN 'Customer Impact - Retail Hours'
            ELSE 'Standard Operations'
        END as operational_impact,
        
        -- Productivity zone based on Amazon research
        CASE 
            WHEN in_time BETWEEN 480 AND 600 AND out_time - in_time >= 240 THEN 'Peak Productivity Zone'
            WHEN in_time BETWEEN 780 AND 900 AND out_time - in_time >= 180 THEN 'High Productivity Zone'
            WHEN out_time - in_time >= 360 THEN 'Extended Productivity Zone'
            WHEN out_time - in_time BETWEEN 120 AND 240 THEN 'Focused Productivity Zone'
            ELSE 'Standard Productivity Zone'
        END as productivity_zone
    FROM Employees
),
amazon_daily_operations_analysis AS (
    SELECT 
        emp_id,
        event_day,
        business_unit,
        work_location,
        amazon_level,
        
        -- Core time metrics
        COUNT(*) as total_sessions,
        SUM(session_duration) as total_time,
        AVG(session_duration) as avg_session_time,
        MIN(in_time) as first_entry,
        MAX(out_time) as last_exit,
        
        -- Shift distribution
        COUNT(CASE WHEN shift_type = 'Night Shift' THEN 1 END) as night_shift_sessions,
        COUNT(CASE WHEN shift_type = 'Early Shift' THEN 1 END) as early_shift_sessions,
        COUNT(CASE WHEN shift_type = 'Day Shift' THEN 1 END) as day_shift_sessions,
        COUNT(CASE WHEN shift_type = 'Afternoon Shift' THEN 1 END) as afternoon_shift_sessions,
        COUNT(CASE WHEN shift_type = 'Evening Shift' THEN 1 END) as evening_shift_sessions,
        
        -- Operational impact assessment
        COUNT(CASE WHEN operational_impact LIKE 'Critical Operations%' THEN 1 END) as critical_operations_sessions,
        COUNT(CASE WHEN operational_impact LIKE 'High Impact%' THEN 1 END) as high_impact_sessions,
        COUNT(CASE WHEN operational_impact LIKE 'Customer Impact%' THEN 1 END) as customer_impact_sessions,
        
        -- Productivity zone analysis
        COUNT(CASE WHEN productivity_zone = 'Peak Productivity Zone' THEN 1 END) as peak_productivity_sessions,
        COUNT(CASE WHEN productivity_zone = 'High Productivity Zone' THEN 1 END) as high_productivity_sessions,
        SUM(CASE WHEN productivity_zone = 'Peak Productivity Zone' THEN session_duration ELSE 0 END) as peak_productivity_time,
        
        -- Amazon operational excellence indicators
        CASE 
            WHEN SUM(session_duration) >= 480 AND COUNT(CASE WHEN operational_impact LIKE 'Critical%' THEN 1 END) > 0
            THEN 'Operational Excellence - Critical Impact Full-Time'
            WHEN SUM(session_duration) >= 360 AND COUNT(CASE WHEN productivity_zone LIKE 'Peak%' THEN 1 END) > 0
            THEN 'Operational Excellence - Peak Productivity Aligned'
            WHEN COUNT(*) <= 2 AND AVG(session_duration) >= 240
            THEN 'Operational Excellence - Efficient Long Sessions'
            WHEN business_unit = 'Fulfillment Centers' AND COUNT(CASE WHEN shift_type = 'Early Shift' THEN 1 END) > 0
            THEN 'Operational Excellence - Fulfillment Optimization'
            ELSE 'Standard Operations'
        END as operational_excellence_level,
        
        -- Work pattern optimization for Amazon business needs
        CASE 
            WHEN business_unit = 'AWS' AND SUM(session_duration) >= 360 AND COUNT(*) <= 2
            THEN 'AWS Optimization: Deep work sessions + infrastructure focus + on-call readiness'
            WHEN business_unit = 'Fulfillment Centers' AND early_shift_sessions > 0
            THEN 'Fulfillment Optimization: Early operations + package processing + delivery preparation'
            WHEN business_unit = 'Prime Video' AND COUNT(*) >= 3 AND AVG(session_duration) >= 120
            THEN 'Prime Video Optimization: Creative collaboration + content review + production coordination'
            WHEN business_unit = 'Customer Service Center' AND customer_impact_sessions > 0
            THEN 'Customer Service Optimization: Customer availability + response times + service quality'
            ELSE 'Standard Optimization: Business unit best practices + efficiency focus'
        END as amazon_optimization_strategy
    FROM amazon_workforce_time_analysis
    GROUP BY emp_id, event_day, business_unit, work_location, amazon_level
),
amazon_leadership_principles_analysis AS (
    SELECT 
        adoa.*,
        
        -- Customer Obsession assessment
        CASE 
            WHEN business_unit IN ('Retail', 'Customer Service Center') AND customer_impact_sessions > 0
            THEN 'Customer Obsession: Direct customer impact + service alignment + customer-first scheduling'
            WHEN work_location = 'Fulfillment Center' AND total_time >= 480
            THEN 'Customer Obsession: Fulfillment excellence + delivery commitment + customer satisfaction enablement'
            WHEN business_unit = 'Prime Video' AND total_sessions >= 2
            THEN 'Customer Obsession: Content quality + viewer experience + entertainment value delivery'
            ELSE 'Customer Obsession: Indirect customer value + internal service + customer-centric mindset'
        END as customer_obsession_alignment,
        
        -- Ownership demonstration
        CASE 
            WHEN amazon_level LIKE '%Principal%' AND total_time >= 600
            THEN 'Ownership: Executive commitment + long-term thinking + strategic responsibility'
            WHEN amazon_level LIKE '%Manager%' AND operational_excellence_level LIKE 'Operational Excellence%'
            THEN 'Ownership: Management accountability + team results + operational responsibility'
            WHEN critical_operations_sessions > 0 AND total_time >= 360
            THEN 'Ownership: Critical operations ownership + reliability + commitment to excellence'
            WHEN total_sessions <= 2 AND avg_session_time >= 240
            THEN 'Ownership: Deep work commitment + sustained focus + quality delivery'
            ELSE 'Ownership: Standard accountability + reliable execution + continuous improvement'
        END as ownership_demonstration,
        
        -- Deliver Results evidence
        CASE 
            WHEN operational_excellence_level LIKE 'Operational Excellence%' AND total_time >= 420
            THEN 'Deliver Results: Exceptional performance + consistent execution + measurable impact'
            WHEN peak_productivity_sessions > 0 AND total_time >= 360
            THEN 'Deliver Results: Peak performance alignment + optimal productivity + quality outcomes'
            WHEN business_unit = 'AWS' AND high_impact_sessions > 0
            THEN 'Deliver Results: Infrastructure reliability + service excellence + customer success'
            WHEN business_unit = 'Fulfillment Centers' AND early_shift_sessions > 0
            THEN 'Deliver Results: Operational excellence + delivery performance + customer satisfaction'
            ELSE 'Deliver Results: Consistent performance + reliable output + steady contribution'
        END as deliver_results_evidence,
        
        -- Think Big potential
        CASE 
            WHEN amazon_level LIKE '%Principal%' AND total_sessions >= 3 AND avg_session_time >= 180
            THEN 'Think Big: Strategic leadership + innovation time + long-term vision + complex problem solving'
            WHEN business_unit = 'Alexa' AND total_time >= 360
            THEN 'Think Big: Innovation focus + future technology + market transformation + bold solutions'
            WHEN business_unit = 'AWS' AND operational_excellence_level LIKE 'Operational Excellence%'
            THEN 'Think Big: Cloud infrastructure + scalable solutions + industry transformation + global impact'
            WHEN work_location = 'Seattle HQ' AND total_time >= 480
            THEN 'Think Big: Corporate strategy + company-wide impact + transformational initiatives'
            ELSE 'Think Big: Incremental innovation + process improvement + growth mindset + future focus'
        END as think_big_potential,
        
        -- Bias for Action demonstration
        CASE 
            WHEN total_sessions >= 4 AND avg_session_time BETWEEN 60 AND 180
            THEN 'Bias for Action: Rapid execution + quick decisions + agile response + fast iteration'
            WHEN critical_operations_sessions > 0 AND first_entry <= 480
            THEN 'Bias for Action: Early execution + proactive approach + immediate response + urgency mindset'
            WHEN business_unit = 'Logistics' AND total_time >= 420
            THEN 'Bias for Action: Operational speed + delivery excellence + rapid problem resolution'
            WHEN peak_productivity_sessions > 0
            THEN 'Bias for Action: Optimal timing + efficient execution + productivity maximization'
            ELSE 'Bias for Action: Standard execution + reliable response + steady progress + consistent action'
        END as bias_for_action_demonstration
    FROM amazon_daily_operations_analysis adoa
)
SELECT 
    event_day as day,
    emp_id,
    total_time,
    business_unit,
    work_location,
    amazon_level,
    operational_excellence_level,
    amazon_optimization_strategy,
    customer_obsession_alignment,
    ownership_demonstration,
    deliver_results_evidence,
    think_big_potential,
    bias_for_action_demonstration,
    
    -- Operational metrics
    total_sessions,
    avg_session_time,
    first_entry,
    last_exit,
    
    -- Shift and productivity analysis
    day_shift_sessions,
    peak_productivity_sessions,
    peak_productivity_time,
    critical_operations_sessions,
    high_impact_sessions,
    customer_impact_sessions
FROM amazon_leadership_principles_analysis
ORDER BY 
    CASE business_unit
        WHEN 'AWS' THEN 1
        WHEN 'Fulfillment Centers' THEN 2
        WHEN 'Prime Video' THEN 3
        WHEN 'Alexa' THEN 4
        WHEN 'Retail' THEN 5
        WHEN 'Logistics' THEN 6
        WHEN 'Advertising' THEN 7
        ELSE 8
    END,
    total_time DESC,
    day,
    emp_id;
```

#### 2. **Real-time Workforce Intelligence and Operational Dashboard**
```sql
-- "Implement real-time workforce intelligence system with predictive analytics and operational alerts"

WITH real_time_workforce_monitoring AS (
    SELECT 
        emp_id,
        event_day,
        in_time,
        out_time,
        out_time - in_time as session_duration,
        
        -- Real-time analysis timestamp
        CURRENT_TIMESTAMP as analysis_timestamp,
        
        -- Live operational status simulation
        CASE 
            WHEN CURRENT_TIME BETWEEN TIME('06:00:00') AND TIME('10:00:00') THEN 'Morning Operations'
            WHEN CURRENT_TIME BETWEEN TIME('10:00:00') AND TIME('14:00:00') THEN 'Peak Operations'
            WHEN CURRENT_TIME BETWEEN TIME('14:00:00') AND TIME('18:00:00') THEN 'Afternoon Operations'
            WHEN CURRENT_TIME BETWEEN TIME('18:00:00') AND TIME('22:00:00') THEN 'Evening Operations'
            ELSE 'Night Operations'
        END as current_operational_period,
        
        -- Capacity utilization simulation
        CASE 
            WHEN emp_id % 7 = 0 THEN 'At Capacity'
            WHEN emp_id % 7 = 1 THEN 'Near Capacity'
            WHEN emp_id % 7 = 2 THEN 'Optimal Utilization'
            WHEN emp_id % 7 = 3 THEN 'Under Utilized'
            WHEN emp_id % 7 = 4 THEN 'Peak Performance'
            WHEN emp_id % 7 = 5 THEN 'Flexible Capacity'
            ELSE 'Standard Utilization'
        END as capacity_status,
        
        -- Performance velocity indicators
        CASE 
            WHEN out_time - in_time >= 480 THEN 'Extended Performance'
            WHEN out_time - in_time >= 360 THEN 'High Performance'
            WHEN out_time - in_time >= 240 THEN 'Standard Performance'
            WHEN out_time - in_time >= 120 THEN 'Focused Performance'
            ELSE 'Short Burst Performance'
        END as performance_velocity,
        
        -- Real-time alerts simulation
        CASE 
            WHEN out_time - in_time > 720 THEN 'ALERT: Extended work session - wellness check needed'
            WHEN in_time <= 240 THEN 'ALERT: Very early start - schedule verification required'
            WHEN out_time >= 1320 THEN 'ALERT: Late night work - work-life balance concern'
            WHEN out_time - in_time <= 30 THEN 'ALERT: Very short session - productivity concern'
            ELSE 'NORMAL: Standard work pattern detected'
        END as real_time_alert,
        
        -- Predictive patterns
        CASE 
            WHEN out_time - in_time >= 420 AND in_time <= 480 THEN 'Pattern: High commitment early starter'
            WHEN out_time - in_time >= 300 AND in_time BETWEEN 480 AND 720 THEN 'Pattern: Consistent professional'
            WHEN out_time - in_time <= 180 AND in_time >= 720 THEN 'Pattern: Flexible short-burst worker'
            ELSE 'Pattern: Variable work style'
        END as predictive_pattern,
        
        -- Team coordination indicators
        CASE 
            WHEN in_time BETWEEN 480 AND 600 THEN 'High Collaboration Window'
            WHEN in_time BETWEEN 600 AND 840 THEN 'Standard Collaboration Window'
            WHEN in_time BETWEEN 840 AND 1080 THEN 'Limited Collaboration Window'
            ELSE 'Minimal Collaboration Window'
        END as collaboration_window
    FROM Employees
),
real_time_analytics_dashboard AS (
    SELECT 
        emp_id,
        event_day,
        current_operational_period,
        
        -- Core performance metrics
        COUNT(*) as total_sessions,
        SUM(session_duration) as total_time,
        AVG(session_duration) as avg_session_duration,
        MAX(session_duration) as longest_session,
        MIN(session_duration) as shortest_session,
        
        -- Real-time status distribution
        COUNT(CASE WHEN capacity_status = 'At Capacity' THEN 1 END) as at_capacity_sessions,
        COUNT(CASE WHEN capacity_status = 'Peak Performance' THEN 1 END) as peak_performance_sessions,
        COUNT(CASE WHEN capacity_status = 'Under Utilized' THEN 1 END) as under_utilized_sessions,
        
        -- Performance velocity analysis
        COUNT(CASE WHEN performance_velocity = 'Extended Performance' THEN 1 END) as extended_performance_sessions,
        COUNT(CASE WHEN performance_velocity = 'High Performance' THEN 1 END) as high_performance_sessions,
        COUNT(CASE WHEN performance_velocity = 'Focused Performance' THEN 1 END) as focused_performance_sessions,
        
        -- Alert aggregation
        COUNT(CASE WHEN real_time_alert LIKE 'ALERT:%' THEN 1 END) as alert_count,
        COUNT(CASE WHEN real_time_alert = 'NORMAL: Standard work pattern detected' THEN 1 END) as normal_sessions,
        
        -- Collaboration analysis
        COUNT(CASE WHEN collaboration_window = 'High Collaboration Window' THEN 1 END) as high_collaboration_sessions,
        COUNT(CASE WHEN collaboration_window = 'Standard Collaboration Window' THEN 1 END) as standard_collaboration_sessions,
        
        -- Predictive insights aggregation
        COUNT(CASE WHEN predictive_pattern LIKE '%High commitment%' THEN 1 END) as high_commitment_patterns,
        COUNT(CASE WHEN predictive_pattern LIKE '%Consistent professional%' THEN 1 END) as consistent_patterns,
        COUNT(CASE WHEN predictive_pattern LIKE '%Flexible%' THEN 1 END) as flexible_patterns,
        
        -- Real-time performance scoring
        (COUNT(CASE WHEN capacity_status = 'Peak Performance' THEN 1 END) * 30) +
        (COUNT(CASE WHEN performance_velocity = 'High Performance' THEN 1 END) * 25) +
        (COUNT(CASE WHEN collaboration_window = 'High Collaboration Window' THEN 1 END) * 20) +
        (COUNT(CASE WHEN real_time_alert = 'NORMAL: Standard work pattern detected' THEN 1 END) * 15) +
        (SUM(session_duration) / 60) as real_time_performance_score,
        
        -- Operational efficiency assessment
        CASE 
            WHEN AVG(session_duration) >= 360 AND COUNT(CASE WHEN real_time_alert LIKE 'ALERT:%' THEN 1 END) = 0
            THEN 'Exceptional Efficiency'
            WHEN AVG(session_duration) >= 240 AND COUNT(CASE WHEN capacity_status = 'Peak Performance' THEN 1 END) > 0
            THEN 'High Efficiency'
            WHEN SUM(session_duration) >= 360 AND COUNT(*) <= 3
            THEN 'Good Efficiency'
            WHEN COUNT(CASE WHEN real_time_alert LIKE 'ALERT:%' THEN 1 END) > 0
            THEN 'Efficiency Concerns'
            ELSE 'Standard Efficiency'
        END as operational_efficiency
    FROM real_time_workforce_monitoring
    GROUP BY emp_id, event_day, current_operational_period
),
predictive_workforce_intelligence AS (
    SELECT 
        rtad.*,
        
        -- Predictive performance modeling
        CASE 
            WHEN real_time_performance_score >= 100 AND operational_efficiency = 'Exceptional Efficiency'
            THEN 'Prediction: Sustained high performance + leadership potential + retention priority'
            WHEN high_performance_sessions >= 2 AND alert_count = 0
            THEN 'Prediction: Consistent high output + reliable performance + promotion candidate'
            WHEN flexible_patterns > 0 AND high_collaboration_sessions > 0
            THEN 'Prediction: Adaptive worker + team player + cross-functional potential'
            WHEN alert_count > 0 AND under_utilized_sessions > 0
            THEN 'Prediction: Performance intervention needed + support required + optimization opportunity'
            ELSE 'Prediction: Stable performance + standard development + routine support'
        END as performance_prediction,
        
        -- Real-time intervention strategies
        CASE 
            WHEN alert_count >= 2
            THEN 'IMMEDIATE INTERVENTION: Multiple alerts active + urgent manager discussion + wellness assessment'
            WHEN under_utilized_sessions > peak_performance_sessions
            THEN 'PRIORITY INTERVENTION: Capacity optimization + workload rebalancing + engagement assessment'
            WHEN extended_performance_sessions > 0 AND alert_count > 0
            THEN 'SCHEDULED INTERVENTION: Work-life balance discussion + schedule optimization + health check'
            WHEN high_commitment_patterns > 0 AND operational_efficiency = 'Exceptional Efficiency'
            THEN 'RECOGNITION INTERVENTION: Performance acknowledgment + career discussion + advancement opportunity'
            ELSE 'STANDARD MONITORING: Continue observation + periodic check-in + support availability'
        END as intervention_strategy,
        
        -- Dynamic resource allocation
        CASE 
            WHEN real_time_performance_score >= 100 AND high_collaboration_sessions >= 2
            THEN 'Resource Allocation: High-value projects + leadership roles + cross-team collaboration + strategic initiatives'
            WHEN peak_performance_sessions >= 2 AND consistent_patterns > 0
            THEN 'Resource Allocation: Complex assignments + skill development + increased responsibility + mentoring opportunities'
            WHEN flexible_patterns > 0 AND standard_collaboration_sessions >= 2
            THEN 'Resource Allocation: Adaptive projects + team coordination + flexible assignments + collaborative work'
            WHEN alert_count = 0 AND operational_efficiency = 'Good Efficiency'
            THEN 'Resource Allocation: Standard projects + steady workload + development opportunities + team participation'
            ELSE 'Resource Allocation: Supportive assignments + basic workload + skill building + close monitoring'
        END as resource_allocation_strategy,
        
        -- Productivity optimization recommendations
        CASE 
            WHEN avg_session_duration >= 360 AND high_collaboration_sessions >= 1
            THEN 'Optimization: Maintain deep work blocks + preserve collaboration windows + optimize peak hours'
            WHEN total_sessions >= 4 AND focused_performance_sessions >= 2
            THEN 'Optimization: Consolidate sessions + reduce fragmentation + create focus blocks + minimize interruptions'
            WHEN collaboration_window = 'Limited Collaboration Window' AND total_time >= 300
            THEN 'Optimization: Expand collaboration hours + team alignment + communication improvement + schedule coordination'
            WHEN alert_count > 0
            THEN 'Optimization: Address alert conditions + schedule adjustment + workload balancing + support enhancement'
            ELSE 'Optimization: Fine-tune current approach + efficiency gains + minor adjustments + performance enhancement'
        END as productivity_optimization,
        
        -- Team integration and collaboration enhancement
        CASE 
            WHEN high_collaboration_sessions >= total_sessions * 0.7
            THEN 'Team Integration: Collaboration champion + team coordination + cross-functional leadership + knowledge sharing'
            WHEN standard_collaboration_sessions >= total_sessions * 0.5
            THEN 'Team Integration: Strong team player + reliable collaboration + consistent availability + supportive member'
            WHEN flexible_patterns > 0 AND total_time >= 300
            THEN 'Team Integration: Adaptive collaborator + flexible support + specialized contribution + situational availability'
            WHEN collaboration_window = 'Minimal Collaboration Window'
            THEN 'Team Integration: Individual contributor + specialized work + asynchronous collaboration + independent execution'
            ELSE 'Team Integration: Standard collaboration + team participation + regular availability + collaborative contribution'
        END as team_integration_strategy
    FROM real_time_analytics_dashboard rtad
)
SELECT 
    event_day as day,
    emp_id,
    total_time,
    current_operational_period,
    operational_efficiency,
    real_time_performance_score,
    performance_prediction,
    intervention_strategy,
    resource_allocation_strategy,
    productivity_optimization,
    team_integration_strategy,
    
    -- Real-time metrics
    total_sessions,
    avg_session_duration,
    alert_count,
    
    -- Performance distribution
    peak_performance_sessions,
    high_performance_sessions,
    focused_performance_sessions,
    
    -- Collaboration analysis
    high_collaboration_sessions,
    standard_collaboration_sessions,
    
    -- Capacity analysis
    at_capacity_sessions,
    under_utilized_sessions,
    
    -- Pattern recognition
    high_commitment_patterns,
    consistent_patterns,
    flexible_patterns
FROM predictive_workforce_intelligence
ORDER BY 
    real_time_performance_score DESC,
    total_time DESC,
    day,
    emp_id;
```

I'll create just one more brief extension and then move to the next questions to maintain momentum:

#### 3. **Global Workforce Compliance and Regulatory Analytics**
```sql
-- "Enterprise compliance system for global workforce regulations and labor law adherence"

WITH global_compliance_analysis AS (
    SELECT 
        emp_id,
        event_day,
        SUM(out_time - in_time) as total_time,
        COUNT(*) as sessions_count,
        MIN(in_time) as earliest_entry,
        MAX(out_time) as latest_exit,
        
        -- Compliance framework simulation
        CASE 
            WHEN emp_id % 4 = 0 THEN 'US Labor Law'
            WHEN emp_id % 4 = 1 THEN 'EU Working Time Directive'
            WHEN emp_id % 4 = 2 THEN 'UK Employment Rights'
            ELSE 'Global Standards'
        END as regulatory_framework,
        
        -- Compliance checks
        CASE 
            WHEN SUM(out_time - in_time) > 480 THEN 'Overtime Required'
            WHEN COUNT(*) > 3 THEN 'Break Compliance Review'
            WHEN MAX(out_time) - MIN(in_time) > 720 THEN 'Extended Day Review'
            ELSE 'Compliant'
        END as compliance_status
    FROM Employees
    GROUP BY emp_id, event_day
)
SELECT 
    event_day as day,
    emp_id,
    total_time,
    regulatory_framework,
    compliance_status,
    sessions_count,
    earliest_entry,
    latest_exit
FROM global_compliance_analysis
ORDER BY total_time DESC;
```

## 🔗 Related LeetCode Questions

1. **#1731 - The Number of Employees Which Report to Each Employee** (Management hierarchy analysis)
2. **#1729 - Find Followers Count** (Basic aggregation patterns)
3. **#1757 - Recyclable and Low Fat Products** (Simple filtering logic)
4. **#1789 - Primary Department for Each Employee** (Employee data analysis)
5. **#1978 - Employees Whose Manager Left the Company** (Complex employee relationships)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Time Calculation**: out_time - in_time for session duration
2. **GROUP BY Aggregation**: Group by employee and day for multi-session handling
3. **SUM Function**: Aggregate multiple sessions per employee per day
4. **Multiple Sessions**: Handle employees with multiple entries/exits per day

### 🚀 **Amazon Interview Tips**
1. **Explain time calculation**: "Each row is one session, subtract in_time from out_time"
2. **Discuss aggregation**: "GROUP BY emp_id and event_day sums all sessions per employee per day"
3. **Address business logic**: "Employees can enter/exit multiple times, need to sum all sessions"
4. **Consider data integrity**: "Guaranteed no overlapping sessions and in_time < out_time"

### 🔧 **Common Patterns**
- Time difference calculation (out_time - in_time)
- GROUP BY multiple columns for multi-dimensional aggregation
- SUM for accumulating session durations
- Multi-session handling within single day

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting GROUP BY** (would calculate individual sessions, not daily totals)
2. **Wrong grouping columns** (missing event_day would sum across all days)
3. **Time unit confusion** (assuming hours vs minutes)
4. **Not handling multiple sessions** (showing each session instead of daily total)

### 🔍 **Performance Considerations**
- INDEX on (emp_id, event_day) for efficient grouping
- Consider partitioning by date for large time-series data
- SUM operations are fast with proper indexing
- Time arithmetic is computationally efficient

### 🎯 **Amazon Leadership Principles Applied**
- **Operational Excellence**: Accurate workforce analytics enable efficient operations management
- **Customer Obsession**: Understanding work patterns helps optimize customer service delivery
- **Ownership**: Comprehensive time tracking enables accountability and performance management
- **Deliver Results**: Workforce optimization drives productivity and business outcomes

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate average daily work hours per employee**
2. **Find employees working overtime (>8 hours) and calculate overtime hours**
3. **Identify peak office hours by counting overlapping sessions**
4. **Calculate total break time between sessions for each employee**

Remember: Workforce time analytics are essential for Amazon's operations across fulfillment centers, customer service, corporate offices, and ensuring compliance with labor regulations globally!