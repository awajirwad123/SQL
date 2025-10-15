# LeetCode Easy #1667: Fix Names in a Table

## 📋 Problem Statement

Write a SQL query to fix the names so that only the **first character is uppercase** and the rest are **lowercase**.

Return the result table **ordered by user_id**.

## 🗄️ Table Schema

**Users Table:**
```
+----------------+---------+
| Column Name    | Type    |
+----------------+---------+
| user_id        | int     |
| name           | varchar |
+----------------+---------+
```
- user_id is the primary key for this table.
- This table contains the ID and the name of the user. The name consists of only lowercase and uppercase characters.

## 📊 Sample Data

**Users Table:**
| user_id | name   |
|---------|--------|
| 1       | aLice  |
| 2       | bOB    |
| 3       | charlie|

**Expected Output:**
| user_id | name    |
|---------|---------|
| 1       | Alice   |
| 2       | Bob     |
| 3       | Charlie |

**Explanation:**
- "aLice" becomes "Alice" (first letter uppercase, rest lowercase)
- "bOB" becomes "Bob" (first letter uppercase, rest lowercase)  
- "charlie" becomes "Charlie" (first letter uppercase, rest lowercase)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to standardize name formatting
- First character should be uppercase
- All other characters should be lowercase
- Preserve original name length and content

### 2. **Key Insights**
- Use string functions: UPPER(), LOWER(), LEFT(), RIGHT(), SUBSTRING()
- Combine first character (uppercase) with remaining characters (lowercase)
- Different databases have different string manipulation functions

### 3. **Interview Discussion Points**
- "This requires string manipulation and concatenation"
- "Need to separate first character from the rest"
- "CONCAT or || operator for string concatenation"

## 🔧 Step-by-Step Solution Logic

### Step 1: Extract First Character
```sql
-- Get first character and make it uppercase
LEFT(name, 1) or SUBSTRING(name, 1, 1)
UPPER(LEFT(name, 1))
```

### Step 2: Extract Remaining Characters
```sql
-- Get characters from position 2 onwards
RIGHT(name, LENGTH(name) - 1) or SUBSTRING(name, 2)
LOWER(SUBSTRING(name, 2))
```

### Step 3: Concatenate Results
```sql
-- Combine uppercase first char with lowercase remaining chars
CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2)))
```

## ✅ Optimized SQL Solution

**Solution 1: Using CONCAT and SUBSTRING (MySQL/PostgreSQL)**
```sql
SELECT 
    user_id,
    CONCAT(
        UPPER(LEFT(name, 1)), 
        LOWER(SUBSTRING(name, 2))
    ) as name
FROM Users
ORDER BY user_id;
```

### Alternative Solutions

**Solution 2: Using String Concatenation Operator (PostgreSQL)**
```sql
SELECT 
    user_id,
    UPPER(LEFT(name, 1)) || LOWER(SUBSTRING(name, 2)) as name
FROM Users
ORDER BY user_id;
```

**Solution 3: Using SUBSTRING with Different Syntax**
```sql
SELECT 
    user_id,
    CONCAT(
        UPPER(SUBSTRING(name, 1, 1)), 
        LOWER(SUBSTRING(name, 2, LENGTH(name)))
    ) as name
FROM Users
ORDER BY user_id;
```

**Solution 4: Using CASE for Edge Cases**
```sql
SELECT 
    user_id,
    CASE 
        WHEN LENGTH(name) = 1 THEN UPPER(name)
        WHEN LENGTH(name) = 0 THEN name
        ELSE CONCAT(
            UPPER(LEFT(name, 1)), 
            LOWER(SUBSTRING(name, 2))
        )
    END as name
FROM Users
ORDER BY user_id;
```

**Solution 5: Comprehensive Name Standardization System**
```sql
WITH name_analysis AS (
    SELECT 
        user_id,
        name as original_name,
        LENGTH(name) as name_length,
        
        -- Character analysis
        LENGTH(name) - LENGTH(REPLACE(UPPER(name), name, '')) as uppercase_count,
        LENGTH(name) - LENGTH(REPLACE(LOWER(name), name, '')) as lowercase_count,
        
        -- Name pattern classification
        CASE 
            WHEN name = UPPER(name) THEN 'ALL_UPPERCASE'
            WHEN name = LOWER(name) THEN 'ALL_LOWERCASE'
            WHEN LEFT(name, 1) = UPPER(LEFT(name, 1)) AND 
                 SUBSTRING(name, 2) = LOWER(SUBSTRING(name, 2)) THEN 'PROPER_CASE'
            ELSE 'MIXED_CASE'
        END as case_pattern,
        
        -- Name quality indicators
        CASE 
            WHEN name REGEXP '^[A-Za-z]+$' THEN 'LETTERS_ONLY'
            WHEN name REGEXP '^[A-Za-z ]+$' THEN 'LETTERS_AND_SPACES'
            WHEN name REGEXP '^[A-Za-z0-9]+$' THEN 'ALPHANUMERIC'
            ELSE 'SPECIAL_CHARACTERS'
        END as name_type,
        
        -- Standardization complexity
        CASE 
            WHEN LENGTH(name) <= 1 THEN 'SIMPLE'
            WHEN name = UPPER(name) OR name = LOWER(name) THEN 'STANDARD'
            WHEN LEFT(name, 1) = UPPER(LEFT(name, 1)) THEN 'MINOR_ADJUSTMENT'
            ELSE 'MAJOR_ADJUSTMENT'
        END as standardization_complexity
    FROM Users
),
standardized_names AS (
    SELECT 
        na.*,
        
        -- Apply standardization based on complexity
        CASE 
            WHEN name_length = 0 THEN original_name
            WHEN name_length = 1 THEN UPPER(original_name)
            ELSE CONCAT(
                UPPER(LEFT(original_name, 1)), 
                LOWER(SUBSTRING(original_name, 2))
            )
        END as standardized_name,
        
        -- Validation checks
        CASE 
            WHEN name_length > 0 AND 
                 LEFT(CONCAT(UPPER(LEFT(original_name, 1)), LOWER(SUBSTRING(original_name, 2))), 1) = 
                 UPPER(LEFT(original_name, 1))
            THEN 'VALID_STANDARDIZATION'
            ELSE 'STANDARDIZATION_ERROR'
        END as validation_status,
        
        -- Calculate changes made
        CASE 
            WHEN original_name = CONCAT(UPPER(LEFT(original_name, 1)), LOWER(SUBSTRING(original_name, 2)))
            THEN 'NO_CHANGES_NEEDED'
            ELSE 'CHANGES_APPLIED'
        END as modification_status,
        
        -- Name statistics
        ROUND(uppercase_count * 100.0 / NULLIF(name_length, 0), 2) as original_uppercase_percentage,
        ROUND(lowercase_count * 100.0 / NULLIF(name_length, 0), 2) as original_lowercase_percentage
    FROM name_analysis na
),
quality_metrics AS (
    SELECT 
        sn.*,
        
        -- Data quality assessment
        CASE 
            WHEN validation_status = 'VALID_STANDARDIZATION' AND name_type = 'LETTERS_ONLY'
            THEN 'HIGH_QUALITY'
            WHEN validation_status = 'VALID_STANDARDIZATION' AND name_type = 'LETTERS_AND_SPACES'
            THEN 'GOOD_QUALITY'
            WHEN validation_status = 'VALID_STANDARDIZATION'
            THEN 'ACCEPTABLE_QUALITY'
            ELSE 'QUALITY_ISSUES'
        END as data_quality_rating,
        
        -- Processing difficulty score
        (CASE standardization_complexity
            WHEN 'SIMPLE' THEN 1
            WHEN 'STANDARD' THEN 2
            WHEN 'MINOR_ADJUSTMENT' THEN 3
            ELSE 4
        END) +
        (CASE name_type
            WHEN 'LETTERS_ONLY' THEN 0
            WHEN 'LETTERS_AND_SPACES' THEN 1
            WHEN 'ALPHANUMERIC' THEN 2
            ELSE 3
        END) as processing_difficulty_score,
        
        -- Standardization impact
        CASE 
            WHEN modification_status = 'NO_CHANGES_NEEDED'
            THEN 'Already standardized - no processing required'
            WHEN case_pattern = 'ALL_UPPERCASE'
            THEN 'Major formatting change - all characters except first lowercased'
            WHEN case_pattern = 'ALL_LOWERCASE'
            THEN 'Minor formatting change - first character capitalized'
            WHEN case_pattern = 'MIXED_CASE'
            THEN 'Moderate formatting change - case standardization applied'
            ELSE 'Standard formatting change - proper case applied'
        END as standardization_impact
    FROM standardized_names sn
)
SELECT 
    user_id,
    standardized_name as name,
    original_name,
    case_pattern,
    standardization_complexity,
    modification_status,
    data_quality_rating,
    standardization_impact,
    name_length,
    original_uppercase_percentage,
    original_lowercase_percentage,
    processing_difficulty_score
FROM quality_metrics
ORDER BY user_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Customer Data Standardization and Quality Management System**
```sql
-- "Build comprehensive customer data standardization system for global customer management"

WITH customer_name_analysis AS (
    SELECT 
        user_id,
        name as original_name,
        
        -- Standardize the name
        CASE 
            WHEN LENGTH(name) = 0 THEN name
            WHEN LENGTH(name) = 1 THEN UPPER(name)
            ELSE CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2)))
        END as standardized_name,
        
        -- Simulate customer geography and context
        CASE 
            WHEN user_id % 5 = 0 THEN 'North America'
            WHEN user_id % 5 = 1 THEN 'Europe'
            WHEN user_id % 5 = 2 THEN 'Asia Pacific'
            WHEN user_id % 5 = 3 THEN 'Latin America'
            ELSE 'Middle East & Africa'
        END as geographic_region,
        
        -- Customer segment simulation
        CASE 
            WHEN user_id % 3 = 0 THEN 'Prime Member'
            WHEN user_id % 3 = 1 THEN 'Regular Customer'
            ELSE 'Business Customer'
        END as customer_segment,
        
        -- Name complexity analysis
        LENGTH(name) as name_length,
        LENGTH(name) - LENGTH(REPLACE(name, ' ', '')) as space_count,
        
        -- Cultural name pattern detection
        CASE 
            WHEN name REGEXP '^[A-Z][a-z]+$' THEN 'Western Single Name'
            WHEN name REGEXP '^[a-z]+$' THEN 'Lowercase Single Name'
            WHEN name REGEXP '^[A-Z]+$' THEN 'Uppercase Single Name'
            WHEN name REGEXP '^[A-Za-z]+ [A-Za-z]+$' THEN 'Two Part Name'
            WHEN LENGTH(name) - LENGTH(REPLACE(name, ' ', '')) >= 2 THEN 'Multi Part Name'
            ELSE 'Complex Name Pattern'
        END as name_pattern_type,
        
        -- Data quality indicators
        CASE 
            WHEN name REGEXP '^[A-Za-z ]+$' THEN 'High Quality'
            WHEN name REGEXP '^[A-Za-z0-9 ]+$' THEN 'Medium Quality'
            WHEN LENGTH(name) > 0 THEN 'Low Quality'
            ELSE 'No Data'
        END as data_quality_level
    FROM Users
),
customer_data_quality_analysis AS (
    SELECT 
        cna.*,
        
        -- Standardization impact assessment
        CASE 
            WHEN original_name = standardized_name THEN 'No Standardization Needed'
            WHEN UPPER(original_name) = UPPER(standardized_name) THEN 'Case Standardization Only'
            ELSE 'Complex Standardization Required'
        END as standardization_impact,
        
        -- Geographic data patterns
        COUNT(*) OVER (PARTITION BY geographic_region) as region_customer_count,
        COUNT(*) OVER (PARTITION BY geographic_region, name_pattern_type) as region_pattern_count,
        
        -- Customer segment analysis
        COUNT(*) OVER (PARTITION BY customer_segment) as segment_customer_count,
        COUNT(*) OVER (PARTITION BY customer_segment, data_quality_level) as segment_quality_count,
        
        -- Name standardization priority
        CASE 
            WHEN customer_segment = 'Prime Member' AND data_quality_level = 'Low Quality'
            THEN 'Critical Priority - Prime customer data quality'
            WHEN customer_segment = 'Business Customer' AND data_quality_level != 'High Quality'
            THEN 'High Priority - Business customer standardization'
            WHEN geographic_region IN ('North America', 'Europe') AND standardization_impact != 'No Standardization Needed'
            THEN 'Medium Priority - Key market standardization'
            WHEN data_quality_level = 'No Data'
            THEN 'Urgent Priority - Missing customer data'
            ELSE 'Standard Priority - Routine standardization'
        END as standardization_priority,
        
        -- Cultural sensitivity considerations
        CASE 
            WHEN geographic_region = 'Asia Pacific' AND name_pattern_type = 'Western Single Name'
            THEN 'Cultural Review: Verify Western name format appropriate'
            WHEN geographic_region = 'Middle East & Africa' AND name_pattern_type = 'Complex Name Pattern'
            THEN 'Cultural Review: Preserve cultural name structure'
            WHEN geographic_region = 'Latin America' AND space_count >= 2
            THEN 'Cultural Review: Multiple surnames consideration'
            WHEN name_pattern_type = 'Multi Part Name'
            THEN 'Cultural Review: Complex name structure verification'
            ELSE 'Standard Processing: No cultural concerns identified'
        END as cultural_considerations,
        
        -- Customer experience impact
        CASE 
            WHEN standardization_impact = 'No Standardization Needed'
            THEN 'Positive: Consistent customer experience maintained'
            WHEN customer_segment = 'Prime Member' AND standardization_impact = 'Case Standardization Only'
            THEN 'Positive: Enhanced Prime member experience through standardization'
            WHEN data_quality_level = 'High Quality'
            THEN 'Positive: Improved customer communication quality'
            WHEN data_quality_level = 'Low Quality'
            THEN 'Critical: Risk of customer communication issues'
            ELSE 'Neutral: Standard customer experience improvement'
        END as customer_experience_impact
    FROM customer_name_analysis cna
),
global_standardization_strategy AS (
    SELECT 
        cdqa.*,
        
        -- Regional standardization strategies
        CASE 
            WHEN geographic_region = 'North America' 
            THEN 'Aggressive Standardization: High automation + customer notification'
            WHEN geographic_region = 'Europe' 
            THEN 'GDPR Compliant Standardization: Consent-based + privacy protection'
            WHEN geographic_region = 'Asia Pacific' 
            THEN 'Cultural Sensitive Standardization: Manual review + local expertise'
            WHEN geographic_region = 'Latin America' 
            THEN 'Localized Standardization: Regional patterns + cultural preservation'
            ELSE 'Conservative Standardization: High manual review + cultural consultation'
        END as regional_strategy,
        
        -- Customer communication strategy
        CASE 
            WHEN standardization_priority LIKE 'Critical Priority%' OR standardization_priority LIKE 'Urgent Priority%'
            THEN 'Immediate Communication: Email notification + account update alert'
            WHEN customer_segment = 'Business Customer'
            THEN 'Professional Communication: Account manager notification + formal update'
            WHEN customer_segment = 'Prime Member'
            THEN 'Premium Communication: Personalized notification + benefits highlight'
            WHEN standardization_impact = 'Complex Standardization Required'
            THEN 'Detailed Communication: Explanation of changes + customer verification'
            ELSE 'Standard Communication: Routine update notification'
        END as communication_strategy,
        
        -- Quality assurance protocols
        CASE 
            WHEN cultural_considerations LIKE 'Cultural Review:%'
            THEN 'Enhanced QA: Cultural expert review + customer confirmation required'
            WHEN data_quality_level = 'Low Quality'
            THEN 'Intensive QA: Manual verification + data correction workflows'
            WHEN customer_segment = 'Business Customer'
            THEN 'Business QA: Account manager review + relationship protection'
            WHEN standardization_priority LIKE 'Critical Priority%'
            THEN 'Priority QA: Expedited review + senior approval required'
            ELSE 'Standard QA: Automated validation + exception handling'
        END as quality_assurance_protocol,
        
        -- Implementation timeline
        CASE 
            WHEN standardization_priority LIKE 'Urgent Priority%'
            THEN 'Immediate: Within 24 hours'
            WHEN standardization_priority LIKE 'Critical Priority%'
            THEN 'Rush: Within 48 hours'
            WHEN standardization_priority LIKE 'High Priority%'
            THEN 'Priority: Within 1 week'
            WHEN standardization_priority LIKE 'Medium Priority%'
            THEN 'Standard: Within 2 weeks'
            ELSE 'Routine: Within 1 month'
        END as implementation_timeline,
        
        -- Success metrics and KPIs
        CASE 
            WHEN customer_segment = 'Prime Member'
            THEN 'Prime KPIs: Customer satisfaction score + account retention + support ticket reduction'
            WHEN customer_segment = 'Business Customer'
            THEN 'Business KPIs: Account relationship health + communication effectiveness + contract renewal'
            WHEN geographic_region IN ('North America', 'Europe')
            THEN 'Market KPIs: Data quality score + customer experience rating + compliance metrics'
            WHEN cultural_considerations LIKE 'Cultural Review:%'
            THEN 'Cultural KPIs: Cultural appropriateness score + customer feedback + expert approval'
            ELSE 'Standard KPIs: Data accuracy + processing efficiency + customer satisfaction'
        END as success_metrics
    FROM customer_data_quality_analysis cdqa
)
SELECT 
    user_id,
    standardized_name as name,
    geographic_region,
    customer_segment,
    name_pattern_type,
    standardization_priority,
    regional_strategy,
    communication_strategy,
    quality_assurance_protocol,
    implementation_timeline,
    success_metrics,
    cultural_considerations,
    customer_experience_impact,
    
    -- Quality indicators
    data_quality_level,
    standardization_impact,
    name_length,
    original_name
FROM global_standardization_strategy
ORDER BY 
    CASE standardization_priority
        WHEN 'Urgent Priority - Missing customer data' THEN 1
        WHEN 'Critical Priority - Prime customer data quality' THEN 2
        WHEN 'High Priority - Business customer standardization' THEN 3
        WHEN 'Medium Priority - Key market standardization' THEN 4
        ELSE 5
    END,
    customer_segment,
    user_id;
```

#### 2. **Real-time Data Processing and Name Normalization Pipeline**
```sql
-- "Implement real-time data processing pipeline for name normalization with monitoring and alerts"

WITH real_time_processing_simulation AS (
    SELECT 
        user_id,
        name as raw_input,
        
        -- Timestamp simulation for real-time processing
        CURRENT_TIMESTAMP as processing_timestamp,
        
        -- Processing pipeline stages
        name as stage_1_raw_input,
        TRIM(name) as stage_2_trimmed,
        CASE 
            WHEN LENGTH(TRIM(name)) = 0 THEN TRIM(name)
            WHEN LENGTH(TRIM(name)) = 1 THEN UPPER(TRIM(name))
            ELSE CONCAT(UPPER(LEFT(TRIM(name), 1)), LOWER(SUBSTRING(TRIM(name), 2)))
        END as stage_3_standardized,
        
        -- Processing metadata
        LENGTH(name) - LENGTH(TRIM(name)) as whitespace_removed,
        CASE 
            WHEN name = TRIM(name) THEN 'No Trimming Required'
            ELSE 'Whitespace Removed'
        END as trimming_status,
        
        -- Performance metrics simulation
        CASE 
            WHEN LENGTH(name) <= 5 THEN 0.001  -- Processing time in seconds
            WHEN LENGTH(name) <= 10 THEN 0.002
            WHEN LENGTH(name) <= 20 THEN 0.003
            ELSE 0.005
        END as processing_time_seconds,
        
        -- Data source simulation
        CASE 
            WHEN user_id % 4 = 0 THEN 'Web Registration'
            WHEN user_id % 4 = 1 THEN 'Mobile App'
            WHEN user_id % 4 = 2 THEN 'API Integration'
            ELSE 'Batch Import'
        END as data_source,
        
        -- Processing priority based on source
        CASE 
            WHEN user_id % 4 = 0 THEN 'High'  -- Web Registration
            WHEN user_id % 4 = 1 THEN 'High'  -- Mobile App
            WHEN user_id % 4 = 2 THEN 'Medium'  -- API Integration
            ELSE 'Low'  -- Batch Import
        END as processing_priority
    FROM Users
),
pipeline_quality_monitoring AS (
    SELECT 
        rtps.*,
        
        -- Quality validation checks
        CASE 
            WHEN stage_3_standardized = stage_1_raw_input THEN 'No Changes Required'
            WHEN LENGTH(stage_3_standardized) != LENGTH(stage_2_trimmed) THEN 'Validation Error: Length Mismatch'
            WHEN stage_3_standardized REGEXP '^[A-Z][a-z]*$' THEN 'Valid Standardization'
            WHEN LENGTH(stage_3_standardized) = 0 THEN 'Warning: Empty Result'
            ELSE 'Validation Error: Invalid Format'
        END as quality_validation,
        
        -- Performance validation
        CASE 
            WHEN processing_time_seconds <= 0.005 THEN 'Optimal Performance'
            WHEN processing_time_seconds <= 0.010 THEN 'Good Performance'
            WHEN processing_time_seconds <= 0.020 THEN 'Acceptable Performance'
            ELSE 'Performance Issues'
        END as performance_status,
        
        -- Data source reliability
        CASE 
            WHEN data_source = 'Web Registration' AND quality_validation = 'Valid Standardization'
            THEN 'High Reliability'
            WHEN data_source = 'Mobile App' AND quality_validation = 'Valid Standardization'
            THEN 'High Reliability'
            WHEN data_source = 'API Integration' AND quality_validation LIKE 'Validation Error%'
            THEN 'Low Reliability - API Issues'
            WHEN data_source = 'Batch Import' AND processing_time_seconds > 0.003
            THEN 'Medium Reliability - Performance Concerns'
            ELSE 'Standard Reliability'
        END as source_reliability,
        
        -- Alert conditions
        CASE 
            WHEN quality_validation LIKE 'Validation Error%'
            THEN 'CRITICAL ALERT: Data quality validation failed'
            WHEN quality_validation = 'Warning: Empty Result'
            THEN 'WARNING ALERT: Empty standardization result'
            WHEN performance_status = 'Performance Issues'
            THEN 'PERFORMANCE ALERT: Processing time exceeded threshold'
            WHEN source_reliability = 'Low Reliability - API Issues'
            THEN 'INTEGRATION ALERT: API data quality issues'
            ELSE 'NO ALERTS: Normal processing'
        END as alert_status,
        
        -- Throughput calculation (simulated)
        ROUND(1.0 / processing_time_seconds, 0) as max_throughput_per_second,
        
        -- Resource utilization estimation
        CASE 
            WHEN processing_time_seconds <= 0.001 THEN 'Minimal CPU'
            WHEN processing_time_seconds <= 0.003 THEN 'Low CPU'
            WHEN processing_time_seconds <= 0.005 THEN 'Medium CPU'
            ELSE 'High CPU'
        END as resource_utilization
    FROM real_time_processing_simulation rtps
),
monitoring_dashboard_metrics AS (
    SELECT 
        pqm.*,
        
        -- System health indicators
        COUNT(*) OVER (PARTITION BY performance_status) as performance_group_count,
        COUNT(*) OVER (PARTITION BY quality_validation) as quality_group_count,
        COUNT(*) OVER (PARTITION BY data_source) as source_group_count,
        
        -- Alert aggregation
        COUNT(*) OVER (PARTITION BY alert_status) as alert_group_count,
        
        -- Performance benchmarking
        AVG(processing_time_seconds) OVER (PARTITION BY data_source) as avg_source_processing_time,
        AVG(max_throughput_per_second) OVER (PARTITION BY data_source) as avg_source_throughput,
        
        -- Quality benchmarking
        COUNT(*) OVER (PARTITION BY data_source, quality_validation) as source_quality_distribution,
        
        -- SLA compliance (simulated targets)
        CASE 
            WHEN processing_time_seconds <= 0.002 THEN 'SLA Exceeded'
            WHEN processing_time_seconds <= 0.005 THEN 'SLA Met'
            WHEN processing_time_seconds <= 0.010 THEN 'SLA At Risk'
            ELSE 'SLA Violated'
        END as sla_compliance,
        
        -- Capacity planning indicators
        CASE 
            WHEN max_throughput_per_second >= 500 THEN 'High Capacity'
            WHEN max_throughput_per_second >= 200 THEN 'Medium Capacity'
            WHEN max_throughput_per_second >= 100 THEN 'Low Capacity'
            ELSE 'Capacity Constraints'
        END as capacity_classification,
        
        -- Operational recommendations
        CASE 
            WHEN alert_status LIKE 'CRITICAL ALERT%'
            THEN 'Immediate Action: Stop processing + investigate data quality issues'
            WHEN alert_status LIKE 'WARNING ALERT%'
            THEN 'Monitor Closely: Increase validation checks + alert threshold adjustment'
            WHEN alert_status LIKE 'PERFORMANCE ALERT%'
            THEN 'Optimize: Resource scaling + algorithm optimization'
            WHEN alert_status LIKE 'INTEGRATION ALERT%'
            THEN 'Fix Integration: API troubleshooting + data source validation'
            ELSE 'Continue: Normal operations + routine monitoring'
        END as operational_recommendation,
        
        -- Auto-scaling recommendations
        CASE 
            WHEN capacity_classification = 'Capacity Constraints' AND processing_priority = 'High'
            THEN 'Scale Up: Add processing nodes + increase resource allocation'
            WHEN capacity_classification = 'High Capacity' AND processing_priority = 'Low'
            THEN 'Scale Down: Reduce resources + optimize cost efficiency'
            WHEN sla_compliance = 'SLA Violated'
            THEN 'Emergency Scale: Immediate resource increase + priority rebalancing'
            WHEN sla_compliance = 'SLA At Risk'
            THEN 'Preventive Scale: Gradual resource increase + performance monitoring'
            ELSE 'Maintain: Current capacity appropriate + monitor trends'
        END as scaling_recommendation
    FROM pipeline_quality_monitoring pqm
),
real_time_analytics_summary AS (
    SELECT 
        mdm.*,
        
        -- Real-time dashboard KPIs
        CASE 
            WHEN alert_group_count > 0 AND alert_status LIKE 'CRITICAL%'
            THEN 'SYSTEM STATUS: CRITICAL - Multiple critical alerts active'
            WHEN alert_group_count > 0 AND alert_status LIKE 'WARNING%'
            THEN 'SYSTEM STATUS: WARNING - Performance monitoring required'
            WHEN sla_compliance = 'SLA Violated'
            THEN 'SYSTEM STATUS: DEGRADED - SLA compliance issues'
            WHEN quality_group_count > 0 AND quality_validation = 'Valid Standardization'
            THEN 'SYSTEM STATUS: HEALTHY - Normal operations'
            ELSE 'SYSTEM STATUS: MONITORING - Routine surveillance'
        END as system_status,
        
        -- Performance trend analysis
        CASE 
            WHEN avg_source_processing_time <= 0.002
            THEN 'TREND: EXCELLENT - Consistent high performance across sources'
            WHEN avg_source_processing_time <= 0.005
            THEN 'TREND: GOOD - Stable performance within targets'
            WHEN avg_source_processing_time <= 0.008
            THEN 'TREND: CONCERNING - Performance degradation detected'
            ELSE 'TREND: CRITICAL - Significant performance issues'
        END as performance_trend,
        
        -- Quality trend analysis
        CASE 
            WHEN quality_validation = 'Valid Standardization' AND source_reliability = 'High Reliability'
            THEN 'QUALITY TREND: STABLE - Consistent data quality maintained'
            WHEN quality_validation = 'Valid Standardization'
            THEN 'QUALITY TREND: GOOD - Quality targets met with minor concerns'
            WHEN quality_validation LIKE 'Warning%'
            THEN 'QUALITY TREND: DECLINING - Quality degradation detected'
            ELSE 'QUALITY TREND: CRITICAL - Quality failure patterns identified'
        END as quality_trend,
        
        -- Cost optimization opportunities
        CASE 
            WHEN capacity_classification = 'High Capacity' AND processing_priority = 'Low'
            THEN 'COST OPTIMIZATION: Over-provisioned resources for low-priority processing'
            WHEN resource_utilization = 'Minimal CPU' AND data_source = 'Batch Import'
            THEN 'COST OPTIMIZATION: Batch processing consolidation opportunity'
            WHEN sla_compliance = 'SLA Exceeded' AND capacity_classification = 'High Capacity'
            THEN 'COST OPTIMIZATION: Performance headroom allows resource reduction'
            ELSE 'COST OPTIMIZATION: Current resource allocation appropriate'
        END as cost_optimization_opportunity
    FROM monitoring_dashboard_metrics mdm
)
SELECT 
    user_id,
    stage_3_standardized as name,
    data_source,
    processing_priority,
    system_status,
    alert_status,
    operational_recommendation,
    scaling_recommendation,
    performance_trend,
    quality_trend,
    cost_optimization_opportunity,
    
    -- Technical metrics
    processing_time_seconds,
    max_throughput_per_second,
    sla_compliance,
    quality_validation,
    resource_utilization,
    
    -- Pipeline details
    stage_1_raw_input,
    stage_2_trimmed,
    trimming_status,
    source_reliability
FROM real_time_analytics_summary
ORDER BY 
    CASE alert_status
        WHEN 'CRITICAL ALERT: Data quality validation failed' THEN 1
        WHEN 'WARNING ALERT: Empty standardization result' THEN 2
        WHEN 'PERFORMANCE ALERT: Processing time exceeded threshold' THEN 3
        WHEN 'INTEGRATION ALERT: API data quality issues' THEN 4
        ELSE 5
    END,
    processing_priority DESC,
    user_id;
```

#### 3. **Enterprise Data Governance and Compliance Management System**
```sql
-- "Design enterprise data governance system for name standardization with compliance tracking and audit trails"

WITH governance_framework_analysis AS (
    SELECT 
        user_id,
        name as original_data,
        
        -- Apply standardization with audit trail
        CASE 
            WHEN LENGTH(name) = 0 THEN name
            WHEN LENGTH(name) = 1 THEN UPPER(name)
            ELSE CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2)))
        END as standardized_data,
        
        -- Data classification
        CASE 
            WHEN name REGEXP '^[A-Za-z ]+$' THEN 'Personal Identifiable Information (PII)'
            WHEN name REGEXP '^[A-Za-z0-9 ]+$' THEN 'Mixed Personal Data'
            WHEN LENGTH(name) > 0 THEN 'Complex Personal Data'
            ELSE 'No Personal Data'
        END as data_classification,
        
        -- Compliance framework requirements
        CASE 
            WHEN user_id % 6 = 0 THEN 'GDPR (Europe)'
            WHEN user_id % 6 = 1 THEN 'CCPA (California)'
            WHEN user_id % 6 = 2 THEN 'PIPEDA (Canada)'
            WHEN user_id % 6 = 3 THEN 'LGPD (Brazil)'
            WHEN user_id % 6 = 4 THEN 'PDPA (Singapore)'
            ELSE 'SOX (US)'
        END as primary_compliance_framework,
        
        -- Data sensitivity level
        CASE 
            WHEN LENGTH(name) = 0 THEN 'No Sensitivity'
            WHEN name REGEXP '^[A-Za-z ]+$' THEN 'Medium Sensitivity'
            WHEN name REGEXP '^[A-Za-z0-9 ]+$' THEN 'High Sensitivity'
            ELSE 'Very High Sensitivity'
        END as sensitivity_level,
        
        -- Processing legal basis (GDPR-style)
        CASE 
            WHEN user_id % 4 = 0 THEN 'Consent'
            WHEN user_id % 4 = 1 THEN 'Contract Performance'
            WHEN user_id % 4 = 2 THEN 'Legitimate Interest'
            ELSE 'Legal Obligation'
        END as processing_legal_basis,
        
        -- Data subject rights impact
        CASE 
            WHEN name != CONCAT(UPPER(LEFT(name, 1)), LOWER(SUBSTRING(name, 2)))
            THEN 'Right to Rectification: Data correction applied'
            ELSE 'Right to Accuracy: Data already compliant'
        END as data_subject_rights_impact
    FROM Users
),
compliance_tracking_system AS (
    SELECT 
        gfa.*,
        
        -- Compliance requirement analysis
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)'
            THEN 'GDPR Requirements: Lawful basis + consent + data minimization + accuracy + accountability'
            WHEN primary_compliance_framework = 'CCPA (California)'
            THEN 'CCPA Requirements: Consumer rights + data transparency + opt-out + non-discrimination'
            WHEN primary_compliance_framework = 'PIPEDA (Canada)'
            THEN 'PIPEDA Requirements: Consent + accountability + transparency + accuracy + safeguards'
            WHEN primary_compliance_framework = 'LGPD (Brazil)'
            THEN 'LGPD Requirements: Lawful basis + data minimization + transparency + security + accountability'
            WHEN primary_compliance_framework = 'PDPA (Singapore)'
            THEN 'PDPA Requirements: Consent + notification + access + accuracy + protection'
            ELSE 'SOX Requirements: Internal controls + accuracy + audit trail + data integrity'
        END as compliance_requirements,
        
        -- Audit trail generation
        CONCAT(
            'AUDIT: ', CURRENT_TIMESTAMP, ' | ',
            'USER: ', user_id, ' | ',
            'ACTION: Name Standardization | ',
            'ORIGINAL: "', original_data, '" | ',
            'STANDARDIZED: "', standardized_data, '" | ',
            'FRAMEWORK: ', primary_compliance_framework, ' | ',
            'BASIS: ', processing_legal_basis, ' | ',
            'CLASSIFICATION: ', data_classification
        ) as audit_trail_entry,
        
        -- Compliance risk assessment
        CASE 
            WHEN sensitivity_level = 'Very High Sensitivity' AND primary_compliance_framework LIKE 'GDPR%'
            THEN 'High Risk: Enhanced GDPR protections required + DPO consultation'
            WHEN sensitivity_level = 'High Sensitivity' AND processing_legal_basis = 'Consent'
            THEN 'Medium Risk: Consent validation required + clear purpose limitation'
            WHEN data_classification = 'Personal Identifiable Information (PII)'
            THEN 'Standard Risk: Standard PII protections + routine compliance monitoring'
            WHEN data_classification = 'No Personal Data'
            THEN 'Low Risk: Minimal compliance requirements + standard data governance'
            ELSE 'Medium Risk: Enhanced monitoring + compliance validation required'
        END as compliance_risk_level,
        
        -- Data governance actions required
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)' AND sensitivity_level = 'Very High Sensitivity'
            THEN 'GDPR Actions: Privacy impact assessment + DPO review + enhanced consent + audit documentation'
            WHEN primary_compliance_framework = 'CCPA (California)' AND data_subject_rights_impact LIKE 'Right to Rectification%'
            THEN 'CCPA Actions: Consumer notification + data correction log + transparency report update'
            WHEN processing_legal_basis = 'Consent' AND standardized_data != original_data
            THEN 'Consent Actions: Re-consent evaluation + purpose limitation review + transparency update'
            WHEN sensitivity_level = 'High Sensitivity'
            THEN 'Standard Actions: Enhanced monitoring + regular compliance review + security validation'
            ELSE 'Routine Actions: Standard governance + periodic review + documentation update'
        END as governance_actions_required,
        
        -- Retention and deletion policies
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)'
            THEN 'GDPR Retention: Purpose-limited + automatic deletion + right to erasure + retention logs'
            WHEN primary_compliance_framework = 'CCPA (California)'
            THEN 'CCPA Retention: Business purpose retention + consumer deletion rights + retention transparency'
            WHEN processing_legal_basis = 'Contract Performance'
            THEN 'Contract Retention: Contract lifecycle + post-contract period + legal requirement compliance'
            WHEN sensitivity_level = 'Very High Sensitivity'
            THEN 'Enhanced Retention: Minimal retention + encrypted storage + secure deletion + audit trail'
            ELSE 'Standard Retention: Business need basis + regular review + secure disposal + documentation'
        END as retention_policy,
        
        -- Privacy by design implementation
        CASE 
            WHEN sensitivity_level = 'Very High Sensitivity'
            THEN 'Privacy by Design: Data minimization + purpose limitation + encryption + access controls + audit logging'
            WHEN primary_compliance_framework LIKE 'GDPR%' OR primary_compliance_framework LIKE 'LGPD%'
            THEN 'Privacy by Design: Built-in privacy + end-to-end protection + transparency + user control'
            WHEN data_classification = 'Personal Identifiable Information (PII)'
            THEN 'Privacy by Design: Standard PII protections + access controls + monitoring + documentation'
            ELSE 'Privacy by Design: Basic privacy controls + data governance + regular review'
        END as privacy_by_design_measures
    FROM governance_framework_analysis gfa
),
regulatory_compliance_dashboard AS (
    SELECT 
        cts.*,
        
        -- Compliance status summary
        CASE 
            WHEN compliance_risk_level = 'High Risk' AND governance_actions_required LIKE 'GDPR Actions%'
            THEN 'COMPLIANCE STATUS: CRITICAL - High-risk GDPR processing requires immediate attention'
            WHEN compliance_risk_level = 'Medium Risk' AND primary_compliance_framework LIKE 'CCPA%'
            THEN 'COMPLIANCE STATUS: MONITORING - CCPA compliance monitoring in progress'
            WHEN compliance_risk_level = 'Standard Risk'
            THEN 'COMPLIANCE STATUS: COMPLIANT - Standard risk controls in place'
            WHEN compliance_risk_level = 'Low Risk'
            THEN 'COMPLIANCE STATUS: OPTIMAL - Low risk with appropriate controls'
            ELSE 'COMPLIANCE STATUS: REVIEW - Compliance assessment required'
        END as compliance_status,
        
        -- Regulatory reporting requirements
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)' AND sensitivity_level = 'Very High Sensitivity'
            THEN 'GDPR Reporting: Data processing register + privacy impact assessment + breach notification procedures'
            WHEN primary_compliance_framework = 'SOX (US)' AND data_subject_rights_impact LIKE 'Right to Rectification%'
            THEN 'SOX Reporting: Internal control documentation + data change audit + management certification'
            WHEN compliance_risk_level = 'High Risk'
            THEN 'Enhanced Reporting: Risk assessment documentation + mitigation plans + executive review'
            WHEN data_classification = 'Personal Identifiable Information (PII)'
            THEN 'Standard Reporting: PII processing logs + compliance monitoring + periodic review'
            ELSE 'Minimal Reporting: Basic documentation + routine compliance check'
        END as regulatory_reporting,
        
        -- Cross-border data transfer considerations
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)'
            THEN 'GDPR Transfers: Adequacy decisions + standard contractual clauses + transfer impact assessments'
            WHEN primary_compliance_framework = 'CCPA (California)'
            THEN 'CCPA Transfers: Service provider agreements + consumer disclosure + opt-out mechanisms'
            WHEN primary_compliance_framework = 'PIPEDA (Canada)'
            THEN 'PIPEDA Transfers: Comparable protection + accountability + consent + safeguards'
            WHEN primary_compliance_framework = 'LGPD (Brazil)'
            THEN 'LGPD Transfers: Adequacy level + standard clauses + authority approval + data subject rights'
            WHEN primary_compliance_framework = 'PDPA (Singapore)'
            THEN 'PDPA Transfers: Prescribed countries + binding corporate rules + consent + accountability'
            ELSE 'SOX Transfers: Internal controls + data integrity + audit trail + management oversight'
        END as cross_border_requirements,
        
        -- Incident response and breach notification
        CASE 
            WHEN sensitivity_level = 'Very High Sensitivity' AND primary_compliance_framework = 'GDPR (Europe)'
            THEN 'GDPR Incident Response: 72-hour authority notification + data subject notification + documentation + assessment'
            WHEN sensitivity_level = 'High Sensitivity' AND primary_compliance_framework = 'CCPA (California)'
            THEN 'CCPA Incident Response: Consumer notification + attorney general notification + remedial measures'
            WHEN compliance_risk_level = 'High Risk'
            THEN 'Enhanced Incident Response: Immediate escalation + comprehensive assessment + regulatory consultation'
            WHEN data_classification = 'Personal Identifiable Information (PII)'
            THEN 'Standard Incident Response: Risk assessment + affected party notification + remediation + documentation'
            ELSE 'Basic Incident Response: Internal assessment + standard procedures + documentation + review'
        END as incident_response_protocol,
        
        -- Ongoing compliance monitoring
        CASE 
            WHEN primary_compliance_framework = 'GDPR (Europe)' AND sensitivity_level = 'Very High Sensitivity'
            THEN 'Continuous GDPR Monitoring: Real-time compliance dashboard + automated alerts + regular DPO review'
            WHEN compliance_risk_level = 'High Risk'
            THEN 'Enhanced Monitoring: Daily compliance checks + risk indicator tracking + executive reporting'
            WHEN data_classification = 'Personal Identifiable Information (PII)'
            THEN 'Standard Monitoring: Weekly compliance review + PII access monitoring + policy compliance check'
            ELSE 'Routine Monitoring: Monthly compliance assessment + basic policy compliance + documentation review'
        END as monitoring_protocol
    FROM compliance_tracking_system cts
)
SELECT 
    user_id,
    standardized_data as name,
    primary_compliance_framework,
    data_classification,
    sensitivity_level,
    compliance_status,
    compliance_risk_level,
    governance_actions_required,
    regulatory_reporting,
    cross_border_requirements,
    incident_response_protocol,
    monitoring_protocol,
    retention_policy,
    privacy_by_design_measures,
    
    -- Audit and tracking
    audit_trail_entry,
    processing_legal_basis,
    data_subject_rights_impact,
    
    -- Original context
    original_data
FROM regulatory_compliance_dashboard
ORDER BY 
    CASE compliance_risk_level
        WHEN 'High Risk' THEN 1
        WHEN 'Medium Risk' THEN 2
        WHEN 'Standard Risk' THEN 3
        ELSE 4
    END,
    primary_compliance_framework,
    user_id;
```

## 🔗 Related LeetCode Questions

1. **#1683 - Invalid Tweets** (String length validation and filtering)
2. **#1484 - Group Sold Products By The Date** (String aggregation and manipulation)
3. **#1795 - Rearrange Products Table** (Data transformation and restructuring)
4. **#1873 - Calculate Special Bonus** (Conditional string operations)
5. **#627 - Swap Salary** (Data modification with conditions)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **String Functions**: UPPER(), LOWER(), LEFT(), RIGHT(), SUBSTRING()
2. **String Concatenation**: CONCAT() or || operator
3. **Conditional Logic**: CASE statements for edge cases
4. **Length Validation**: Handle empty strings and single characters

### 🚀 **Amazon Interview Tips**
1. **Explain string manipulation**: "Break name into first character and remaining characters"
2. **Discuss edge cases**: "Handle empty strings, single characters, and whitespace"
3. **Address performance**: "String functions are generally efficient for this use case"
4. **Consider internationalization**: "Different languages may have different capitalization rules"

### 🔧 **Common Patterns**
- String function combination (UPPER + LEFT + LOWER + SUBSTRING)
- CONCAT for string building
- CASE statements for edge case handling
- ORDER BY for consistent output

### ⚠️ **Common Mistakes to Avoid**
1. **Not handling edge cases** (empty strings, single characters)
2. **Wrong string function usage** (using RIGHT instead of SUBSTRING)
3. **Database-specific syntax** (different CONCAT methods across databases)
4. **Forgetting ORDER BY** (output order requirements)

### 🔍 **Performance Considerations**
- String functions are generally fast for this type of operation
- Consider indexing if this operation is part of frequent queries
- CONCAT is usually more readable than || operator
- Minimal performance impact for small to medium datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Standardized names improve customer communication and experience
- **Operational Excellence**: Consistent data formatting reduces errors and improves system reliability
- **Innovation**: Advanced data governance systems enable compliance and privacy protection
- **Bias for Action**: Quick standardization prevents data quality issues from accumulating

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Capitalize each word in a full name (Title Case)**
2. **Convert names to all uppercase or all lowercase**
3. **Handle names with special characters or numbers**
4. **Implement name standardization with cultural considerations**

Remember: Data standardization is crucial for Amazon's global customer base, ensuring consistent communication, personalization, and regulatory compliance across all customer touchpoints!