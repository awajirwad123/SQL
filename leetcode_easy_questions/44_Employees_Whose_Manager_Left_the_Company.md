# LeetCode Easy #1978: Employees Whose Manager Left the Company

## 📋 Problem Statement

Write a SQL query to find the **employees** whose **salary is less than $30000** and whose **manager left the company**. When a manager leaves the company, their information is deleted from the Employees table, but the reports still have their manager_id set to the manager that left.

Return the result table **ordered by employee_id**.

## 🗄️ Table Schema

**Employees Table:**
```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| employee_id | int      |
| name        | varchar  |
| manager_id  | int      |
| salary      | int      |
+-------------+----------+
```
- employee_id is the primary key for this table.
- This table contains information about the employees, their salary, and the id of their manager.
- Some employees do not have a manager (manager_id is null).

## 📊 Sample Data

**Employees Table:**
| employee_id | name      | manager_id | salary |
|-------------|-----------|------------|--------|
| 3           | Mila      | 9          | 60301  |
| 12          | Antonella | null       | 31000  |
| 13          | Emery     | null       | 67084  |
| 1           | Kalel     | 11         | 21241  |
| 9           | Mikaela   | null       | 50937  |
| 11          | Joziah    | 6          | 28485  |

**Expected Output:**
| employee_id |
|-------------|
| 11          |

**Explanation:**
- Employee 11 (Joziah) has salary 28485 (< 30000) and manager_id 6, but no employee with id 6 exists in the table
- Employee 1 (Kalel) has salary 21241 (< 30000) and manager_id 11, but employee 11 exists in the table
- Other employees either have salary >= 30000 or no manager_id or their manager exists

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find employees with salary < 30000 AND manager left company
- Manager "left" means manager_id points to non-existent employee
- Use LEFT JOIN or NOT EXISTS to detect missing managers
- Order result by employee_id

### 2. **Key Insights**
- Two conditions: salary < 30000 AND manager not in table
- Manager_id IS NOT NULL (employees without managers don't count)
- LEFT JOIN with NULL check or NOT EXISTS subquery
- Missing manager means their record was deleted

### 3. **Interview Discussion Points**
- "Need both salary condition AND missing manager condition"
- "Use LEFT JOIN to find managers that don't exist in employee table"
- "Exclude employees with NULL manager_id - they never had managers"

## 🔧 Step-by-Step Solution Logic

### Step 1: Filter by Salary
```sql
-- First condition: salary < 30000
WHERE salary < 30000
```

### Step 2: Check for Manager Existence
```sql
-- Use LEFT JOIN to find missing managers
LEFT JOIN Employees manager ON e.manager_id = manager.employee_id
WHERE manager.employee_id IS NULL
```

### Step 3: Exclude NULL Manager_ID
```sql
-- Only consider employees who had managers
AND e.manager_id IS NOT NULL
```

### Step 4: Order Results
```sql
-- Order by employee_id as requested
ORDER BY employee_id
```

## ✅ Optimized SQL Solution

**Solution 1: LEFT JOIN Approach**
```sql
SELECT e.employee_id
FROM Employees e
LEFT JOIN Employees manager ON e.manager_id = manager.employee_id
WHERE e.salary < 30000
  AND e.manager_id IS NOT NULL
  AND manager.employee_id IS NULL
ORDER BY e.employee_id;
```

### Alternative Solutions

**Solution 2: NOT EXISTS Subquery**
```sql
SELECT employee_id
FROM Employees e
WHERE salary < 30000
  AND manager_id IS NOT NULL
  AND NOT EXISTS (
      SELECT 1 
      FROM Employees m 
      WHERE m.employee_id = e.manager_id
  )
ORDER BY employee_id;
```

**Solution 3: NOT IN Approach (with NULL handling)**
```sql
SELECT employee_id
FROM Employees
WHERE salary < 30000
  AND manager_id IS NOT NULL
  AND manager_id NOT IN (
      SELECT employee_id 
      FROM Employees 
      WHERE employee_id IS NOT NULL
  )
ORDER BY employee_id;
```

**Solution 4: Window Function Analysis**
```sql
SELECT employee_id
FROM (
    SELECT 
        e.employee_id,
        e.salary,
        e.manager_id,
        m.employee_id as manager_exists
    FROM Employees e
    LEFT JOIN Employees m ON e.manager_id = m.employee_id
) emp_analysis
WHERE salary < 30000
  AND manager_id IS NOT NULL
  AND manager_exists IS NULL
ORDER BY employee_id;
```

**Solution 5: Comprehensive Employee Retention and Organizational Impact Analysis**
```sql
WITH employee_manager_analysis AS (
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        e.salary,
        m.employee_id as manager_exists,
        m.name as manager_name,
        m.salary as manager_salary,
        
        -- Employee classification
        CASE 
            WHEN e.manager_id IS NULL THEN 'Top Level Executive'
            WHEN m.employee_id IS NOT NULL THEN 'Active Reporting Employee'
            WHEN m.employee_id IS NULL THEN 'Orphaned Employee'
        END as employee_status,
        
        -- Salary tier classification
        CASE 
            WHEN e.salary >= 60000 THEN 'High Salary Tier'
            WHEN e.salary >= 40000 THEN 'Mid Salary Tier'
            WHEN e.salary >= 30000 THEN 'Standard Salary Tier'
            ELSE 'Low Salary Tier'
        END as salary_tier,
        
        -- Risk assessment for orphaned employees
        CASE 
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary < 30000 THEN 'High Risk - Low Paid Orphaned'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary >= 30000 THEN 'Medium Risk - Well Paid Orphaned'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NOT NULL THEN 'Low Risk - Active Management'
            WHEN e.manager_id IS NULL THEN 'Executive Level - No Risk'
        END as retention_risk,
        
        -- Department simulation based on employee_id
        CASE 
            WHEN e.employee_id % 7 = 0 THEN 'Engineering'
            WHEN e.employee_id % 7 = 1 THEN 'Sales'
            WHEN e.employee_id % 7 = 2 THEN 'Marketing'
            WHEN e.employee_id % 7 = 3 THEN 'Operations'
            WHEN e.employee_id % 7 = 4 THEN 'Human Resources'
            WHEN e.employee_id % 7 = 5 THEN 'Finance'
            ELSE 'Customer Service'
        END as department,
        
        -- Experience level simulation
        CASE 
            WHEN e.salary >= 50000 THEN 'Senior Level'
            WHEN e.salary >= 35000 THEN 'Mid Level'
            WHEN e.salary >= 25000 THEN 'Junior Level'
            ELSE 'Entry Level'
        END as experience_level,
        
        -- Performance impact assessment
        CASE 
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL THEN 'Performance Impact: No direct supervision + unclear accountability + potential productivity loss'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NOT NULL THEN 'Performance Impact: Active supervision + clear accountability + structured performance'
            WHEN e.manager_id IS NULL THEN 'Performance Impact: Self-directed + high accountability + executive performance'
        END as performance_impact,
        
        -- Career development concerns
        CASE 
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary < 30000 
            THEN 'Career Development: Critical concern + mentorship gap + advancement uncertainty + skill development risk'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL 
            THEN 'Career Development: Moderate concern + guidance gap + development coordination needed'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NOT NULL 
            THEN 'Career Development: Structured path + active mentorship + clear advancement + skill development'
            ELSE 'Career Development: Executive track + self-directed + strategic development + leadership focus'
        END as career_development_status,
        
        -- Compensation fairness analysis
        CASE 
            WHEN e.salary < 30000 AND e.manager_id IS NOT NULL AND m.employee_id IS NULL
            THEN 'Compensation Concern: Below market + management gap + retention risk + immediate review needed'
            WHEN e.salary < 30000 AND e.manager_id IS NOT NULL AND m.employee_id IS NOT NULL
            THEN 'Compensation Concern: Below market + active management + development focus + structured improvement'
            WHEN e.salary >= 30000 AND e.salary < 40000
            THEN 'Compensation Analysis: Market competitive + standard range + regular review + performance-based adjustment'
            ELSE 'Compensation Analysis: Above market + competitive advantage + retention strength + performance reward'
        END as compensation_analysis
    FROM Employees e
    LEFT JOIN Employees m ON e.manager_id = m.employee_id
),
organizational_health_assessment AS (
    SELECT 
        ema.*,
        
        -- Count of employees per category for organizational insights
        COUNT(*) OVER() as total_employees,
        COUNT(*) OVER(PARTITION BY employee_status) as employees_per_status,
        COUNT(*) OVER(PARTITION BY retention_risk) as employees_per_risk_level,
        COUNT(*) OVER(PARTITION BY department) as department_size,
        COUNT(*) OVER(PARTITION BY salary_tier) as salary_tier_size,
        
        -- Management span analysis
        COUNT(*) OVER(PARTITION BY manager_id) as direct_reports_count,
        
        -- Organizational structure insights
        CASE 
            WHEN employee_status = 'Orphaned Employee' AND retention_risk = 'High Risk - Low Paid Orphaned'
            THEN 'CRITICAL: Immediate intervention required + management assignment + compensation review + retention action'
            WHEN employee_status = 'Orphaned Employee' AND retention_risk = 'Medium Risk - Well Paid Orphaned'
            THEN 'HIGH PRIORITY: Management assignment + career planning + performance review + development focus'
            WHEN employee_status = 'Active Reporting Employee' AND salary_tier = 'Low Salary Tier'
            THEN 'MODERATE PRIORITY: Compensation review + performance evaluation + development opportunity + career planning'
            WHEN employee_status = 'Top Level Executive'
            THEN 'EXECUTIVE FOCUS: Strategic leadership + organizational development + succession planning + performance excellence'
            ELSE 'STANDARD: Regular management + performance monitoring + development support + career progression'
        END as organizational_action_priority,
        
        -- HR intervention strategies
        CASE 
            WHEN retention_risk = 'High Risk - Low Paid Orphaned'
            THEN 'HR Strategy: Emergency intervention + immediate manager assignment + compensation adjustment + retention bonus + career counseling'
            WHEN retention_risk = 'Medium Risk - Well Paid Orphaned'
            THEN 'HR Strategy: Priority management assignment + career development planning + performance review + mentorship program'
            WHEN retention_risk = 'Low Risk - Active Management' AND salary_tier = 'Low Salary Tier'
            THEN 'HR Strategy: Compensation review + performance improvement + skill development + promotion pathway'
            WHEN retention_risk = 'Executive Level - No Risk'
            THEN 'HR Strategy: Executive development + succession planning + strategic leadership + organizational vision'
            ELSE 'HR Strategy: Standard HR support + regular check-ins + performance management + career development'
        END as hr_intervention_strategy,
        
        -- Business continuity risk
        CASE 
            WHEN employee_status = 'Orphaned Employee' AND department_size <= 3
            THEN 'Business Risk: High - Small team + management gap + knowledge transfer risk + operational disruption'
            WHEN employee_status = 'Orphaned Employee' AND experience_level = 'Senior Level'
            THEN 'Business Risk: High - Senior expertise + management gap + decision-making void + project impact'
            WHEN employee_status = 'Orphaned Employee'
            THEN 'Business Risk: Medium - Management gap + supervision void + accountability unclear + performance uncertainty'
            WHEN salary_tier = 'Low Salary Tier' AND direct_reports_count > 0
            THEN 'Business Risk: Low-Medium - Compensation disparity + management responsibility + team dynamics'
            ELSE 'Business Risk: Low - Stable structure + clear management + operational continuity + performance clarity'
        END as business_continuity_risk,
        
        -- Leadership development opportunities
        CASE 
            WHEN employee_status = 'Active Reporting Employee' AND salary >= 40000 AND direct_reports_count = 0
            THEN 'Leadership Opportunity: Management readiness + leadership potential + promotion candidate + development investment'
            WHEN employee_status = 'Top Level Executive' AND department_size >= 5
            THEN 'Leadership Opportunity: Executive excellence + team leadership + organizational impact + strategic contribution'
            WHEN employee_status = 'Orphaned Employee' AND experience_level = 'Senior Level'
            THEN 'Leadership Opportunity: Acting management + interim leadership + step-up opportunity + crisis leadership'
            WHEN retention_risk = 'Low Risk - Active Management' AND salary >= 35000
            THEN 'Leadership Opportunity: Team leadership + project management + cross-functional coordination + skill development'
            ELSE 'Leadership Opportunity: Individual contributor + skill building + expertise development + performance excellence'
        END as leadership_development_opportunity
    FROM employee_manager_analysis ema
)
SELECT 
    employee_id,
    name,
    manager_id,
    salary,
    department,
    employee_status,
    salary_tier,
    retention_risk,
    organizational_action_priority,
    hr_intervention_strategy,
    business_continuity_risk,
    leadership_development_opportunity,
    performance_impact,
    career_development_status,
    compensation_analysis,
    
    -- Organizational metrics
    total_employees,
    employees_per_status,
    department_size,
    direct_reports_count,
    experience_level
FROM organizational_health_assessment
WHERE retention_risk = 'High Risk - Low Paid Orphaned'
ORDER BY 
    CASE organizational_action_priority
        WHEN 'CRITICAL: Immediate intervention required + management assignment + compensation review + retention action' THEN 1
        WHEN 'HIGH PRIORITY: Management assignment + career planning + performance review + development focus' THEN 2
        ELSE 3
    END,
    salary ASC,
    employee_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Organizational Resilience and Leadership Continuity Platform**
```sql
-- "Design comprehensive organizational resilience system for Amazon's complex hierarchical structure and succession planning"

WITH amazon_organizational_analysis AS (
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        e.salary,
        m.employee_id as manager_exists,
        m.name as manager_name,
        
        -- Amazon level classification
        CASE 
            WHEN e.salary >= 200000 THEN 'L8-L10 Principal/VP/Director'
            WHEN e.salary >= 150000 THEN 'L7 Senior Manager/Principal Engineer'
            WHEN e.salary >= 100000 THEN 'L6 Manager/Senior Engineer'
            WHEN e.salary >= 70000 THEN 'L5 Senior SDE/Specialist'
            WHEN e.salary >= 50000 THEN 'L4 SDE II/Professional'
            WHEN e.salary >= 35000 THEN 'L3 SDE I/Associate'
            WHEN e.salary >= 25000 THEN 'L2 Support/Junior'
            ELSE 'L1 Entry Level'
        END as amazon_level,
        
        -- Amazon business unit simulation
        CASE 
            WHEN e.employee_id % 10 = 0 THEN 'AWS'
            WHEN e.employee_id % 10 = 1 THEN 'Prime Video & Studios'
            WHEN e.employee_id % 10 = 2 THEN 'Alexa & Voice'
            WHEN e.employee_id % 10 = 3 THEN 'Retail & Marketplace'
            WHEN e.employee_id % 10 = 4 THEN 'Fulfillment & Logistics'
            WHEN e.employee_id % 10 = 5 THEN 'Advertising & DSP'
            WHEN e.employee_id % 10 = 6 THEN 'Devices & Hardware'
            WHEN e.employee_id % 10 = 7 THEN 'International'
            WHEN e.employee_id % 10 = 8 THEN 'Corporate Functions'
            ELSE 'Emerging Technologies'
        END as amazon_business_unit,
        
        -- Organizational impact level
        CASE 
            WHEN e.manager_id IS NULL THEN 'Executive Leadership'
            WHEN m.employee_id IS NOT NULL THEN 'Active Contributor'
            WHEN m.employee_id IS NULL AND e.salary >= 50000 THEN 'High-Impact Orphaned'
            WHEN m.employee_id IS NULL AND e.salary < 30000 THEN 'At-Risk Orphaned'
            ELSE 'Standard Orphaned'
        END as organizational_impact,
        
        -- Amazon Leadership Principles risk assessment
        CASE 
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary < 30000
            THEN 'CRITICAL RISK: Ownership gap + customer obsession risk + delivery impact + leadership void'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary >= 30000
            THEN 'HIGH RISK: Leadership gap + ownership transfer needed + succession planning + continuity concern'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NOT NULL
            THEN 'LOW RISK: Active leadership + clear ownership + customer focus + delivery assurance'
            ELSE 'EXECUTIVE LEVEL: Strategic leadership + organizational ownership + vision setting + long-term thinking'
        END as leadership_principles_risk,
        
        -- Business continuity for Amazon scale
        CASE 
            WHEN amazon_business_unit IN ('AWS', 'Retail & Marketplace') AND m.employee_id IS NULL AND e.manager_id IS NOT NULL
            THEN 'MISSION CRITICAL: Core business impact + revenue risk + customer experience + immediate escalation'
            WHEN amazon_business_unit IN ('Prime Video & Studios', 'Alexa & Voice') AND m.employee_id IS NULL AND e.manager_id IS NOT NULL
            THEN 'HIGH BUSINESS IMPACT: Innovation disruption + product development + customer engagement + strategic concern'
            WHEN amazon_business_unit IN ('Fulfillment & Logistics') AND m.employee_id IS NULL AND e.manager_id IS NOT NULL
            THEN 'OPERATIONAL CRITICAL: Supply chain risk + delivery impact + customer satisfaction + operational disruption'
            WHEN m.employee_id IS NULL AND e.manager_id IS NOT NULL
            THEN 'BUSINESS IMPACT: Functional disruption + team coordination + project delivery + performance management'
            ELSE 'STABLE OPERATIONS: Normal business continuity + active management + clear accountability + performance tracking'
        END as amazon_business_continuity,
        
        -- Career development and progression at Amazon scale
        CASE 
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND e.salary < 30000
            THEN 'Career Crisis: Mentorship void + advancement blocked + skill development gap + retention emergency'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND amazon_level LIKE '%L3%'
            THEN 'Development Risk: Early career disruption + learning gap + progression uncertainty + guidance needed'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND amazon_level LIKE '%L4-L5%'
            THEN 'Professional Impact: Mid-career disruption + leadership development + advancement planning + interim management'
            WHEN e.manager_id IS NOT NULL AND m.employee_id IS NULL AND amazon_level LIKE '%L6%'
            THEN 'Leadership Opportunity: Acting management + step-up potential + leadership development + succession candidate'
            ELSE 'Standard Development: Structured career path + active mentorship + clear progression + leadership pipeline'
        END as amazon_career_development,
        
        -- Customer obsession impact
        CASE 
            WHEN amazon_business_unit IN ('Retail & Marketplace', 'Customer Service') AND organizational_impact = 'At-Risk Orphaned'
            THEN 'Customer Impact: Direct customer experience risk + service quality concern + satisfaction threat + immediate attention'
            WHEN amazon_business_unit IN ('AWS', 'Prime Video & Studios') AND organizational_impact LIKE '%Orphaned%'
            THEN 'Customer Impact: Product experience risk + innovation delay + feature development + customer value delivery'
            WHEN amazon_business_unit IN ('Fulfillment & Logistics') AND organizational_impact LIKE '%Orphaned%'
            THEN 'Customer Impact: Delivery experience risk + operational efficiency + customer convenience + logistics excellence'
            WHEN organizational_impact = 'Active Contributor'
            THEN 'Customer Impact: Positive customer focus + active contribution + customer-centric decisions + value delivery'
            ELSE 'Customer Impact: Strategic customer vision + long-term customer value + customer experience innovation + market leadership'
        END as customer_obsession_impact
    FROM Employees e
    LEFT JOIN Employees m ON e.manager_id = m.employee_id
),
amazon_succession_planning AS (
    SELECT 
        aoa.*,
        
        -- Succession planning assessment
        CASE 
            WHEN organizational_impact = 'At-Risk Orphaned' AND amazon_level LIKE '%L1-L3%'
            THEN 'Succession: Entry/Junior level + immediate manager assignment + basic supervision + standard development'
            WHEN organizational_impact = 'High-Impact Orphaned' AND amazon_level LIKE '%L4-L5%'
            THEN 'Succession: Mid-level impact + acting manager needed + leadership development + succession opportunity'
            WHEN organizational_impact = 'High-Impact Orphaned' AND amazon_level LIKE '%L6%'
            THEN 'Succession: Management level + interim leadership role + succession planning + organizational development'
            WHEN organizational_impact = 'Executive Leadership'
            THEN 'Succession: Executive succession + organizational continuity + strategic leadership + long-term planning'
            ELSE 'Succession: Standard progression + regular development + performance management + career advancement'
        END as succession_planning_strategy,
        
        -- Amazon-specific intervention based on Leadership Principles
        CASE 
            WHEN leadership_principles_risk = 'CRITICAL RISK: Ownership gap + customer obsession risk + delivery impact + leadership void'
            THEN 'LP Intervention: IMMEDIATE - Assign ownership + restore customer focus + ensure delivery + establish leadership + emergency response'
            WHEN leadership_principles_risk = 'HIGH RISK: Leadership gap + ownership transfer needed + succession planning + continuity concern'
            THEN 'LP Intervention: PRIORITY - Leadership transition + ownership clarification + succession execution + continuity planning'
            WHEN amazon_business_continuity = 'MISSION CRITICAL: Core business impact + revenue risk + customer experience + immediate escalation'
            THEN 'LP Intervention: ESCALATED - Executive attention + business continuity + revenue protection + customer experience assurance'
            WHEN customer_obsession_impact LIKE '%Direct customer experience risk%'
            THEN 'LP Intervention: CUSTOMER FOCUS - Customer experience protection + service continuity + satisfaction assurance + quality maintenance'
            ELSE 'LP Intervention: STANDARD - Regular leadership development + ownership clarity + customer focus + continuous improvement'
        END as amazon_leadership_intervention,
        
        -- Bias for Action - immediate response recommendations
        CASE 
            WHEN organizational_impact = 'At-Risk Orphaned' AND amazon_business_unit IN ('AWS', 'Retail & Marketplace')
            THEN 'Bias for Action: IMMEDIATE (24hrs) - Emergency manager assignment + business continuity + customer protection + escalation protocol'
            WHEN leadership_principles_risk LIKE 'CRITICAL RISK%'
            THEN 'Bias for Action: URGENT (48hrs) - Leadership assignment + ownership transfer + customer assurance + performance continuity'
            WHEN amazon_business_continuity LIKE 'MISSION CRITICAL%'
            THEN 'Bias for Action: HIGH PRIORITY (72hrs) - Business impact mitigation + operational continuity + strategic intervention'
            WHEN organizational_impact LIKE '%Orphaned%'
            THEN 'Bias for Action: STANDARD (1-2 weeks) - Management assignment + development planning + performance review + career guidance'
            ELSE 'Bias for Action: ROUTINE - Regular check-ins + performance monitoring + development support + career progression'
        END as bias_for_action_timeline,
        
        -- Think Big - long-term organizational development
        CASE 
            WHEN amazon_level LIKE '%L6%' AND organizational_impact LIKE '%Orphaned%'
            THEN 'Think Big: Leadership pipeline development + succession planning + organizational resilience + management excellence + future leaders'
            WHEN amazon_business_unit IN ('AWS', 'Alexa & Voice', 'Emerging Technologies') AND organizational_impact LIKE '%Orphaned%'
            THEN 'Think Big: Innovation leadership + technology advancement + future capabilities + competitive advantage + market transformation'
            WHEN customer_obsession_impact LIKE '%Strategic customer vision%'
            THEN 'Think Big: Customer experience revolution + market leadership + long-term value + customer-centric innovation + industry transformation'
            WHEN organizational_impact = 'Executive Leadership'
            THEN 'Think Big: Organizational transformation + global expansion + industry leadership + long-term vision + sustainable growth'
            ELSE 'Think Big: Professional development + skill advancement + career growth + contribution expansion + organizational value'
        END as think_big_development
    FROM amazon_organizational_analysis aoa
)
SELECT 
    employee_id,
    name,
    manager_id,
    salary,
    amazon_level,
    amazon_business_unit,
    organizational_impact,
    leadership_principles_risk,
    amazon_business_continuity,
    amazon_career_development,
    customer_obsession_impact,
    succession_planning_strategy,
    amazon_leadership_intervention,
    bias_for_action_timeline,
    think_big_development
FROM amazon_succession_planning
WHERE organizational_impact = 'At-Risk Orphaned'
ORDER BY 
    CASE amazon_business_continuity
        WHEN 'MISSION CRITICAL: Core business impact + revenue risk + customer experience + immediate escalation' THEN 1
        WHEN 'OPERATIONAL CRITICAL: Supply chain risk + delivery impact + customer satisfaction + operational disruption' THEN 2
        WHEN 'HIGH BUSINESS IMPACT: Innovation disruption + product development + customer engagement + strategic concern' THEN 3
        ELSE 4
    END,
    salary ASC,
    employee_id;
```

I'll create one final brief extension and then complete the last question:

#### 2. **Real-time Organizational Health Monitoring and Alert System**
```sql
-- "Real-time organizational health monitoring with automated alerts and intervention triggers"

WITH real_time_org_health AS (
    SELECT 
        e.employee_id,
        e.salary,
        e.manager_id,
        CASE WHEN m.employee_id IS NULL AND e.manager_id IS NOT NULL THEN 1 ELSE 0 END as orphaned_flag,
        
        -- Real-time alert levels
        CASE 
            WHEN e.salary < 30000 AND m.employee_id IS NULL AND e.manager_id IS NOT NULL THEN 'CRITICAL ALERT'
            WHEN e.salary < 40000 AND m.employee_id IS NULL AND e.manager_id IS NOT NULL THEN 'HIGH ALERT'
            WHEN m.employee_id IS NULL AND e.manager_id IS NOT NULL THEN 'MEDIUM ALERT'
            ELSE 'NORMAL STATUS'
        END as alert_level,
        
        -- Automated intervention triggers
        CASE 
            WHEN e.salary < 30000 AND m.employee_id IS NULL AND e.manager_id IS NOT NULL 
            THEN 'AUTO-TRIGGER: Emergency HR intervention + immediate manager assignment + retention review'
            WHEN m.employee_id IS NULL AND e.manager_id IS NOT NULL 
            THEN 'AUTO-TRIGGER: Manager assignment + performance review + development planning'
            ELSE 'AUTO-TRIGGER: Standard monitoring + regular check-ins + performance tracking'
        END as auto_intervention
    FROM Employees e
    LEFT JOIN Employees m ON e.manager_id = m.employee_id
)
SELECT 
    employee_id,
    alert_level,
    auto_intervention
FROM real_time_org_health
WHERE orphaned_flag = 1 AND salary < 30000
ORDER BY salary ASC, employee_id;
```

## 🔗 Related LeetCode Questions

1. **#1731 - The Number of Employees Which Report to Each Employee** (Employee hierarchy relationships)
2. **#1789 - Primary Department for Each Employee** (Employee organizational structure)
3. **#1741 - Find Total Time Spent by Each Employee** (Employee performance analysis)
4. **#1729 - Find Followers Count** (Relationship counting and analysis)
5. **#1777 - Product's Price for Each Store** (Missing data analysis patterns)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Missing Data Detection**: Use LEFT JOIN with NULL check or NOT EXISTS
2. **Multiple Conditions**: Combine salary filter with missing manager condition
3. **NULL Handling**: Distinguish between NULL manager_id and missing manager records
4. **Referential Integrity**: Detect broken foreign key relationships

### 🚀 **Amazon Interview Tips**
1. **Explain missing data logic**: "Manager left means their record deleted but references remain"
2. **Discuss multiple conditions**: "Need both salary < 30000 AND manager missing"
3. **Address NULL handling**: "Exclude employees with NULL manager_id - they never had managers"
4. **Consider data quality**: "This query helps identify orphaned records needing attention"

### 🔧 **Common Patterns**
- LEFT JOIN with NULL check for missing references
- NOT EXISTS subquery for non-matching records
- Multiple WHERE conditions with AND logic
- Referential integrity validation queries

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting NULL manager_id check** (including employees who never had managers)
2. **Using INNER JOIN** (would exclude the missing managers we want to find)
3. **NOT IN with NULLs** (can cause unexpected behavior if employee_id contains NULL)
4. **Wrong condition order** (not optimizing filter sequence for performance)

### 🔍 **Performance Considerations**
- INDEX on manager_id for efficient LEFT JOIN
- INDEX on salary for fast salary filtering
- Consider filtering salary first before JOIN for better performance
- NOT EXISTS can be faster than LEFT JOIN for large datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Ownership**: Take responsibility for employees who lost management structure
- **Customer Obsession**: Ensure employee well-being to maintain customer service quality
- **Bias for Action**: Quickly identify and address organizational gaps
- **Deliver Results**: Maintain organizational effectiveness through proper management structure

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find all orphaned employees regardless of salary**
2. **Calculate average salary of orphaned vs non-orphaned employees**
3. **Find managers who have orphaned employees reporting to them**
4. **List employees whose manager earns less than them**

Remember: Organizational health monitoring like this is crucial for Amazon's massive workforce management, ensuring no employee falls through the cracks when managers leave or organizational changes occur!