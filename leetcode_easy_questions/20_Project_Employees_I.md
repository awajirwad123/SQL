# LeetCode Easy #1075: Project Employees I

## 📋 Problem Statement

Write a SQL query that reports the **average experience years** of all the employees for each project, **rounded to 2 decimal places**.

Return the result table in **any order**.

## 🗄️ Table Schema

**Project Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| project_id  | int     |
| employee_id | int     |
+-------------+---------+
```
- `(project_id, employee_id)` is the primary key of this table.
- Each row of this table indicates that the employee with `employee_id` is working on the project with `project_id`.

**Employee Table:**
```
+------------------+---------+
| Column Name      | Type    |
+------------------+---------+
| employee_id      | int     |
| name             | varchar |
| experience_years | int     |
+------------------+---------+
```
- `employee_id` is the primary key of this table.
- Each row of this table contains information about one employee.

## 📊 Sample Data

**Project Table:**
| project_id | employee_id |
|------------|-------------|
| 1          | 1           |
| 1          | 2           |
| 1          | 3           |
| 2          | 1           |
| 2          | 4           |

**Employee Table:**
| employee_id | name  | experience_years |
|-------------|-------|------------------|
| 1           | Khaled| 3                |
| 2           | Ali   | 2                |
| 3           | John  | 1                |
| 4           | Doe   | 2                |

**Expected Output:**
| project_id | average_years |
|------------|---------------|
| 1          | 2.00          |
| 2          | 2.50          |

**Explanation:** 
- Project 1: Employees 1(3 years), 2(2 years), 3(1 year) → Average = (3+2+1)/3 = 2.00
- Project 2: Employees 1(3 years), 4(2 years) → Average = (3+2)/2 = 2.50

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to join Project and Employee tables
- Calculate average experience per project
- GROUP BY project_id and use AVG function
- Round result to 2 decimal places

### 2. **Key Insights**
- Project table links employees to projects
- Employee table contains experience data
- JOIN on employee_id to get experience for each project
- GROUP BY project_id to calculate per-project averages

### 3. **Interview Discussion Points**
- "This is a JOIN with aggregation problem"
- "I need to group by project and average the experience years"
- "ROUND function ensures 2 decimal places in output"

## 🔧 Step-by-Step Solution Logic

### Step 1: Join Tables
```sql
-- JOIN Project and Employee on employee_id
-- This connects project assignments to employee experience
```

### Step 2: Group by Project
```sql
-- GROUP BY project_id to aggregate per project
-- This creates groups for each project
```

### Step 3: Calculate Average and Round
```sql
-- AVG(experience_years) for average experience
-- ROUND(..., 2) for 2 decimal places
```

## ✅ Optimized SQL Solution

**Solution 1: Basic JOIN with AVG**
```sql
SELECT 
    p.project_id,
    ROUND(AVG(e.experience_years), 2) as average_years
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

### Alternative Solutions

**Solution 2: With Explicit Casting**
```sql
SELECT 
    p.project_id,
    ROUND(AVG(CAST(e.experience_years AS DECIMAL)), 2) as average_years
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

**Solution 3: With Additional Project Statistics**
```sql
SELECT 
    p.project_id,
    COUNT(*) as team_size,
    ROUND(AVG(e.experience_years), 2) as average_years,
    MIN(e.experience_years) as min_experience,
    MAX(e.experience_years) as max_experience
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

**Solution 4: Using Window Functions**
```sql
WITH project_experience AS (
    SELECT 
        p.project_id,
        e.experience_years,
        AVG(e.experience_years) OVER (PARTITION BY p.project_id) as avg_exp
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
)
SELECT DISTINCT 
    project_id,
    ROUND(avg_exp, 2) as average_years
FROM project_experience;
```

**Solution 5: With Team Experience Distribution**
```sql
SELECT 
    p.project_id,
    ROUND(AVG(e.experience_years), 2) as average_years,
    ROUND(STDDEV(e.experience_years), 2) as experience_std_dev,
    CASE 
        WHEN STDDEV(e.experience_years) <= 1 THEN 'Homogeneous Team'
        WHEN STDDEV(e.experience_years) <= 2 THEN 'Balanced Team'
        ELSE 'Diverse Team'
    END as team_composition
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Team Analytics**
```sql
-- "Provide detailed team composition analysis for each project"

WITH team_analytics AS (
    SELECT 
        p.project_id,
        COUNT(*) as team_size,
        AVG(e.experience_years) as avg_experience,
        MIN(e.experience_years) as min_experience,
        MAX(e.experience_years) as max_experience,
        STDDEV(e.experience_years) as experience_variance,
        SUM(e.experience_years) as total_team_experience,
        COUNT(CASE WHEN e.experience_years >= 5 THEN 1 END) as senior_count,
        COUNT(CASE WHEN e.experience_years BETWEEN 2 AND 4 THEN 1 END) as mid_level_count,
        COUNT(CASE WHEN e.experience_years < 2 THEN 1 END) as junior_count
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id
),
team_classifications AS (
    SELECT 
        *,
        ROUND(avg_experience, 2) as average_years,
        ROUND(experience_variance, 2) as experience_std_dev,
        ROUND(senior_count * 100.0 / team_size, 2) as senior_percentage,
        ROUND(junior_count * 100.0 / team_size, 2) as junior_percentage,
        CASE 
            WHEN avg_experience >= 4 THEN 'Senior Team'
            WHEN avg_experience >= 2.5 THEN 'Experienced Team'
            WHEN avg_experience >= 1.5 THEN 'Mixed Team'
            ELSE 'Junior Team'
        END as team_seniority_level,
        CASE 
            WHEN experience_variance <= 0.5 THEN 'Very Homogeneous'
            WHEN experience_variance <= 1.0 THEN 'Homogeneous'
            WHEN experience_variance <= 2.0 THEN 'Balanced'
            ELSE 'Highly Diverse'
        END as diversity_level
    FROM team_analytics
)
SELECT 
    project_id,
    team_size,
    average_years,
    min_experience,
    max_experience,
    experience_std_dev,
    team_seniority_level,
    diversity_level,
    senior_count,
    mid_level_count,
    junior_count,
    senior_percentage,
    junior_percentage,
    total_team_experience
FROM team_classifications
ORDER BY average_years DESC, team_size DESC;
```

#### 2. **Project Complexity and Team Matching**
```sql
-- "Analyze if team experience matches project requirements"

-- Assuming we have project complexity data
WITH project_requirements AS (
    SELECT 
        project_id,
        CASE 
            WHEN project_id <= 2 THEN 'High Complexity'
            WHEN project_id <= 5 THEN 'Medium Complexity'
            ELSE 'Low Complexity'
        END as complexity_level,
        CASE 
            WHEN project_id <= 2 THEN 3.5
            WHEN project_id <= 5 THEN 2.5
            ELSE 1.5
        END as required_avg_experience
    FROM (SELECT DISTINCT project_id FROM Project) projects
),
team_analysis AS (
    SELECT 
        p.project_id,
        ROUND(AVG(e.experience_years), 2) as average_years,
        COUNT(*) as team_size,
        MIN(e.experience_years) as min_experience,
        MAX(e.experience_years) as max_experience
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id
),
project_assessment AS (
    SELECT 
        ta.project_id,
        ta.average_years,
        ta.team_size,
        pr.complexity_level,
        pr.required_avg_experience,
        CASE 
            WHEN ta.average_years >= pr.required_avg_experience THEN 'Well Matched'
            WHEN ta.average_years >= pr.required_avg_experience * 0.8 THEN 'Adequate'
            ELSE 'Under-experienced'
        END as team_project_fit,
        ROUND(ta.average_years - pr.required_avg_experience, 2) as experience_gap,
        CASE 
            WHEN ta.team_size >= 5 THEN 'Large Team'
            WHEN ta.team_size >= 3 THEN 'Medium Team'
            ELSE 'Small Team'
        END as team_size_category
    FROM team_analysis ta
    JOIN project_requirements pr ON ta.project_id = pr.project_id
)
SELECT 
    project_id,
    complexity_level,
    average_years,
    required_avg_experience,
    team_project_fit,
    experience_gap,
    team_size,
    team_size_category,
    CASE 
        WHEN team_project_fit = 'Well Matched' AND team_size >= 3 THEN 'Optimal Setup'
        WHEN team_project_fit = 'Adequate' THEN 'Manageable Setup'
        WHEN experience_gap < -1 THEN 'Risk: Significant Skill Gap'
        ELSE 'Review Required'
    END as project_risk_assessment
FROM project_assessment
ORDER BY 
    CASE team_project_fit 
        WHEN 'Under-experienced' THEN 1
        WHEN 'Adequate' THEN 2
        ELSE 3
    END,
    experience_gap ASC;
```

#### 3. **Employee Utilization and Workload Analysis**
```sql
-- "Analyze employee utilization across projects"

WITH employee_workload AS (
    SELECT 
        e.employee_id,
        e.name,
        e.experience_years,
        COUNT(p.project_id) as project_count,
        GROUP_CONCAT(p.project_id ORDER BY p.project_id) as assigned_projects
    FROM Employee e
    LEFT JOIN Project p ON e.employee_id = p.employee_id
    GROUP BY e.employee_id, e.name, e.experience_years
),
workload_analysis AS (
    SELECT 
        *,
        CASE 
            WHEN project_count = 0 THEN 'Unassigned'
            WHEN project_count = 1 THEN 'Single Project'
            WHEN project_count = 2 THEN 'Dual Projects'
            ELSE 'Multi-Project'
        END as workload_category,
        CASE 
            WHEN experience_years >= 5 AND project_count <= 1 THEN 'Under-utilized Senior'
            WHEN experience_years <= 2 AND project_count >= 3 THEN 'Over-utilized Junior'
            WHEN project_count = 0 THEN 'Available Resource'
            ELSE 'Balanced Allocation'
        END as utilization_status
    FROM employee_workload
),
project_averages AS (
    SELECT 
        p.project_id,
        ROUND(AVG(e.experience_years), 2) as average_years
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id
)
SELECT 
    wa.employee_id,
    wa.name,
    wa.experience_years,
    wa.project_count,
    wa.assigned_projects,
    wa.workload_category,
    wa.utilization_status,
    CASE 
        WHEN wa.project_count > 0 THEN
            ROUND(AVG(pa.average_years), 2)
        ELSE NULL
    END as avg_team_experience_in_projects
FROM workload_analysis wa
LEFT JOIN Project p ON FIND_IN_SET(p.project_id, wa.assigned_projects) > 0
LEFT JOIN project_averages pa ON p.project_id = pa.project_id
GROUP BY wa.employee_id, wa.name, wa.experience_years, wa.project_count, 
         wa.assigned_projects, wa.workload_category, wa.utilization_status
ORDER BY wa.experience_years DESC, wa.project_count DESC;
```

#### 4. **Performance Optimization for Large Datasets**
```sql
-- "Optimize for large-scale project and employee data"

-- Create indexes for efficient JOINs and GROUP BY
-- CREATE INDEX idx_project_employee ON Project(employee_id, project_id);
-- CREATE INDEX idx_employee_experience ON Employee(experience_years);

-- Optimized query with execution plan analysis
EXPLAIN ANALYZE
SELECT 
    p.project_id,
    ROUND(AVG(e.experience_years), 2) as average_years
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
GROUP BY p.project_id;

-- For very large datasets, consider pre-aggregated materialized views
-- CREATE MATERIALIZED VIEW project_team_stats AS
-- SELECT 
--     p.project_id,
--     COUNT(*) as team_size,
--     AVG(e.experience_years) as avg_experience,
--     MIN(e.experience_years) as min_experience,
--     MAX(e.experience_years) as max_experience,
--     STDDEV(e.experience_years) as experience_variance
-- FROM Project p
-- INNER JOIN Employee e ON p.employee_id = e.employee_id
-- GROUP BY p.project_id;

-- Partitioned analysis for better performance
WITH recent_projects AS (
    SELECT DISTINCT project_id 
    FROM Project 
    WHERE project_id >= (SELECT MAX(project_id) - 100 FROM Project)
)
SELECT 
    p.project_id,
    ROUND(AVG(e.experience_years), 2) as average_years,
    COUNT(*) as team_size
FROM Project p
INNER JOIN Employee e ON p.employee_id = e.employee_id
INNER JOIN recent_projects rp ON p.project_id = rp.project_id
GROUP BY p.project_id
ORDER BY p.project_id;
```

#### 5. **Project Success Prediction Model**
```sql
-- "Predict project success based on team composition"

WITH project_metrics AS (
    SELECT 
        p.project_id,
        COUNT(*) as team_size,
        AVG(e.experience_years) as avg_experience,
        MIN(e.experience_years) as min_experience,
        MAX(e.experience_years) as max_experience,
        STDDEV(e.experience_years) as experience_variance,
        COUNT(CASE WHEN e.experience_years >= 5 THEN 1 END) as senior_members,
        COUNT(CASE WHEN e.experience_years < 2 THEN 1 END) as junior_members
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id
),
success_indicators AS (
    SELECT 
        *,
        ROUND(avg_experience, 2) as average_years,
        -- Success scoring algorithm
        (CASE 
            WHEN avg_experience >= 4 THEN 30
            WHEN avg_experience >= 3 THEN 25
            WHEN avg_experience >= 2 THEN 20
            ELSE 10
        END) +
        (CASE 
            WHEN team_size BETWEEN 3 AND 6 THEN 20
            WHEN team_size BETWEEN 2 AND 8 THEN 15
            ELSE 5
        END) +
        (CASE 
            WHEN experience_variance <= 1.5 THEN 15
            WHEN experience_variance <= 2.5 THEN 10
            ELSE 5
        END) +
        (CASE 
            WHEN senior_members >= 1 AND junior_members <= team_size / 2 THEN 15
            WHEN senior_members >= 1 THEN 10
            ELSE 5
        END) +
        (CASE 
            WHEN min_experience >= 1 THEN 10
            ELSE 5
        END) as success_score
    FROM project_metrics
),
success_predictions AS (
    SELECT 
        *,
        CASE 
            WHEN success_score >= 80 THEN 'Very High'
            WHEN success_score >= 65 THEN 'High'
            WHEN success_score >= 50 THEN 'Medium'
            WHEN success_score >= 35 THEN 'Low'
            ELSE 'Very Low'
        END as success_probability,
        CASE 
            WHEN success_score < 50 THEN 
                CASE 
                    WHEN avg_experience < 2 THEN 'Add senior members'
                    WHEN team_size < 3 THEN 'Increase team size'
                    WHEN experience_variance > 3 THEN 'Balance skill levels'
                    ELSE 'Review team composition'
                END
            ELSE 'Team composition is adequate'
        END as recommendation
    FROM success_indicators
)
SELECT 
    project_id,
    average_years,
    team_size,
    success_score,
    success_probability,
    recommendation,
    senior_members,
    junior_members,
    ROUND(experience_variance, 2) as skill_diversity
FROM success_predictions
ORDER BY success_score DESC, average_years DESC;
```

#### 6. **Resource Planning and Optimization**
```sql
-- "Optimize team allocation across projects"

WITH current_allocation AS (
    SELECT 
        p.project_id,
        ROUND(AVG(e.experience_years), 2) as current_avg_experience,
        COUNT(*) as current_team_size,
        SUM(e.experience_years) as total_experience_points
    FROM Project p
    INNER JOIN Employee e ON p.employee_id = e.employee_id
    GROUP BY p.project_id
),
available_resources AS (
    SELECT 
        e.employee_id,
        e.name,
        e.experience_years,
        COALESCE(current_projects.project_count, 0) as current_projects
    FROM Employee e
    LEFT JOIN (
        SELECT employee_id, COUNT(*) as project_count
        FROM Project 
        GROUP BY employee_id
    ) current_projects ON e.employee_id = current_projects.employee_id
),
optimization_opportunities AS (
    SELECT 
        ca.project_id,
        ca.current_avg_experience,
        ca.current_team_size,
        -- Find potential additions from available resources
        (SELECT COUNT(*) FROM available_resources WHERE current_projects <= 1) as available_light_load,
        (SELECT AVG(experience_years) FROM available_resources WHERE current_projects <= 1) as avg_available_exp,
        CASE 
            WHEN ca.current_avg_experience < 2.5 AND ca.current_team_size < 5 THEN 'Needs senior reinforcement'
            WHEN ca.current_team_size < 3 THEN 'Understaffed'
            WHEN ca.current_avg_experience > 4 AND ca.current_team_size > 6 THEN 'Over-resourced'
            ELSE 'Adequately staffed'
        END as staffing_assessment
    FROM current_allocation ca
)
SELECT 
    project_id,
    current_avg_experience as average_years,
    current_team_size,
    staffing_assessment,
    available_light_load,
    ROUND(avg_available_exp, 2) as avg_available_experience,
    CASE 
        WHEN staffing_assessment LIKE '%reinforcement%' THEN 'Priority: Add experienced member'
        WHEN staffing_assessment = 'Understaffed' THEN 'Priority: Increase team size'
        WHEN staffing_assessment = 'Over-resourced' THEN 'Consider redistribution'
        ELSE 'Monitor current state'
    END as action_recommendation
FROM optimization_opportunities
ORDER BY 
    CASE staffing_assessment
        WHEN 'Needs senior reinforcement' THEN 1
        WHEN 'Understaffed' THEN 2
        WHEN 'Over-resourced' THEN 3
        ELSE 4
    END,
    current_avg_experience ASC;
```

## 🔗 Related LeetCode Questions

1. **#1068 - Product Sales Analysis I** (Basic JOIN operations)
2. **#586 - Customer Placing the Largest Number of Orders** (GROUP BY with aggregation)
3. **#1084 - Sales Analysis III** (Complex aggregation with JOIN)
4. **#1141 - User Activity for the Past 30 Days I** (GROUP BY with date filtering)
5. **#1179 - Reformat Department Table** (Advanced GROUP BY techniques)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **JOIN with GROUP BY**: Combining tables and aggregating results
2. **AVG Function**: Calculating averages with proper rounding
3. **ROUND Function**: Formatting decimal places for presentation
4. **Aggregation with Multiple Tables**: Working with normalized data

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I round to exactly 2 decimal places?"
2. **Discuss performance**: "INDEX on JOIN columns improves GROUP BY performance"
3. **Consider edge cases**: "What about projects with no employees?"
4. **Think business context**: "This helps assess project team strength and capability"

### 🔧 **Common Patterns**
- JOIN followed by GROUP BY for aggregation across tables
- AVG() function with ROUND() for proper formatting
- COUNT() for team size analysis
- Window functions for advanced analytics

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting to round** the average to specified decimal places
2. **Using OUTER JOIN** when INNER JOIN is appropriate
3. **Not handling NULL values** in experience_years
4. **Performance issues** without proper indexing on JOIN columns

### 🔍 **Performance Considerations**
- Composite index on (employee_id, project_id) for efficient JOINs
- Index on experience_years for faster aggregation
- Consider materialized views for frequently accessed team statistics
- Partitioning for very large project datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Hire and Develop the Best**: Analyzing team composition and experience
- **Operational Excellence**: Efficient team allocation and project management
- **Data-Driven Decisions**: Using team metrics for project planning
- **Customer Obsession**: Ensuring adequate team expertise for project success

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find projects with average experience above 3 years**
2. **Calculate median experience per project (not just average)**
3. **Find the most and least experienced team member per project**
4. **Calculate experience variance within each project team**

Remember: Team composition analysis and resource optimization are crucial for Amazon's project management, workforce planning, delivery team formation, and ensuring project success across all business units!