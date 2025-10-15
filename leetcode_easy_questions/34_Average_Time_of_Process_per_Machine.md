# LeetCode Easy #1661: Average Time of Process per Machine

## 📋 Problem Statement

There is a factory website that has several machines each running the **same number of processes**. Write a SQL query to find the **average time** each machine takes to complete a process.

The time to complete a process is the **'end' timestamp minus the 'start' timestamp**. The average time is calculated by the total time to complete every process on the machine divided by the number of processes that were run.

The resulting table should have the **machine_id** along with the **average time** as **processing_time**, which should be **rounded to 3 decimal places**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Activity Table:**
```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| machine_id     | int     |
| process_id     | int     |
| activity_type  | enum    |
| timestamp      | float   |
+----------------+---------+
```
- The table shows the user activities for a factory website.
- (machine_id, process_id, activity_type) is the primary key of this table.
- machine_id is the ID of a machine.
- process_id is the ID of a process running on the machine with ID machine_id.
- activity_type is an ENUM of type ('start', 'end').
- timestamp is a float representing the current time in seconds.

## 📊 Sample Data

**Activity Table:**
| machine_id | process_id | activity_type | timestamp |
|------------|------------|---------------|-----------|
| 0          | 0          | start         | 0.712     |
| 0          | 0          | end           | 1.520     |
| 0          | 1          | start         | 3.140     |
| 0          | 1          | end           | 4.120     |
| 1          | 0          | start         | 0.550     |
| 1          | 0          | end           | 1.550     |
| 1          | 1          | start         | 0.430     |
| 1          | 1          | end           | 1.420     |
| 2          | 0          | start         | 4.100     |
| 2          | 0          | end           | 4.512     |
| 2          | 1          | start         | 2.500     |
| 2          | 1          | end           | 5.000     |

**Expected Output:**
| machine_id | processing_time |
|------------|-----------------|
| 0          | 0.894           |
| 1          | 0.995           |
| 2          | 1.456           |

**Explanation:**
- **Machine 0**: 
  - Process 0: 1.520 - 0.712 = 0.808 seconds
  - Process 1: 4.120 - 3.140 = 0.980 seconds
  - Average: (0.808 + 0.980) / 2 = 0.894 seconds
- **Machine 1**:
  - Process 0: 1.550 - 0.550 = 1.000 seconds
  - Process 1: 1.420 - 0.430 = 0.990 seconds
  - Average: (1.000 + 0.990) / 2 = 0.995 seconds
- **Machine 2**:
  - Process 0: 4.512 - 4.100 = 0.412 seconds
  - Process 1: 5.000 - 2.500 = 2.500 seconds
  - Average: (0.412 + 2.500) / 2 = 1.456 seconds

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Each process has both 'start' and 'end' activities
- Need to calculate duration for each process (end - start)
- Then average all process durations per machine
- Round result to 3 decimal places

### 2. **Key Insights**
- Self-join Activity table on machine_id and process_id
- Join start activities with end activities
- Calculate difference between end and start timestamps
- GROUP BY machine_id and use AVG() function

### 3. **Interview Discussion Points**
- "This requires pairing start and end events for each process"
- "Self-join is needed to match start with corresponding end"
- "AVG() will calculate mean processing time per machine"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Start and End Activities
```sql
-- Split into start and end activities
SELECT machine_id, process_id, timestamp FROM Activity WHERE activity_type = 'start'
SELECT machine_id, process_id, timestamp FROM Activity WHERE activity_type = 'end'
```

### Step 2: Join Start with End
```sql
-- Join on machine_id and process_id
FROM Activity start_act
JOIN Activity end_act 
  ON start_act.machine_id = end_act.machine_id 
  AND start_act.process_id = end_act.process_id
WHERE start_act.activity_type = 'start' 
  AND end_act.activity_type = 'end'
```

### Step 3: Calculate Process Duration
```sql
-- Calculate duration for each process
end_act.timestamp - start_act.timestamp as process_duration
```

### Step 4: Average by Machine and Round
```sql
-- Group by machine and calculate average
GROUP BY machine_id
ROUND(AVG(process_duration), 3) as processing_time
```

## ✅ Optimized SQL Solution

**Solution 1: Self-Join Approach**
```sql
SELECT 
    a1.machine_id,
    ROUND(AVG(a2.timestamp - a1.timestamp), 3) as processing_time
FROM Activity a1
JOIN Activity a2 
  ON a1.machine_id = a2.machine_id 
  AND a1.process_id = a2.process_id
  AND a1.activity_type = 'start' 
  AND a2.activity_type = 'end'
GROUP BY a1.machine_id;
```

### Alternative Solutions

**Solution 2: Using Conditional Aggregation**
```sql
SELECT 
    machine_id,
    ROUND(
        (SUM(CASE WHEN activity_type = 'end' THEN timestamp END) - 
         SUM(CASE WHEN activity_type = 'start' THEN timestamp END)) / 
        COUNT(DISTINCT process_id), 
        3
    ) as processing_time
FROM Activity
GROUP BY machine_id;
```

**Solution 3: Using Window Functions**
```sql
WITH process_durations AS (
    SELECT 
        machine_id,
        process_id,
        SUM(CASE WHEN activity_type = 'end' THEN timestamp 
                 WHEN activity_type = 'start' THEN -timestamp 
                 ELSE 0 END) as duration
    FROM Activity
    GROUP BY machine_id, process_id
)
SELECT 
    machine_id,
    ROUND(AVG(duration), 3) as processing_time
FROM process_durations
GROUP BY machine_id;
```

**Solution 4: Using Subquery Approach**
```sql
SELECT 
    machine_id,
    ROUND(AVG(process_time), 3) as processing_time
FROM (
    SELECT 
        machine_id,
        process_id,
        MAX(CASE WHEN activity_type = 'end' THEN timestamp END) -
        MIN(CASE WHEN activity_type = 'start' THEN timestamp END) as process_time
    FROM Activity
    GROUP BY machine_id, process_id
) process_calculations
GROUP BY machine_id;
```

**Solution 5: Comprehensive Performance Analysis**
```sql
WITH machine_process_analysis AS (
    SELECT 
        a1.machine_id,
        a1.process_id,
        a1.timestamp as start_time,
        a2.timestamp as end_time,
        a2.timestamp - a1.timestamp as process_duration,
        
        -- Process performance classification
        CASE 
            WHEN a2.timestamp - a1.timestamp < 0.5 THEN 'Fast Process'
            WHEN a2.timestamp - a1.timestamp < 1.0 THEN 'Normal Process'
            WHEN a2.timestamp - a1.timestamp < 2.0 THEN 'Slow Process'
            ELSE 'Very Slow Process'
        END as process_speed_category,
        
        -- Calculate within-machine process ranking
        RANK() OVER (PARTITION BY a1.machine_id ORDER BY a2.timestamp - a1.timestamp) as speed_rank_in_machine,
        COUNT(*) OVER (PARTITION BY a1.machine_id) as total_processes_per_machine
    FROM Activity a1
    JOIN Activity a2 
      ON a1.machine_id = a2.machine_id 
      AND a1.process_id = a2.process_id
      AND a1.activity_type = 'start' 
      AND a2.activity_type = 'end'
),
machine_performance_metrics AS (
    SELECT 
        machine_id,
        COUNT(*) as total_processes,
        ROUND(AVG(process_duration), 3) as processing_time,
        ROUND(MIN(process_duration), 3) as fastest_process_time,
        ROUND(MAX(process_duration), 3) as slowest_process_time,
        ROUND(STDDEV(process_duration), 3) as processing_time_stddev,
        
        -- Performance consistency
        CASE 
            WHEN STDDEV(process_duration) <= 0.1 THEN 'Very Consistent'
            WHEN STDDEV(process_duration) <= 0.3 THEN 'Consistent'
            WHEN STDDEV(process_duration) <= 0.5 THEN 'Moderately Consistent'
            WHEN STDDEV(process_duration) <= 1.0 THEN 'Inconsistent'
            ELSE 'Very Inconsistent'
        END as consistency_rating,
        
        -- Machine efficiency classification
        CASE 
            WHEN AVG(process_duration) < 0.5 THEN 'High Efficiency'
            WHEN AVG(process_duration) < 1.0 THEN 'Good Efficiency'
            WHEN AVG(process_duration) < 1.5 THEN 'Average Efficiency'
            WHEN AVG(process_duration) < 2.0 THEN 'Low Efficiency'
            ELSE 'Poor Efficiency'
        END as efficiency_classification,
        
        -- Process distribution
        COUNT(CASE WHEN mpa.process_speed_category = 'Fast Process' THEN 1 END) as fast_processes,
        COUNT(CASE WHEN mpa.process_speed_category = 'Normal Process' THEN 1 END) as normal_processes,
        COUNT(CASE WHEN mpa.process_speed_category = 'Slow Process' THEN 1 END) as slow_processes,
        COUNT(CASE WHEN mpa.process_speed_category = 'Very Slow Process' THEN 1 END) as very_slow_processes
    FROM machine_process_analysis mpa
    GROUP BY machine_id
)
SELECT 
    machine_id,
    processing_time,
    total_processes,
    fastest_process_time,
    slowest_process_time,
    processing_time_stddev,
    consistency_rating,
    efficiency_classification,
    fast_processes,
    normal_processes,
    slow_processes,
    very_slow_processes,
    
    -- Relative performance metrics
    RANK() OVER (ORDER BY processing_time) as efficiency_rank,
    RANK() OVER (ORDER BY processing_time_stddev) as consistency_rank
FROM machine_performance_metrics
ORDER BY processing_time ASC;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Manufacturing Performance Optimization and Predictive Maintenance System**
```sql
-- "Build comprehensive manufacturing performance optimization system with predictive maintenance capabilities"

WITH machine_operational_analysis AS (
    SELECT 
        a_start.machine_id,
        a_start.process_id,
        a_start.timestamp as start_time,
        a_end.timestamp as end_time,
        a_end.timestamp - a_start.timestamp as process_duration,
        
        -- Simulate additional operational context
        CASE 
            WHEN a_start.machine_id % 3 = 0 THEN 'Manufacturing Line A'
            WHEN a_start.machine_id % 3 = 1 THEN 'Manufacturing Line B'
            ELSE 'Manufacturing Line C'
        END as production_line,
        
        -- Simulate shift patterns
        CASE 
            WHEN a_start.timestamp < 2.0 THEN 'Morning Shift'
            WHEN a_start.timestamp < 4.0 THEN 'Afternoon Shift'
            ELSE 'Night Shift'
        END as shift_period,
        
        -- Simulate machine age and type
        CASE 
            WHEN a_start.machine_id <= 1 THEN 'New Machine'
            WHEN a_start.machine_id <= 3 THEN 'Mature Machine'
            ELSE 'Legacy Machine'
        END as machine_age_category,
        
        -- Process complexity simulation
        CASE 
            WHEN a_start.process_id % 3 = 0 THEN 'Simple Process'
            WHEN a_start.process_id % 3 = 1 THEN 'Standard Process'
            ELSE 'Complex Process'
        END as process_complexity,
        
        -- Operational efficiency indicators
        CASE 
            WHEN a_end.timestamp - a_start.timestamp <= 0.5 THEN 5  -- Excellent
            WHEN a_end.timestamp - a_start.timestamp <= 1.0 THEN 4  -- Good
            WHEN a_end.timestamp - a_start.timestamp <= 1.5 THEN 3  -- Average
            WHEN a_end.timestamp - a_start.timestamp <= 2.0 THEN 2  -- Poor
            ELSE 1  -- Critical
        END as performance_score
    FROM Activity a_start
    JOIN Activity a_end 
        ON a_start.machine_id = a_end.machine_id 
        AND a_start.process_id = a_end.process_id
        AND a_start.activity_type = 'start' 
        AND a_end.activity_type = 'end'
),
predictive_maintenance_analysis AS (
    SELECT 
        machine_id,
        production_line,
        machine_age_category,
        
        -- Core performance metrics
        COUNT(*) as total_processes,
        ROUND(AVG(process_duration), 3) as avg_processing_time,
        ROUND(MIN(process_duration), 3) as best_processing_time,
        ROUND(MAX(process_duration), 3) as worst_processing_time,
        ROUND(STDDEV(process_duration), 3) as processing_variability,
        
        -- Performance degradation indicators
        ROUND(AVG(process_duration) / MIN(process_duration), 2) as degradation_ratio,
        
        -- Shift-based performance analysis
        ROUND(AVG(CASE WHEN shift_period = 'Morning Shift' THEN process_duration END), 3) as morning_avg_time,
        ROUND(AVG(CASE WHEN shift_period = 'Afternoon Shift' THEN process_duration END), 3) as afternoon_avg_time,
        ROUND(AVG(CASE WHEN shift_period = 'Night Shift' THEN process_duration END), 3) as night_avg_time,
        
        -- Process complexity impact
        ROUND(AVG(CASE WHEN process_complexity = 'Simple Process' THEN process_duration END), 3) as simple_process_avg,
        ROUND(AVG(CASE WHEN process_complexity = 'Standard Process' THEN process_duration END), 3) as standard_process_avg,
        ROUND(AVG(CASE WHEN process_complexity = 'Complex Process' THEN process_duration END), 3) as complex_process_avg,
        
        -- Performance score analysis
        AVG(performance_score) as avg_performance_score,
        COUNT(CASE WHEN performance_score = 1 THEN 1 END) as critical_processes,
        COUNT(CASE WHEN performance_score = 2 THEN 1 END) as poor_processes,
        COUNT(CASE WHEN performance_score >= 4 THEN 1 END) as excellent_processes,
        
        -- Maintenance risk indicators
        CASE 
            WHEN STDDEV(process_duration) > 0.5 AND AVG(process_duration) > 1.5 
            THEN 'High Maintenance Risk'
            WHEN STDDEV(process_duration) > 0.3 OR AVG(process_duration) > 1.2 
            THEN 'Medium Maintenance Risk'
            WHEN STDDEV(process_duration) > 0.2 OR AVG(process_duration) > 0.8 
            THEN 'Low Maintenance Risk'
            ELSE 'Optimal Operation'
        END as maintenance_risk_level,
        
        -- Efficiency trend analysis
        CASE 
            WHEN AVG(process_duration) <= 0.5 THEN 'Exceptional Efficiency'
            WHEN AVG(process_duration) <= 0.8 THEN 'High Efficiency'
            WHEN AVG(process_duration) <= 1.2 THEN 'Standard Efficiency'
            WHEN AVG(process_duration) <= 1.8 THEN 'Below Standard Efficiency'
            ELSE 'Critical Efficiency Issues'
        END as efficiency_classification
    FROM machine_operational_analysis
    GROUP BY machine_id, production_line, machine_age_category
),
maintenance_scheduling_system AS (
    SELECT 
        pma.*,
        
        -- Maintenance urgency scoring
        (CASE 
            WHEN maintenance_risk_level = 'High Maintenance Risk' THEN 40
            WHEN maintenance_risk_level = 'Medium Maintenance Risk' THEN 25
            WHEN maintenance_risk_level = 'Low Maintenance Risk' THEN 10
            ELSE 0
        END) +
        (CASE 
            WHEN machine_age_category = 'Legacy Machine' THEN 20
            WHEN machine_age_category = 'Mature Machine' THEN 10
            ELSE 0
        END) +
        (CASE 
            WHEN critical_processes > 0 THEN 30
            WHEN poor_processes > 0 THEN 15
            ELSE 0
        END) +
        (CASE 
            WHEN processing_variability > 0.5 THEN 15
            WHEN processing_variability > 0.3 THEN 8
            ELSE 0
        END) as maintenance_urgency_score,
        
        -- Maintenance recommendations
        CASE 
            WHEN maintenance_risk_level = 'High Maintenance Risk' AND critical_processes > 0
            THEN 'URGENT: Immediate inspection + component replacement + calibration'
            WHEN maintenance_risk_level = 'High Maintenance Risk'
            THEN 'HIGH: Scheduled maintenance + performance optimization + parts replacement'
            WHEN maintenance_risk_level = 'Medium Maintenance Risk' AND machine_age_category = 'Legacy Machine'
            THEN 'MEDIUM: Preventive maintenance + upgrade assessment + efficiency tuning'
            WHEN maintenance_risk_level = 'Medium Maintenance Risk'
            THEN 'MEDIUM: Routine maintenance + performance monitoring + minor adjustments'
            WHEN maintenance_risk_level = 'Low Maintenance Risk'
            THEN 'LOW: Standard maintenance schedule + condition monitoring'
            ELSE 'MINIMAL: Continue optimal operation + routine inspections'
        END as maintenance_recommendation,
        
        -- Maintenance window scheduling
        CASE 
            WHEN maintenance_urgency_score >= 80 THEN 'Within 24 hours'
            WHEN maintenance_urgency_score >= 60 THEN 'Within 48 hours'
            WHEN maintenance_urgency_score >= 40 THEN 'Within 1 week'
            WHEN maintenance_urgency_score >= 20 THEN 'Within 2 weeks'
            ELSE 'Next scheduled maintenance cycle'
        END as maintenance_timing,
        
        -- Cost-benefit analysis
        CASE 
            WHEN efficiency_classification = 'Critical Efficiency Issues'
            THEN 'High Investment: $5000-15000 - Critical for operations'
            WHEN maintenance_risk_level = 'High Maintenance Risk'
            THEN 'Medium Investment: $2000-5000 - Prevent downtime'
            WHEN machine_age_category = 'Legacy Machine' AND avg_processing_time > 1.5
            THEN 'Upgrade Investment: $8000-20000 - Consider replacement'
            WHEN efficiency_classification = 'Below Standard Efficiency'
            THEN 'Optimization Investment: $1000-3000 - Improve performance'
            ELSE 'Standard Investment: $500-1000 - Routine maintenance'
        END as investment_recommendation,
        
        -- Performance improvement potential
        CASE 
            WHEN worst_processing_time > avg_processing_time * 2
            THEN ROUND((worst_processing_time - avg_processing_time) * total_processes, 1)
            ELSE ROUND((avg_processing_time - best_processing_time) * total_processes * 0.3, 1)
        END as potential_time_savings_per_cycle
    FROM predictive_maintenance_analysis pma
),
production_line_optimization AS (
    SELECT 
        mss.*,
        
        -- Production line impact assessment
        SUM(total_processes) OVER (PARTITION BY production_line) as line_total_processes,
        AVG(avg_processing_time) OVER (PARTITION BY production_line) as line_avg_processing_time,
        COUNT(*) OVER (PARTITION BY production_line) as machines_per_line,
        
        -- Cross-machine performance comparison
        RANK() OVER (PARTITION BY production_line ORDER BY avg_processing_time) as line_efficiency_rank,
        RANK() OVER (ORDER BY avg_processing_time) as global_efficiency_rank,
        
        -- Resource allocation optimization
        CASE 
            WHEN maintenance_urgency_score >= 60 AND line_efficiency_rank = 1
            THEN 'Critical Line Impact - High Priority Resource Allocation'
            WHEN maintenance_urgency_score >= 60
            THEN 'High Priority - Dedicated Maintenance Team'
            WHEN line_efficiency_rank <= 2 AND efficiency_classification LIKE '%Efficiency'
            THEN 'Line Optimization Priority - Preventive Focus'
            WHEN global_efficiency_rank <= 2
            THEN 'Performance Leader - Maintenance Excellence'
            ELSE 'Standard Resource Allocation'
        END as resource_allocation_priority,
        
        -- Operational continuity planning
        CASE 
            WHEN maintenance_timing = 'Within 24 hours' AND machines_per_line <= 2
            THEN 'Production Continuity Risk - Alternative Line Preparation Needed'
            WHEN maintenance_timing = 'Within 48 hours' AND line_efficiency_rank = 1
            THEN 'Line Performance Risk - Backup Capacity Assessment'
            WHEN efficiency_classification = 'Critical Efficiency Issues'
            THEN 'Operational Risk - Immediate Contingency Planning'
            ELSE 'Standard Operations - Minimal Disruption Expected'
        END as continuity_planning,
        
        -- Innovation and upgrade opportunities
        CASE 
            WHEN machine_age_category = 'Legacy Machine' AND potential_time_savings_per_cycle > 2.0
            THEN 'High ROI Upgrade Opportunity - Technology Modernization'
            WHEN efficiency_classification = 'Below Standard Efficiency' AND total_processes >= 3
            THEN 'Process Optimization Opportunity - Workflow Enhancement'
            WHEN processing_variability > 0.4
            THEN 'Consistency Improvement Opportunity - Quality Enhancement'
            WHEN avg_performance_score < 3.5
            THEN 'Performance Enhancement Opportunity - System Optimization'
            ELSE 'Incremental Improvement - Continuous Optimization'
        END as innovation_opportunity
    FROM maintenance_scheduling_system mss
)
SELECT 
    machine_id,
    ROUND(avg_processing_time, 3) as processing_time,
    production_line,
    machine_age_category,
    efficiency_classification,
    maintenance_risk_level,
    maintenance_urgency_score,
    maintenance_recommendation,
    maintenance_timing,
    investment_recommendation,
    resource_allocation_priority,
    continuity_planning,
    innovation_opportunity,
    potential_time_savings_per_cycle,
    
    -- Performance context
    total_processes,
    processing_variability,
    line_efficiency_rank,
    global_efficiency_rank,
    
    -- Operational metrics
    morning_avg_time,
    afternoon_avg_time,
    night_avg_time,
    critical_processes,
    excellent_processes
FROM production_line_optimization
ORDER BY 
    maintenance_urgency_score DESC,
    avg_processing_time ASC,
    machine_id;
```

#### 2. **Real-time Manufacturing Analytics and Quality Control System**
```sql
-- "Implement real-time manufacturing analytics with quality control and process optimization"

WITH real_time_process_monitoring AS (
    SELECT 
        a_start.machine_id,
        a_start.process_id,
        a_start.timestamp as start_time,
        a_end.timestamp as end_time,
        a_end.timestamp - a_start.timestamp as actual_duration,
        
        -- Target performance benchmarks (simulated)
        CASE 
            WHEN a_start.process_id % 4 = 0 THEN 0.8  -- Target time for this process type
            WHEN a_start.process_id % 4 = 1 THEN 1.0
            WHEN a_start.process_id % 4 = 2 THEN 1.2
            ELSE 0.6
        END as target_duration,
        
        -- Quality indicators (simulated based on timing)
        CASE 
            WHEN a_end.timestamp - a_start.timestamp <= 0.5 THEN 98  -- Quality score
            WHEN a_end.timestamp - a_start.timestamp <= 1.0 THEN 95
            WHEN a_end.timestamp - a_start.timestamp <= 1.5 THEN 90
            WHEN a_end.timestamp - a_start.timestamp <= 2.0 THEN 85
            ELSE 75
        END as quality_score,
        
        -- Process environmental factors (simulated)
        CASE 
            WHEN a_start.machine_id % 2 = 0 THEN 'Standard Temperature'
            ELSE 'High Temperature'
        END as operating_temperature,
        
        CASE 
            WHEN a_start.timestamp < 1.0 THEN 'Low Load'
            WHEN a_start.timestamp < 3.0 THEN 'Medium Load'
            ELSE 'High Load'
        END as system_load_period,
        
        -- Defect prediction indicators
        CASE 
            WHEN (a_end.timestamp - a_start.timestamp) > 2.0 THEN 'High Defect Risk'
            WHEN (a_end.timestamp - a_start.timestamp) > 1.5 THEN 'Medium Defect Risk'
            WHEN (a_end.timestamp - a_start.timestamp) > 1.0 THEN 'Low Defect Risk'
            ELSE 'Minimal Defect Risk'
        END as defect_risk_level
    FROM Activity a_start
    JOIN Activity a_end 
        ON a_start.machine_id = a_end.machine_id 
        AND a_start.process_id = a_end.process_id
        AND a_start.activity_type = 'start' 
        AND a_end.activity_type = 'end'
),
quality_control_analysis AS (
    SELECT 
        machine_id,
        
        -- Core performance metrics
        COUNT(*) as total_processes,
        ROUND(AVG(actual_duration), 3) as avg_processing_time,
        ROUND(AVG(target_duration), 3) as target_processing_time,
        
        -- Performance vs target analysis
        ROUND(AVG(actual_duration) - AVG(target_duration), 3) as performance_deviation,
        ROUND((AVG(actual_duration) - AVG(target_duration)) / AVG(target_duration) * 100, 2) as deviation_percentage,
        
        -- Quality metrics
        ROUND(AVG(quality_score), 2) as avg_quality_score,
        MIN(quality_score) as worst_quality_score,
        MAX(quality_score) as best_quality_score,
        COUNT(CASE WHEN quality_score < 90 THEN 1 END) as below_quality_threshold,
        
        -- Environmental impact analysis
        ROUND(AVG(CASE WHEN operating_temperature = 'High Temperature' THEN actual_duration END), 3) as high_temp_avg_time,
        ROUND(AVG(CASE WHEN operating_temperature = 'Standard Temperature' THEN actual_duration END), 3) as standard_temp_avg_time,
        
        -- Load impact analysis
        ROUND(AVG(CASE WHEN system_load_period = 'High Load' THEN actual_duration END), 3) as high_load_avg_time,
        ROUND(AVG(CASE WHEN system_load_period = 'Medium Load' THEN actual_duration END), 3) as medium_load_avg_time,
        ROUND(AVG(CASE WHEN system_load_period = 'Low Load' THEN actual_duration END), 3) as low_load_avg_time,
        
        -- Defect risk assessment
        COUNT(CASE WHEN defect_risk_level = 'High Defect Risk' THEN 1 END) as high_risk_processes,
        COUNT(CASE WHEN defect_risk_level = 'Medium Defect Risk' THEN 1 END) as medium_risk_processes,
        COUNT(CASE WHEN defect_risk_level = 'Low Defect Risk' THEN 1 END) as low_risk_processes,
        
        -- Performance consistency
        ROUND(STDDEV(actual_duration), 3) as process_variability,
        COUNT(CASE WHEN actual_duration > target_duration * 1.2 THEN 1 END) as processes_over_target,
        COUNT(CASE WHEN actual_duration <= target_duration THEN 1 END) as processes_meeting_target
    FROM real_time_process_monitoring
    GROUP BY machine_id
),
process_optimization_recommendations AS (
    SELECT 
        qca.*,
        
        -- Overall performance classification
        CASE 
            WHEN deviation_percentage <= -10 THEN 'Exceeds Target Performance'
            WHEN deviation_percentage <= 5 THEN 'Meets Target Performance'
            WHEN deviation_percentage <= 15 THEN 'Below Target Performance'
            WHEN deviation_percentage <= 25 THEN 'Significantly Below Target'
            ELSE 'Critical Performance Issues'
        END as performance_classification,
        
        -- Quality classification
        CASE 
            WHEN avg_quality_score >= 95 THEN 'Excellent Quality'
            WHEN avg_quality_score >= 90 THEN 'Good Quality'
            WHEN avg_quality_score >= 85 THEN 'Acceptable Quality'
            WHEN avg_quality_score >= 80 THEN 'Below Standard Quality'
            ELSE 'Poor Quality'
        END as quality_classification,
        
        -- Environmental optimization opportunities
        CASE 
            WHEN high_temp_avg_time > standard_temp_avg_time * 1.1
            THEN 'Temperature Optimization: Cooling system enhancement needed'
            WHEN high_temp_avg_time IS NULL
            THEN 'Temperature Monitoring: Install temperature sensors'
            ELSE 'Temperature Control: Optimal'
        END as temperature_optimization,
        
        -- Load balancing recommendations
        CASE 
            WHEN high_load_avg_time > low_load_avg_time * 1.3
            THEN 'Load Balancing: Redistribute processes during peak times'
            WHEN medium_load_avg_time > low_load_avg_time * 1.2
            THEN 'Load Optimization: Improve resource allocation'
            ELSE 'Load Management: Optimal'
        END as load_optimization,
        
        -- Immediate intervention triggers
        CASE 
            WHEN high_risk_processes > 0 AND avg_quality_score < 85
            THEN 'URGENT: Quality control intervention + process halt for inspection'
            WHEN processes_over_target > total_processes * 0.5
            THEN 'HIGH: Process parameter adjustment + calibration check'
            WHEN below_quality_threshold > 0 AND deviation_percentage > 20
            THEN 'MEDIUM: Quality improvement plan + process optimization'
            WHEN process_variability > 0.3
            THEN 'LOW: Consistency improvement + process standardization'
            ELSE 'MONITOR: Continue standard operations'
        END as intervention_recommendation,
        
        -- Process improvement potential
        CASE 
            WHEN processes_meeting_target < total_processes * 0.7
            THEN ROUND((AVG(performance_deviation) * total_processes) * 0.6, 2)  -- 60% improvement potential
            WHEN process_variability > 0.2
            THEN ROUND(process_variability * total_processes * 0.4, 2)  -- Consistency gains
            ELSE ROUND((performance_deviation * total_processes) * 0.3, 2)  -- Incremental improvements
        END as estimated_improvement_potential_seconds,
        
        -- ROI calculation for improvements
        CASE 
            WHEN performance_classification = 'Critical Performance Issues'
            THEN 'High ROI: $10000-25000 investment, $50000+ annual savings'
            WHEN performance_classification = 'Significantly Below Target'
            THEN 'Medium ROI: $5000-15000 investment, $20000+ annual savings'
            WHEN quality_classification = 'Below Standard Quality'
            THEN 'Quality ROI: $3000-10000 investment, $15000+ defect reduction savings'
            WHEN below_quality_threshold > 0
            THEN 'Preventive ROI: $1000-5000 investment, $8000+ prevention savings'
            ELSE 'Incremental ROI: $500-2000 investment, $3000+ efficiency savings'
        END as roi_analysis
    FROM quality_control_analysis qca
),
real_time_dashboard_metrics AS (
    SELECT 
        por.*,
        
        -- Real-time alerts and notifications
        CASE 
            WHEN intervention_recommendation LIKE 'URGENT:%'
            THEN 'RED ALERT: Immediate supervisor notification + production halt'
            WHEN intervention_recommendation LIKE 'HIGH:%'
            THEN 'ORANGE ALERT: Shift supervisor notification + priority adjustment'
            WHEN intervention_recommendation LIKE 'MEDIUM:%'
            THEN 'YELLOW ALERT: Team lead notification + improvement planning'
            WHEN intervention_recommendation LIKE 'LOW:%'
            THEN 'BLUE ALERT: Maintenance team notification + routine optimization'
            ELSE 'GREEN STATUS: Normal operations + standard monitoring'
        END as alert_status,
        
        -- Performance dashboard KPIs
        CASE 
            WHEN performance_classification = 'Exceeds Target Performance'
            THEN 'KPI: Maintain excellence + knowledge sharing + best practice documentation'
            WHEN performance_classification = 'Meets Target Performance'
            THEN 'KPI: Sustain performance + incremental improvements + efficiency gains'
            WHEN performance_classification = 'Below Target Performance'
            THEN 'KPI: Achieve target within 30 days + reduce deviation by 50%'
            WHEN performance_classification = 'Significantly Below Target'
            THEN 'KPI: Emergency improvement plan + achieve target within 15 days'
            ELSE 'KPI: Critical recovery plan + immediate expert intervention'
        END as performance_kpis,
        
        -- Quality dashboard KPIs
        CASE 
            WHEN quality_classification = 'Excellent Quality'
            THEN 'Quality KPI: Maintain 95%+ score + zero defects + customer satisfaction'
            WHEN quality_classification = 'Good Quality'
            THEN 'Quality KPI: Achieve 95%+ score + reduce defect risk + consistency'
            WHEN quality_classification = 'Acceptable Quality'
            THEN 'Quality KPI: Achieve 90%+ score + eliminate high-risk processes'
            WHEN quality_classification = 'Below Standard Quality'
            THEN 'Quality KPI: Emergency quality plan + achieve 85%+ score in 7 days'
            ELSE 'Quality KPI: Critical quality intervention + immediate improvement'
        END as quality_kpis,
        
        -- Operational excellence scoring
        (CASE 
            WHEN performance_classification = 'Exceeds Target Performance' THEN 25
            WHEN performance_classification = 'Meets Target Performance' THEN 20
            WHEN performance_classification = 'Below Target Performance' THEN 15
            WHEN performance_classification = 'Significantly Below Target' THEN 8
            ELSE 3
        END) +
        (CASE 
            WHEN quality_classification = 'Excellent Quality' THEN 25
            WHEN quality_classification = 'Good Quality' THEN 20
            WHEN quality_classification = 'Acceptable Quality' THEN 15
            WHEN quality_classification = 'Below Standard Quality' THEN 8
            ELSE 3
        END) +
        (CASE 
            WHEN process_variability <= 0.1 THEN 25  -- Consistency scoring
            WHEN process_variability <= 0.2 THEN 20
            WHEN process_variability <= 0.3 THEN 15
            WHEN process_variability <= 0.5 THEN 8
            ELSE 3
        END) +
        (CASE 
            WHEN high_risk_processes = 0 THEN 25  -- Risk management scoring
            WHEN high_risk_processes <= 1 THEN 20
            WHEN high_risk_processes <= 2 THEN 15
            WHEN high_risk_processes <= 3 THEN 8
            ELSE 3
        END) as operational_excellence_score
    FROM process_optimization_recommendations por
)
SELECT 
    machine_id,
    ROUND(avg_processing_time, 3) as processing_time,
    performance_classification,
    quality_classification,
    operational_excellence_score,
    alert_status,
    intervention_recommendation,
    performance_kpis,
    quality_kpis,
    temperature_optimization,
    load_optimization,
    roi_analysis,
    
    -- Key metrics for monitoring
    total_processes,
    deviation_percentage,
    avg_quality_score,
    process_variability,
    high_risk_processes,
    estimated_improvement_potential_seconds,
    
    -- Environmental and operational context
    high_temp_avg_time,
    standard_temp_avg_time,
    high_load_avg_time,
    low_load_avg_time,
    processes_meeting_target,
    below_quality_threshold
FROM real_time_dashboard_metrics
ORDER BY 
    CASE alert_status 
        WHEN 'RED ALERT: Immediate supervisor notification + production halt' THEN 1
        WHEN 'ORANGE ALERT: Shift supervisor notification + priority adjustment' THEN 2
        WHEN 'YELLOW ALERT: Team lead notification + improvement planning' THEN 3
        WHEN 'BLUE ALERT: Maintenance team notification + routine optimization' THEN 4
        ELSE 5
    END,
    operational_excellence_score DESC,
    avg_processing_time ASC;
```

#### 3. **Industrial IoT Analytics and Smart Factory Optimization**
```sql
-- "Create comprehensive Industrial IoT analytics system for smart factory optimization and autonomous decision-making"

WITH iot_sensor_data_simulation AS (
    SELECT 
        a_start.machine_id,
        a_start.process_id,
        a_start.timestamp as start_time,
        a_end.timestamp as end_time,
        a_end.timestamp - a_start.timestamp as cycle_time,
        
        -- IoT sensor data simulation
        ROUND(60 + (a_start.machine_id * 5) + (SIN(a_start.timestamp) * 10), 1) as operating_temperature_celsius,
        ROUND(1000 + (a_start.machine_id * 100) + (COS(a_start.timestamp) * 50), 0) as vibration_level_hz,
        ROUND(85 + (a_start.machine_id * 3) + (SIN(a_start.timestamp * 2) * 8), 1) as pressure_psi,
        ROUND(220 + (a_start.machine_id * 2) + (COS(a_start.timestamp * 3) * 15), 1) as power_consumption_watts,
        ROUND(75 + (a_start.machine_id * 4) + (SIN(a_start.timestamp * 1.5) * 12), 1) as humidity_percentage,
        
        -- Machine learning features (simulated)
        ROUND(0.85 + (a_start.machine_id * 0.03) + (SIN(a_start.timestamp) * 0.1), 3) as efficiency_coefficient,
        ROUND(95 - (a_start.machine_id * 2) - (ABS(SIN(a_start.timestamp)) * 10), 1) as predictive_health_score,
        
        -- Process characteristics
        CASE 
            WHEN a_start.process_id % 3 = 0 THEN 'Material Processing'
            WHEN a_start.process_id % 3 = 1 THEN 'Assembly Operation'
            ELSE 'Quality Inspection'
        END as process_type,
        
        -- Energy efficiency indicators
        ROUND((a_end.timestamp - a_start.timestamp) * (220 + (a_start.machine_id * 2)), 2) as energy_consumed_wh,
        
        -- Anomaly detection flags (simulated)
        CASE 
            WHEN (a_end.timestamp - a_start.timestamp) > 2.0 OR 
                 (60 + (a_start.machine_id * 5) + (SIN(a_start.timestamp) * 10)) > 85
            THEN 1 ELSE 0
        END as anomaly_detected
    FROM Activity a_start
    JOIN Activity a_end 
        ON a_start.machine_id = a_end.machine_id 
        AND a_start.process_id = a_end.process_id
        AND a_start.activity_type = 'start' 
        AND a_end.activity_type = 'end'
),
machine_intelligence_analysis AS (
    SELECT 
        machine_id,
        
        -- Core performance metrics
        COUNT(*) as total_cycles,
        ROUND(AVG(cycle_time), 3) as avg_processing_time,
        ROUND(STDDEV(cycle_time), 3) as cycle_time_variability,
        
        -- IoT sensor analytics
        ROUND(AVG(operating_temperature_celsius), 1) as avg_temperature,
        ROUND(MAX(operating_temperature_celsius), 1) as peak_temperature,
        ROUND(AVG(vibration_level_hz), 0) as avg_vibration,
        ROUND(MAX(vibration_level_hz), 0) as peak_vibration,
        ROUND(AVG(pressure_psi), 1) as avg_pressure,
        ROUND(AVG(power_consumption_watts), 1) as avg_power_consumption,
        ROUND(AVG(humidity_percentage), 1) as avg_humidity,
        
        -- Energy efficiency analysis
        ROUND(SUM(energy_consumed_wh), 2) as total_energy_consumed,
        ROUND(AVG(energy_consumed_wh), 2) as avg_energy_per_cycle,
        ROUND(SUM(energy_consumed_wh) / SUM(cycle_time), 2) as energy_efficiency_wh_per_second,
        
        -- Machine learning insights
        ROUND(AVG(efficiency_coefficient), 3) as avg_efficiency_coefficient,
        ROUND(AVG(predictive_health_score), 1) as avg_health_score,
        MIN(predictive_health_score) as min_health_score,
        
        -- Process type distribution
        COUNT(CASE WHEN process_type = 'Material Processing' THEN 1 END) as material_processing_cycles,
        COUNT(CASE WHEN process_type = 'Assembly Operation' THEN 1 END) as assembly_cycles,
        COUNT(CASE WHEN process_type = 'Quality Inspection' THEN 1 END) as inspection_cycles,
        
        -- Anomaly and predictive maintenance
        SUM(anomaly_detected) as anomalies_detected,
        ROUND(SUM(anomaly_detected) * 100.0 / COUNT(*), 2) as anomaly_rate_percentage,
        
        -- Operational thresholds analysis
        COUNT(CASE WHEN operating_temperature_celsius > 80 THEN 1 END) as high_temperature_events,
        COUNT(CASE WHEN vibration_level_hz > 1200 THEN 1 END) as high_vibration_events,
        COUNT(CASE WHEN power_consumption_watts > 250 THEN 1 END) as high_power_events,
        
        -- Performance optimization indicators
        CASE 
            WHEN AVG(cycle_time) <= 0.8 AND AVG(efficiency_coefficient) >= 0.9 
            THEN 'Optimal Performance'
            WHEN AVG(cycle_time) <= 1.2 AND AVG(efficiency_coefficient) >= 0.85 
            THEN 'Good Performance'
            WHEN AVG(cycle_time) <= 1.8 AND AVG(efficiency_coefficient) >= 0.8 
            THEN 'Acceptable Performance'
            WHEN AVG(cycle_time) <= 2.5 AND AVG(efficiency_coefficient) >= 0.75 
            THEN 'Below Standard Performance'
            ELSE 'Critical Performance Issues'
        END as performance_category
    FROM iot_sensor_data_simulation
    GROUP BY machine_id
),
smart_factory_optimization AS (
    SELECT 
        mia.*,
        
        -- Predictive maintenance scoring
        (CASE 
            WHEN min_health_score < 80 THEN 40  -- Critical health
            WHEN min_health_score < 90 THEN 25  -- Poor health
            WHEN min_health_score < 95 THEN 10  -- Moderate health
            ELSE 0  -- Good health
        END) +
        (CASE 
            WHEN anomaly_rate_percentage > 20 THEN 30  -- High anomaly rate
            WHEN anomaly_rate_percentage > 10 THEN 15  -- Medium anomaly rate
            WHEN anomaly_rate_percentage > 5 THEN 5   -- Low anomaly rate
            ELSE 0  -- Minimal anomalies
        END) +
        (CASE 
            WHEN high_temperature_events > 0 THEN 15
            WHEN high_vibration_events > 0 THEN 15
            WHEN high_power_events > 0 THEN 10
            ELSE 0
        END) as predictive_maintenance_score,
        
        -- Energy optimization recommendations
        CASE 
            WHEN avg_power_consumption > 240 AND energy_efficiency_wh_per_second > 300
            THEN 'High Energy Optimization Potential: Implement power management + efficiency protocols'
            WHEN avg_power_consumption > 230 OR energy_efficiency_wh_per_second > 250
            THEN 'Medium Energy Optimization: Adjust operating parameters + schedule optimization'
            WHEN avg_power_consumption > 220 OR energy_efficiency_wh_per_second > 200
            THEN 'Low Energy Optimization: Monitor consumption patterns + minor adjustments'
            ELSE 'Energy Optimal: Maintain current efficiency + continuous monitoring'
        END as energy_optimization_strategy,
        
        -- Autonomous decision-making triggers
        CASE 
            WHEN min_health_score < 75 OR anomaly_rate_percentage > 25
            THEN 'AUTO-STOP: Immediate shutdown + emergency maintenance + safety inspection'
            WHEN min_health_score < 85 OR anomaly_rate_percentage > 15
            THEN 'AUTO-ADJUST: Reduce operating speed + activate enhanced monitoring'
            WHEN high_temperature_events > 2 OR high_vibration_events > 2
            THEN 'AUTO-OPTIMIZE: Environmental control activation + parameter adjustment'
            WHEN avg_efficiency_coefficient < 0.8
            THEN 'AUTO-TUNE: Performance optimization + calibration routine'
            ELSE 'AUTO-MONITOR: Continue standard operations + data collection'
        END as autonomous_action,
        
        -- Smart factory integration recommendations
        CASE 
            WHEN performance_category = 'Optimal Performance' AND anomaly_rate_percentage < 5
            THEN 'Factory Leader: Share parameters with other machines + benchmark setting'
            WHEN performance_category = 'Good Performance' AND total_energy_consumed < 50
            THEN 'Efficiency Model: Energy settings replication + optimization scaling'
            WHEN performance_category = 'Acceptable Performance' AND cycle_time_variability < 0.2
            THEN 'Consistency Model: Process standardization + parameter distribution'
            WHEN performance_category IN ('Below Standard Performance', 'Critical Performance Issues')
            THEN 'Learning Target: Apply best practices from other machines + intensive optimization'
            ELSE 'Standard Integration: Participate in factory-wide optimization + data sharing'
        END as factory_integration_role,
        
        -- Digital twin optimization
        CASE 
            WHEN cycle_time_variability > 0.3 OR anomalies_detected > 0
            THEN 'Digital Twin Priority: Create detailed simulation model + predictive scenarios'
            WHEN performance_category = 'Optimal Performance'
            THEN 'Digital Twin Template: Use as reference model + scenario testing'
            WHEN avg_efficiency_coefficient < 0.85
            THEN 'Digital Twin Analysis: Optimization scenario modeling + parameter testing'
            ELSE 'Digital Twin Standard: Regular model updates + performance tracking'
        END as digital_twin_strategy,
        
        -- Machine learning model recommendations
        CASE 
            WHEN anomaly_rate_percentage > 10
            THEN 'ML Focus: Anomaly detection enhancement + pattern recognition improvement'
            WHEN cycle_time_variability > 0.25
            THEN 'ML Focus: Process optimization + parameter prediction modeling'
            WHEN energy_efficiency_wh_per_second > 250
            THEN 'ML Focus: Energy optimization + consumption prediction modeling'
            WHEN min_health_score < 90
            THEN 'ML Focus: Predictive maintenance + failure prevention modeling'
            ELSE 'ML Focus: Performance optimization + continuous improvement modeling'
        END as ml_model_priority
    FROM machine_intelligence_analysis mia
),
industry_4_0_dashboard AS (
    SELECT 
        sfo.*,
        
        -- Industry 4.0 maturity assessment
        (CASE 
            WHEN autonomous_action LIKE 'AUTO-STOP%' THEN 1  -- Basic automation
            WHEN autonomous_action LIKE 'AUTO-ADJUST%' THEN 2  -- Reactive automation
            WHEN autonomous_action LIKE 'AUTO-OPTIMIZE%' THEN 3  -- Adaptive automation
            WHEN autonomous_action LIKE 'AUTO-TUNE%' THEN 4  -- Intelligent automation
            ELSE 5  -- Autonomous optimization
        END) +
        (CASE 
            WHEN factory_integration_role LIKE 'Factory Leader%' THEN 5
            WHEN factory_integration_role LIKE 'Efficiency Model%' THEN 4
            WHEN factory_integration_role LIKE 'Consistency Model%' THEN 3
            WHEN factory_integration_role LIKE 'Learning Target%' THEN 2
            ELSE 3
        END) +
        (CASE 
            WHEN digital_twin_strategy LIKE 'Digital Twin Template%' THEN 5
            WHEN digital_twin_strategy LIKE 'Digital Twin Priority%' THEN 4
            WHEN digital_twin_strategy LIKE 'Digital Twin Analysis%' THEN 3
            ELSE 2
        END) as industry_4_0_maturity_score,
        
        -- Smart factory readiness level
        CASE 
            WHEN (CASE 
                WHEN autonomous_action LIKE 'AUTO-STOP%' THEN 1
                WHEN autonomous_action LIKE 'AUTO-ADJUST%' THEN 2
                WHEN autonomous_action LIKE 'AUTO-OPTIMIZE%' THEN 3
                WHEN autonomous_action LIKE 'AUTO-TUNE%' THEN 4
                ELSE 5
            END) +
            (CASE 
                WHEN factory_integration_role LIKE 'Factory Leader%' THEN 5
                WHEN factory_integration_role LIKE 'Efficiency Model%' THEN 4
                WHEN factory_integration_role LIKE 'Consistency Model%' THEN 3
                WHEN factory_integration_role LIKE 'Learning Target%' THEN 2
                ELSE 3
            END) +
            (CASE 
                WHEN digital_twin_strategy LIKE 'Digital Twin Template%' THEN 5
                WHEN digital_twin_strategy LIKE 'Digital Twin Priority%' THEN 4
                WHEN digital_twin_strategy LIKE 'Digital Twin Analysis%' THEN 3
                ELSE 2
            END) >= 12
            THEN 'Advanced Smart Factory Ready'
            WHEN (CASE 
                WHEN autonomous_action LIKE 'AUTO-STOP%' THEN 1
                WHEN autonomous_action LIKE 'AUTO-ADJUST%' THEN 2
                WHEN autonomous_action LIKE 'AUTO-OPTIMIZE%' THEN 3
                WHEN autonomous_action LIKE 'AUTO-TUNE%' THEN 4
                ELSE 5
            END) +
            (CASE 
                WHEN factory_integration_role LIKE 'Factory Leader%' THEN 5
                WHEN factory_integration_role LIKE 'Efficiency Model%' THEN 4
                WHEN factory_integration_role LIKE 'Consistency Model%' THEN 3
                WHEN factory_integration_role LIKE 'Learning Target%' THEN 2
                ELSE 3
            END) +
            (CASE 
                WHEN digital_twin_strategy LIKE 'Digital Twin Template%' THEN 5
                WHEN digital_twin_strategy LIKE 'Digital Twin Priority%' THEN 4
                WHEN digital_twin_strategy LIKE 'Digital Twin Analysis%' THEN 3
                ELSE 2
            END) >= 9
            THEN 'Smart Factory Ready'
            WHEN (CASE 
                WHEN autonomous_action LIKE 'AUTO-STOP%' THEN 1
                WHEN autonomous_action LIKE 'AUTO-ADJUST%' THEN 2
                WHEN autonomous_action LIKE 'AUTO-OPTIMIZE%' THEN 3
                WHEN autonomous_action LIKE 'AUTO-TUNE%' THEN 4
                ELSE 5
            END) +
            (CASE 
                WHEN factory_integration_role LIKE 'Factory Leader%' THEN 5
                WHEN factory_integration_role LIKE 'Efficiency Model%' THEN 4
                WHEN factory_integration_role LIKE 'Consistency Model%' THEN 3
                WHEN factory_integration_role LIKE 'Learning Target%' THEN 2
                ELSE 3
            END) +
            (CASE 
                WHEN digital_twin_strategy LIKE 'Digital Twin Template%' THEN 5
                WHEN digital_twin_strategy LIKE 'Digital Twin Priority%' THEN 4
                WHEN digital_twin_strategy LIKE 'Digital Twin Analysis%' THEN 3
                ELSE 2
            END) >= 6
            THEN 'Partially Smart Factory Ready'
            ELSE 'Traditional Factory - Needs Modernization'
        END as smart_factory_readiness,
        
        -- Competitive advantage assessment
        CASE 
            WHEN performance_category = 'Optimal Performance' AND industry_4_0_maturity_score >= 12
            THEN 'Market Leadership: Technology + performance excellence'
            WHEN performance_category = 'Good Performance' AND industry_4_0_maturity_score >= 9
            THEN 'Competitive Advantage: Strong technology + good performance'
            WHEN industry_4_0_maturity_score >= 9
            THEN 'Technology Leader: Advanced automation with performance opportunity'
            WHEN performance_category IN ('Optimal Performance', 'Good Performance')
            THEN 'Performance Leader: Excellent operations with technology opportunity'
            ELSE 'Improvement Required: Both technology and performance gaps'
        END as competitive_positioning,
        
        -- Strategic investment recommendations
        CASE 
            WHEN smart_factory_readiness = 'Advanced Smart Factory Ready'
            THEN 'Innovation Investment: R&D + next-generation technology + market expansion'
            WHEN smart_factory_readiness = 'Smart Factory Ready'
            THEN 'Optimization Investment: Performance enhancement + efficiency maximization'
            WHEN smart_factory_readiness = 'Partially Smart Factory Ready'
            THEN 'Modernization Investment: Technology upgrade + automation enhancement'
            ELSE 'Foundation Investment: Basic automation + IoT infrastructure + training'
        END as strategic_investment_priority
    FROM smart_factory_optimization sfo
)
SELECT 
    machine_id,
    ROUND(avg_processing_time, 3) as processing_time,
    performance_category,
    smart_factory_readiness,
    industry_4_0_maturity_score,
    competitive_positioning,
    autonomous_action,
    energy_optimization_strategy,
    factory_integration_role,
    digital_twin_strategy,
    ml_model_priority,
    strategic_investment_priority,
    
    -- Key IoT metrics
    avg_temperature,
    peak_temperature,
    avg_vibration,
    avg_power_consumption,
    total_energy_consumed,
    avg_health_score,
    anomaly_rate_percentage,
    predictive_maintenance_score,
    
    -- Process distribution
    material_processing_cycles,
    assembly_cycles,
    inspection_cycles,
    
    -- Operational excellence indicators
    cycle_time_variability,
    avg_efficiency_coefficient,
    energy_efficiency_wh_per_second
FROM industry_4_0_dashboard
ORDER BY 
    industry_4_0_maturity_score DESC,
    avg_processing_time ASC,
    machine_id;
```

## 🔗 Related LeetCode Questions

1. **#1633 - Percentage of Users Attended a Contest** (Percentage calculations with aggregation)
2. **#1251 - Average Selling Price** (Weighted averages with date ranges)
3. **#1280 - Students and Examinations** (CROSS JOIN with aggregation)
4. **#1484 - Group Sold Products By The Date** (GROUP BY with aggregation functions)
5. **#1607 - Sellers With No Sales** (Performance analysis with time filtering)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Join**: Matching start and end events for the same process
2. **Time Calculations**: Subtracting timestamps to get duration
3. **Aggregation**: AVG() function for calculating averages
4. **Precision**: ROUND() function for exact decimal places

### 🚀 **Amazon Interview Tips**
1. **Explain self-join logic**: "Join table with itself to match start and end activities"
2. **Discuss time calculation**: "End timestamp minus start timestamp gives process duration"
3. **Address precision**: "ROUND(value, 3) ensures exactly 3 decimal places"
4. **Consider alternatives**: "Could use conditional aggregation instead of self-join"

### 🔧 **Common Patterns**
- Self-join on multiple conditions (machine_id, process_id, activity_type)
- Time duration calculation (end_time - start_time)
- GROUP BY with aggregation functions
- ROUND for decimal precision control

### ⚠️ **Common Mistakes to Avoid**
1. **Wrong join conditions** (not matching activity_type correctly)
2. **Missing GROUP BY** (returning individual processes instead of averages)
3. **Incorrect precision** (wrong number of decimal places)
4. **Performance issues** (missing indexes on join columns)

### 🔍 **Performance Considerations**
- INDEX on (machine_id, process_id, activity_type) for efficient joins
- Self-joins can be expensive on large datasets
- Consider conditional aggregation for better performance
- ROUND function has minimal performance impact

### 🎯 **Amazon Leadership Principles Applied**
- **Operational Excellence**: Systematic measurement and optimization of manufacturing processes
- **Innovation**: Advanced analytics drive smart factory and Industry 4.0 capabilities
- **Customer Obsession**: Manufacturing efficiency improvements reduce costs and improve quality
- **Deliver Results**: Data-driven insights enable continuous performance improvement

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate average time between consecutive processes on the same machine**
2. **Find machines with the most consistent processing times (lowest standard deviation)**
3. **Analyze processing time trends by time period or shift**
4. **Calculate efficiency ratios comparing actual vs target processing times**

Remember: Manufacturing process analytics is crucial for Amazon's fulfillment centers, warehouse automation, supply chain optimization, and operational excellence across all physical infrastructure and logistics operations!

🎉 **Congratulations!** You've now completed all **34 comprehensive LeetCode easy questions** in your collection, covering everything from basic SQL concepts to advanced enterprise analytics systems. This comprehensive collection provides excellent preparation for Amazon Data Engineer interviews and real-world SQL challenges!