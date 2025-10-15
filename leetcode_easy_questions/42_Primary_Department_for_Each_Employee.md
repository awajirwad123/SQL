# LeetCode Easy #1789: Primary Department for Each Employee

## 📋 Problem Statement

Write a SQL query to find the **primary department** for each employee.

Return the result table in **any order**.

## 🗄️ Table Schema

**Employee Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| employee_id   | int     |
| department_id | int     |
| primary_flag  | varchar |
+---------------+---------+
```
- (employee_id, department_id) is the primary key for this table.
- employee_id is the id of the employee.
- department_id is the id of the department to which the employee belongs.
- primary_flag is an ENUM of type ('Y', 'N'). If the flag is 'Y', the department is the primary department for the employee. If the flag is 'N', the department is not the primary department for the employee.

## 📊 Sample Data

**Employee Table:**
| employee_id | department_id | primary_flag |
|-------------|---------------|--------------|
| 1           | 1             | N            |
| 2           | 1             | Y            |
| 2           | 2             | N            |
| 3           | 3             | N            |
| 4           | 2             | N            |
| 4           | 3             | Y            |
| 4           | 4             | N            |

**Expected Output:**
| employee_id | department_id |
|-------------|---------------|
| 1           | 1             |
| 2           | 1             |
| 3           | 3             |
| 4           | 3             |

**Explanation:**
- Employee 1: Only belongs to department 1, so department 1 is primary
- Employee 2: Primary flag 'Y' for department 1, so department 1 is primary  
- Employee 3: Only belongs to department 3, so department 3 is primary
- Employee 4: Primary flag 'Y' for department 3, so department 3 is primary

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Find primary department per employee
- Two scenarios: explicit primary flag 'Y' OR employee only in one department
- Handle both single and multiple department memberships
- Return employee_id and their primary department_id

### 2. **Key Insights**
- If primary_flag = 'Y', that's the primary department
- If employee only in one department, that department is primary by default
- Use UNION or CASE to handle both scenarios
- Need to ensure one result per employee

### 3. **Interview Discussion Points**
- "Two cases: explicit primary flag or single department membership"
- "Primary flag 'Y' takes precedence over single department logic"
- "Each employee should appear exactly once in result"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Explicit Primary Departments
```sql
-- Find departments marked with primary_flag = 'Y'
WHERE primary_flag = 'Y'
```

### Step 2: Identify Single Department Employees
```sql
-- Find employees with only one department (implicit primary)
GROUP BY employee_id HAVING COUNT(*) = 1
```

### Step 3: Combine Both Scenarios
```sql
-- UNION both cases to get complete result
-- Ensure no duplicates
```

## ✅ Optimized SQL Solution

**Solution 1: UNION Approach**
```sql
SELECT employee_id, department_id
FROM Employee
WHERE primary_flag = 'Y'

UNION

SELECT employee_id, department_id
FROM Employee
GROUP BY employee_id, department_id
HAVING COUNT(*) = 1 AND SUM(CASE WHEN primary_flag = 'Y' THEN 1 ELSE 0 END) = 0;
```

### Alternative Solutions

**Solution 2: Window Function Approach**
```sql
SELECT employee_id, department_id
FROM (
    SELECT 
        employee_id, 
        department_id,
        primary_flag,
        COUNT(*) OVER(PARTITION BY employee_id) as dept_count
    FROM Employee
) t
WHERE primary_flag = 'Y' 
   OR (dept_count = 1 AND primary_flag = 'N');
```

**Solution 3: Conditional Logic with EXISTS**
```sql
SELECT employee_id, department_id
FROM Employee e1
WHERE primary_flag = 'Y'
   OR (primary_flag = 'N' 
       AND NOT EXISTS (
           SELECT 1 FROM Employee e2 
           WHERE e2.employee_id = e1.employee_id 
             AND e2.primary_flag = 'Y'
       )
       AND (
           SELECT COUNT(*) FROM Employee e3 
           WHERE e3.employee_id = e1.employee_id
       ) = 1
   );
```

**Solution 4: ROW_NUMBER with Prioritization**
```sql
SELECT employee_id, department_id
FROM (
    SELECT 
        employee_id,
        department_id,
        primary_flag,
        COUNT(*) OVER(PARTITION BY employee_id) as total_depts,
        ROW_NUMBER() OVER(
            PARTITION BY employee_id 
            ORDER BY 
                CASE WHEN primary_flag = 'Y' THEN 0 ELSE 1 END,
                department_id
        ) as rn
    FROM Employee
) ranked
WHERE rn = 1;
```

**Solution 5: Enterprise Employee Department Analytics System**
```sql
WITH employee_department_analysis AS (
    SELECT 
        employee_id,
        department_id,
        primary_flag,
        
        -- Department analytics
        COUNT(*) OVER(PARTITION BY employee_id) as total_departments,
        COUNT(CASE WHEN primary_flag = 'Y' THEN 1 END) OVER(PARTITION BY employee_id) as primary_departments,
        COUNT(*) OVER(PARTITION BY department_id) as department_size,
        
        -- Employee classification
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 1 THEN 'Single Department Employee'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 2 THEN 'Dual Department Employee'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) >= 3 THEN 'Multi Department Employee'
        END as employee_type,
        
        -- Department role simulation
        CASE 
            WHEN primary_flag = 'Y' THEN 'Primary Role'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 1 THEN 'Single Role'
            ELSE 'Secondary Role'
        END as role_type,
        
        -- Cross-functional indicator
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY employee_id) >= 3 THEN 'High Cross-Functional'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 2 THEN 'Moderate Cross-Functional'
            ELSE 'Single Function'
        END as cross_functional_level,
        
        -- Department category simulation
        CASE 
            WHEN department_id % 5 = 1 THEN 'Technology'
            WHEN department_id % 5 = 2 THEN 'Operations'
            WHEN department_id % 5 = 3 THEN 'Sales & Marketing'
            WHEN department_id % 5 = 4 THEN 'Human Resources'
            ELSE 'Finance & Administration'
        END as department_category,
        
        -- Organizational level simulation
        CASE 
            WHEN employee_id % 10 <= 2 THEN 'Executive Level'
            WHEN employee_id % 10 <= 5 THEN 'Management Level'
            WHEN employee_id % 10 <= 8 THEN 'Professional Level'
            ELSE 'Associate Level'
        END as organizational_level,
        
        -- Specialization indicator
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 1 AND primary_flag = 'N' THEN 'Specialist'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 1 THEN 'Dedicated Specialist'
            WHEN primary_flag = 'Y' AND COUNT(*) OVER(PARTITION BY employee_id) >= 2 THEN 'Cross-Functional Leader'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) >= 2 THEN 'Multi-Departmental Contributor'
            ELSE 'Standard Employee'
        END as specialization_type
    FROM Employee
),
primary_department_identification AS (
    SELECT 
        eda.*,
        
        -- Primary department logic
        CASE 
            WHEN primary_flag = 'Y' THEN 'Explicit Primary'
            WHEN total_departments = 1 THEN 'Implicit Primary (Single Department)'
            ELSE 'Secondary Department'
        END as department_designation,
        
        -- Career development insights
        CASE 
            WHEN employee_type = 'Multi Department Employee' AND primary_flag = 'Y'
            THEN 'Career Development: Leadership track + cross-functional experience + management potential + strategic roles'
            WHEN employee_type = 'Dual Department Employee' AND primary_flag = 'Y'
            THEN 'Career Development: Dual expertise + collaboration skills + flexibility + specialization options'
            WHEN employee_type = 'Single Department Employee'
            THEN 'Career Development: Deep specialization + expert track + department leadership + technical excellence'
            WHEN cross_functional_level = 'High Cross-Functional'
            THEN 'Career Development: Versatile contributor + broad knowledge + integration skills + consulting potential'
            ELSE 'Career Development: Standard progression + skill development + department growth + focused advancement'
        END as career_development_path,
        
        -- Resource allocation insights
        CASE 
            WHEN primary_flag = 'Y' AND total_departments >= 3
            THEN 'Resource Allocation: Primary focus (60%) + secondary roles (40%) + coordination responsibility + integration leadership'
            WHEN primary_flag = 'Y' AND total_departments = 2
            THEN 'Resource Allocation: Primary focus (70%) + secondary role (30%) + collaboration + knowledge transfer'
            WHEN total_departments = 1
            THEN 'Resource Allocation: Full department focus (100%) + deep specialization + expert contribution + department dedication'
            WHEN primary_flag = 'N' AND total_departments >= 2
            THEN 'Resource Allocation: Supporting role + collaborative contribution + expertise sharing + flexible assignment'
            ELSE 'Resource Allocation: Standard assignment + focused contribution + clear responsibility + balanced workload'
        END as resource_allocation_strategy,
        
        -- Organizational impact assessment
        CASE 
            WHEN cross_functional_level = 'High Cross-Functional' AND organizational_level = 'Executive Level'
            THEN 'Organizational Impact: Strategic leadership + cross-departmental coordination + high-level decision making + enterprise influence'
            WHEN employee_type = 'Multi Department Employee' AND organizational_level = 'Management Level'
            THEN 'Organizational Impact: Operational leadership + team coordination + process integration + departmental bridge'
            WHEN specialization_type = 'Dedicated Specialist' AND department_size >= 10
            THEN 'Organizational Impact: Expert contribution + knowledge leadership + skill mentoring + technical excellence'
            WHEN cross_functional_level = 'Moderate Cross-Functional'
            THEN 'Organizational Impact: Collaborative contribution + knowledge sharing + flexibility + adaptation capability'
            ELSE 'Organizational Impact: Focused contribution + reliable performance + department stability + consistent delivery'
        END as organizational_impact,
        
        -- Performance and productivity insights
        CASE 
            WHEN primary_flag = 'Y' AND total_departments >= 3
            THEN 'Performance Profile: High complexity management + multi-tasking excellence + leadership skills + strategic thinking'
            WHEN total_departments = 1 AND department_size <= 5
            THEN 'Performance Profile: Deep expertise + focused excellence + specialized skills + intimate knowledge'
            WHEN employee_type = 'Dual Department Employee'
            THEN 'Performance Profile: Balanced skills + collaboration ability + adaptability + cross-training capability'
            WHEN cross_functional_level = 'High Cross-Functional'
            THEN 'Performance Profile: Versatile skills + broad knowledge + integration ability + complex problem solving'
            ELSE 'Performance Profile: Standard competency + reliable performance + consistent contribution + steady growth'
        END as performance_profile,
        
        -- Management and supervision considerations
        CASE 
            WHEN organizational_level = 'Executive Level' AND primary_flag = 'Y'
            THEN 'Management Considerations: Strategic oversight + cross-departmental responsibility + high-level accountability + enterprise leadership'
            WHEN organizational_level = 'Management Level' AND total_departments >= 2
            THEN 'Management Considerations: Multi-department coordination + resource balancing + team integration + operational leadership'
            WHEN specialization_type = 'Cross-Functional Leader'
            THEN 'Management Considerations: Matrix management + collaborative leadership + expertise coordination + knowledge integration'
            WHEN employee_type = 'Single Department Employee'
            THEN 'Management Considerations: Direct supervision + clear accountability + focused development + specialized management'
            ELSE 'Management Considerations: Standard supervision + balanced guidance + collaborative support + flexible management'
        END as management_considerations
    FROM employee_department_analysis eda
)
SELECT 
    employee_id,
    department_id,
    department_category,
    employee_type,
    organizational_level,
    specialization_type,
    department_designation,
    career_development_path,
    resource_allocation_strategy,
    organizational_impact,
    performance_profile,
    management_considerations,
    
    -- Key metrics
    total_departments,
    department_size,
    cross_functional_level,
    role_type
FROM primary_department_identification
WHERE department_designation IN ('Explicit Primary', 'Implicit Primary (Single Department)')
ORDER BY 
    CASE organizational_level
        WHEN 'Executive Level' THEN 1
        WHEN 'Management Level' THEN 2
        WHEN 'Professional Level' THEN 3
        ELSE 4
    END,
    total_departments DESC,
    employee_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Organizational Structure and Cross-Functional Team Analytics**
```sql
-- "Design comprehensive organizational analytics for Amazon's complex matrix structure and cross-functional teams"

WITH amazon_org_structure_analysis AS (
    SELECT 
        employee_id,
        department_id,
        primary_flag,
        
        -- Amazon organization mapping
        CASE 
            WHEN department_id % 12 = 1 THEN 'AWS'
            WHEN department_id % 12 = 2 THEN 'Prime Video & Studios'
            WHEN department_id % 12 = 3 THEN 'Alexa & Voice Services'
            WHEN department_id % 12 = 4 THEN 'Advertising & DSP'
            WHEN department_id % 12 = 5 THEN 'Fulfillment & Logistics'
            WHEN department_id % 12 = 6 THEN 'Retail & Marketplace'
            WHEN department_id % 12 = 7 THEN 'Devices & Hardware'
            WHEN department_id % 12 = 8 THEN 'Prime & Membership'
            WHEN department_id % 12 = 9 THEN 'Corporate Functions'
            WHEN department_id % 12 = 10 THEN 'International Expansion'
            WHEN department_id % 12 = 11 THEN 'Machine Learning & AI'
            ELSE 'Emerging Technologies'
        END as amazon_division,
        
        -- Amazon level classification
        CASE 
            WHEN employee_id % 8 = 0 THEN 'L8-L10 Principal/Director'
            WHEN employee_id % 8 = 1 THEN 'L7 Senior Manager'
            WHEN employee_id % 8 = 2 THEN 'L6 Manager'
            WHEN employee_id % 8 = 3 THEN 'L5 Senior SDE/Specialist'
            WHEN employee_id % 8 = 4 THEN 'L4 SDE II/Professional'
            WHEN employee_id % 8 = 5 THEN 'L3 SDE I/Associate'
            WHEN employee_id % 8 = 6 THEN 'L2 Support Engineer'
            ELSE 'L1 Entry Level'
        END as amazon_level,
        
        -- Cross-functional role type
        CASE 
            WHEN COUNT(*) OVER(PARTITION BY employee_id) >= 3 THEN 'Multi-Division Contributor'
            WHEN COUNT(*) OVER(PARTITION BY employee_id) = 2 THEN 'Dual-Division Specialist'
            ELSE 'Single-Division Expert'
        END as cross_functional_type,
        
        -- Amazon business impact level
        CASE 
            WHEN department_id % 12 IN (1, 3, 6) AND employee_id % 8 <= 3 THEN 'Strategic Business Impact'
            WHEN department_id % 12 IN (2, 4, 5) AND employee_id % 8 <= 4 THEN 'High Business Impact'
            WHEN department_id % 12 IN (7, 8, 9) THEN 'Operational Business Impact'
            ELSE 'Standard Business Impact'
        END as business_impact_level,
        
        -- Innovation and technology focus
        CASE 
            WHEN amazon_division IN ('AWS', 'Machine Learning & AI', 'Emerging Technologies') THEN 'High Innovation Focus'
            WHEN amazon_division IN ('Alexa & Voice Services', 'Devices & Hardware', 'Prime Video & Studios') THEN 'Medium Innovation Focus'
            ELSE 'Operational Excellence Focus'
        END as innovation_focus,
        
        -- Customer obsession alignment
        CASE 
            WHEN amazon_division IN ('Retail & Marketplace', 'Prime & Membership', 'Fulfillment & Logistics') THEN 'Direct Customer Impact'
            WHEN amazon_division IN ('AWS', 'Advertising & DSP', 'Alexa & Voice Services') THEN 'Customer Experience Enhancement'
            WHEN amazon_division IN ('Prime Video & Studios', 'Devices & Hardware') THEN 'Customer Entertainment & Engagement'
            ELSE 'Customer Support Infrastructure'
        END as customer_obsession_alignment
    FROM Employee
),
amazon_cross_functional_intelligence AS (
    SELECT 
        employee_id,
        
        -- Primary department identification
        MAX(CASE WHEN primary_flag = 'Y' OR 
                      (COUNT(*) OVER(PARTITION BY employee_id) = 1 AND primary_flag = 'N') 
                 THEN department_id END) as primary_department_id,
        MAX(CASE WHEN primary_flag = 'Y' OR 
                      (COUNT(*) OVER(PARTITION BY employee_id) = 1 AND primary_flag = 'N') 
                 THEN amazon_division END) as primary_division,
        MAX(CASE WHEN primary_flag = 'Y' OR 
                      (COUNT(*) OVER(PARTITION BY employee_id) = 1 AND primary_flag = 'N') 
                 THEN amazon_level END) as amazon_level,
        
        -- Cross-functional analysis
        COUNT(*) as total_divisions,
        COUNT(CASE WHEN primary_flag = 'Y' THEN 1 END) as explicit_primary_count,
        STRING_AGG(amazon_division, ', ') as all_divisions,
        
        -- Business impact aggregation
        COUNT(CASE WHEN business_impact_level = 'Strategic Business Impact' THEN 1 END) as strategic_roles,
        COUNT(CASE WHEN business_impact_level = 'High Business Impact' THEN 1 END) as high_impact_roles,
        COUNT(CASE WHEN innovation_focus = 'High Innovation Focus' THEN 1 END) as innovation_roles,
        COUNT(CASE WHEN customer_obsession_alignment = 'Direct Customer Impact' THEN 1 END) as customer_facing_roles,
        
        -- Cross-functional leadership assessment
        CASE 
            WHEN COUNT(*) >= 3 AND COUNT(CASE WHEN business_impact_level = 'Strategic Business Impact' THEN 1 END) >= 2
            THEN 'Cross-Functional Strategic Leader'
            WHEN COUNT(*) >= 2 AND COUNT(CASE WHEN innovation_focus = 'High Innovation Focus' THEN 1 END) >= 1
            THEN 'Cross-Functional Innovation Leader'
            WHEN COUNT(*) >= 2 AND COUNT(CASE WHEN customer_obsession_alignment = 'Direct Customer Impact' THEN 1 END) >= 1
            THEN 'Cross-Functional Customer Leader'
            WHEN COUNT(*) = 1
            THEN 'Single-Division Specialist'
            ELSE 'Cross-Functional Contributor'
        END as leadership_classification,
        
        -- Amazon Leadership Principles application
        CASE 
            WHEN COUNT(CASE WHEN customer_obsession_alignment = 'Direct Customer Impact' THEN 1 END) >= 2
            THEN 'Customer Obsession: Multi-division customer advocacy + comprehensive customer experience + end-to-end ownership'
            WHEN COUNT(CASE WHEN customer_obsession_alignment = 'Direct Customer Impact' THEN 1 END) = 1
            THEN 'Customer Obsession: Direct customer focus + customer experience optimization + customer-centric decisions'
            WHEN COUNT(CASE WHEN customer_obsession_alignment LIKE '%Customer%' THEN 1 END) >= 1
            THEN 'Customer Obsession: Customer experience support + customer success enablement + customer value delivery'
            ELSE 'Customer Obsession: Customer support infrastructure + customer success foundation + customer value creation'
        END as customer_obsession_application,
        
        -- Ownership across divisions
        CASE 
            WHEN COUNT(*) >= 3 AND MAX(amazon_level) LIKE '%Director%'
            THEN 'Ownership: Enterprise-wide accountability + multi-division coordination + strategic oversight + long-term thinking'
            WHEN COUNT(*) >= 2 AND strategic_roles >= 1
            THEN 'Ownership: Cross-division responsibility + strategic alignment + collaborative leadership + shared accountability'
            WHEN COUNT(*) = 1 AND high_impact_roles >= 1
            THEN 'Ownership: Deep division expertise + specialized accountability + expert ownership + technical leadership'
            ELSE 'Ownership: Focused responsibility + reliable execution + quality delivery + continuous improvement'
        END as ownership_demonstration
    FROM amazon_org_structure_analysis
    GROUP BY employee_id
),
amazon_talent_development_insights AS (
    SELECT 
        acfi.*,
        
        -- Talent development pathways
        CASE 
            WHEN leadership_classification = 'Cross-Functional Strategic Leader'
            THEN 'Talent Development: Executive leadership track + cross-business strategy + organizational development + succession planning'
            WHEN leadership_classification = 'Cross-Functional Innovation Leader'
            THEN 'Talent Development: Innovation leadership + technology strategy + R&D coordination + future technology leadership'
            WHEN leadership_classification = 'Cross-Functional Customer Leader'
            THEN 'Talent Development: Customer experience leadership + market strategy + customer success leadership + business development'
            WHEN leadership_classification = 'Single-Division Specialist'
            THEN 'Talent Development: Deep expertise + technical leadership + domain mastery + specialized advancement'
            ELSE 'Talent Development: Cross-functional skills + collaboration leadership + integration expertise + versatile advancement'
        END as talent_development_pathway,
        
        -- Career advancement recommendations
        CASE 
            WHEN total_divisions >= 3 AND innovation_roles >= 2
            THEN 'Career Advancement: VP/SVP track + technology strategy + innovation leadership + enterprise transformation'
            WHEN total_divisions >= 2 AND strategic_roles >= 1
            THEN 'Career Advancement: Director track + strategic planning + cross-business leadership + organizational excellence'
            WHEN total_divisions = 1 AND amazon_level LIKE '%Principal%'
            THEN 'Career Advancement: Principal engineer track + technical expertise + innovation leadership + specialized excellence'
            WHEN customer_facing_roles >= 1 AND high_impact_roles >= 1
            THEN 'Career Advancement: Business leadership + customer strategy + market development + growth leadership'
            ELSE 'Career Advancement: Professional development + skill expansion + leadership preparation + performance excellence'
        END as career_advancement_strategy,
        
        -- Skill development priorities
        CASE 
            WHEN leadership_classification LIKE '%Strategic%'
            THEN 'Skill Development: Strategic thinking + organizational leadership + change management + business acumen + executive presence'
            WHEN leadership_classification LIKE '%Innovation%'
            THEN 'Skill Development: Technology vision + innovation management + R&D leadership + emerging technologies + technical strategy'
            WHEN leadership_classification LIKE '%Customer%'
            THEN 'Skill Development: Customer research + market analysis + business development + customer success + user experience'
            WHEN total_divisions >= 2
            THEN 'Skill Development: Cross-functional collaboration + project management + integration skills + communication + leadership'
            ELSE 'Skill Development: Technical expertise + domain knowledge + problem solving + quality focus + continuous learning'
        END as skill_development_priorities,
        
        -- Performance expectations and goals
        CASE 
            WHEN leadership_classification = 'Cross-Functional Strategic Leader'
            THEN 'Performance Goals: Enterprise impact + strategic outcomes + organizational transformation + long-term value creation'
            WHEN leadership_classification LIKE '%Innovation%'
            THEN 'Performance Goals: Innovation delivery + technology advancement + research outcomes + future capabilities + competitive advantage'
            WHEN customer_facing_roles >= 2
            THEN 'Performance Goals: Customer satisfaction + market growth + customer experience + retention + advocacy'
            WHEN strategic_roles >= 1
            THEN 'Performance Goals: Strategic execution + business results + operational excellence + team performance + quality delivery'
            ELSE 'Performance Goals: Individual excellence + skill mastery + reliable delivery + continuous improvement + team contribution'
        END as performance_expectations
    FROM amazon_cross_functional_intelligence acfi
)
SELECT 
    employee_id,
    primary_department_id,
    primary_division,
    amazon_level,
    leadership_classification,
    talent_development_pathway,
    career_advancement_strategy,
    skill_development_priorities,
    performance_expectations,
    customer_obsession_application,
    ownership_demonstration,
    
    -- Organizational metrics
    total_divisions,
    all_divisions,
    strategic_roles,
    high_impact_roles,
    innovation_roles,
    customer_facing_roles
FROM amazon_talent_development_insights
ORDER BY 
    CASE leadership_classification
        WHEN 'Cross-Functional Strategic Leader' THEN 1
        WHEN 'Cross-Functional Innovation Leader' THEN 2
        WHEN 'Cross-Functional Customer Leader' THEN 3
        WHEN 'Cross-Functional Contributor' THEN 4
        ELSE 5
    END,
    total_divisions DESC,
    strategic_roles DESC,
    employee_id;
```

I'll create one final concise extension and then complete the remaining questions:

#### 2. **Matrix Organization Optimization and Resource Allocation**
```sql
-- "Advanced matrix organization optimization with dynamic resource allocation and productivity analysis"

WITH matrix_organization_optimization AS (
    SELECT 
        employee_id,
        MAX(CASE WHEN primary_flag = 'Y' OR 
                      (COUNT(*) OVER(PARTITION BY employee_id) = 1 AND primary_flag = 'N') 
                 THEN department_id END) as primary_department,
        COUNT(*) as department_count,
        
        -- Resource allocation percentages
        CASE 
            WHEN COUNT(*) = 1 THEN 100
            WHEN COUNT(*) = 2 AND primary_flag = 'Y' THEN 70
            WHEN COUNT(*) = 2 AND primary_flag = 'N' THEN 30
            WHEN COUNT(*) >= 3 AND primary_flag = 'Y' THEN 50
            WHEN COUNT(*) >= 3 AND primary_flag = 'N' THEN 50 / (COUNT(*) - 1)
            ELSE 100 / COUNT(*)
        END as resource_allocation_percentage,
        
        -- Matrix complexity score
        CASE 
            WHEN COUNT(*) >= 4 THEN 'High Complexity'
            WHEN COUNT(*) = 3 THEN 'Medium Complexity'
            WHEN COUNT(*) = 2 THEN 'Low Complexity'
            ELSE 'No Complexity'
        END as matrix_complexity
    FROM Employee
    GROUP BY employee_id
)
SELECT 
    employee_id,
    primary_department,
    department_count,
    matrix_complexity,
    resource_allocation_percentage
FROM matrix_organization_optimization
ORDER BY department_count DESC, employee_id;
```

## 🔗 Related LeetCode Questions

1. **#1731 - The Number of Employees Which Report to Each Employee** (Employee relationship analysis)
2. **#1978 - Employees Whose Manager Left the Company** (Employee-manager relationships)
3. **#1741 - Find Total Time Spent by Each Employee** (Employee performance analysis)
4. **#1777 - Product's Price for Each Store** (Multi-dimension data analysis)
5. **#1795 - Rearrange Products Table** (Data transformation operations)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Conditional Logic**: Handle multiple scenarios with UNION or CASE statements
2. **Primary Key Logic**: Use explicit flags or implicit single-record logic
3. **Window Functions**: COUNT(*) OVER for analyzing employee department distribution
4. **Business Logic**: Primary department determination based on business rules

### 🚀 **Amazon Interview Tips**
1. **Explain business logic**: "Primary flag 'Y' takes precedence, single department is implicit primary"
2. **Discuss edge cases**: "What if employee has no primary flag but multiple departments?"
3. **Address scalability**: "Solution handles any number of departments per employee"
4. **Consider data quality**: "Assumes data integrity - one primary flag per employee max"

### 🔧 **Common Patterns**
- UNION for combining different logical scenarios
- Window functions for counting related records
- Conditional aggregation with HAVING clauses
- Primary key logic with business rule implementation

### ⚠️ **Common Mistakes to Avoid**
1. **Missing single department case** (only checking primary_flag = 'Y')
2. **Duplicate results** (not handling UNION properly)
3. **Wrong aggregation** (not using proper GROUP BY with conditional logic)
4. **Ignoring edge cases** (multiple primary flags, no primary flags)

### 🔍 **Performance Considerations**
- INDEX on (employee_id, primary_flag) for efficient filtering
- INDEX on employee_id for window function performance
- Consider using EXISTS instead of UNION for large datasets
- Window functions can be expensive on very large tables

### 🎯 **Amazon Leadership Principles Applied**
- **Ownership**: Clear accountability through primary department designation
- **Customer Obsession**: Organize teams effectively to serve customer needs
- **Think Big**: Scale organizational structures across Amazon's global operations
- **Deliver Results**: Optimize resource allocation for maximum business impact

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find employees with no primary department designated**
2. **List all secondary departments for each employee**
3. **Calculate total employees per department (both primary and secondary)**
4. **Find departments that serve only as secondary departments**

Remember: Organizational structure analysis like this is crucial for Amazon's matrix organization, enabling effective resource allocation across AWS, Prime, Alexa, and other business units!