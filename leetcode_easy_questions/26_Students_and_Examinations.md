# LeetCode Easy #1280: Students and Examinations

## 📋 Problem Statement

Write a SQL query to find the number of times each student attended each exam.

Return the result table **ordered by** `student_id` **and** `subject_name`.

## 🗄️ Table Schema

**Students Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
```
- student_id is the primary key for this table.
- Each row contains the ID and the name of one student in the school.

**Subjects Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| subject_name  | varchar |
+---------------+---------+
```
- subject_name is the primary key for this table.
- Each row contains the name of one subject in the school.

**Examinations Table:**
```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student_id   | int     |
| subject_name | varchar |
+--------------+---------+
```
- There is no primary key for this table. It may contain duplicates.
- Each row indicates that a student with student_id attended the exam of subject_name.

## 📊 Sample Data

**Students Table:**
| student_id | student_name |
|------------|--------------|
| 1          | Alice        |
| 2          | Bob          |
| 13         | John         |
| 6          | Alex         |

**Subjects Table:**
| subject_name |
|--------------|
| Math         |
| Physics      |
| Programming  |

**Examinations Table:**
| student_id | subject_name |
|------------|--------------|
| 1          | Math         |
| 1          | Physics      |
| 1          | Programming  |
| 2          | Programming  |
| 1          | Physics      |
| 1          | Math         |
| 13         | Math         |
| 13         | Programming  |
| 13         | Physics      |
| 2          | Math         |
| 1          | Math         |

**Expected Output:**
| student_id | student_name | subject_name | attended_exams |
|------------|--------------|--------------|----------------|
| 1          | Alice        | Math         | 3              |
| 1          | Alice        | Physics      | 2              |
| 1          | Alice        | Programming  | 1              |
| 2          | Bob          | Math         | 1              |
| 2          | Bob          | Physics      | 0              |
| 2          | Bob          | Programming  | 1              |
| 6          | Alex         | Math         | 0              |
| 6          | Alex         | Physics      | 0              |
| 6          | Alex         | Programming  | 0              |
| 13         | John         | Math         | 1              |
| 13         | John         | Physics      | 1              |
| 13         | John         | Programming  | 1              |

**Explanation:**
- Alice (student_id=1) attended Math 3 times, Physics 2 times, Programming 1 time
- Bob (student_id=2) attended Math 1 time, Programming 1 time, Physics 0 times
- Alex (student_id=6) didn't attend any exams (0 for all subjects)
- John (student_id=13) attended each subject 1 time

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need all combinations of students and subjects (Cartesian product)
- Count attendance for each student-subject combination
- Include students who didn't attend any exams (zeros)
- Sort by student_id and subject_name

### 2. **Key Insights**
- CROSS JOIN to get all student-subject combinations
- LEFT JOIN to preserve combinations with zero attendance
- COUNT with proper grouping
- Handle NULL values from LEFT JOIN

### 3. **Interview Discussion Points**
- "This requires a complete matrix of students × subjects"
- "Need CROSS JOIN to generate all possible combinations"
- "LEFT JOIN ensures we capture zero attendance"

## 🔧 Step-by-Step Solution Logic

### Step 1: Create Student-Subject Matrix
```sql
-- CROSS JOIN Students and Subjects
-- This creates all possible student-subject combinations
```

### Step 2: Left Join with Examinations
```sql
-- LEFT JOIN with Examinations table
-- Preserves all combinations, even with no exam attendance
```

### Step 3: Count Attendances
```sql
-- COUNT(e.student_id) counts actual exam records
-- NULL values from LEFT JOIN are not counted
```

### Step 4: Sort Results
```sql
-- ORDER BY student_id, subject_name
-- Ensure consistent result ordering
```

## ✅ Optimized SQL Solution

**Solution 1: CROSS JOIN with LEFT JOIN**
```sql
SELECT 
    s.student_id,
    s.student_name,
    sub.subject_name,
    COUNT(e.student_id) as attended_exams
FROM Students s
CROSS JOIN Subjects sub
LEFT JOIN Examinations e ON s.student_id = e.student_id 
    AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

### Alternative Solutions

**Solution 2: Using CTE for Clarity**
```sql
WITH student_subject_matrix AS (
    SELECT 
        s.student_id,
        s.student_name,
        sub.subject_name
    FROM Students s
    CROSS JOIN Subjects sub
)
SELECT 
    ssm.student_id,
    ssm.student_name,
    ssm.subject_name,
    COUNT(e.student_id) as attended_exams
FROM student_subject_matrix ssm
LEFT JOIN Examinations e ON ssm.student_id = e.student_id 
    AND ssm.subject_name = e.subject_name
GROUP BY ssm.student_id, ssm.student_name, ssm.subject_name
ORDER BY ssm.student_id, ssm.subject_name;
```

**Solution 3: With Detailed Analytics**
```sql
SELECT 
    s.student_id,
    s.student_name,
    sub.subject_name,
    COUNT(e.student_id) as attended_exams,
    CASE 
        WHEN COUNT(e.student_id) = 0 THEN 'No Attendance'
        WHEN COUNT(e.student_id) = 1 THEN 'Low Attendance'
        WHEN COUNT(e.student_id) <= 3 THEN 'Moderate Attendance'
        ELSE 'High Attendance'
    END as attendance_category,
    ROUND(COUNT(e.student_id) * 100.0 / (
        SELECT COUNT(*) FROM Examinations e2 WHERE e2.subject_name = sub.subject_name
    ), 2) as subject_attendance_percentage
FROM Students s
CROSS JOIN Subjects sub
LEFT JOIN Examinations e ON s.student_id = e.student_id 
    AND sub.subject_name = e.subject_name
GROUP BY s.student_id, s.student_name, sub.subject_name
ORDER BY s.student_id, sub.subject_name;
```

**Solution 4: Including Overall Statistics**
```sql
WITH base_results AS (
    SELECT 
        s.student_id,
        s.student_name,
        sub.subject_name,
        COUNT(e.student_id) as attended_exams
    FROM Students s
    CROSS JOIN Subjects sub
    LEFT JOIN Examinations e ON s.student_id = e.student_id 
        AND sub.subject_name = e.subject_name
    GROUP BY s.student_id, s.student_name, sub.subject_name
),
student_totals AS (
    SELECT 
        student_id,
        SUM(attended_exams) as total_exams_attended,
        AVG(attended_exams) as avg_exams_per_subject,
        COUNT(CASE WHEN attended_exams > 0 THEN 1 END) as subjects_with_attendance
    FROM base_results
    GROUP BY student_id
),
subject_totals AS (
    SELECT 
        subject_name,
        SUM(attended_exams) as total_subject_attendance,
        AVG(attended_exams) as avg_attendance_per_student,
        COUNT(CASE WHEN attended_exams > 0 THEN 1 END) as students_with_attendance
    FROM base_results
    GROUP BY subject_name
)
SELECT 
    br.student_id,
    br.student_name,
    br.subject_name,
    br.attended_exams,
    st.total_exams_attended,
    ROUND(st.avg_exams_per_subject, 2) as student_avg_per_subject,
    sbt.total_subject_attendance,
    ROUND(sbt.avg_attendance_per_student, 2) as subject_avg_per_student
FROM base_results br
JOIN student_totals st ON br.student_id = st.student_id
JOIN subject_totals sbt ON br.subject_name = sbt.subject_name
ORDER BY br.student_id, br.subject_name;
```

**Solution 5: Performance Optimized Version**
```sql
-- For large datasets, pre-aggregate examinations
WITH exam_counts AS (
    SELECT 
        student_id,
        subject_name,
        COUNT(*) as attended_exams
    FROM Examinations
    GROUP BY student_id, subject_name
)
SELECT 
    s.student_id,
    s.student_name,
    sub.subject_name,
    COALESCE(ec.attended_exams, 0) as attended_exams
FROM Students s
CROSS JOIN Subjects sub
LEFT JOIN exam_counts ec ON s.student_id = ec.student_id 
    AND sub.subject_name = ec.subject_name
ORDER BY s.student_id, sub.subject_name;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Academic Performance Analysis**
```sql
-- "Analyze student performance patterns and engagement across subjects"

WITH detailed_attendance AS (
    SELECT 
        s.student_id,
        s.student_name,
        sub.subject_name,
        COUNT(e.student_id) as attended_exams,
        
        -- Time-based analysis (assuming exam_date exists)
        -- COUNT(CASE WHEN e.exam_date >= CURRENT_DATE - INTERVAL 30 DAY THEN 1 END) as recent_exams,
        -- MIN(e.exam_date) as first_exam_date,
        -- MAX(e.exam_date) as last_exam_date,
        
        CASE 
            WHEN COUNT(e.student_id) = 0 THEN 0
            ELSE 1
        END as subject_engagement_flag
    FROM Students s
    CROSS JOIN Subjects sub
    LEFT JOIN Examinations e ON s.student_id = e.student_id 
        AND sub.subject_name = e.subject_name
    GROUP BY s.student_id, s.student_name, sub.subject_name
),
student_performance_metrics AS (
    SELECT 
        student_id,
        student_name,
        SUM(attended_exams) as total_exams,
        COUNT(subject_name) as total_subjects,
        SUM(subject_engagement_flag) as engaged_subjects,
        AVG(attended_exams) as avg_exams_per_subject,
        STDDEV(attended_exams) as exam_consistency,
        MAX(attended_exams) as max_subject_exams,
        MIN(attended_exams) as min_subject_exams,
        
        CASE 
            WHEN SUM(subject_engagement_flag) = COUNT(subject_name) THEN 'Fully Engaged'
            WHEN SUM(subject_engagement_flag) >= COUNT(subject_name) * 0.7 THEN 'Highly Engaged'
            WHEN SUM(subject_engagement_flag) >= COUNT(subject_name) * 0.3 THEN 'Moderately Engaged'
            ELSE 'Low Engagement'
        END as engagement_level,
        
        CASE 
            WHEN STDDEV(attended_exams) <= 1 THEN 'Consistent Performer'
            WHEN STDDEV(attended_exams) <= 2 THEN 'Moderate Variation'
            ELSE 'Inconsistent Performer'
        END as consistency_profile
    FROM detailed_attendance
    GROUP BY student_id, student_name
),
subject_popularity_metrics AS (
    SELECT 
        subject_name,
        SUM(attended_exams) as total_attendance,
        COUNT(student_id) as total_students,
        SUM(CASE WHEN attended_exams > 0 THEN 1 ELSE 0 END) as active_students,
        AVG(attended_exams) as avg_attendance_per_student,
        STDDEV(attended_exams) as attendance_variation,
        
        ROUND(SUM(CASE WHEN attended_exams > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(student_id), 2) as participation_rate,
        
        CASE 
            WHEN AVG(attended_exams) >= 2.5 THEN 'High Interest Subject'
            WHEN AVG(attended_exams) >= 1.5 THEN 'Moderate Interest Subject'
            WHEN AVG(attended_exams) >= 0.5 THEN 'Low Interest Subject'
            ELSE 'Very Low Interest Subject'
        END as subject_popularity
    FROM detailed_attendance
    GROUP BY subject_name
),
personalized_recommendations AS (
    SELECT 
        da.student_id,
        da.student_name,
        da.subject_name,
        da.attended_exams,
        spm.engagement_level,
        spm.consistency_profile,
        spm.avg_exams_per_subject as student_avg,
        sp.avg_attendance_per_student as subject_avg,
        sp.participation_rate,
        
        CASE 
            WHEN da.attended_exams = 0 AND sp.participation_rate > 50 
            THEN 'Encourage participation in popular subject'
            WHEN da.attended_exams > 0 AND da.attended_exams < spm.avg_exams_per_subject * 0.5 
            THEN 'Focus more attention on this subject'
            WHEN da.attended_exams > sp.avg_attendance_per_student * 1.5 
            THEN 'Strong performer - consider advanced topics'
            WHEN da.attended_exams = 0 AND spm.engagement_level = 'Low Engagement' 
            THEN 'Requires academic intervention'
            ELSE 'Continue current approach'
        END as recommendation,
        
        CASE 
            WHEN da.attended_exams >= sp.avg_attendance_per_student * 1.2 THEN 'Above Average'
            WHEN da.attended_exams >= sp.avg_attendance_per_student * 0.8 THEN 'Average'
            ELSE 'Below Average'
        END as relative_performance
    FROM detailed_attendance da
    JOIN student_performance_metrics spm ON da.student_id = spm.student_id
    JOIN subject_popularity_metrics sp ON da.subject_name = sp.subject_name
)
SELECT 
    student_id,
    student_name,
    subject_name,
    attended_exams,
    engagement_level,
    consistency_profile,
    ROUND(student_avg, 2) as student_avg_per_subject,
    ROUND(subject_avg, 2) as subject_avg_per_student,
    participation_rate,
    relative_performance,
    recommendation
FROM personalized_recommendations
ORDER BY student_id, 
    CASE relative_performance 
        WHEN 'Below Average' THEN 1 
        WHEN 'Average' THEN 2 
        WHEN 'Above Average' THEN 3 
    END,
    subject_name;
```

#### 2. **Exam Scheduling and Capacity Analysis**
```sql
-- "Optimize exam scheduling based on student attendance patterns"

WITH exam_frequency_analysis AS (
    SELECT 
        subject_name,
        student_id,
        COUNT(*) as total_attempts,
        -- Assuming we have exam_date column for time analysis
        -- COUNT(DISTINCT DATE(exam_date)) as unique_exam_days,
        -- DATEDIFF(MAX(exam_date), MIN(exam_date)) + 1 as engagement_span_days,
        
        ROW_NUMBER() OVER (PARTITION BY subject_name ORDER BY COUNT(*) DESC) as attendance_rank_in_subject,
        ROW_NUMBER() OVER (PARTITION BY student_id ORDER BY COUNT(*) DESC) as subject_preference_rank
    FROM Examinations
    GROUP BY subject_name, student_id
),
subject_scheduling_metrics AS (
    SELECT 
        subject_name,
        COUNT(DISTINCT student_id) as unique_attendees,
        SUM(total_attempts) as total_exam_sessions,
        AVG(total_attempts) as avg_attempts_per_student,
        STDDEV(total_attempts) as attempt_variation,
        MAX(total_attempts) as max_attempts_by_student,
        MIN(total_attempts) as min_attempts_by_student,
        
        -- Capacity planning metrics
        ROUND(SUM(total_attempts) / COUNT(DISTINCT student_id), 2) as sessions_per_active_student,
        
        CASE 
            WHEN STDDEV(total_attempts) > AVG(total_attempts) * 0.5 THEN 'High Variation'
            WHEN STDDEV(total_attempts) > AVG(total_attempts) * 0.3 THEN 'Moderate Variation'
            ELSE 'Low Variation'
        END as demand_consistency,
        
        CASE 
            WHEN AVG(total_attempts) >= 3 THEN 'High Demand'
            WHEN AVG(total_attempts) >= 2 THEN 'Moderate Demand'
            WHEN AVG(total_attempts) >= 1 THEN 'Low Demand'
            ELSE 'Very Low Demand'
        END as demand_level
    FROM exam_frequency_analysis
    GROUP BY subject_name
),
student_scheduling_patterns AS (
    SELECT 
        student_id,
        COUNT(DISTINCT subject_name) as subjects_attempted,
        SUM(total_attempts) as total_exam_attempts,
        AVG(total_attempts) as avg_attempts_per_subject,
        MAX(total_attempts) as max_subject_attempts,
        
        -- Preference analysis
        COUNT(CASE WHEN subject_preference_rank = 1 THEN 1 END) as preferred_subjects,
        
        CASE 
            WHEN COUNT(DISTINCT subject_name) = (SELECT COUNT(*) FROM Subjects) THEN 'All Subjects'
            WHEN COUNT(DISTINCT subject_name) >= (SELECT COUNT(*) FROM Subjects) * 0.7 THEN 'Most Subjects'
            WHEN COUNT(DISTINCT subject_name) >= (SELECT COUNT(*) FROM Subjects) * 0.3 THEN 'Some Subjects'
            ELSE 'Few Subjects'
        END as subject_breadth,
        
        CASE 
            WHEN AVG(total_attempts) >= 3 THEN 'Intensive Student'
            WHEN AVG(total_attempts) >= 2 THEN 'Regular Student'
            WHEN AVG(total_attempts) >= 1 THEN 'Casual Student'
            ELSE 'Minimal Participation'
        END as participation_intensity
    FROM exam_frequency_analysis
    GROUP BY student_id
),
optimal_scheduling_analysis AS (
    SELECT 
        s.subject_name,
        s.unique_attendees,
        s.total_exam_sessions,
        s.avg_attempts_per_student,
        s.demand_level,
        s.demand_consistency,
        
        -- Calculate optimal exam frequency
        CASE 
            WHEN s.demand_level = 'High Demand' THEN 
                GREATEST(s.total_exam_sessions / 4, s.unique_attendees * 2)  -- Weekly sessions
            WHEN s.demand_level = 'Moderate Demand' THEN 
                GREATEST(s.total_exam_sessions / 8, s.unique_attendees)      -- Bi-weekly sessions
            ELSE 
                s.total_exam_sessions / 12                                   -- Monthly sessions
        END as recommended_weekly_sessions,
        
        -- Resource allocation
        CEIL(s.unique_attendees / 30.0) as recommended_exam_rooms,  -- Assuming 30 students per room
        
        CASE 
            WHEN s.demand_consistency = 'High Variation' THEN 'Flexible scheduling needed'
            WHEN s.unique_attendees > 50 AND s.demand_level = 'High Demand' THEN 'Multiple session times'
            WHEN s.unique_attendees < 10 THEN 'Consider combining with other subjects'
            ELSE 'Standard scheduling'
        END as scheduling_recommendation,
        
        -- Prioritization for resource allocation
        ROW_NUMBER() OVER (ORDER BY s.total_exam_sessions DESC, s.unique_attendees DESC) as resource_priority
    FROM subject_scheduling_metrics s
),
cross_subject_conflicts AS (
    SELECT 
        e1.student_id,
        e1.subject_name as subject1,
        e2.subject_name as subject2,
        COUNT(*) as concurrent_exam_attempts
        -- Add exam_date analysis for actual scheduling conflicts
        -- COUNT(CASE WHEN DATE(e1.exam_date) = DATE(e2.exam_date) THEN 1 END) as same_day_conflicts
    FROM Examinations e1
    JOIN Examinations e2 ON e1.student_id = e2.student_id 
        AND e1.subject_name < e2.subject_name  -- Avoid duplicate pairs
    GROUP BY e1.student_id, e1.subject_name, e2.subject_name
    HAVING COUNT(*) >= 2  -- Students taking both subjects frequently
)
SELECT 
    osa.subject_name,
    osa.unique_attendees,
    osa.total_exam_sessions,
    ROUND(osa.avg_attempts_per_student, 2) as avg_attempts_per_student,
    osa.demand_level,
    osa.demand_consistency,
    ROUND(osa.recommended_weekly_sessions, 0) as recommended_weekly_sessions,
    osa.recommended_exam_rooms,
    osa.scheduling_recommendation,
    osa.resource_priority,
    
    -- Popular subject combinations (for avoiding conflicts)
    COALESCE(csc.popular_combinations, 'No major conflicts') as common_subject_combinations
FROM optimal_scheduling_analysis osa
LEFT JOIN (
    SELECT 
        subject1 as subject_name,
        GROUP_CONCAT(CONCAT(subject2, ' (', concurrent_exam_attempts, ' students)') 
                    ORDER BY concurrent_exam_attempts DESC 
                    SEPARATOR '; ') as popular_combinations
    FROM cross_subject_conflicts
    WHERE concurrent_exam_attempts >= 3
    GROUP BY subject1
    
    UNION ALL
    
    SELECT 
        subject2 as subject_name,
        GROUP_CONCAT(CONCAT(subject1, ' (', concurrent_exam_attempts, ' students)') 
                    ORDER BY concurrent_exam_attempts DESC 
                    SEPARATOR '; ') as popular_combinations
    FROM cross_subject_conflicts
    WHERE concurrent_exam_attempts >= 3
    GROUP BY subject2
) csc ON osa.subject_name = csc.subject_name
ORDER BY osa.resource_priority;
```

#### 3. **Student Success Prediction and Risk Assessment**
```sql
-- "Identify students at risk and predict academic outcomes"

WITH student_engagement_patterns AS (
    SELECT 
        s.student_id,
        s.student_name,
        COUNT(DISTINCT sub.subject_name) as total_available_subjects,
        COUNT(DISTINCT e.subject_name) as subjects_attempted,
        SUM(CASE WHEN e.subject_name IS NOT NULL THEN 1 ELSE 0 END) as total_exam_attempts,
        
        -- Engagement metrics
        ROUND(COUNT(DISTINCT e.subject_name) * 100.0 / COUNT(DISTINCT sub.subject_name), 2) as subject_coverage_percentage,
        
        AVG(exam_counts.subject_exam_count) as avg_exams_per_attempted_subject,
        STDDEV(exam_counts.subject_exam_count) as exam_consistency,
        MAX(exam_counts.subject_exam_count) as max_subject_attempts,
        MIN(exam_counts.subject_exam_count) as min_subject_attempts,
        
        -- Time-based patterns (assuming exam timestamps)
        -- DATEDIFF(MAX(e.exam_date), MIN(e.exam_date)) + 1 as engagement_span_days,
        -- COUNT(DISTINCT DATE(e.exam_date)) as unique_active_days,
        
        CASE 
            WHEN COUNT(DISTINCT e.subject_name) = 0 THEN 'No Engagement'
            WHEN COUNT(DISTINCT e.subject_name) = COUNT(DISTINCT sub.subject_name) THEN 'Full Engagement'
            WHEN COUNT(DISTINCT e.subject_name) >= COUNT(DISTINCT sub.subject_name) * 0.7 THEN 'High Engagement'
            WHEN COUNT(DISTINCT e.subject_name) >= COUNT(DISTINCT sub.subject_name) * 0.3 THEN 'Moderate Engagement'
            ELSE 'Low Engagement'
        END as engagement_level
    FROM Students s
    CROSS JOIN Subjects sub
    LEFT JOIN Examinations e ON s.student_id = e.student_id
    LEFT JOIN (
        SELECT student_id, subject_name, COUNT(*) as subject_exam_count
        FROM Examinations
        GROUP BY student_id, subject_name
    ) exam_counts ON s.student_id = exam_counts.student_id AND sub.subject_name = exam_counts.subject_name
    GROUP BY s.student_id, s.student_name
),
risk_assessment AS (
    SELECT 
        *,
        -- Risk scoring (0-100, higher = more risk)
        CASE 
            WHEN engagement_level = 'No Engagement' THEN 90
            WHEN engagement_level = 'Low Engagement' THEN 70
            WHEN engagement_level = 'Moderate Engagement' THEN 40
            WHEN engagement_level = 'High Engagement' THEN 20
            ELSE 10
        END +
        CASE 
            WHEN avg_exams_per_attempted_subject < 1 THEN 20
            WHEN avg_exams_per_attempted_subject < 2 THEN 10
            ELSE 0
        END +
        CASE 
            WHEN exam_consistency > 2 THEN 15  -- High inconsistency
            WHEN exam_consistency > 1 THEN 10
            ELSE 0
        END as risk_score,
        
        -- Success prediction factors
        CASE 
            WHEN subject_coverage_percentage >= 80 AND avg_exams_per_attempted_subject >= 2 THEN 'High Success Probability'
            WHEN subject_coverage_percentage >= 60 AND avg_exams_per_attempted_subject >= 1.5 THEN 'Moderate Success Probability'
            WHEN subject_coverage_percentage >= 30 AND avg_exams_per_attempted_subject >= 1 THEN 'Low Success Probability'
            ELSE 'At Risk'
        END as success_prediction,
        
        -- Intervention recommendations
        CASE 
            WHEN subjects_attempted = 0 THEN 'Immediate academic intervention required'
            WHEN subject_coverage_percentage < 30 THEN 'Academic counseling recommended'
            WHEN avg_exams_per_attempted_subject < 1 THEN 'Study skills support needed'
            WHEN exam_consistency > 2 THEN 'Consistency coaching required'
            ELSE 'Continue monitoring'
        END as intervention_recommendation
    FROM student_engagement_patterns
),
peer_comparison AS (
    SELECT 
        ra.*,
        AVG(ra.total_exam_attempts) OVER () as class_avg_attempts,
        AVG(ra.subject_coverage_percentage) OVER () as class_avg_coverage,
        
        PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY ra.total_exam_attempts) OVER () as q1_attempts,
        PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY ra.total_exam_attempts) OVER () as q3_attempts,
        
        CASE 
            WHEN ra.total_exam_attempts >= PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY ra.total_exam_attempts) OVER () 
            THEN 'Top Quartile'
            WHEN ra.total_exam_attempts >= PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY ra.total_exam_attempts) OVER () 
            THEN 'Above Median'
            WHEN ra.total_exam_attempts >= PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY ra.total_exam_attempts) OVER () 
            THEN 'Below Median'
            ELSE 'Bottom Quartile'
        END as peer_ranking,
        
        ROW_NUMBER() OVER (ORDER BY ra.risk_score DESC) as risk_priority_rank
    FROM risk_assessment ra
),
actionable_insights AS (
    SELECT 
        pc.*,
        CASE 
            WHEN pc.risk_score >= 80 THEN 'Critical - Immediate Action Required'
            WHEN pc.risk_score >= 60 THEN 'High Risk - Intervention Needed'
            WHEN pc.risk_score >= 40 THEN 'Moderate Risk - Monitor Closely'
            WHEN pc.risk_score >= 20 THEN 'Low Risk - Standard Support'
            ELSE 'Minimal Risk - Self-Directed'
        END as risk_category,
        
        CASE 
            WHEN pc.peer_ranking = 'Bottom Quartile' AND pc.engagement_level = 'No Engagement' 
            THEN 'Schedule immediate meeting with academic advisor'
            WHEN pc.peer_ranking = 'Bottom Quartile' 
            THEN 'Enroll in academic support program'
            WHEN pc.engagement_level = 'Low Engagement' AND pc.avg_exams_per_attempted_subject < 1 
            THEN 'Assign peer mentor'
            WHEN pc.exam_consistency > 2 
            THEN 'Provide time management and study planning resources'
            WHEN pc.subject_coverage_percentage < 50 
            THEN 'Career counseling to align subject selection with goals'
            ELSE 'Continue with standard academic monitoring'
        END as specific_action_plan,
        
        -- Timeline for intervention
        CASE 
            WHEN pc.risk_score >= 80 THEN 'Within 1 week'
            WHEN pc.risk_score >= 60 THEN 'Within 2 weeks'
            WHEN pc.risk_score >= 40 THEN 'Within 1 month'
            ELSE 'Next semester review'
        END as action_timeline
    FROM peer_comparison pc
)
SELECT 
    student_id,
    student_name,
    subjects_attempted,
    total_exam_attempts,
    ROUND(subject_coverage_percentage, 1) as subject_coverage_percentage,
    ROUND(avg_exams_per_attempted_subject, 2) as avg_exams_per_subject,
    engagement_level,
    success_prediction,
    risk_score,
    risk_category,
    peer_ranking,
    risk_priority_rank,
    intervention_recommendation,
    specific_action_plan,
    action_timeline
FROM actionable_insights
ORDER BY risk_score DESC, total_exam_attempts ASC;
```

#### 4. **Advanced Analytics Dashboard Metrics**
```sql
-- "Create comprehensive dashboard metrics for academic administration"

WITH comprehensive_metrics AS (
    -- All student-subject combinations with attendance
    SELECT 
        s.student_id,
        s.student_name,
        sub.subject_name,
        COUNT(e.student_id) as attended_exams,
        CASE WHEN COUNT(e.student_id) > 0 THEN 1 ELSE 0 END as is_active_in_subject
    FROM Students s
    CROSS JOIN Subjects sub
    LEFT JOIN Examinations e ON s.student_id = e.student_id 
        AND sub.subject_name = e.subject_name
    GROUP BY s.student_id, s.student_name, sub.subject_name
),
dashboard_student_metrics AS (
    SELECT 
        student_id,
        student_name,
        SUM(attended_exams) as total_exams,
        SUM(is_active_in_subject) as active_subjects,
        COUNT(*) as total_subjects_available,
        AVG(attended_exams) as avg_exams_per_subject,
        MAX(attended_exams) as max_subject_exams,
        STDDEV(attended_exams) as exam_consistency_score,
        
        -- Performance categories
        CASE 
            WHEN SUM(attended_exams) >= 10 THEN 'High Performer'
            WHEN SUM(attended_exams) >= 5 THEN 'Moderate Performer'
            WHEN SUM(attended_exams) >= 1 THEN 'Low Performer'
            ELSE 'Non Performer'
        END as performance_tier,
        
        ROUND(SUM(is_active_in_subject) * 100.0 / COUNT(*), 2) as subject_engagement_rate
    FROM comprehensive_metrics
    GROUP BY student_id, student_name
),
dashboard_subject_metrics AS (
    SELECT 
        subject_name,
        SUM(attended_exams) as total_attendance,
        SUM(is_active_in_subject) as active_students,
        COUNT(*) as total_students_enrolled,
        AVG(attended_exams) as avg_exams_per_student,
        MAX(attended_exams) as max_student_exams,
        STDDEV(attended_exams) as attendance_variation,
        
        ROUND(SUM(is_active_in_subject) * 100.0 / COUNT(*), 2) as participation_rate,
        
        CASE 
            WHEN AVG(attended_exams) >= 3 THEN 'High Demand'
            WHEN AVG(attended_exams) >= 2 THEN 'Moderate Demand' 
            WHEN AVG(attended_exams) >= 1 THEN 'Low Demand'
            ELSE 'Very Low Demand'
        END as demand_level
    FROM comprehensive_metrics
    GROUP BY subject_name
),
institutional_kpis AS (
    SELECT 
        COUNT(DISTINCT student_id) as total_students,
        COUNT(DISTINCT subject_name) as total_subjects,
        SUM(attended_exams) as total_exam_instances,
        
        -- Overall engagement metrics
        ROUND(AVG(subject_engagement_rate), 2) as avg_student_engagement_rate,
        ROUND(AVG(total_exams), 2) as avg_exams_per_student,
        
        -- Distribution analysis
        COUNT(CASE WHEN performance_tier = 'High Performer' THEN 1 END) as high_performers,
        COUNT(CASE WHEN performance_tier = 'Moderate Performer' THEN 1 END) as moderate_performers,
        COUNT(CASE WHEN performance_tier = 'Low Performer' THEN 1 END) as low_performers,
        COUNT(CASE WHEN performance_tier = 'Non Performer' THEN 1 END) as non_performers,
        
        -- Risk indicators
        COUNT(CASE WHEN total_exams = 0 THEN 1 END) as students_with_no_exams,
        COUNT(CASE WHEN active_subjects <= 1 THEN 1 END) as students_limited_engagement,
        
        -- Performance ratios
        ROUND(COUNT(CASE WHEN performance_tier IN ('High Performer', 'Moderate Performer') THEN 1 END) * 100.0 / COUNT(*), 2) as success_rate,
        ROUND(COUNT(CASE WHEN total_exams = 0 THEN 1 END) * 100.0 / COUNT(*), 2) as disengagement_rate
    FROM dashboard_student_metrics
),
trend_indicators AS (
    SELECT 
        'Overall Institution' as metric_category,
        'Total Students' as metric_name,
        CAST(total_students AS VARCHAR) as metric_value,
        'Count' as metric_unit,
        
        CASE 
            WHEN total_students >= 100 THEN 'Large Institution'
            WHEN total_students >= 50 THEN 'Medium Institution'
            ELSE 'Small Institution'
        END as performance_indicator
    FROM institutional_kpis
    
    UNION ALL
    
    SELECT 
        'Overall Institution' as metric_category,
        'Average Engagement Rate' as metric_name,
        CONCAT(CAST(avg_student_engagement_rate AS VARCHAR), '%') as metric_value,
        'Percentage' as metric_unit,
        
        CASE 
            WHEN avg_student_engagement_rate >= 80 THEN 'Excellent Engagement'
            WHEN avg_student_engagement_rate >= 60 THEN 'Good Engagement'
            WHEN avg_student_engagement_rate >= 40 THEN 'Fair Engagement'
            ELSE 'Poor Engagement'
        END as performance_indicator
    FROM institutional_kpis
    
    UNION ALL
    
    SELECT 
        'Overall Institution' as metric_category,
        'Success Rate' as metric_name,
        CONCAT(CAST(success_rate AS VARCHAR), '%') as metric_value,
        'Percentage' as metric_unit,
        
        CASE 
            WHEN success_rate >= 80 THEN 'High Success Rate'
            WHEN success_rate >= 60 THEN 'Moderate Success Rate'
            WHEN success_rate >= 40 THEN 'Low Success Rate'
            ELSE 'Critical Success Rate'
        END as performance_indicator
    FROM institutional_kpis
    
    UNION ALL
    
    SELECT 
        'Risk Management' as metric_category,
        'Disengagement Rate' as metric_name,
        CONCAT(CAST(disengagement_rate AS VARCHAR), '%') as metric_value,
        'Percentage' as metric_unit,
        
        CASE 
            WHEN disengagement_rate <= 10 THEN 'Low Risk'
            WHEN disengagement_rate <= 25 THEN 'Moderate Risk'
            WHEN disengagement_rate <= 40 THEN 'High Risk'
            ELSE 'Critical Risk'
        END as performance_indicator
    FROM institutional_kpis
),
actionable_recommendations AS (
    SELECT 
        ik.*,
        CASE 
            WHEN ik.disengagement_rate > 25 THEN 'Implement early intervention programs'
            WHEN ik.avg_student_engagement_rate < 50 THEN 'Review curriculum and teaching methods'
            WHEN ik.success_rate < 60 THEN 'Enhance academic support services'
            WHEN ik.students_with_no_exams > ik.total_students * 0.2 THEN 'Mandatory academic check-ins'
            ELSE 'Continue current practices with minor optimizations'
        END as primary_recommendation,
        
        CASE 
            WHEN ik.non_performers > ik.total_students * 0.3 THEN 'High Priority'
            WHEN ik.students_limited_engagement > ik.total_students * 0.2 THEN 'Medium Priority'
            ELSE 'Low Priority'
        END as intervention_priority
    FROM institutional_kpis ik
)
SELECT 
    ti.metric_category,
    ti.metric_name,
    ti.metric_value,
    ti.metric_unit,
    ti.performance_indicator,
    ar.primary_recommendation,
    ar.intervention_priority
FROM trend_indicators ti
CROSS JOIN actionable_recommendations ar
ORDER BY 
    CASE ti.metric_category 
        WHEN 'Overall Institution' THEN 1 
        WHEN 'Risk Management' THEN 2 
        ELSE 3 
    END,
    ti.metric_name;
```

## 🔗 Related LeetCode Questions

1. **#1068 - Product Sales Analysis I** (Basic JOIN operations)
2. **#1581 - Customer Who Visited but Did Not Make Any Transactions** (LEFT JOIN with counting)
3. **#1693 - Daily Leads and Partners** (GROUP BY with counting)
4. **#1148 - Article Views I** (DISTINCT with filtering)
5. **#1378 - Replace Employee ID With The Unique Identifier** (JOIN operations)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **CROSS JOIN**: Creating all possible combinations
2. **LEFT JOIN**: Preserving all combinations including zeros
3. **COUNT with JOIN**: Proper aggregation with NULL handling
4. **Matrix Generation**: Student × Subject complete coverage

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Should I include students who never attended any exams?"
2. **Explain CROSS JOIN**: "This creates the complete student-subject matrix"
3. **Handle zeros properly**: "LEFT JOIN ensures we capture zero attendance"
4. **Discuss performance**: "CROSS JOIN can be expensive with large tables"

### 🔧 **Common Patterns**
- CROSS JOIN for complete combinations
- LEFT JOIN to preserve all combinations
- COUNT with NULL handling from outer joins
- GROUP BY with multiple columns

### ⚠️ **Common Mistakes to Avoid**
1. **Using INNER JOIN** (missing zero attendance records)
2. **Wrong COUNT function** (using COUNT(*) instead of COUNT(column))
3. **Missing ORDER BY** (inconsistent result ordering)
4. **Performance issues** (not optimizing CROSS JOIN)

### 🔍 **Performance Considerations**
- CROSS JOIN can be expensive for large tables
- Index on (student_id, subject_name) for join optimization
- Consider materialized views for frequently accessed combinations
- Pre-aggregate when dealing with large examination datasets

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Complete academic tracking improves student outcomes
- **Data-Driven Decisions**: Comprehensive attendance analytics guide academic policies
- **Operational Excellence**: Systematic tracking of all student-subject combinations
- **Innovation**: Advanced analytics for personalized academic interventions

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Include exam scores and calculate averages**
2. **Add time-based analysis (monthly/quarterly attendance)**
3. **Find students with perfect attendance**
4. **Calculate attendance rates by demographics**

Remember: Complete matrix reporting is crucial for Amazon's educational technology platforms, learning management systems, certification tracking, and comprehensive student analytics across diverse learning programs!