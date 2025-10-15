# LeetCode Easy #1378: Replace Employee ID With The Unique Identifier

## 📋 Problem Statement

Write a SQL query to show the **unique ID** of each user, If a user does not have a unique ID replace it with **null**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Employees Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| name          | varchar |
+---------------+---------+
```
- id is the primary key for this table.
- Each row contains the id and the name of an employee in a company.

**EmployeeUNI Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| unique_id     | int     |
+---------------+---------+
```
- (id, unique_id) is the primary key for this table.
- Each row contains the id and the corresponding unique id of an employee in the company.

## 📊 Sample Data

**Employees Table:**
| id | name     |
|----|----------|
| 1  | Alice    |
| 7  | Bob      |
| 11 | Meir     |
| 90 | Winston  |

**EmployeeUNI Table:**
| id | unique_id |
|----|-----------|
| 3  | 1         |
| 11 | 2         |
| 90 | 3         |

**Expected Output:**
| unique_id | name    |
|-----------|---------|
| null      | Alice   |
| null      | Bob     |
| 2         | Meir    |
| 3         | Winston |

**Explanation:**
- Alice (id=1) and Bob (id=7) have no corresponding unique_id, so they get null
- Meir (id=11) has unique_id=2
- Winston (id=90) has unique_id=3
- Employee with id=3 in EmployeeUNI has no corresponding employee record (ignored)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to include all employees, even those without unique IDs
- Replace missing unique IDs with NULL
- LEFT JOIN preserves all employees
- Some unique IDs may not have corresponding employees

### 2. **Key Insights**
- LEFT JOIN to preserve all employee records
- NULL values appear automatically for non-matching records
- Only employees should appear in results, not orphaned unique IDs
- Simple two-table join operation

### 3. **Interview Discussion Points**
- "This is a classic LEFT JOIN scenario"
- "We want to preserve all employees regardless of unique ID existence"
- "NULL handling is automatic with LEFT JOIN"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Base Table
```sql
-- Employees is the base table (we want all employees)
-- Use LEFT JOIN to preserve all employee records
```

### Step 2: Join with Unique IDs
```sql
-- LEFT JOIN EmployeeUNI ON matching id
-- Non-matching employees will have NULL unique_id
```

### Step 3: Select Required Columns
```sql
-- SELECT unique_id (or NULL), name
-- Order doesn't matter per problem statement
```

## ✅ Optimized SQL Solution

**Solution 1: Basic LEFT JOIN**
```sql
SELECT 
    eu.unique_id,
    e.name
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

### Alternative Solutions

**Solution 2: With Explicit NULL Handling**
```sql
SELECT 
    COALESCE(eu.unique_id, NULL) as unique_id,
    e.name
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

**Solution 3: Using CASE Statement**
```sql
SELECT 
    CASE 
        WHEN eu.unique_id IS NOT NULL THEN eu.unique_id 
        ELSE NULL 
    END as unique_id,
    e.name
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

**Solution 4: With Additional Employee Information**
```sql
SELECT 
    e.id as employee_id,
    e.name,
    eu.unique_id,
    CASE 
        WHEN eu.unique_id IS NOT NULL THEN 'Has Unique ID'
        ELSE 'Missing Unique ID'
    END as id_status
FROM Employees e
LEFT JOIN EmployeeUNI eu ON e.id = eu.id;
```

**Solution 5: Comprehensive Employee Directory**
```sql
WITH employee_status AS (
    SELECT 
        e.id as employee_id,
        e.name,
        eu.unique_id,
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Active'
            ELSE 'Pending ID Assignment'
        END as status,
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 1
            ELSE 0
        END as has_unique_id_flag
    FROM Employees e
    LEFT JOIN EmployeeUNI eu ON e.id = eu.id
)
SELECT 
    unique_id,
    name,
    status,
    employee_id
FROM employee_status
ORDER BY 
    has_unique_id_flag DESC,  -- Show employees with unique IDs first
    name;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Employee Identity Management System**
```sql
-- "Design a complete employee identity and access management analysis"

WITH employee_identity_analysis AS (
    SELECT 
        e.id as employee_id,
        e.name as employee_name,
        eu.unique_id,
        
        -- Identity status classification
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Complete Identity'
            ELSE 'Incomplete Identity'
        END as identity_status,
        
        -- Access level determination
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Full Access'
            ELSE 'Limited Access'
        END as access_level,
        
        -- Priority for ID assignment
        ROW_NUMBER() OVER (
            PARTITION BY CASE WHEN eu.unique_id IS NULL THEN 1 ELSE 0 END 
            ORDER BY e.id
        ) as assignment_priority
    FROM Employees e
    LEFT JOIN EmployeeUNI eu ON e.id = eu.id
),
orphaned_unique_ids AS (
    -- Find unique IDs without corresponding employees
    SELECT 
        eu.id as orphaned_employee_id,
        eu.unique_id as orphaned_unique_id,
        'Orphaned Unique ID' as issue_type
    FROM EmployeeUNI eu
    LEFT JOIN Employees e ON eu.id = e.id
    WHERE e.id IS NULL
),
identity_metrics AS (
    SELECT 
        COUNT(*) as total_employees,
        COUNT(unique_id) as employees_with_unique_id,
        COUNT(*) - COUNT(unique_id) as employees_without_unique_id,
        ROUND(COUNT(unique_id) * 100.0 / COUNT(*), 2) as identity_completion_rate,
        
        -- Security metrics
        COUNT(CASE WHEN access_level = 'Full Access' THEN 1 END) as full_access_employees,
        COUNT(CASE WHEN access_level = 'Limited Access' THEN 1 END) as limited_access_employees
    FROM employee_identity_analysis
),
security_recommendations AS (
    SELECT 
        eia.*,
        im.identity_completion_rate,
        im.employees_without_unique_id,
        
        CASE 
            WHEN eia.identity_status = 'Incomplete Identity' AND eia.assignment_priority <= 5 
            THEN 'High Priority ID Assignment'
            WHEN eia.identity_status = 'Incomplete Identity' AND eia.assignment_priority <= 20 
            THEN 'Medium Priority ID Assignment'
            WHEN eia.identity_status = 'Incomplete Identity' 
            THEN 'Standard ID Assignment'
            WHEN eia.identity_status = 'Complete Identity' 
            THEN 'Monitor for compliance'
            ELSE 'No action required'
        END as recommendation,
        
        CASE 
            WHEN im.identity_completion_rate < 70 THEN 'Critical - Bulk ID Assignment Needed'
            WHEN im.identity_completion_rate < 85 THEN 'Action Required - Schedule ID Assignment'
            WHEN im.identity_completion_rate < 95 THEN 'Monitor - Gradual ID Assignment'
            ELSE 'Excellent - Maintain Current Process'
        END as system_health_status
    FROM employee_identity_analysis eia
    CROSS JOIN identity_metrics im
),
compliance_report AS (
    SELECT 
        'Employee Identity Management' as category,
        'Total Employees' as metric,
        CAST(total_employees AS VARCHAR) as value,
        'Count' as unit,
        CASE 
            WHEN total_employees > 1000 THEN 'Enterprise Scale'
            WHEN total_employees > 100 THEN 'Large Organization'
            WHEN total_employees > 20 THEN 'Medium Organization'
            ELSE 'Small Organization'
        END as classification
    FROM identity_metrics
    
    UNION ALL
    
    SELECT 
        'Employee Identity Management' as category,
        'Identity Completion Rate' as metric,
        CONCAT(CAST(identity_completion_rate AS VARCHAR), '%') as value,
        'Percentage' as unit,
        CASE 
            WHEN identity_completion_rate >= 95 THEN 'Excellent Compliance'
            WHEN identity_completion_rate >= 85 THEN 'Good Compliance'
            WHEN identity_completion_rate >= 70 THEN 'Fair Compliance'
            ELSE 'Poor Compliance'
        END as classification
    FROM identity_metrics
    
    UNION ALL
    
    SELECT 
        'Security Risk Assessment' as category,
        'Employees Without IDs' as metric,
        CAST(employees_without_unique_id AS VARCHAR) as value,
        'Count' as unit,
        CASE 
            WHEN employees_without_unique_id = 0 THEN 'No Risk'
            WHEN employees_without_unique_id <= 5 THEN 'Low Risk'
            WHEN employees_without_unique_id <= 20 THEN 'Medium Risk'
            ELSE 'High Risk'
        END as classification
    FROM identity_metrics
)
SELECT 
    sr.employee_id,
    sr.employee_name,
    sr.unique_id,
    sr.identity_status,
    sr.access_level,
    sr.assignment_priority,
    sr.recommendation,
    sr.system_health_status,
    
    -- Additional context from orphaned IDs
    CASE 
        WHEN EXISTS (SELECT 1 FROM orphaned_unique_ids) 
        THEN 'Check for orphaned unique IDs in system'
        ELSE 'No orphaned unique IDs detected'
    END as data_integrity_status
FROM security_recommendations sr
ORDER BY 
    CASE sr.identity_status WHEN 'Incomplete Identity' THEN 1 ELSE 2 END,
    sr.assignment_priority;
```

#### 2. **Employee Onboarding and Lifecycle Management**
```sql
-- "Track employee onboarding progress and identity lifecycle"

WITH employee_onboarding_pipeline AS (
    SELECT 
        e.id as employee_id,
        e.name as employee_name,
        eu.unique_id,
        
        -- Onboarding stage analysis
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Stage 3: Identity Assigned'
            ELSE 'Stage 2: Identity Pending'
        END as onboarding_stage,
        
        -- Assumed additional onboarding data (in real scenario, would come from other tables)
        CASE 
            WHEN e.id % 4 = 0 THEN 'IT Equipment Assigned'
            WHEN e.id % 4 = 1 THEN 'IT Equipment Pending'
            WHEN e.id % 4 = 2 THEN 'IT Equipment Not Required'
            ELSE 'IT Equipment Status Unknown'
        END as equipment_status,
        
        CASE 
            WHEN e.id % 3 = 0 THEN 'Training Completed'
            WHEN e.id % 3 = 1 THEN 'Training In Progress'
            ELSE 'Training Not Started'
        END as training_status,
        
        CASE 
            WHEN eu.unique_id IS NOT NULL AND e.id % 2 = 0 THEN 'Access Provisioned'
            WHEN eu.unique_id IS NOT NULL THEN 'Access Provisioning'
            ELSE 'Access Pending Identity'
        END as access_provisioning_status
    FROM Employees e
    LEFT JOIN EmployeeUNI eu ON e.id = eu.id
),
onboarding_completion_analysis AS (
    SELECT 
        *,
        -- Overall onboarding progress scoring
        (CASE WHEN unique_id IS NOT NULL THEN 25 ELSE 0 END) +
        (CASE WHEN equipment_status = 'IT Equipment Assigned' THEN 25 
              WHEN equipment_status = 'IT Equipment Not Required' THEN 25 
              ELSE 0 END) +
        (CASE WHEN training_status = 'Training Completed' THEN 25 ELSE 0 END) +
        (CASE WHEN access_provisioning_status = 'Access Provisioned' THEN 25 ELSE 0 END) as onboarding_score,
        
        -- Completion timeline estimation
        CASE 
            WHEN unique_id IS NOT NULL AND equipment_status LIKE '%Assigned%' AND training_status = 'Training Completed' 
            THEN 'Onboarding Complete'
            WHEN unique_id IS NOT NULL AND (equipment_status LIKE '%Pending%' OR training_status != 'Training Completed')
            THEN 'Onboarding 75% Complete'
            WHEN unique_id IS NOT NULL 
            THEN 'Onboarding 50% Complete'
            ELSE 'Onboarding <50% Complete'
        END as completion_status,
        
        -- Bottleneck identification
        CASE 
            WHEN unique_id IS NULL THEN 'Identity Assignment Bottleneck'
            WHEN equipment_status LIKE '%Pending%' THEN 'Equipment Assignment Bottleneck'
            WHEN training_status = 'Training Not Started' THEN 'Training Initiation Bottleneck'
            WHEN access_provisioning_status LIKE '%Pending%' THEN 'Access Provisioning Bottleneck'
            ELSE 'No Major Bottlenecks'
        END as primary_bottleneck
    FROM employee_onboarding_pipeline
),
department_simulation AS (
    -- Simulate department assignments for more realistic analysis
    SELECT 
        oca.*,
        CASE 
            WHEN oca.employee_id % 5 = 0 THEN 'Engineering'
            WHEN oca.employee_id % 5 = 1 THEN 'Sales'
            WHEN oca.employee_id % 5 = 2 THEN 'Marketing'
            WHEN oca.employee_id % 5 = 3 THEN 'Operations'
            ELSE 'Human Resources'
        END as department,
        
        CASE 
            WHEN oca.employee_id % 3 = 0 THEN 'Senior'
            WHEN oca.employee_id % 3 = 1 THEN 'Mid-Level'
            ELSE 'Junior'
        END as seniority_level
    FROM onboarding_completion_analysis oca
),
departmental_analytics AS (
    SELECT 
        department,
        COUNT(*) as total_employees,
        AVG(onboarding_score) as avg_onboarding_score,
        COUNT(CASE WHEN completion_status = 'Onboarding Complete' THEN 1 END) as fully_onboarded,
        COUNT(CASE WHEN unique_id IS NULL THEN 1 END) as employees_without_id,
        
        -- Department-specific metrics
        ROUND(COUNT(CASE WHEN completion_status = 'Onboarding Complete' THEN 1 END) * 100.0 / COUNT(*), 2) as completion_rate,
        
        -- Most common bottleneck per department
        (SELECT primary_bottleneck 
         FROM department_simulation ds2 
         WHERE ds2.department = ds.department 
         GROUP BY primary_bottleneck 
         ORDER BY COUNT(*) DESC 
         LIMIT 1) as most_common_bottleneck
    FROM department_simulation ds
    GROUP BY department
),
actionable_onboarding_insights AS (
    SELECT 
        ds.employee_id,
        ds.employee_name,
        ds.department,
        ds.seniority_level,
        ds.unique_id,
        ds.onboarding_score,
        ds.completion_status,
        ds.primary_bottleneck,
        da.completion_rate as dept_completion_rate,
        da.most_common_bottleneck as dept_common_bottleneck,
        
        -- Personalized recommendations
        CASE 
            WHEN ds.primary_bottleneck = 'Identity Assignment Bottleneck' AND ds.seniority_level = 'Senior'
            THEN 'URGENT: Expedite unique ID assignment for senior employee'
            WHEN ds.primary_bottleneck = 'Identity Assignment Bottleneck'
            THEN 'Schedule unique ID assignment within 2 business days'
            WHEN ds.primary_bottleneck = 'Equipment Assignment Bottleneck' AND ds.department = 'Engineering'
            THEN 'Priority equipment provisioning for technical role'
            WHEN ds.primary_bottleneck = 'Training Initiation Bottleneck'
            THEN 'Enroll in next available training cohort'
            WHEN ds.completion_status = 'Onboarding Complete'
            THEN 'Monitor post-onboarding integration'
            ELSE 'Follow standard onboarding process'
        END as personalized_action,
        
        -- Timeline estimation
        CASE 
            WHEN ds.onboarding_score >= 75 THEN 'Complete within 1 week'
            WHEN ds.onboarding_score >= 50 THEN 'Complete within 2 weeks'
            WHEN ds.onboarding_score >= 25 THEN 'Complete within 1 month'
            ELSE 'Requires immediate attention - timeline uncertain'
        END as estimated_completion_timeline
    FROM department_simulation ds
    JOIN departmental_analytics da ON ds.department = da.department
)
SELECT 
    employee_id,
    employee_name,
    department,
    seniority_level,
    unique_id,
    onboarding_score,
    completion_status,
    primary_bottleneck,
    personalized_action,
    estimated_completion_timeline,
    ROUND(dept_completion_rate, 2) as department_completion_rate,
    dept_common_bottleneck as department_common_issue
FROM actionable_onboarding_insights
ORDER BY 
    CASE seniority_level WHEN 'Senior' THEN 1 WHEN 'Mid-Level' THEN 2 ELSE 3 END,
    onboarding_score ASC,  -- Show employees needing most attention first
    employee_id;
```

#### 3. **Identity Security and Audit Trail**
```sql
-- "Implement comprehensive identity security monitoring and audit capabilities"

WITH identity_security_assessment AS (
    SELECT 
        e.id as employee_id,
        e.name as employee_name,
        eu.unique_id,
        
        -- Security classification
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Identified'
            ELSE 'Unidentified'
        END as security_status,
        
        -- Risk assessment based on ID patterns
        CASE 
            WHEN eu.unique_id IS NULL THEN 'High Risk - No Identity'
            WHEN eu.unique_id = e.id THEN 'Medium Risk - Predictable ID Pattern'
            WHEN eu.unique_id < 10 THEN 'Medium Risk - Low Entropy ID'
            ELSE 'Low Risk - Secure ID'
        END as risk_assessment,
        
        -- Access control implications
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Privileged Access Eligible'
            ELSE 'Guest Access Only'
        END as access_control_level,
        
        -- Compliance status
        CASE 
            WHEN eu.unique_id IS NOT NULL THEN 'Compliant'
            ELSE 'Non-Compliant'
        END as compliance_status
    FROM Employees e
    LEFT JOIN EmployeeUNI eu ON e.id = eu.id
),
security_anomaly_detection AS (
    SELECT 
        isa.*,
        
        -- Duplicate unique ID detection
        CASE 
            WHEN isa.unique_id IS NOT NULL AND (
                SELECT COUNT(*) 
                FROM EmployeeUNI eu2 
                WHERE eu2.unique_id = isa.unique_id
            ) > 1 THEN 'CRITICAL: Duplicate Unique ID'
            ELSE 'No Duplicate ID Issues'
        END as duplicate_id_status,
        
        -- ID sequence analysis
        LAG(isa.unique_id) OVER (ORDER BY isa.unique_id) as prev_unique_id,
        LEAD(isa.unique_id) OVER (ORDER BY isa.unique_id) as next_unique_id,
        
        -- Gap analysis in ID sequences
        CASE 
            WHEN isa.unique_id IS NOT NULL AND 
                 LAG(isa.unique_id) OVER (ORDER BY isa.unique_id) IS NOT NULL AND
                 isa.unique_id - LAG(isa.unique_id) OVER (ORDER BY isa.unique_id) > 1
            THEN 'ID Sequence Gap Detected'
            ELSE 'Normal ID Sequence'
        END as sequence_analysis
    FROM identity_security_assessment isa
),
audit_trail_simulation AS (
    -- Simulate audit events for demonstration
    SELECT 
        sad.*,
        
        -- Simulated last access timestamp
        CASE 
            WHEN sad.unique_id IS NOT NULL THEN 
                DATE_SUB(CURRENT_TIMESTAMP, INTERVAL (sad.unique_id % 30) DAY)
            ELSE NULL
        END as last_access_date,
        
        -- Simulated access frequency
        CASE 
            WHEN sad.unique_id IS NOT NULL THEN (sad.unique_id % 10) + 1
            ELSE 0
        END as monthly_access_count,
        
        -- Account status inference
        CASE 
            WHEN sad.unique_id IS NOT NULL AND (sad.unique_id % 10) >= 5 THEN 'Active'
            WHEN sad.unique_id IS NOT NULL AND (sad.unique_id % 10) >= 2 THEN 'Inactive'
            WHEN sad.unique_id IS NOT NULL THEN 'Dormant'
            ELSE 'Not Provisioned'
        END as account_activity_status
    FROM security_anomaly_detection sad
),
comprehensive_security_report AS (
    SELECT 
        ats.*,
        
        -- Combined risk scoring (0-100, higher = more risk)
        CASE ats.risk_assessment
            WHEN 'High Risk - No Identity' THEN 80
            WHEN 'Medium Risk - Predictable ID Pattern' THEN 50
            WHEN 'Medium Risk - Low Entropy ID' THEN 40
            ELSE 10
        END +
        CASE ats.duplicate_id_status
            WHEN 'CRITICAL: Duplicate Unique ID' THEN 20
            ELSE 0
        END +
        CASE ats.account_activity_status
            WHEN 'Dormant' THEN 15
            WHEN 'Inactive' THEN 10
            WHEN 'Not Provisioned' THEN 5
            ELSE 0
        END as total_risk_score,
        
        -- Security recommendations
        CASE 
            WHEN ats.duplicate_id_status LIKE 'CRITICAL%' 
            THEN 'IMMEDIATE: Resolve duplicate unique ID conflict'
            WHEN ats.risk_assessment = 'High Risk - No Identity' AND ats.account_activity_status = 'Not Provisioned'
            THEN 'URGENT: Provision secure unique ID immediately'
            WHEN ats.risk_assessment LIKE 'Medium Risk%' 
            THEN 'Schedule ID security review and possible regeneration'
            WHEN ats.account_activity_status = 'Dormant' 
            THEN 'Review account necessity - consider deactivation'
            WHEN ats.compliance_status = 'Non-Compliant' 
            THEN 'Bring into compliance within 48 hours'
            ELSE 'Continue standard monitoring'
        END as security_recommendation,
        
        -- Incident classification
        CASE 
            WHEN ats.duplicate_id_status LIKE 'CRITICAL%' THEN 'Security Incident'
            WHEN ats.risk_assessment = 'High Risk - No Identity' THEN 'Compliance Violation'
            WHEN ats.account_activity_status = 'Dormant' THEN 'Access Review Required'
            ELSE 'Normal Operation'
        END as incident_classification
    FROM audit_trail_simulation ats
),
security_metrics_summary AS (
    SELECT 
        COUNT(*) as total_employees,
        COUNT(unique_id) as employees_with_ids,
        COUNT(CASE WHEN compliance_status = 'Compliant' THEN 1 END) as compliant_employees,
        COUNT(CASE WHEN risk_assessment = 'High Risk - No Identity' THEN 1 END) as high_risk_employees,
        COUNT(CASE WHEN duplicate_id_status LIKE 'CRITICAL%' THEN 1 END) as duplicate_id_incidents,
        COUNT(CASE WHEN account_activity_status = 'Dormant' THEN 1 END) as dormant_accounts,
        
        AVG(total_risk_score) as avg_risk_score,
        MAX(total_risk_score) as max_risk_score,
        
        ROUND(COUNT(unique_id) * 100.0 / COUNT(*), 2) as identity_coverage_rate,
        ROUND(COUNT(CASE WHEN compliance_status = 'Compliant' THEN 1 END) * 100.0 / COUNT(*), 2) as compliance_rate
    FROM comprehensive_security_report
)
SELECT 
    csr.employee_id,
    csr.employee_name,
    csr.unique_id,
    csr.security_status,
    csr.risk_assessment,
    csr.compliance_status,
    csr.account_activity_status,
    csr.total_risk_score,
    csr.duplicate_id_status,
    csr.security_recommendation,
    csr.incident_classification,
    
    -- Context from organization-wide metrics
    sms.compliance_rate as organization_compliance_rate,
    CASE 
        WHEN csr.total_risk_score > sms.avg_risk_score * 1.5 THEN 'Above Average Risk'
        WHEN csr.total_risk_score < sms.avg_risk_score * 0.5 THEN 'Below Average Risk'
        ELSE 'Average Risk Level'
    END as relative_risk_assessment
FROM comprehensive_security_report csr
CROSS JOIN security_metrics_summary sms
ORDER BY 
    CASE csr.incident_classification 
        WHEN 'Security Incident' THEN 1 
        WHEN 'Compliance Violation' THEN 2 
        WHEN 'Access Review Required' THEN 3 
        ELSE 4 
    END,
    csr.total_risk_score DESC,
    csr.employee_id;
```

#### 4. **Advanced Identity Analytics and Reporting**
```sql
-- "Create executive dashboard and advanced analytics for identity management"

WITH identity_analytics_base AS (
    SELECT 
        e.id as employee_id,
        e.name as employee_name,
        eu.unique_id,
        
        -- Basic classifications
        CASE WHEN eu.unique_id IS NOT NULL THEN 1 ELSE 0 END as has_unique_id,
        CASE WHEN eu.unique_id IS NOT NULL THEN 'Active' ELSE 'Pending' END as identity_status,
        
        -- Simulated additional attributes for analytics
        CASE 
            WHEN e.id % 5 = 0 THEN 'Engineering'
            WHEN e.id % 5 = 1 THEN 'Sales'
            WHEN e.id % 5 = 2 THEN 'Marketing'
            WHEN e.id % 5 = 3 THEN 'Operations'
            ELSE 'Human Resources'
        END as department,
        
        CASE 
            WHEN e.id % 4 = 0 THEN 'Manager'
            WHEN e.id % 4 = 1 THEN 'Senior'
            WHEN e.id % 4 = 2 THEN 'Mid-Level'
            ELSE 'Junior'
        END as role_level,
        
        CASE 
            WHEN e.id <= 30 THEN 'Tenure_5plus_years'
            WHEN e.id <= 60 THEN 'Tenure_2to5_years'
            WHEN e.id <= 90 THEN 'Tenure_1to2_years'
            ELSE 'Tenure_under1_year'
        END as tenure_category
    FROM Employees e
    LEFT JOIN EmployeeUNI eu ON e.id = eu.id
),
executive_dashboard_metrics AS (
    SELECT 
        -- Overall organization metrics
        COUNT(*) as total_workforce,
        SUM(has_unique_id) as employees_with_identity,
        COUNT(*) - SUM(has_unique_id) as employees_pending_identity,
        ROUND(SUM(has_unique_id) * 100.0 / COUNT(*), 2) as identity_completion_percentage,
        
        -- Department breakdown
        COUNT(CASE WHEN department = 'Engineering' THEN 1 END) as engineering_total,
        SUM(CASE WHEN department = 'Engineering' THEN has_unique_id ELSE 0 END) as engineering_with_id,
        
        COUNT(CASE WHEN department = 'Sales' THEN 1 END) as sales_total,
        SUM(CASE WHEN department = 'Sales' THEN has_unique_id ELSE 0 END) as sales_with_id,
        
        COUNT(CASE WHEN department = 'Marketing' THEN 1 END) as marketing_total,
        SUM(CASE WHEN department = 'Marketing' THEN has_unique_id ELSE 0 END) as marketing_with_id,
        
        COUNT(CASE WHEN department = 'Operations' THEN 1 END) as operations_total,
        SUM(CASE WHEN department = 'Operations' THEN has_unique_id ELSE 0 END) as operations_with_id,
        
        COUNT(CASE WHEN department = 'Human Resources' THEN 1 END) as hr_total,
        SUM(CASE WHEN department = 'Human Resources' THEN has_unique_id ELSE 0 END) as hr_with_id,
        
        -- Role level analysis
        COUNT(CASE WHEN role_level = 'Manager' THEN 1 END) as managers_total,
        SUM(CASE WHEN role_level = 'Manager' THEN has_unique_id ELSE 0 END) as managers_with_id,
        
        COUNT(CASE WHEN role_level = 'Senior' THEN 1 END) as senior_total,
        SUM(CASE WHEN role_level = 'Senior' THEN has_unique_id ELSE 0 END) as senior_with_id
    FROM identity_analytics_base
),
departmental_analysis AS (
    SELECT 
        department,
        COUNT(*) as dept_total_employees,
        SUM(has_unique_id) as dept_employees_with_id,
        ROUND(SUM(has_unique_id) * 100.0 / COUNT(*), 2) as dept_completion_rate,
        
        -- Role distribution within department
        COUNT(CASE WHEN role_level = 'Manager' THEN 1 END) as dept_managers,
        COUNT(CASE WHEN role_level = 'Senior' THEN 1 END) as dept_senior,
        COUNT(CASE WHEN role_level = 'Mid-Level' THEN 1 END) as dept_mid_level,
        COUNT(CASE WHEN role_level = 'Junior' THEN 1 END) as dept_junior,
        
        -- Identity completion by role within department
        ROUND(SUM(CASE WHEN role_level = 'Manager' THEN has_unique_id ELSE 0 END) * 100.0 / 
              NULLIF(COUNT(CASE WHEN role_level = 'Manager' THEN 1 END), 0), 2) as mgr_completion_rate,
        ROUND(SUM(CASE WHEN role_level = 'Senior' THEN has_unique_id ELSE 0 END) * 100.0 / 
              NULLIF(COUNT(CASE WHEN role_level = 'Senior' THEN 1 END), 0), 2) as senior_completion_rate,
        
        -- Department ranking
        ROW_NUMBER() OVER (ORDER BY SUM(has_unique_id) * 100.0 / COUNT(*) DESC) as dept_rank_by_completion
    FROM identity_analytics_base
    GROUP BY department
),
trend_analysis AS (
    SELECT 
        tenure_category,
        COUNT(*) as tenure_group_total,
        SUM(has_unique_id) as tenure_group_with_id,
        ROUND(SUM(has_unique_id) * 100.0 / COUNT(*), 2) as tenure_completion_rate,
        
        -- Cross-analysis: tenure vs role vs identity
        COUNT(CASE WHEN role_level = 'Manager' AND has_unique_id = 1 THEN 1 END) as tenure_managers_with_id,
        COUNT(CASE WHEN role_level = 'Manager' THEN 1 END) as tenure_managers_total
    FROM identity_analytics_base
    GROUP BY tenure_category
),
executive_summary_report AS (
    SELECT 
        'Executive Summary' as report_section,
        'Overall Identity Completion' as metric_name,
        CONCAT(CAST(identity_completion_percentage AS VARCHAR), '%') as metric_value,
        
        CASE 
            WHEN identity_completion_percentage >= 95 THEN 'Excellent - Industry Leading'
            WHEN identity_completion_percentage >= 85 THEN 'Good - Above Average'
            WHEN identity_completion_percentage >= 70 THEN 'Fair - Needs Improvement'
            ELSE 'Poor - Critical Action Required'
        END as performance_assessment,
        
        CASE 
            WHEN identity_completion_percentage >= 95 THEN 'Maintain excellence, monitor for new hires'
            WHEN identity_completion_percentage >= 85 THEN 'Target remaining 15% for completion within 30 days'
            WHEN identity_completion_percentage >= 70 THEN 'Implement accelerated ID assignment program'
            ELSE 'Emergency identity provisioning initiative required'
        END as executive_recommendation
    FROM executive_dashboard_metrics
    
    UNION ALL
    
    SELECT 
        'Department Performance' as report_section,
        'Best Performing Department' as metric_name,
        (SELECT department FROM departmental_analysis WHERE dept_rank_by_completion = 1) as metric_value,
        
        CONCAT('Completion Rate: ', 
               CAST((SELECT dept_completion_rate FROM departmental_analysis WHERE dept_rank_by_completion = 1) AS VARCHAR), 
               '%') as performance_assessment,
        
        'Use as best practice model for other departments' as executive_recommendation
    FROM executive_dashboard_metrics
    
    UNION ALL
    
    SELECT 
        'Risk Assessment' as report_section,
        'Employees Pending Identity' as metric_name,
        CAST(employees_pending_identity AS VARCHAR) as metric_value,
        
        CASE 
            WHEN employees_pending_identity = 0 THEN 'No Risk - Complete Coverage'
            WHEN employees_pending_identity <= 5 THEN 'Low Risk - Manageable Gap'
            WHEN employees_pending_identity <= 20 THEN 'Medium Risk - Requires Attention'
            ELSE 'High Risk - Significant Security Gap'
        END as performance_assessment,
        
        CASE 
            WHEN employees_pending_identity = 0 THEN 'Continue monitoring new hire processes'
            WHEN employees_pending_identity <= 5 THEN 'Complete within 1 week'
            WHEN employees_pending_identity <= 20 THEN 'Dedicated task force for 2-week completion'
            ELSE 'Emergency response team - 1-week deadline'
        END as executive_recommendation
    FROM executive_dashboard_metrics
)
SELECT 
    report_section,
    metric_name,
    metric_value,
    performance_assessment,
    executive_recommendation
FROM executive_summary_report
ORDER BY 
    CASE report_section 
        WHEN 'Executive Summary' THEN 1 
        WHEN 'Department Performance' THEN 2 
        WHEN 'Risk Assessment' THEN 3 
        ELSE 4 
    END,
    metric_name;
```

## 🔗 Related LeetCode Questions

1. **#175 - Combine Two Tables** (Basic LEFT JOIN)
2. **#181 - Employees Earning More Than Their Managers** (Self JOIN)
3. **#183 - Customers Who Never Placed an Order** (LEFT JOIN with NULL)
4. **#577 - Employee Bonus** (LEFT JOIN with aggregation)
5. **#1068 - Product Sales Analysis I** (INNER JOIN operations)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **LEFT JOIN**: Preserving all records from the left table
2. **NULL Handling**: Automatic NULL for non-matching records
3. **Primary vs Foreign Keys**: Understanding table relationships
4. **Data Preservation**: Keeping all employees regardless of unique ID

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I include all employees even without unique IDs?"
2. **Explain JOIN choice**: "LEFT JOIN preserves all employee records"
3. **Handle edge cases**: "What about employees with multiple unique IDs?"
4. **Business context**: "This supports employee directory and access management"

### 🔧 **Common Patterns**
- LEFT JOIN for optional relationships
- NULL handling in JOIN results
- Primary table preservation
- Optional attribute mapping

### ⚠️ **Common Mistakes to Avoid**
1. **Using INNER JOIN** (loses employees without unique IDs)
2. **Wrong table order** (putting EmployeeUNI as base table)
3. **Unnecessary NULL handling** (COALESCE when not needed)
4. **Missing edge cases** (not considering orphaned unique IDs)

### 🔍 **Performance Considerations**
- Index on id columns for efficient joins
- Consider covering indexes for frequently queried columns
- Monitor for orphaned records in lookup tables
- Use EXPLAIN PLAN to verify join strategy

### 🎯 **Amazon Leadership Principles Applied**
- **Security First**: Proper identity management for access control
- **Operational Excellence**: Complete employee directory maintenance
- **Customer Obsession**: Reliable employee information for internal customers
- **Data-Driven Decisions**: Comprehensive identity analytics for HR planning

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find employees with multiple unique IDs**
2. **Include employee count per unique ID**
3. **Show orphaned unique IDs**
4. **Add employee status and department information**

Remember: Identity management is fundamental to Amazon's employee systems, access control, security frameworks, and organizational analytics across global operations and diverse workforce management needs!