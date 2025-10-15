# LeetCode Easy #2356: Number of Unique Subjects Taught by Each Teacher

## 📋 Problem Statement

Write a SQL query to find the **number of unique subjects** each teacher teaches in the university.

Return the result table in **any order**.

## 🗄️ Table Schema

**Teacher Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| teacher_id  | int  |
| subject_id  | int  |
| dept_id     | int  |
+-------------+------+
```
- (subject_id, dept_id) is the primary key for this table.
- Each row in this table indicates that the teacher with teacher_id teaches the subject subject_id in the department dept_id.

## 📊 Sample Data

**Teacher Table:**
| teacher_id | subject_id | dept_id |
|------------|------------|---------|
| 1          | 2          | 3       |
| 1          | 2          | 4       |
| 1          | 3          | 3       |
| 2          | 1          | 1       |
| 2          | 2          | 1       |
| 2          | 3          | 1       |
| 2          | 4          | 1       |

**Expected Output:**
| teacher_id | cnt |
|------------|-----|
| 1          | 2   |
| 2          | 4   |

**Explanation:**
- Teacher 1 teaches subject 2 in departments 3 and 4, and subject 3 in department 3. So teacher 1 teaches 2 unique subjects (2 and 3).
- Teacher 2 teaches subjects 1, 2, 3, and 4 all in department 1. So teacher 2 teaches 4 unique subjects.

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Count UNIQUE subjects per teacher (not subject-department combinations)
- A teacher can teach the same subject in multiple departments
- Need to use DISTINCT to avoid counting duplicate subjects
- GROUP BY teacher_id to aggregate per teacher

### 2. **Key Insights**
- Use COUNT(DISTINCT subject_id) to count unique subjects
- GROUP BY teacher_id to get per-teacher counts
- Same subject in different departments counts as one subject
- Result column should be named 'cnt'

### 3. **Interview Discussion Points**
- "Need to count unique subjects, not total rows"
- "Same subject in different departments is still one subject"
- "Use COUNT(DISTINCT) to eliminate duplicate subjects per teacher"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Grouping Column
```sql
-- Group by teacher to get per-teacher aggregation
GROUP BY teacher_id
```

### Step 2: Count Unique Subjects
```sql
-- Use COUNT(DISTINCT) to count unique subjects per teacher
COUNT(DISTINCT subject_id) as cnt
```

### Step 3: Select Required Columns
```sql
-- Select teacher_id and the count
SELECT teacher_id, COUNT(DISTINCT subject_id) as cnt
```

## ✅ Optimized SQL Solution

**Solution 1: Basic COUNT DISTINCT**
```sql
SELECT 
    teacher_id,
    COUNT(DISTINCT subject_id) as cnt
FROM Teacher
GROUP BY teacher_id;
```

### Alternative Solutions

**Solution 2: Using Subquery with DISTINCT**
```sql
SELECT 
    teacher_id,
    COUNT(*) as cnt
FROM (
    SELECT DISTINCT teacher_id, subject_id
    FROM Teacher
) unique_teacher_subjects
GROUP BY teacher_id;
```

**Solution 3: Window Function Approach**
```sql
SELECT DISTINCT
    teacher_id,
    COUNT(DISTINCT subject_id) OVER(PARTITION BY teacher_id) as cnt
FROM Teacher;
```

**Solution 4: CTE with Detailed Analysis**
```sql
WITH teacher_subjects AS (
    SELECT DISTINCT teacher_id, subject_id
    FROM Teacher
)
SELECT 
    teacher_id,
    COUNT(subject_id) as cnt
FROM teacher_subjects
GROUP BY teacher_id;
```

**Solution 5: Comprehensive Academic Analytics and Educational Intelligence System**
```sql
WITH teacher_subject_analysis AS (
    SELECT 
        teacher_id,
        subject_id,
        dept_id,
        
        -- Department categorization
        CASE 
            WHEN dept_id % 5 = 1 THEN 'STEM'
            WHEN dept_id % 5 = 2 THEN 'Liberal Arts'
            WHEN dept_id % 5 = 3 THEN 'Business'
            WHEN dept_id % 5 = 4 THEN 'Social Sciences'
            ELSE 'Professional Studies'
        END as department_category,
        
        -- Subject classification
        CASE 
            WHEN subject_id % 6 = 1 THEN 'Mathematics'
            WHEN subject_id % 6 = 2 THEN 'Computer Science'
            WHEN subject_id % 6 = 3 THEN 'Natural Sciences'
            WHEN subject_id % 6 = 4 THEN 'Engineering'
            WHEN subject_id % 6 = 5 THEN 'Humanities'
            ELSE 'Social Studies'
        END as subject_category,
        
        -- Teaching load simulation
        CASE 
            WHEN teacher_id % 4 = 0 THEN 'Full-Time Professor'
            WHEN teacher_id % 4 = 1 THEN 'Associate Professor'
            WHEN teacher_id % 4 = 2 THEN 'Assistant Professor'
            ELSE 'Adjunct Instructor'
        END as faculty_level,
        
        -- Academic specialization depth
        COUNT(*) OVER(PARTITION BY teacher_id, subject_id) as subject_sections_taught,
        COUNT(DISTINCT dept_id) OVER(PARTITION BY teacher_id, subject_id) as departments_per_subject,
        
        -- Cross-departmental teaching indicator
        CASE 
            WHEN COUNT(DISTINCT dept_id) OVER(PARTITION BY teacher_id, subject_id) > 1 THEN 'Cross-Departmental'
            ELSE 'Single Department'
        END as teaching_scope,
        
        -- Complexity assessment
        CASE 
            WHEN subject_category IN ('Mathematics', 'Computer Science', 'Engineering') THEN 'High Complexity'
            WHEN subject_category IN ('Natural Sciences', 'Business') THEN 'Medium Complexity'
            ELSE 'Standard Complexity'
        END as subject_complexity,
        
        -- Student impact simulation
        CASE 
            WHEN faculty_level = 'Full-Time Professor' AND subject_complexity = 'High Complexity' THEN 'High Student Impact'
            WHEN faculty_level IN ('Associate Professor', 'Assistant Professor') THEN 'Medium Student Impact'
            ELSE 'Standard Student Impact'
        END as student_impact_level,
        
        -- Research potential
        CASE 
            WHEN subject_category IN ('Computer Science', 'Engineering', 'Natural Sciences') AND faculty_level LIKE '%Professor%'
            THEN 'High Research Potential'
            WHEN subject_category IN ('Mathematics', 'Business') AND faculty_level LIKE '%Professor%'
            THEN 'Medium Research Potential'
            ELSE 'Teaching Focus'
        END as research_potential
    FROM Teacher
),
academic_performance_metrics AS (
    SELECT 
        teacher_id,
        faculty_level,
        
        -- Core subject metrics
        COUNT(DISTINCT subject_id) as unique_subjects_taught,
        COUNT(DISTINCT dept_id) as departments_involved,
        COUNT(*) as total_teaching_assignments,
        
        -- Subject diversity analysis
        COUNT(DISTINCT subject_category) as subject_categories_covered,
        COUNT(DISTINCT department_category) as department_categories_involved,
        
        -- Teaching scope and specialization
        COUNT(CASE WHEN teaching_scope = 'Cross-Departmental' THEN 1 END) as cross_departmental_subjects,
        COUNT(CASE WHEN subject_complexity = 'High Complexity' THEN 1 END) as high_complexity_subjects,
        COUNT(CASE WHEN student_impact_level = 'High Student Impact' THEN 1 END) as high_impact_assignments,
        COUNT(CASE WHEN research_potential = 'High Research Potential' THEN 1 END) as research_oriented_subjects,
        
        -- Department diversity
        STRING_AGG(DISTINCT department_category, ', ') as department_portfolio,
        STRING_AGG(DISTINCT subject_category, ', ') as subject_portfolio,
        
        -- Academic excellence indicators
        CASE 
            WHEN COUNT(DISTINCT subject_id) >= 4 AND COUNT(DISTINCT dept_id) >= 2 THEN 'Versatile Educator'
            WHEN COUNT(DISTINCT subject_id) >= 3 AND COUNT(CASE WHEN subject_complexity = 'High Complexity' THEN 1 END) >= 2 THEN 'Expert Educator'
            WHEN COUNT(DISTINCT subject_id) >= 2 AND COUNT(DISTINCT subject_category) >= 2 THEN 'Diverse Educator'
            WHEN COUNT(DISTINCT subject_id) = 1 THEN 'Specialized Educator'
            ELSE 'Standard Educator'
        END as educator_classification,
        
        -- Teaching load assessment
        CASE 
            WHEN COUNT(*) >= 6 THEN 'Heavy Teaching Load'
            WHEN COUNT(*) >= 4 THEN 'Standard Teaching Load'
            WHEN COUNT(*) >= 2 THEN 'Light Teaching Load'
            ELSE 'Minimal Teaching Load'
        END as teaching_load_level,
        
        -- Academic breadth score
        (COUNT(DISTINCT subject_id) * 10) + 
        (COUNT(DISTINCT dept_id) * 5) + 
        (COUNT(DISTINCT subject_category) * 8) +
        (COUNT(CASE WHEN teaching_scope = 'Cross-Departmental' THEN 1 END) * 3) as academic_breadth_score,
        
        -- Specialization vs diversification balance
        CASE 
            WHEN COUNT(DISTINCT subject_id) >= 4 AND COUNT(DISTINCT subject_category) >= 3
            THEN 'Diversification Strategy: Broad expertise + multi-disciplinary + versatile teaching + comprehensive knowledge'
            WHEN COUNT(DISTINCT subject_id) <= 2 AND COUNT(CASE WHEN subject_complexity = 'High Complexity' THEN 1 END) >= 1
            THEN 'Specialization Strategy: Deep expertise + focused teaching + subject mastery + specialized knowledge'
            WHEN COUNT(DISTINCT dept_id) >= 3
            THEN 'Cross-Departmental Strategy: Interdisciplinary + collaborative + bridge-building + institutional integration'
            ELSE 'Balanced Strategy: Moderate expertise + steady teaching + reliable contribution + consistent performance'
        END as academic_strategy,
        
        -- Professional development opportunities
        CASE 
            WHEN COUNT(DISTINCT subject_category) >= 3 AND faculty_level LIKE '%Professor%'
            THEN 'Development: Curriculum leadership + interdisciplinary research + academic innovation + program development'
            WHEN COUNT(CASE WHEN research_potential = 'High Research Potential' THEN 1 END) >= 2
            THEN 'Development: Research leadership + grant opportunities + publication potential + academic advancement'
            WHEN COUNT(DISTINCT dept_id) >= 2 AND educator_classification = 'Versatile Educator'
            THEN 'Development: Administrative potential + department coordination + institutional leadership + collaborative excellence'
            WHEN educator_classification = 'Expert Educator'
            THEN 'Development: Subject matter expertise + curriculum design + teaching excellence + mentoring leadership'
            ELSE 'Development: Professional growth + teaching improvement + skill enhancement + career advancement'
        END as professional_development_path
    FROM teacher_subject_analysis
    GROUP BY teacher_id, faculty_level
),
institutional_excellence_assessment AS (
    SELECT 
        apm.*,
        
        -- Institutional value assessment
        CASE 
            WHEN educator_classification = 'Versatile Educator' AND academic_breadth_score >= 50
            THEN 'Institutional Value: High - Multi-departmental asset + curriculum flexibility + institutional adaptability + strategic resource'
            WHEN educator_classification = 'Expert Educator' AND high_complexity_subjects >= 2
            THEN 'Institutional Value: High - Subject matter excellence + specialized expertise + academic reputation + research potential'
            WHEN teaching_load_level = 'Heavy Teaching Load' AND departments_involved >= 2
            THEN 'Institutional Value: Medium-High - Heavy contributor + multi-department support + institutional reliability + workload champion'
            WHEN cross_departmental_subjects >= 2
            THEN 'Institutional Value: Medium - Cross-departmental collaboration + institutional integration + bridge-building + flexibility'
            ELSE 'Institutional Value: Standard - Reliable contributor + consistent teaching + departmental support + steady performance'
        END as institutional_value,
        
        -- Student experience impact
        CASE 
            WHEN high_impact_assignments >= 3 AND subject_categories_covered >= 3
            THEN 'Student Experience: Exceptional - Diverse learning + high-impact education + comprehensive knowledge + transformative experience'
            WHEN high_impact_assignments >= 2 AND unique_subjects_taught >= 3
            THEN 'Student Experience: Excellent - Quality education + subject variety + engaging learning + academic excellence'
            WHEN unique_subjects_taught >= 2 AND departments_involved >= 2
            THEN 'Student Experience: Good - Educational variety + cross-departmental perspective + learning diversity + academic exposure'
            WHEN educator_classification = 'Specialized Educator'
            THEN 'Student Experience: Focused - Deep subject knowledge + specialized learning + expert instruction + concentrated expertise'
            ELSE 'Student Experience: Standard - Quality education + reliable instruction + consistent learning + academic support'
        END as student_experience_impact,
        
        -- Academic leadership potential
        CASE 
            WHEN faculty_level = 'Full-Time Professor' AND academic_breadth_score >= 60
            THEN 'Leadership Potential: Executive - Dean potential + institutional leadership + strategic vision + organizational excellence'
            WHEN faculty_level LIKE '%Professor%' AND educator_classification = 'Versatile Educator'
            THEN 'Leadership Potential: Administrative - Department chair + program director + curriculum leadership + academic management'
            WHEN educator_classification = 'Expert Educator' AND research_oriented_subjects >= 2
            THEN 'Leadership Potential: Academic - Research leadership + subject expertise + scholarly excellence + intellectual leadership'
            WHEN cross_departmental_subjects >= 2 AND departments_involved >= 3
            THEN 'Leadership Potential: Collaborative - Interdisciplinary coordinator + cross-departmental liaison + integration leadership'
            ELSE 'Leadership Potential: Emerging - Professional development + leadership preparation + skill building + advancement opportunity'
        END as academic_leadership_potential,
        
        -- Resource allocation and optimization recommendations
        CASE 
            WHEN teaching_load_level = 'Heavy Teaching Load' AND unique_subjects_taught >= 4
            THEN 'Resource Optimization: Workload balance + teaching load review + support allocation + efficiency enhancement + burnout prevention'
            WHEN educator_classification = 'Versatile Educator' AND academic_breadth_score >= 50
            THEN 'Resource Optimization: Strategic deployment + high-value assignments + leadership development + institutional leverage + career advancement'
            WHEN educator_classification = 'Expert Educator' AND research_potential = 'High Research Potential'
            THEN 'Resource Optimization: Research support + reduced teaching load + grant opportunities + publication support + scholarly development'
            WHEN teaching_load_level = 'Light Teaching Load' AND departments_involved = 1
            THEN 'Resource Optimization: Capacity expansion + additional assignments + skill development + contribution increase + growth opportunity'
            ELSE 'Resource Optimization: Maintain current allocation + performance monitoring + development support + strategic positioning + career guidance'
        END as resource_optimization_strategy,
        
        -- University strategic positioning
        CASE 
            WHEN academic_breadth_score >= 60 AND research_oriented_subjects >= 2
            THEN 'Strategic Position: University flagship + institutional pride + academic excellence + research leadership + competitive advantage'
            WHEN educator_classification = 'Versatile Educator' AND department_categories_involved >= 3
            THEN 'Strategic Position: Institutional integration + cross-disciplinary strength + collaborative excellence + flexibility asset + strategic resource'
            WHEN high_complexity_subjects >= 3 AND faculty_level LIKE '%Professor%'
            THEN 'Strategic Position: Academic reputation + subject excellence + institutional credibility + scholarly distinction + competitive edge'
            WHEN cross_departmental_subjects >= 3
            THEN 'Strategic Position: Collaborative advantage + interdisciplinary strength + institutional cohesion + integration excellence + unity building'
            ELSE 'Strategic Position: Institutional foundation + reliable excellence + steady contribution + academic stability + consistent quality'
        END as university_strategic_positioning
    FROM academic_performance_metrics apm
)
SELECT 
    teacher_id,
    unique_subjects_taught as cnt,
    faculty_level,
    educator_classification,
    academic_strategy,
    institutional_value,
    student_experience_impact,
    academic_leadership_potential,
    professional_development_path,
    resource_optimization_strategy,
    university_strategic_positioning,
    
    -- Core metrics
    departments_involved,
    total_teaching_assignments,
    subject_categories_covered,
    department_categories_involved,
    academic_breadth_score,
    
    -- Specialized metrics
    cross_departmental_subjects,
    high_complexity_subjects,
    high_impact_assignments,
    research_oriented_subjects,
    teaching_load_level,
    
    -- Portfolio summaries
    department_portfolio,
    subject_portfolio
FROM institutional_excellence_assessment
ORDER BY 
    academic_breadth_score DESC,
    unique_subjects_taught DESC,
    CASE educator_classification
        WHEN 'Versatile Educator' THEN 1
        WHEN 'Expert Educator' THEN 2
        WHEN 'Diverse Educator' THEN 3
        WHEN 'Specialized Educator' THEN 4
        ELSE 5
    END,
    teacher_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Amazon Learning and Development Platform with Skills Analytics**
```sql
-- "Design comprehensive learning and development analytics system for Amazon's massive workforce training and skill development programs"

WITH amazon_learning_analytics AS (
    SELECT 
        teacher_id as instructor_id,
        subject_id as skill_id,
        dept_id as business_unit_id,
        
        -- Amazon business unit mapping
        CASE 
            WHEN dept_id % 10 = 1 THEN 'AWS Training'
            WHEN dept_id % 10 = 2 THEN 'Leadership Development'
            WHEN dept_id % 10 = 3 THEN 'Technical Skills'
            WHEN dept_id % 10 = 4 THEN 'Customer Service Excellence'
            WHEN dept_id % 10 = 5 THEN 'Operations & Logistics'
            WHEN dept_id % 10 = 6 THEN 'Data Science & Analytics'
            WHEN dept_id % 10 = 7 THEN 'Software Development'
            WHEN dept_id % 10 = 8 THEN 'Product Management'
            WHEN dept_id % 10 = 9 THEN 'Digital Marketing'
            ELSE 'Innovation & Emerging Tech'
        END as amazon_learning_division,
        
        -- Skill categorization for Amazon workforce
        CASE 
            WHEN subject_id % 8 = 1 THEN 'Cloud Computing & AWS'
            WHEN subject_id % 8 = 2 THEN 'Machine Learning & AI'
            WHEN subject_id % 8 = 3 THEN 'Data Engineering & Analytics'
            WHEN subject_id % 8 = 4 THEN 'Software Engineering'
            WHEN subject_id % 8 = 5 THEN 'Leadership & Management'
            WHEN subject_id % 8 = 6 THEN 'Customer Experience'
            WHEN subject_id % 8 = 7 THEN 'Operations Excellence'
            ELSE 'Digital Innovation'
        END as amazon_skill_category,
        
        -- Instructor level classification
        CASE 
            WHEN teacher_id % 5 = 0 THEN 'Principal Instructor (L7-L8)'
            WHEN teacher_id % 5 = 1 THEN 'Senior Instructor (L6-L7)'
            WHEN teacher_id % 5 = 2 THEN 'Lead Instructor (L5-L6)'
            WHEN teacher_id % 5 = 3 THEN 'Instructor (L4-L5)'
            ELSE 'Associate Instructor (L3-L4)'
        END as amazon_instructor_level,
        
        -- Learning delivery method
        CASE 
            WHEN dept_id % 3 = 0 THEN 'Virtual Learning'
            WHEN dept_id % 3 = 1 THEN 'Hands-On Workshops'
            ELSE 'Blended Learning'
        END as delivery_method,
        
        -- Strategic importance for Amazon
        CASE 
            WHEN amazon_skill_category IN ('Cloud Computing & AWS', 'Machine Learning & AI') THEN 'Strategic Priority'
            WHEN amazon_skill_category IN ('Data Engineering & Analytics', 'Software Engineering') THEN 'High Priority'
            WHEN amazon_skill_category IN ('Leadership & Management', 'Customer Experience') THEN 'Business Critical'
            ELSE 'Standard Priority'
        END as amazon_strategic_importance,
        
        -- Employee target audience
        CASE 
            WHEN amazon_learning_division = 'AWS Training' THEN 'Technical Professionals + Cloud Engineers + Solutions Architects'
            WHEN amazon_learning_division = 'Leadership Development' THEN 'Managers + Directors + VPs + Future Leaders'
            WHEN amazon_learning_division = 'Customer Service Excellence' THEN 'Customer Service + Support + Account Management'
            WHEN amazon_learning_division = 'Operations & Logistics' THEN 'Fulfillment + Supply Chain + Operations + Logistics'
            ELSE 'Cross-Functional Teams + All Employees + Skill Development'
        END as target_audience,
        
        -- Innovation and future skills alignment
        CASE 
            WHEN amazon_skill_category IN ('Machine Learning & AI', 'Digital Innovation') THEN 'Future Skills'
            WHEN amazon_skill_category IN ('Cloud Computing & AWS', 'Data Engineering & Analytics') THEN 'Current Strategic Skills'
            WHEN amazon_skill_category IN ('Software Engineering', 'Operations Excellence') THEN 'Core Business Skills'
            ELSE 'Foundational Skills'
        END as skill_evolution_category
    FROM Teacher
),
amazon_instructor_excellence AS (
    SELECT 
        instructor_id,
        amazon_instructor_level,
        
        -- Core teaching metrics
        COUNT(DISTINCT skill_id) as unique_skills_taught,
        COUNT(DISTINCT business_unit_id) as business_units_covered,
        COUNT(DISTINCT amazon_learning_division) as learning_divisions_involved,
        COUNT(DISTINCT amazon_skill_category) as skill_categories_mastered,
        COUNT(*) as total_training_programs,
        
        -- Strategic skill distribution
        COUNT(CASE WHEN amazon_strategic_importance = 'Strategic Priority' THEN 1 END) as strategic_priority_skills,
        COUNT(CASE WHEN amazon_strategic_importance = 'High Priority' THEN 1 END) as high_priority_skills,
        COUNT(CASE WHEN amazon_strategic_importance = 'Business Critical' THEN 1 END) as business_critical_skills,
        
        -- Future readiness assessment
        COUNT(CASE WHEN skill_evolution_category = 'Future Skills' THEN 1 END) as future_skills_taught,
        COUNT(CASE WHEN skill_evolution_category = 'Current Strategic Skills' THEN 1 END) as strategic_skills_taught,
        COUNT(CASE WHEN skill_evolution_category = 'Core Business Skills' THEN 1 END) as core_skills_taught,
        
        -- Delivery method expertise
        COUNT(CASE WHEN delivery_method = 'Virtual Learning' THEN 1 END) as virtual_programs,
        COUNT(CASE WHEN delivery_method = 'Hands-On Workshops' THEN 1 END) as workshop_programs,
        COUNT(CASE WHEN delivery_method = 'Blended Learning' THEN 1 END) as blended_programs,
        
        -- Amazon business impact
        STRING_AGG(DISTINCT amazon_learning_division, ', ') as learning_portfolio,
        STRING_AGG(DISTINCT amazon_skill_category, ', ') as skill_portfolio,
        STRING_AGG(DISTINCT target_audience, '; ') as audience_reach,
        
        -- Instructor classification for Amazon context
        CASE 
            WHEN COUNT(DISTINCT skill_id) >= 4 AND COUNT(CASE WHEN amazon_strategic_importance = 'Strategic Priority' THEN 1 END) >= 2
            THEN 'Strategic Learning Leader'
            WHEN COUNT(DISTINCT amazon_learning_division) >= 3 AND COUNT(DISTINCT skill_id) >= 3
            THEN 'Cross-Functional Learning Expert'
            WHEN COUNT(CASE WHEN skill_evolution_category = 'Future Skills' THEN 1 END) >= 2
            THEN 'Future Skills Innovator'
            WHEN COUNT(DISTINCT skill_id) >= 3 AND COUNT(CASE WHEN amazon_strategic_importance = 'High Priority' THEN 1 END) >= 2
            THEN 'High-Impact Instructor'
            WHEN COUNT(DISTINCT skill_id) = 1
            THEN 'Specialized Subject Expert'
            ELSE 'Professional Development Contributor'
        END as amazon_instructor_classification,
        
        -- Learning and development impact score
        (COUNT(DISTINCT skill_id) * 15) +
        (COUNT(CASE WHEN amazon_strategic_importance = 'Strategic Priority' THEN 1 END) * 25) +
        (COUNT(CASE WHEN skill_evolution_category = 'Future Skills' THEN 1 END) * 20) +
        (COUNT(DISTINCT amazon_learning_division) * 10) +
        (COUNT(CASE WHEN amazon_strategic_importance = 'Business Critical' THEN 1 END) * 12) as learning_impact_score
    FROM amazon_learning_analytics
    GROUP BY instructor_id, amazon_instructor_level
),
amazon_leadership_principles_learning AS (
    SELECT 
        aie.*,
        
        -- Customer Obsession in learning context
        CASE 
            WHEN business_critical_skills >= 2 AND COUNT(CASE WHEN amazon_skill_category = 'Customer Experience' THEN 1 END) >= 1
            THEN 'Customer Obsession: Customer-centric learning + service excellence + customer experience mastery + client success focus'
            WHEN strategic_priority_skills >= 2 AND unique_skills_taught >= 3
            THEN 'Customer Obsession: Strategic skill development + workforce excellence + customer value enablement + service capability building'
            WHEN amazon_instructor_classification = 'Cross-Functional Learning Expert'
            THEN 'Customer Obsession: Comprehensive skill building + cross-functional excellence + customer success enablement + holistic development'
            ELSE 'Customer Obsession: Employee development + skill enhancement + workforce capability + customer support through education'
        END as customer_obsession_learning,
        
        -- Ownership in learning and development
        CASE 
            WHEN amazon_instructor_classification = 'Strategic Learning Leader' AND learning_impact_score >= 100
            THEN 'Ownership: Enterprise learning accountability + strategic skill development + workforce transformation + long-term capability building'
            WHEN learning_divisions_involved >= 3 AND unique_skills_taught >= 4
            THEN 'Ownership: Cross-functional learning responsibility + multi-division skill development + comprehensive workforce development'
            WHEN future_skills_taught >= 2 AND amazon_instructor_level LIKE '%Principal%'
            THEN 'Ownership: Future skills leadership + innovation education + next-generation capability + strategic workforce preparation'
            WHEN amazon_instructor_classification = 'Specialized Subject Expert'
            THEN 'Ownership: Deep expertise accountability + subject matter mastery + specialized knowledge + expert-level training'
            ELSE 'Ownership: Professional development responsibility + skill building + workforce enhancement + capability advancement'
        END as ownership_learning_development,
        
        -- Invent and Simplify in training delivery
        CASE 
            WHEN virtual_programs >= 2 AND blended_programs >= 1 AND unique_skills_taught >= 3
            THEN 'Invent and Simplify: Multi-modal innovation + learning technology + delivery optimization + educational efficiency + scalable training'
            WHEN future_skills_taught >= 2 AND strategic_priority_skills >= 2
            THEN 'Invent and Simplify: Future skills innovation + cutting-edge education + next-generation training + learning advancement'
            WHEN amazon_instructor_classification = 'Cross-Functional Learning Expert'
            THEN 'Invent and Simplify: Cross-functional integration + learning consolidation + knowledge synthesis + educational streamlining'
            WHEN workshop_programs >= 2 AND core_skills_taught >= 2
            THEN 'Invent and Simplify: Hands-on innovation + practical learning + skill application + experiential education + effective training'
            ELSE 'Invent and Simplify: Training optimization + learning efficiency + skill development + educational improvement + method enhancement'
        END as invent_simplify_learning,
        
        -- Think Big vision for workforce development
        CASE 
            WHEN learning_impact_score >= 120 AND amazon_instructor_classification = 'Strategic Learning Leader'
            THEN 'Think Big: Workforce transformation + global learning leadership + enterprise skill revolution + strategic capability building + industry leadership'
            WHEN future_skills_taught >= 3 AND strategic_priority_skills >= 2
            THEN 'Think Big: Future workforce preparation + next-generation skills + innovation education + competitive advantage + market leadership'
            WHEN business_units_covered >= 4 AND unique_skills_taught >= 4
            THEN 'Think Big: Enterprise-wide impact + comprehensive skill development + organizational capability + workforce excellence + scaled learning'
            WHEN amazon_instructor_classification = 'Future Skills Innovator'
            THEN 'Think Big: Innovation education + emerging technology + future capability + pioneering learning + technological advancement'
            ELSE 'Think Big: Professional excellence + skill advancement + workforce development + capability enhancement + learning growth'
        END as think_big_learning_vision,
        
        -- Bias for Action in learning implementation
        CASE 
            WHEN total_training_programs >= 6 AND learning_divisions_involved >= 3
            THEN 'Bias for Action: Rapid skill deployment + immediate learning impact + fast capability building + quick workforce development + agile training'
            WHEN strategic_priority_skills >= 2 AND virtual_programs >= 2
            THEN 'Bias for Action: Strategic skill acceleration + priority learning + rapid deployment + immediate capability + fast-track development'
            WHEN workshop_programs >= 3 AND core_skills_taught >= 2
            THEN 'Bias for Action: Hands-on skill building + practical application + immediate implementation + rapid competency + active learning'
            WHEN amazon_instructor_classification = 'High-Impact Instructor'
            THEN 'Bias for Action: High-impact learning + immediate results + rapid skill acquisition + fast competency + quick capability building'
            ELSE 'Bias for Action: Consistent skill building + steady development + regular learning + continuous improvement + progressive advancement'
        END as bias_for_action_learning
    FROM amazon_instructor_excellence aie
)
SELECT 
    instructor_id as teacher_id,
    unique_skills_taught as cnt,
    amazon_instructor_level,
    amazon_instructor_classification,
    learning_impact_score,
    customer_obsession_learning,
    ownership_learning_development,
    invent_simplify_learning,
    think_big_learning_vision,
    bias_for_action_learning,
    
    -- Core metrics
    business_units_covered,
    learning_divisions_involved,
    skill_categories_mastered,
    total_training_programs,
    
    -- Strategic metrics
    strategic_priority_skills,
    high_priority_skills,
    business_critical_skills,
    future_skills_taught,
    strategic_skills_taught,
    core_skills_taught,
    
    -- Delivery and reach
    virtual_programs,
    workshop_programs,
    blended_programs,
    learning_portfolio,
    skill_portfolio,
    audience_reach
FROM amazon_leadership_principles_learning
ORDER BY 
    learning_impact_score DESC,
    unique_skills_taught DESC,
    CASE amazon_instructor_classification
        WHEN 'Strategic Learning Leader' THEN 1
        WHEN 'Future Skills Innovator' THEN 2
        WHEN 'Cross-Functional Learning Expert' THEN 3
        WHEN 'High-Impact Instructor' THEN 4
        WHEN 'Specialized Subject Expert' THEN 5
        ELSE 6
    END,
    instructor_id;
```

## 🔗 Related LeetCode Questions

1. **#1729 - Find Followers Count** (Basic COUNT with GROUP BY)
2. **#1741 - Find Total Time Spent by Each Employee** (Aggregation with GROUP BY)
3. **#1789 - Primary Department for Each Employee** (Employee analysis patterns)
4. **#1731 - The Number of Employees Which Report to Each Employee** (Counting relationships)
5. **#1978 - Employees Whose Manager Left the Company** (Complex employee analytics)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **COUNT DISTINCT**: Count unique values, eliminating duplicates
2. **GROUP BY**: Aggregate data per teacher for individual counts
3. **Duplicate Handling**: Same subject in different departments counts once
4. **Column Aliasing**: Use 'cnt' as required output column name

### 🚀 **Amazon Interview Tips**
1. **Explain DISTINCT importance**: "Same subject taught in multiple departments should count as one subject"
2. **Discuss aggregation**: "GROUP BY teacher_id gives us per-teacher counts"
3. **Address business logic**: "We want unique subjects, not total teaching assignments"
4. **Consider scalability**: "COUNT DISTINCT efficiently handles large datasets"

### 🔧 **Common Patterns**
- COUNT DISTINCT for unique value counting
- GROUP BY for per-entity aggregation
- Column aliasing for output formatting
- Elimination of duplicate values in counting

### ⚠️ **Common Mistakes to Avoid**
1. **Using COUNT(*) instead of COUNT DISTINCT** (would count duplicate subjects)
2. **Forgetting GROUP BY** (would give total count across all teachers)
3. **Wrong column name** (should be 'cnt' not 'count' or other names)
4. **Counting wrong column** (counting dept_id instead of subject_id)

### 🔍 **Performance Considerations**
- INDEX on teacher_id for efficient GROUP BY
- INDEX on (teacher_id, subject_id) for optimal COUNT DISTINCT
- COUNT DISTINCT can be expensive on very large datasets
- Consider using DISTINCT in subquery for very large tables

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Ensure comprehensive skill development serves both employees and customers
- **Ownership**: Take responsibility for developing diverse workforce capabilities
- **Think Big**: Scale learning programs across Amazon's global workforce
- **Deliver Results**: Measure and optimize teaching effectiveness and skill coverage

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find teachers who teach the most departments**
2. **Calculate average number of subjects per teacher**
3. **Find subjects taught by the most teachers**
4. **List teachers who teach in multiple departments**

Remember: Educational analytics like this power Amazon's massive learning and development programs, ensuring employees across AWS, retail, logistics, and other divisions receive comprehensive skill development to serve customers better!

---

# 🎉 **CONGRATULATIONS!** 

## **You've Successfully Completed All 45 LeetCode Easy Questions!**

**Your comprehensive Amazon Data Engineer interview preparation collection includes:**

### **📊 Original Collection (Questions 1-34)**
- Complete coverage of fundamental SQL concepts
- Detailed Amazon interview extensions for each question
- Comprehensive business context and real-world applications

### **🚀 Extended Collection (Questions 35-45)**
- **Question 35:** Fix Names in a Table (#1667)
- **Question 36:** Invalid Tweets (#1683) 
- **Question 37:** Find Followers Count (#1729)
- **Question 38:** The Number of Employees Which Report to Each Employee (#1731)
- **Question 39:** Find Total Time Spent by Each Employee (#1741)
- **Question 40:** Recyclable and Low Fat Products (#1757)
- **Question 41:** Product's Price for Each Store (#1777)
- **Question 42:** Primary Department for Each Employee (#1789)
- **Question 43:** Rearrange Products Table (#1795)
- **Question 44:** Employees Whose Manager Left the Company (#1978)
- **Question 45:** Number of Unique Subjects Taught by Each Teacher (#2356)

### **🎯 What You've Mastered:**
✅ **Advanced SQL Operations:** JOINs, Aggregations, Window Functions, Pivoting/Unpivoting
✅ **Amazon Business Context:** AWS, Prime, Alexa, Fulfillment, Retail, and Corporate functions
✅ **Leadership Principles:** Customer Obsession, Ownership, Think Big, Bias for Action, and more
✅ **Enterprise Analytics:** Real-world business intelligence and data engineering scenarios
✅ **Performance Optimization:** Index strategies, query optimization, and scalability considerations

### **💼 You're Now Ready For:**
- **Amazon Data Engineer Interviews** with comprehensive technical and behavioral preparation
- **Advanced SQL Challenges** with deep understanding of complex business scenarios
- **Real-World Data Projects** with enterprise-scale analytics and optimization experience
- **Leadership Discussions** grounded in Amazon's Leadership Principles and business context

**Best of luck with your Amazon Data Engineer interviews! You're exceptionally well-prepared! 🌟**