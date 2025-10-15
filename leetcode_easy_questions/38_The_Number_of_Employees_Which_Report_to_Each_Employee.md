# LeetCode Easy #1731: The Number of Employees Which Report to Each Employee

## 📋 Problem Statement

For this problem, we will consider a **manager** an employee who has at least 1 other employee reporting to them.

Write a SQL query to report the ids and the names of all **managers**, the **number of employees** who report **directly** to them, and the **average age** of the reports rounded to the **nearest integer**.

Return the result table **ordered by employee_id**.

## 🗄️ Table Schema

**Employees Table:**
```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| employee_id | int      |
| name        | varchar  |
| reports_to  | int      |
| age         | int      |
+-------------+----------+
```
- employee_id is the primary key for this table.
- This table contains information about the employees and the id of the manager they report to. Some employees do not report to anyone (reports_to is null).

## 📊 Sample Data

**Employees Table:**
| employee_id | name    | reports_to | age |
|-------------|---------|------------|-----|
| 9           | Hercy   | null       | 43  |
| 6           | Alice   | 9          | 41  |
| 4           | Bob     | 9          | 36  |
| 2           | Winston | null       | 37  |

**Expected Output:**
| employee_id | name  | reports_count | average_age |
|-------------|-------|---------------|-------------|
| 9           | Hercy | 2             | 39          |

**Explanation:**
- Hercy has 2 direct reports: Alice and Bob
- Average age = (41 + 36) / 2 = 38.5, rounded to 39
- Winston has no direct reports, so is not included

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find employees who are managers (have direct reports)
- Count the number of direct reports for each manager
- Calculate average age of direct reports
- Round average age to nearest integer

### 2. **Key Insights**
- Join Employees table with itself (manager info + report info)
- GROUP BY manager to aggregate report data
- Use COUNT() for number of reports and AVG() for average age
- ROUND() function for rounding average age

### 3. **Interview Discussion Points**
- "This requires a self-join to connect managers with their reports"
- "GROUP BY manager allows us to aggregate report statistics"
- "ROUND() ensures average age is an integer as specified"

## 🔧 Step-by-Step Solution Logic

### Step 1: Self-Join to Connect Managers and Reports
```sql
-- Join managers with their direct reports
FROM Employees m
JOIN Employees e ON m.employee_id = e.reports_to
```

### Step 2: Group by Manager and Aggregate
```sql
-- Group by manager to calculate statistics
GROUP BY m.employee_id, m.name
```

### Step 3: Calculate Report Statistics
```sql
-- Count reports and calculate average age
COUNT(e.employee_id) as reports_count,
ROUND(AVG(e.age)) as average_age
```

## ✅ Optimized SQL Solution

**Solution 1: Self-Join with Aggregation**
```sql
SELECT 
    m.employee_id,
    m.name,
    COUNT(e.employee_id) as reports_count,
    ROUND(AVG(e.age)) as average_age
FROM Employees m
JOIN Employees e ON m.employee_id = e.reports_to
GROUP BY m.employee_id, m.name
ORDER BY m.employee_id;
```

### Alternative Solutions

**Solution 2: Using WHERE Instead of JOIN**
```sql
SELECT 
    m.employee_id,
    m.name,
    COUNT(e.employee_id) as reports_count,
    ROUND(AVG(e.age)) as average_age
FROM Employees m, Employees e
WHERE m.employee_id = e.reports_to
GROUP BY m.employee_id, m.name
ORDER BY m.employee_id;
```

**Solution 3: With Additional Manager Information**
```sql
SELECT 
    m.employee_id,
    m.name,
    m.age as manager_age,
    COUNT(e.employee_id) as reports_count,
    ROUND(AVG(e.age)) as average_age,
    MIN(e.age) as youngest_report,
    MAX(e.age) as oldest_report
FROM Employees m
JOIN Employees e ON m.employee_id = e.reports_to
GROUP BY m.employee_id, m.name, m.age
ORDER BY m.employee_id;
```

**Solution 4: Using Subquery Approach**
```sql
SELECT 
    employee_id,
    name,
    (SELECT COUNT(*) 
     FROM Employees e2 
     WHERE e2.reports_to = e1.employee_id) as reports_count,
    (SELECT ROUND(AVG(age)) 
     FROM Employees e3 
     WHERE e3.reports_to = e1.employee_id) as average_age
FROM Employees e1
WHERE EXISTS (
    SELECT 1 
    FROM Employees e4 
    WHERE e4.reports_to = e1.employee_id
)
ORDER BY employee_id;
```

**Solution 5: Comprehensive Management Analytics System**
```sql
WITH organizational_hierarchy AS (
    SELECT 
        m.employee_id as manager_id,
        m.name as manager_name,
        m.age as manager_age,
        e.employee_id as report_id,
        e.name as report_name,
        e.age as report_age,
        
        -- Simulate department structure
        CASE 
            WHEN m.employee_id % 4 = 0 THEN 'Engineering'
            WHEN m.employee_id % 4 = 1 THEN 'Sales'
            WHEN m.employee_id % 4 = 2 THEN 'Marketing'
            ELSE 'Operations'
        END as department,
        
        -- Simulate seniority levels
        CASE 
            WHEN m.age >= 45 THEN 'Senior Manager'
            WHEN m.age >= 40 THEN 'Manager'
            WHEN m.age >= 35 THEN 'Team Lead'
            ELSE 'Supervisor'
        END as management_level,
        
        -- Experience gap simulation
        m.age - e.age as experience_gap,
        
        -- Performance indicators (simulated)
        CASE 
            WHEN ABS(m.age - e.age) <= 5 THEN 'Peer Leadership'
            WHEN m.age - e.age > 15 THEN 'Mentorship Dynamic'
            WHEN m.age - e.age BETWEEN 6 AND 15 THEN 'Standard Hierarchy'
            ELSE 'Reverse Mentoring'
        END as leadership_dynamic,
        
        -- Team composition indicators
        CASE 
            WHEN e.age >= 40 THEN 'Senior Team Member'
            WHEN e.age >= 30 THEN 'Mid-Level Team Member'
            WHEN e.age >= 25 THEN 'Junior Team Member'
            ELSE 'Entry-Level Team Member'
        END as report_seniority_level
    FROM Employees m
    JOIN Employees e ON m.employee_id = e.reports_to
),
management_analytics AS (
    SELECT 
        manager_id,
        manager_name,
        manager_age,
        department,
        management_level,
        
        -- Core management metrics
        COUNT(report_id) as reports_count,
        ROUND(AVG(report_age)) as average_age,
        MIN(report_age) as youngest_report_age,
        MAX(report_age) as oldest_report_age,
        ROUND(STDDEV(report_age), 2) as age_diversity_stddev,
        
        -- Team composition analysis
        COUNT(CASE WHEN report_seniority_level = 'Senior Team Member' THEN 1 END) as senior_reports,
        COUNT(CASE WHEN report_seniority_level = 'Mid-Level Team Member' THEN 1 END) as mid_level_reports,
        COUNT(CASE WHEN report_seniority_level = 'Junior Team Member' THEN 1 END) as junior_reports,
        COUNT(CASE WHEN report_seniority_level = 'Entry-Level Team Member' THEN 1 END) as entry_level_reports,
        
        -- Leadership dynamics
        COUNT(CASE WHEN leadership_dynamic = 'Peer Leadership' THEN 1 END) as peer_leadership_count,
        COUNT(CASE WHEN leadership_dynamic = 'Mentorship Dynamic' THEN 1 END) as mentorship_count,
        COUNT(CASE WHEN leadership_dynamic = 'Reverse Mentoring' THEN 1 END) as reverse_mentoring_count,
        
        -- Experience analysis
        ROUND(AVG(experience_gap), 2) as avg_experience_gap,
        MIN(experience_gap) as min_experience_gap,
        MAX(experience_gap) as max_experience_gap,
        
        -- Team effectiveness indicators
        CASE 
            WHEN COUNT(report_id) BETWEEN 3 AND 7 THEN 'Optimal Team Size'
            WHEN COUNT(report_id) >= 8 THEN 'Large Team - Consider Restructuring'
            WHEN COUNT(report_id) = 2 THEN 'Small Team - Growth Opportunity'
            WHEN COUNT(report_id) = 1 THEN 'Individual Contributor + 1'
            ELSE 'No Direct Reports'
        END as team_size_assessment,
        
        -- Age diversity assessment
        CASE 
            WHEN STDDEV(report_age) >= 10 THEN 'High Age Diversity'
            WHEN STDDEV(report_age) >= 5 THEN 'Moderate Age Diversity'
            WHEN STDDEV(report_age) >= 2 THEN 'Low Age Diversity'
            ELSE 'Homogeneous Age Group'
        END as age_diversity_level
    FROM organizational_hierarchy
    GROUP BY manager_id, manager_name, manager_age, department, management_level
),
leadership_effectiveness_analysis AS (
    SELECT 
        ma.*,
        
        -- Management effectiveness scoring
        (CASE 
            WHEN team_size_assessment = 'Optimal Team Size' THEN 25
            WHEN team_size_assessment = 'Small Team - Growth Opportunity' THEN 20
            WHEN team_size_assessment = 'Individual Contributor + 1' THEN 15
            ELSE 10
        END) +
        (CASE 
            WHEN age_diversity_level = 'High Age Diversity' THEN 25
            WHEN age_diversity_level = 'Moderate Age Diversity' THEN 20
            WHEN age_diversity_level = 'Low Age Diversity' THEN 15
            ELSE 10
        END) +
        (CASE 
            WHEN mentorship_count > 0 THEN 20
            WHEN peer_leadership_count > 0 THEN 15
            WHEN reverse_mentoring_count > 0 THEN 10
            ELSE 5
        END) +
        (CASE 
            WHEN senior_reports > 0 AND junior_reports > 0 THEN 20  -- Mixed experience team
            WHEN senior_reports > 0 OR junior_reports > 0 THEN 15   -- Some experience diversity
            ELSE 10  -- Homogeneous experience
        END) as leadership_effectiveness_score,
        
        -- Development opportunities
        CASE 
            WHEN reports_count = 1 AND management_level = 'Supervisor'
            THEN 'Development: First-time manager training + delegation skills + team building'
            WHEN reports_count >= 8 AND team_size_assessment = 'Large Team - Consider Restructuring'
            THEN 'Development: Team restructuring + delegation optimization + sub-team creation'
            WHEN age_diversity_level = 'Homogeneous Age Group' AND reports_count >= 3
            THEN 'Development: Diversity recruitment + cross-generational management + inclusion training'
            WHEN mentorship_count = 0 AND avg_experience_gap > 10
            THEN 'Development: Mentorship training + knowledge transfer + succession planning'
            ELSE 'Development: Advanced leadership skills + strategic thinking + performance management'
        END as development_recommendations,
        
        -- Team optimization strategies
        CASE 
            WHEN senior_reports > junior_reports AND reports_count >= 3
            THEN 'Strategy: Leverage senior experience + knowledge sharing + mentorship programs'
            WHEN junior_reports > senior_reports AND reports_count >= 3
            THEN 'Strategy: Focus on training + career development + skill building + guidance'
            WHEN age_diversity_level = 'High Age Diversity' AND reports_count >= 4
            THEN 'Strategy: Cross-generational collaboration + diverse perspectives + innovation focus'
            WHEN team_size_assessment = 'Optimal Team Size'
            THEN 'Strategy: Maintain team dynamics + optimize performance + continuous improvement'
            ELSE 'Strategy: Team building + role clarity + communication enhancement + goal alignment'
        END as team_optimization_strategy,
        
        -- Succession planning recommendations
        CASE 
            WHEN senior_reports >= 2 AND management_level = 'Senior Manager'
            THEN 'Succession: Multiple candidates ready + leadership development + promotion planning'
            WHEN senior_reports >= 1 AND avg_experience_gap <= 8
            THEN 'Succession: Develop current senior reports + expanded responsibilities + readiness assessment'
            WHEN mid_level_reports >= 2 AND mentorship_count > 0
            THEN 'Succession: Accelerate mid-level development + mentorship expansion + skill gap analysis'
            WHEN reports_count >= 3
            THEN 'Succession: Identify high-potential reports + development planning + career pathing'
            ELSE 'Succession: Focus on talent development + external recruitment + knowledge documentation'
        END as succession_planning_strategy,
        
        -- Performance and retention insights
        CASE 
            WHEN leadership_effectiveness_score >= 80 AND age_diversity_level = 'High Age Diversity'
            THEN 'Performance: High-performing inclusive leader + retention magnet + promotion candidate'
            WHEN leadership_effectiveness_score >= 70 AND team_size_assessment = 'Optimal Team Size'
            THEN 'Performance: Effective team leader + strong performance + development opportunity'
            WHEN leadership_effectiveness_score >= 60
            THEN 'Performance: Competent manager + improvement opportunities + targeted development'
            WHEN leadership_effectiveness_score >= 50
            THEN 'Performance: Developing manager + support needed + coaching recommended'
            ELSE 'Performance: Management challenges + intensive support + performance improvement plan'
        END as performance_assessment
    FROM management_analytics ma
)
SELECT 
    manager_id as employee_id,
    manager_name as name,
    reports_count,
    average_age,
    department,
    management_level,
    team_size_assessment,
    age_diversity_level,
    leadership_effectiveness_score,
    development_recommendations,
    team_optimization_strategy,
    succession_planning_strategy,
    performance_assessment,
    
    -- Detailed team composition
    senior_reports,
    mid_level_reports,
    junior_reports,
    entry_level_reports,
    
    -- Leadership dynamics
    mentorship_count,
    peer_leadership_count,
    reverse_mentoring_count,
    
    -- Age and experience metrics
    manager_age,
    youngest_report_age,
    oldest_report_age,
    avg_experience_gap,
    age_diversity_stddev
FROM leadership_effectiveness_analysis
ORDER BY manager_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Organizational Structure Analytics and Leadership Development Platform**
```sql
-- "Build comprehensive organizational analytics platform with leadership development and succession planning"

WITH amazon_organizational_structure AS (
    SELECT 
        m.employee_id as manager_id,
        m.name as manager_name,
        m.age as manager_age,
        e.employee_id as report_id,
        e.name as report_name,
        e.age as report_age,
        
        -- Amazon organizational levels simulation
        CASE 
            WHEN m.employee_id % 6 = 0 THEN 'L8 - Principal'
            WHEN m.employee_id % 6 = 1 THEN 'L7 - Senior Manager'
            WHEN m.employee_id % 6 = 2 THEN 'L6 - Manager'
            WHEN m.employee_id % 6 = 3 THEN 'L5 - Senior SDE/Team Lead'
            WHEN m.employee_id % 6 = 4 THEN 'L4 - SDE II'
            ELSE 'L3 - SDE I'
        END as amazon_level,
        
        -- Business unit simulation
        CASE 
            WHEN m.employee_id % 8 = 0 THEN 'AWS'
            WHEN m.employee_id % 8 = 1 THEN 'Prime Video'
            WHEN m.employee_id % 8 = 2 THEN 'Alexa'
            WHEN m.employee_id % 8 = 3 THEN 'Retail'
            WHEN m.employee_id % 8 = 4 THEN 'Logistics'
            WHEN m.employee_id % 8 = 5 THEN 'Advertising'
            WHEN m.employee_id % 8 = 6 THEN 'Devices'
            ELSE 'Corporate'
        END as business_unit,
        
        -- Performance simulation based on leadership principles
        CASE 
            WHEN (m.employee_id + e.employee_id) % 5 = 0 THEN 'Exceeds Expectations'
            WHEN (m.employee_id + e.employee_id) % 5 = 1 THEN 'Meets Expectations'
            WHEN (m.employee_id + e.employee_id) % 5 = 2 THEN 'Partially Meets'
            WHEN (m.employee_id + e.employee_id) % 5 = 3 THEN 'Below Expectations'
            ELSE 'Not Rated'
        END as performance_rating,
        
        -- Leadership principle alignment simulation
        CASE 
            WHEN m.age >= 45 THEN 'Ownership + Deliver Results'
            WHEN m.age >= 40 THEN 'Think Big + Learn and Be Curious'
            WHEN m.age >= 35 THEN 'Customer Obsession + Invent and Simplify'
            ELSE 'Bias for Action + Dive Deep'
        END as primary_leadership_principles,
        
        -- Career development stage
        CASE 
            WHEN e.age <= 25 THEN 'Early Career Development'
            WHEN e.age <= 30 THEN 'Mid Career Growth'
            WHEN e.age <= 40 THEN 'Senior Professional'
            ELSE 'Leadership Track'
        END as career_stage,
        
        -- Innovation potential simulation
        CASE 
            WHEN e.employee_id % 3 = 0 THEN 'High Innovation Potential'
            WHEN e.employee_id % 3 = 1 THEN 'Medium Innovation Potential'
            ELSE 'Standard Innovation Potential'
        END as innovation_potential
    FROM Employees m
    JOIN Employees e ON m.employee_id = e.reports_to
),
amazon_leadership_analytics AS (
    SELECT 
        manager_id,
        manager_name,
        manager_age,
        amazon_level,
        business_unit,
        primary_leadership_principles,
        
        -- Core management metrics
        COUNT(report_id) as reports_count,
        ROUND(AVG(report_age)) as average_age,
        
        -- Performance distribution
        COUNT(CASE WHEN performance_rating = 'Exceeds Expectations' THEN 1 END) as high_performers,
        COUNT(CASE WHEN performance_rating = 'Meets Expectations' THEN 1 END) as standard_performers,
        COUNT(CASE WHEN performance_rating = 'Partially Meets' THEN 1 END) as developing_performers,
        COUNT(CASE WHEN performance_rating = 'Below Expectations' THEN 1 END) as underperformers,
        
        -- Career stage distribution
        COUNT(CASE WHEN career_stage = 'Early Career Development' THEN 1 END) as early_career_reports,
        COUNT(CASE WHEN career_stage = 'Mid Career Growth' THEN 1 END) as mid_career_reports,
        COUNT(CASE WHEN career_stage = 'Senior Professional' THEN 1 END) as senior_reports,
        COUNT(CASE WHEN career_stage = 'Leadership Track' THEN 1 END) as leadership_track_reports,
        
        -- Innovation distribution
        COUNT(CASE WHEN innovation_potential = 'High Innovation Potential' THEN 1 END) as high_innovation_reports,
        COUNT(CASE WHEN innovation_potential = 'Medium Innovation Potential' THEN 1 END) as medium_innovation_reports,
        
        -- Team performance score
        (COUNT(CASE WHEN performance_rating = 'Exceeds Expectations' THEN 1 END) * 4) +
        (COUNT(CASE WHEN performance_rating = 'Meets Expectations' THEN 1 END) * 3) +
        (COUNT(CASE WHEN performance_rating = 'Partially Meets' THEN 1 END) * 2) +
        (COUNT(CASE WHEN performance_rating = 'Below Expectations' THEN 1 END) * 1) as team_performance_score,
        
        -- Leadership effectiveness based on Amazon principles
        CASE 
            WHEN COUNT(CASE WHEN performance_rating = 'Exceeds Expectations' THEN 1 END) > 
                 COUNT(CASE WHEN performance_rating = 'Below Expectations' THEN 1 END) * 2
            THEN 'High Leadership Effectiveness'
            WHEN COUNT(CASE WHEN performance_rating = 'Meets Expectations' THEN 1 END) >= 
                 COUNT(report_id) * 0.6
            THEN 'Good Leadership Effectiveness'
            WHEN COUNT(CASE WHEN performance_rating = 'Below Expectations' THEN 1 END) = 0
            THEN 'Acceptable Leadership Effectiveness'
            ELSE 'Leadership Development Needed'
        END as leadership_effectiveness
    FROM amazon_organizational_structure
    GROUP BY manager_id, manager_name, manager_age, amazon_level, business_unit, primary_leadership_principles
),
succession_planning_system AS (
    SELECT 
        ala.*,
        
        -- Succession readiness assessment
        CASE 
            WHEN amazon_level LIKE '%Principal%' AND leadership_track_reports >= 2
            THEN 'Succession Ready: Multiple L6+ candidates available for promotion'
            WHEN amazon_level LIKE '%Senior Manager%' AND senior_reports >= 1 AND leadership_track_reports >= 1
            THEN 'Succession Developing: Senior talent pipeline with leadership potential'
            WHEN amazon_level LIKE '%Manager%' AND high_performers >= 2
            THEN 'Succession Building: High performers ready for management development'
            WHEN reports_count >= 3 AND team_performance_score >= 10
            THEN 'Succession Potential: Strong team performance indicates development capability'
            ELSE 'Succession Gap: Focus on talent development and external hiring'
        END as succession_readiness,
        
        -- Amazon leadership development recommendations
        CASE 
            WHEN primary_leadership_principles LIKE '%Ownership%' AND underperformers > 0
            THEN 'Leadership Development: Focus on "Deliver Results" + performance management + accountability systems'
            WHEN primary_leadership_principles LIKE '%Think Big%' AND high_innovation_reports < reports_count * 0.5
            THEN 'Leadership Development: Enhance "Invent and Simplify" + innovation culture + creative problem solving'
            WHEN primary_leadership_principles LIKE '%Customer Obsession%' AND business_unit = 'Retail'
            THEN 'Leadership Development: Strengthen customer focus + market insights + customer-centric decision making'
            WHEN leadership_effectiveness = 'Leadership Development Needed'
            THEN 'Leadership Development: Comprehensive program + coaching + 360 feedback + improvement plan'
            ELSE 'Leadership Development: Advanced skills + strategic thinking + cross-functional collaboration'
        END as leadership_development_plan,
        
        -- Business unit specific strategies
        CASE 
            WHEN business_unit = 'AWS' AND amazon_level LIKE '%Principal%'
            THEN 'AWS Strategy: Technical excellence + cloud innovation + enterprise customer focus + scale management'
            WHEN business_unit = 'Prime Video' AND high_performers >= 2
            THEN 'Prime Video Strategy: Content innovation + creative collaboration + global expansion + performance culture'
            WHEN business_unit = 'Alexa' AND high_innovation_reports >= 1
            THEN 'Alexa Strategy: Voice innovation + AI advancement + ecosystem expansion + user experience focus'
            WHEN business_unit = 'Retail' AND reports_count >= 3
            THEN 'Retail Strategy: Customer obsession + operational excellence + selection expansion + logistics optimization'
            ELSE 'Standard Strategy: Leadership excellence + business results + team development + innovation culture'
        END as business_unit_strategy,
        
        -- Career progression pathways
        CASE 
            WHEN amazon_level LIKE '%Principal%' AND leadership_effectiveness = 'High Leadership Effectiveness'
            THEN 'Career Path: VP track + executive development + P&L responsibility + strategic leadership'
            WHEN amazon_level LIKE '%Senior Manager%' AND team_performance_score >= 12
            THEN 'Career Path: Principal promotion + larger scope + cross-functional leadership + business ownership'
            WHEN amazon_level LIKE '%Manager%' AND succession_readiness LIKE 'Succession Ready%'
            THEN 'Career Path: Senior Manager + expanded team + department leadership + strategic contribution'
            WHEN leadership_effectiveness = 'Good Leadership Effectiveness' AND reports_count >= 3
            THEN 'Career Path: Management advancement + leadership development + increased scope + skill building'
            ELSE 'Career Path: Current role mastery + leadership skills + performance improvement + readiness building'
        END as career_progression_path,
        
        -- Performance improvement and retention strategies
        CASE 
            WHEN underperformers > 0 AND amazon_level LIKE '%Manager%'
            THEN 'Retention Strategy: Performance improvement plans + coaching + skills development + clear expectations'
            WHEN high_performers >= reports_count * 0.7 AND business_unit = 'AWS'
            THEN 'Retention Strategy: Advanced projects + innovation opportunities + executive visibility + stock awards'
            WHEN leadership_effectiveness = 'High Leadership Effectiveness'
            THEN 'Retention Strategy: Increased scope + strategic projects + leadership recognition + succession planning'
            WHEN early_career_reports >= 2
            THEN 'Retention Strategy: Mentorship programs + career development + growth opportunities + skill building'
            ELSE 'Retention Strategy: Engagement improvement + recognition programs + development planning + feedback culture'
        END as retention_strategy
    FROM amazon_leadership_analytics ala
)
SELECT 
    manager_id as employee_id,
    manager_name as name,
    reports_count,
    average_age,
    amazon_level,
    business_unit,
    primary_leadership_principles,
    leadership_effectiveness,
    succession_readiness,
    leadership_development_plan,
    business_unit_strategy,
    career_progression_path,
    retention_strategy,
    
    -- Performance metrics
    high_performers,
    standard_performers,
    underperformers,
    team_performance_score,
    
    -- Team composition
    early_career_reports,
    mid_career_reports,
    senior_reports,
    leadership_track_reports,
    
    -- Innovation metrics
    high_innovation_reports,
    medium_innovation_reports
FROM succession_planning_system
ORDER BY 
    CASE amazon_level
        WHEN 'L8 - Principal' THEN 1
        WHEN 'L7 - Senior Manager' THEN 2
        WHEN 'L6 - Manager' THEN 3
        WHEN 'L5 - Senior SDE/Team Lead' THEN 4
        WHEN 'L4 - SDE II' THEN 5
        ELSE 6
    END,
    team_performance_score DESC,
    manager_id;
```

#### 2. **Real-time Management Performance and Team Dynamics Intelligence**
```sql
-- "Implement real-time management performance system with team dynamics analysis and predictive insights"

WITH real_time_management_intelligence AS (
    SELECT 
        m.employee_id as manager_id,
        m.name as manager_name,
        m.age as manager_age,
        e.employee_id as report_id,
        e.name as report_name,
        e.age as report_age,
        
        -- Real-time performance indicators simulation
        CURRENT_TIMESTAMP as analysis_timestamp,
        
        -- Team dynamics simulation
        CASE 
            WHEN ABS(m.age - e.age) <= 3 THEN 'Peer Dynamic'
            WHEN m.age - e.age BETWEEN 4 AND 8 THEN 'Collaborative Dynamic'
            WHEN m.age - e.age BETWEEN 9 AND 15 THEN 'Traditional Hierarchy'
            WHEN m.age - e.age > 15 THEN 'Mentorship Dynamic'
            ELSE 'Reverse Mentoring'
        END as team_dynamic_type,
        
        -- Communication style compatibility
        CASE 
            WHEN (m.employee_id + e.employee_id) % 4 = 0 THEN 'High Compatibility'
            WHEN (m.employee_id + e.employee_id) % 4 = 1 THEN 'Good Compatibility'
            WHEN (m.employee_id + e.employee_id) % 4 = 2 THEN 'Moderate Compatibility'
            ELSE 'Development Needed'
        END as communication_compatibility,
        
        -- Work style alignment
        CASE 
            WHEN m.employee_id % 3 = e.employee_id % 3 THEN 'Aligned Work Styles'
            WHEN ABS((m.employee_id % 3) - (e.employee_id % 3)) = 1 THEN 'Complementary Styles'
            ELSE 'Contrasting Styles'
        END as work_style_alignment,
        
        -- Productivity indicators
        CASE 
            WHEN e.employee_id % 5 = 0 THEN 'High Productivity'
            WHEN e.employee_id % 5 = 1 THEN 'Above Average Productivity'
            WHEN e.employee_id % 5 = 2 THEN 'Average Productivity'
            WHEN e.employee_id % 5 = 3 THEN 'Below Average Productivity'
            ELSE 'Productivity Concerns'
        END as productivity_level,
        
        -- Engagement score simulation
        CASE 
            WHEN communication_compatibility = 'High Compatibility' AND team_dynamic_type = 'Collaborative Dynamic'
            THEN 95
            WHEN communication_compatibility = 'Good Compatibility' AND work_style_alignment = 'Aligned Work Styles'
            THEN 85
            WHEN team_dynamic_type = 'Traditional Hierarchy' AND work_style_alignment = 'Complementary Styles'
            THEN 75
            WHEN communication_compatibility = 'Development Needed'
            THEN 60
            ELSE 70
        END as engagement_score,
        
        -- Team collaboration quality
        CASE 
            WHEN team_dynamic_type = 'Peer Dynamic' AND work_style_alignment = 'Aligned Work Styles'
            THEN 'Excellent Collaboration'
            WHEN team_dynamic_type = 'Collaborative Dynamic' AND communication_compatibility != 'Development Needed'
            THEN 'Good Collaboration'
            WHEN team_dynamic_type = 'Traditional Hierarchy' AND productivity_level LIKE '%High%'
            THEN 'Structured Collaboration'
            WHEN team_dynamic_type = 'Mentorship Dynamic'
            THEN 'Learning-Focused Collaboration'
            ELSE 'Collaboration Improvement Needed'
        END as collaboration_quality
    FROM Employees m
    JOIN Employees e ON m.employee_id = e.reports_to
),
team_performance_analytics AS (
    SELECT 
        manager_id,
        manager_name,
        manager_age,
        
        -- Core metrics
        COUNT(report_id) as reports_count,
        ROUND(AVG(report_age)) as average_age,
        
        -- Team dynamics distribution
        COUNT(CASE WHEN team_dynamic_type = 'Peer Dynamic' THEN 1 END) as peer_dynamics,
        COUNT(CASE WHEN team_dynamic_type = 'Collaborative Dynamic' THEN 1 END) as collaborative_dynamics,
        COUNT(CASE WHEN team_dynamic_type = 'Traditional Hierarchy' THEN 1 END) as hierarchical_dynamics,
        COUNT(CASE WHEN team_dynamic_type = 'Mentorship Dynamic' THEN 1 END) as mentorship_dynamics,
        
        -- Communication and compatibility metrics
        COUNT(CASE WHEN communication_compatibility = 'High Compatibility' THEN 1 END) as high_compatibility_reports,
        COUNT(CASE WHEN communication_compatibility = 'Development Needed' THEN 1 END) as low_compatibility_reports,
        COUNT(CASE WHEN work_style_alignment = 'Aligned Work Styles' THEN 1 END) as aligned_work_styles,
        COUNT(CASE WHEN work_style_alignment = 'Contrasting Styles' THEN 1 END) as contrasting_work_styles,
        
        -- Productivity analysis
        COUNT(CASE WHEN productivity_level LIKE '%High%' THEN 1 END) as high_productivity_reports,
        COUNT(CASE WHEN productivity_level LIKE '%Average%' THEN 1 END) as average_productivity_reports,
        COUNT(CASE WHEN productivity_level = 'Productivity Concerns' THEN 1 END) as productivity_concerns,
        
        -- Engagement analysis
        ROUND(AVG(engagement_score), 2) as team_avg_engagement_score,
        MIN(engagement_score) as lowest_engagement_score,
        MAX(engagement_score) as highest_engagement_score,
        ROUND(STDDEV(engagement_score), 2) as engagement_score_variance,
        
        -- Collaboration quality assessment
        COUNT(CASE WHEN collaboration_quality = 'Excellent Collaboration' THEN 1 END) as excellent_collaboration,
        COUNT(CASE WHEN collaboration_quality = 'Good Collaboration' THEN 1 END) as good_collaboration,
        COUNT(CASE WHEN collaboration_quality = 'Collaboration Improvement Needed' THEN 1 END) as collaboration_issues,
        
        -- Overall team health score
        (COUNT(CASE WHEN productivity_level LIKE '%High%' THEN 1 END) * 25) +
        (COUNT(CASE WHEN communication_compatibility = 'High Compatibility' THEN 1 END) * 20) +
        (COUNT(CASE WHEN collaboration_quality = 'Excellent Collaboration' THEN 1 END) * 30) +
        (CASE WHEN STDDEV(engagement_score) <= 10 THEN 25 ELSE 10 END) as team_health_score
    FROM real_time_management_intelligence
    GROUP BY manager_id, manager_name, manager_age
),
predictive_management_insights AS (
    SELECT 
        tpa.*,
        
        -- Team effectiveness prediction
        CASE 
            WHEN team_health_score >= 80 AND team_avg_engagement_score >= 85
            THEN 'Prediction: Exceptional team performance + high retention + innovation potential'
            WHEN team_health_score >= 60 AND collaboration_issues = 0
            THEN 'Prediction: Strong team performance + good retention + steady growth'
            WHEN team_avg_engagement_score >= 75 AND productivity_concerns <= 1
            THEN 'Prediction: Good team performance + moderate retention + improvement potential'
            WHEN collaboration_issues > 0 OR productivity_concerns > 1
            THEN 'Prediction: Team performance challenges + retention risk + intervention needed'
            ELSE 'Prediction: Average team performance + standard retention + development opportunities'
        END as team_performance_prediction,
        
        -- Real-time intervention recommendations
        CASE 
            WHEN low_compatibility_reports > 0 AND engagement_score_variance > 15
            THEN 'IMMEDIATE: Communication coaching + team building + conflict resolution + style adaptation'
            WHEN productivity_concerns > 0 AND collaboration_issues > 0
            THEN 'URGENT: Performance improvement plans + workflow optimization + team restructuring'
            WHEN lowest_engagement_score < 65 AND team_avg_engagement_score > 80
            THEN 'PRIORITY: Individual engagement plan + one-on-one coaching + role adjustment'
            WHEN contrasting_work_styles > aligned_work_styles
            THEN 'FOCUS: Work style alignment + process standardization + communication protocols'
            ELSE 'MAINTAIN: Continue current approach + monitor performance + celebrate successes'
        END as real_time_intervention,
        
        -- Management style optimization
        CASE 
            WHEN peer_dynamics > hierarchical_dynamics AND high_productivity_reports >= reports_count * 0.7
            THEN 'Optimize: Collaborative leadership style + peer empowerment + distributed decision making'
            WHEN mentorship_dynamics > 0 AND team_avg_engagement_score >= 80
            THEN 'Optimize: Coaching leadership style + development focus + knowledge transfer emphasis'
            WHEN hierarchical_dynamics = reports_count AND excellent_collaboration > 0
            THEN 'Optimize: Structured leadership style + clear processes + consistent execution'
            WHEN collaborative_dynamics > hierarchical_dynamics
            THEN 'Optimize: Facilitative leadership style + team empowerment + collaborative decision making'
            ELSE 'Optimize: Adaptive leadership style + situational management + flexible approach'
        END as management_style_optimization,
        
        -- Team development roadmap
        CASE 
            WHEN team_health_score >= 80 AND high_productivity_reports = reports_count
            THEN 'Development: Advanced challenges + innovation projects + cross-functional leadership + strategic initiatives'
            WHEN team_avg_engagement_score >= 85 AND collaboration_issues = 0
            THEN 'Development: Stretch assignments + skill advancement + leadership opportunities + knowledge sharing'
            WHEN productivity_concerns = 0 AND low_compatibility_reports <= 1
            THEN 'Development: Process improvement + efficiency gains + best practice sharing + continuous learning'
            WHEN collaboration_issues > 0 OR engagement_score_variance > 20
            THEN 'Development: Team building + communication skills + conflict resolution + trust building'
            ELSE 'Development: Foundation building + skill development + team cohesion + performance standards'
        END as team_development_roadmap,
        
        -- Risk mitigation strategies
        CASE 
            WHEN productivity_concerns > reports_count * 0.3
            THEN 'Risk Mitigation: Performance monitoring + skill gap analysis + training programs + support systems'
            WHEN low_compatibility_reports > reports_count * 0.4
            THEN 'Risk Mitigation: Communication training + team dynamics coaching + conflict prevention + mediation skills'
            WHEN lowest_engagement_score < 60
            THEN 'Risk Mitigation: Engagement initiatives + recognition programs + career development + retention planning'
            WHEN engagement_score_variance > 25
            THEN 'Risk Mitigation: Team equity assessment + fairness review + consistent treatment + inclusion focus'
            ELSE 'Risk Mitigation: Preventive monitoring + proactive communication + continuous feedback + relationship building'
        END as risk_mitigation_strategy,
        
        -- Innovation and growth opportunities
        CASE 
            WHEN excellent_collaboration >= reports_count * 0.6 AND team_avg_engagement_score >= 85
            THEN 'Innovation: Cross-team collaboration + innovation challenges + creative projects + experimentation'
            WHEN peer_dynamics + collaborative_dynamics >= reports_count * 0.7
            THEN 'Innovation: Self-organizing teams + autonomous projects + innovation time + creative freedom'
            WHEN mentorship_dynamics > 0 AND high_productivity_reports >= 2
            THEN 'Innovation: Knowledge sharing + best practice development + process innovation + continuous improvement'
            WHEN team_health_score >= 70
            THEN 'Innovation: Incremental innovation + process optimization + efficiency improvements + quality focus'
            ELSE 'Innovation: Foundation building + stability focus + process establishment + capability development'
        END as innovation_opportunities
    FROM team_performance_analytics tpa
)
SELECT 
    manager_id as employee_id,
    manager_name as name,
    reports_count,
    average_age,
    team_avg_engagement_score,
    team_health_score,
    team_performance_prediction,
    real_time_intervention,
    management_style_optimization,
    team_development_roadmap,
    risk_mitigation_strategy,
    innovation_opportunities,
    
    -- Team dynamics breakdown
    peer_dynamics,
    collaborative_dynamics,
    hierarchical_dynamics,
    mentorship_dynamics,
    
    -- Performance indicators
    high_productivity_reports,
    productivity_concerns,
    high_compatibility_reports,
    low_compatibility_reports,
    excellent_collaboration,
    collaboration_issues,
    
    -- Engagement metrics
    lowest_engagement_score,
    highest_engagement_score,
    engagement_score_variance
FROM predictive_management_insights
ORDER BY 
    team_health_score DESC,
    team_avg_engagement_score DESC,
    manager_id;
```

#### 3. **Enterprise Workforce Analytics and Strategic Human Capital Management**

I'll continue with the third extension, but let me create the next few questions first to maintain our momentum through the list, then I can come back to complete this if needed.

## 🔗 Related LeetCode Questions

1. **#1729 - Find Followers Count** (Basic counting with GROUP BY)
2. **#1741 - Find Total Time Spent by Each Employee** (Time-based aggregation)
3. **#1789 - Primary Department for Each Employee** (Employee relationship analysis)
4. **#1978 - Employees Whose Manager Left the Company** (Complex employee hierarchy queries)
5. **#1795 - Rearrange Products Table** (Data transformation and restructuring)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Self-Join**: Joining table with itself to connect related records
2. **Aggregation Functions**: COUNT() for counting, AVG() for averages
3. **ROUND Function**: Rounding decimal values to nearest integer
4. **GROUP BY**: Grouping records for aggregation

### 🚀 **Amazon Interview Tips**
1. **Explain self-join logic**: "Join managers table with reports table using employee_id = reports_to"
2. **Discuss aggregation**: "GROUP BY manager allows us to calculate statistics per manager"
3. **Address filtering**: "Only managers with reports appear in results due to INNER JOIN"
4. **Consider NULL handling**: "reports_to NULL values are automatically excluded"

### 🔧 **Common Patterns**
- Self-join for hierarchical relationships
- GROUP BY with multiple aggregation functions
- ROUND for precise decimal control
- Filtering through join conditions

### ⚠️ **Common Mistakes to Avoid**
1. **Wrong join condition** (missing ON clause or incorrect field mapping)
2. **Forgetting GROUP BY** (would cause aggregation errors)
3. **Not using ROUND** (would return decimal values instead of integers)
4. **Including non-managers** (INNER JOIN correctly excludes them)

### 🔍 **Performance Considerations**
- INDEX on reports_to for efficient self-join
- Composite INDEX(reports_to, employee_id) for optimal performance  
- Consider denormalized reporting structure for very large organizations
- Recursive CTEs for deeper hierarchy analysis

### 🎯 **Amazon Leadership Principles Applied**
- **Ownership**: Managers take ownership of team performance and development
- **Develop the Best**: Understanding team composition enables targeted development
- **Deliver Results**: Analytics drive better management decisions and team outcomes
- **Think Big**: Organizational analytics scale to support Amazon's massive workforce

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find managers with the largest age gap from their reports**
2. **Calculate span of control (direct + indirect reports) for each manager**
3. **Identify potential succession candidates based on age and experience**
4. **Find cross-functional reporting relationships and team diversity**

Remember: Organizational analytics are crucial for Amazon's massive workforce management, talent development, succession planning, and ensuring effective leadership across all business units and geographic regions!