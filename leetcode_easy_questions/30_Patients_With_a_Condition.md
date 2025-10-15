# LeetCode Easy #1527: Patients With a Condition

## 📋 Problem Statement

Write a SQL query to report the **patient_id**, **patient_name** and **conditions** of the patients who have Type I Diabetes.

Type I Diabetes always starts with **DIAB1** prefix.

Return the result table in **any order**.

## 🗄️ Table Schema

**Patients Table:**
```
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| patient_id   | int     |
| patient_name | varchar |
| conditions   | varchar |
+--------------+---------+
```
- patient_id is the primary key for this table.
- conditions contains 0 or more code separated by spaces.
- This table contains information of the patients in the hospital.

## 📊 Sample Data

**Patients Table:**
| patient_id | patient_name | conditions          |
|------------|--------------|---------------------|
| 1          | Daniel       | YFEV COUGH          |
| 2          | Alice        |                     |
| 3          | Bob          | DIAB1 MYOP          |
| 4          | George       | ACNE DIAB1          |
| 5          | Alain        | DIAB1 ACNE MYOP     |
| 6          | Kate         | COUGH DIAB1 FLU     |
| 7          | Tracy        | ASMA DIAB11 DIP     |
| 8          | Sarah        | DIAB100 MYOP        |
| 9          | John         | DIAB1               |

**Expected Output:**
| patient_id | patient_name | conditions          |
|------------|--------------|---------------------|
| 3          | Bob          | DIAB1 MYOP          |
| 4          | George       | ACNE DIAB1          |
| 5          | Alain        | DIAB1 ACNE MYOP     |
| 6          | Kate         | COUGH DIAB1 FLU     |
| 9          | John         | DIAB1               |

**Explanation:**
- **Patient 1 (Daniel)**: ❌ Has "YFEV COUGH" - no DIAB1 condition
- **Patient 2 (Alice)**: ❌ Empty conditions - no conditions listed
- **Patient 3 (Bob)**: ✅ Has "DIAB1 MYOP" - starts with DIAB1
- **Patient 4 (George)**: ✅ Has "ACNE DIAB1" - contains DIAB1 as separate word
- **Patient 5 (Alain)**: ✅ Has "DIAB1 ACNE MYOP" - starts with DIAB1
- **Patient 6 (Kate)**: ✅ Has "COUGH DIAB1 FLU" - contains DIAB1 as separate word
- **Patient 7 (Tracy)**: ❌ Has "ASMA DIAB11 DIP" - DIAB11 is not DIAB1
- **Patient 8 (Sarah)**: ❌ Has "DIAB100 MYOP" - DIAB100 is not DIAB1
- **Patient 9 (John)**: ✅ Has "DIAB1" - exactly matches DIAB1

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to find patients with Type I Diabetes (DIAB1 prefix)
- DIAB1 must be a complete word, not part of another code
- Can appear at beginning, middle, or end of conditions string
- Must handle empty conditions gracefully

### 2. **Key Insights**
- DIAB1 must be surrounded by word boundaries
- Could be: "DIAB1", " DIAB1", "DIAB1 ", " DIAB1 "
- Cannot be: "DIAB11", "DIAB100", "SOMEDIAB1"
- Need pattern matching with word boundaries

### 3. **Interview Discussion Points**
- "This requires pattern matching with word boundaries"
- "DIAB1 could appear anywhere in the conditions string"
- "Need to ensure it's a complete word, not a substring"

## 🔧 Step-by-Step Solution Logic

### Step 1: Understand Word Boundaries
```sql
-- DIAB1 as complete word means:
-- - At start: "DIAB1 ..."
-- - In middle: "... DIAB1 ..."
-- - At end: "... DIAB1"
-- - Alone: "DIAB1"
```

### Step 2: Pattern Matching Options
```sql
-- Option 1: Multiple LIKE conditions
-- Option 2: REGEXP with word boundaries
-- Option 3: String manipulation functions
```

### Step 3: Handle Edge Cases
```sql
-- Empty conditions
-- Multiple occurrences of DIAB1
-- Similar codes like DIAB11, DIAB100
```

### Step 4: Apply Solution
```sql
-- Use appropriate pattern matching
-- Return patient information
```

## ✅ Optimized SQL Solution

**Solution 1: REGEXP with Word Boundaries (MySQL)**
```sql
SELECT 
    patient_id,
    patient_name,
    conditions
FROM Patients
WHERE conditions REGEXP '\\bDIAB1\\b';
```

### Alternative Solutions

**Solution 2: Multiple LIKE Conditions (More Compatible)**
```sql
SELECT 
    patient_id,
    patient_name,
    conditions
FROM Patients
WHERE conditions LIKE 'DIAB1 %'      -- Starts with DIAB1
   OR conditions LIKE '% DIAB1'       -- Ends with DIAB1
   OR conditions LIKE '% DIAB1 %'     -- Contains DIAB1 in middle
   OR conditions = 'DIAB1';           -- Exactly DIAB1
```

**Solution 3: Using REGEXP with Alternative Patterns**
```sql
SELECT 
    patient_id,
    patient_name,
    conditions
FROM Patients
WHERE conditions REGEXP '^DIAB1( |$)|( DIAB1 )|( DIAB1$)';
```

**Solution 4: PostgreSQL Compatible**
```sql
SELECT 
    patient_id,
    patient_name,
    conditions
FROM Patients
WHERE conditions ~ '\\yDIAB1\\y';
```

**Solution 5: Comprehensive Solution with Pattern Analysis**
```sql
SELECT 
    patient_id,
    patient_name,
    conditions,
    CASE 
        WHEN conditions LIKE 'DIAB1 %' THEN 'DIAB1 at start'
        WHEN conditions LIKE '% DIAB1' THEN 'DIAB1 at end'
        WHEN conditions LIKE '% DIAB1 %' THEN 'DIAB1 in middle'
        WHEN conditions = 'DIAB1' THEN 'DIAB1 only'
        ELSE 'No DIAB1 pattern'
    END as diab1_position
FROM Patients
WHERE conditions LIKE 'DIAB1 %'
   OR conditions LIKE '% DIAB1'
   OR conditions LIKE '% DIAB1 %'
   OR conditions = 'DIAB1'
ORDER BY patient_id;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Advanced Medical Condition Analysis and Patient Care System**
```sql
-- "Design a comprehensive medical condition analysis system for patient care"

WITH condition_parsing AS (
    SELECT 
        patient_id,
        patient_name,
        conditions,
        
        -- DIAB1 analysis
        CASE 
            WHEN conditions REGEXP '\\bDIAB1\\b' THEN 1
            ELSE 0
        END as has_type1_diabetes,
        
        -- Parse individual conditions
        CASE 
            WHEN conditions IS NULL OR conditions = '' THEN 0
            ELSE (LENGTH(conditions) - LENGTH(REPLACE(conditions, ' ', '')) + 1)
        END as condition_count,
        
        -- Common condition checks
        CASE WHEN conditions REGEXP '\\bHYPT\\b' THEN 1 ELSE 0 END as has_hypertension,
        CASE WHEN conditions REGEXP '\\bDIAB2\\b' THEN 1 ELSE 0 END as has_type2_diabetes,
        CASE WHEN conditions REGEXP '\\bCOUGH\\b' THEN 1 ELSE 0 END as has_cough,
        CASE WHEN conditions REGEXP '\\bFEVER\\b|\\bYFEV\\b' THEN 1 ELSE 0 END as has_fever,
        CASE WHEN conditions REGEXP '\\bACNE\\b' THEN 1 ELSE 0 END as has_acne,
        CASE WHEN conditions REGEXP '\\bMYOP\\b' THEN 1 ELSE 0 END as has_myopia,
        CASE WHEN conditions REGEXP '\\bASMA\\b|\\bASHMA\\b' THEN 1 ELSE 0 END as has_asthma,
        CASE WHEN conditions REGEXP '\\bFLU\\b' THEN 1 ELSE 0 END as has_flu,
        
        -- Diabetes classification
        CASE 
            WHEN conditions REGEXP '\\bDIAB1\\b' AND conditions REGEXP '\\bDIAB2\\b' 
            THEN 'Both Type 1 and Type 2'
            WHEN conditions REGEXP '\\bDIAB1\\b' 
            THEN 'Type 1 Diabetes'
            WHEN conditions REGEXP '\\bDIAB2\\b' 
            THEN 'Type 2 Diabetes'
            WHEN conditions REGEXP '\\bDIAB\\b' 
            THEN 'Unspecified Diabetes'
            ELSE 'No Diabetes'
        END as diabetes_classification,
        
        -- Condition severity assessment
        CASE 
            WHEN condition_count >= 4 THEN 'High Complexity'
            WHEN condition_count = 3 THEN 'Medium Complexity'
            WHEN condition_count = 2 THEN 'Low Complexity'
            WHEN condition_count = 1 THEN 'Single Condition'
            ELSE 'No Conditions'
        END as complexity_level,
        
        -- Comorbidity analysis for diabetes patients
        CASE 
            WHEN conditions REGEXP '\\bDIAB1\\b' AND 
                 (conditions REGEXP '\\bHYPT\\b' OR conditions REGEXP '\\bMYOP\\b')
            THEN 'Type 1 Diabetes with Comorbidities'
            WHEN conditions REGEXP '\\bDIAB1\\b'
            THEN 'Type 1 Diabetes Only'
            ELSE 'No Type 1 Diabetes'
        END as diab1_comorbidity_status
    FROM Patients
),
medical_risk_assessment AS (
    SELECT 
        cp.*,
        
        -- Risk scoring for Type 1 Diabetes patients
        CASE 
            WHEN has_type1_diabetes = 1 THEN
                10 + -- Base Type 1 Diabetes risk
                (has_hypertension * 15) + -- Cardiovascular risk
                (has_type2_diabetes * 20) + -- Dual diabetes complexity
                (has_cough * 5) + -- Respiratory concerns
                (has_fever * 8) + -- Infection risk
                (has_asthma * 12) + -- Respiratory comorbidity
                (CASE WHEN condition_count >= 4 THEN 10 ELSE 0 END) -- Multiple condition complexity
            ELSE 0
        END as diabetes_risk_score,
        
        -- Care priority classification
        CASE 
            WHEN has_type1_diabetes = 1 AND complexity_level = 'High Complexity' 
            THEN 'Critical Care Priority'
            WHEN has_type1_diabetes = 1 AND (has_hypertension = 1 OR has_type2_diabetes = 1)
            THEN 'High Care Priority'
            WHEN has_type1_diabetes = 1 AND complexity_level = 'Medium Complexity'
            THEN 'Medium Care Priority'
            WHEN has_type1_diabetes = 1 
            THEN 'Standard Care Priority'
            ELSE 'General Population'
        END as care_priority,
        
        -- Treatment recommendations
        CASE 
            WHEN has_type1_diabetes = 1 AND has_type2_diabetes = 1 
            THEN 'Endocrinologist consultation required - Complex diabetes management'
            WHEN has_type1_diabetes = 1 AND has_hypertension = 1 
            THEN 'Diabetes + Cardiology care coordination needed'
            WHEN has_type1_diabetes = 1 AND has_asthma = 1 
            THEN 'Diabetes + Pulmonology care coordination needed'
            WHEN has_type1_diabetes = 1 AND condition_count >= 3 
            THEN 'Multidisciplinary care team assignment'
            WHEN has_type1_diabetes = 1 
            THEN 'Standard diabetes care protocol'
            ELSE 'General medical care'
        END as treatment_recommendation,
        
        -- Monitoring frequency
        CASE 
            WHEN diabetes_risk_score >= 50 THEN 'Weekly monitoring'
            WHEN diabetes_risk_score >= 30 THEN 'Bi-weekly monitoring'
            WHEN diabetes_risk_score >= 15 THEN 'Monthly monitoring'
            WHEN has_type1_diabetes = 1 THEN 'Quarterly monitoring'
            ELSE 'Annual checkup'
        END as monitoring_frequency
    FROM condition_parsing cp
),
care_coordination_analysis AS (
    SELECT 
        mra.*,
        
        -- Specialist referral needs
        CASE 
            WHEN has_type1_diabetes = 1 THEN
                'Endocrinologist' ||
                CASE WHEN has_hypertension = 1 THEN ', Cardiologist' ELSE '' END ||
                CASE WHEN has_asthma = 1 THEN ', Pulmonologist' ELSE '' END ||
                CASE WHEN has_myopia = 1 THEN ', Ophthalmologist' ELSE '' END ||
                CASE WHEN condition_count >= 4 THEN ', Care Coordinator' ELSE '' END
            ELSE 'Primary Care'
        END as specialist_referrals,
        
        -- Medication management complexity
        CASE 
            WHEN has_type1_diabetes = 1 AND has_type2_diabetes = 1 
            THEN 'Complex Insulin + Oral Medications'
            WHEN has_type1_diabetes = 1 AND has_hypertension = 1 
            THEN 'Insulin + Blood Pressure Medications'
            WHEN has_type1_diabetes = 1 
            THEN 'Insulin Management'
            ELSE 'Standard Medication Management'
        END as medication_complexity,
        
        -- Care cost estimation
        CASE 
            WHEN care_priority = 'Critical Care Priority' THEN 'High Cost ($8000-12000/year)'
            WHEN care_priority = 'High Care Priority' THEN 'Medium-High Cost ($5000-8000/year)'
            WHEN care_priority = 'Medium Care Priority' THEN 'Medium Cost ($3000-5000/year)'
            WHEN care_priority = 'Standard Care Priority' THEN 'Standard Cost ($2000-3000/year)'
            ELSE 'Low Cost (<$2000/year)'
        END as estimated_annual_cost,
        
        -- Insurance prior authorization needs
        CASE 
            WHEN has_type1_diabetes = 1 AND condition_count >= 3 
            THEN 'Multiple prior authorizations likely needed'
            WHEN has_type1_diabetes = 1 
            THEN 'Insulin prior authorization required'
            ELSE 'Standard coverage'
        END as insurance_considerations
    FROM medical_risk_assessment mra
),
patient_outcomes_prediction AS (
    SELECT 
        cca.*,
        
        -- Health outcome predictions
        CASE 
            WHEN diabetes_risk_score >= 60 THEN 'High Risk - Frequent Hospitalizations Likely'
            WHEN diabetes_risk_score >= 40 THEN 'Medium Risk - Occasional Emergency Care'
            WHEN diabetes_risk_score >= 20 THEN 'Low Risk - Stable with Regular Care'
            WHEN has_type1_diabetes = 1 THEN 'Very Low Risk - Well Managed Diabetes'
            ELSE 'General Population Risk'
        END as health_outcome_prediction,
        
        -- Quality of life assessment
        CASE 
            WHEN complexity_level = 'High Complexity' AND has_type1_diabetes = 1 
            THEN 'Significantly Impacted - Comprehensive Support Needed'
            WHEN has_type1_diabetes = 1 AND condition_count >= 2 
            THEN 'Moderately Impacted - Support Services Recommended'
            WHEN has_type1_diabetes = 1 
            THEN 'Mildly Impacted - Education and Resources Helpful'
            ELSE 'Standard Quality of Life'
        END as quality_of_life_assessment,
        
        -- Patient education priorities
        CASE 
            WHEN has_type1_diabetes = 1 THEN
                'Insulin Management, Blood Sugar Monitoring' ||
                CASE WHEN has_hypertension = 1 THEN ', Blood Pressure Control' ELSE '' END ||
                CASE WHEN has_asthma = 1 THEN ', Respiratory Care' ELSE '' END ||
                CASE WHEN condition_count >= 3 THEN ', Medication Coordination' ELSE '' END
            ELSE 'General Health Education'
        END as education_priorities
    FROM care_coordination_analysis cca
)
SELECT 
    patient_id,
    patient_name,
    conditions,
    diabetes_classification,
    complexity_level,
    care_priority,
    diabetes_risk_score,
    treatment_recommendation,
    monitoring_frequency,
    specialist_referrals,
    medication_complexity,
    health_outcome_prediction,
    quality_of_life_assessment,
    education_priorities,
    estimated_annual_cost,
    insurance_considerations
FROM patient_outcomes_prediction
WHERE has_type1_diabetes = 1
ORDER BY 
    diabetes_risk_score DESC,
    CASE care_priority 
        WHEN 'Critical Care Priority' THEN 1
        WHEN 'High Care Priority' THEN 2
        WHEN 'Medium Care Priority' THEN 3
        WHEN 'Standard Care Priority' THEN 4
        ELSE 5
    END,
    patient_id;
```

#### 2. **Medical Coding and Healthcare Analytics System**
```sql
-- "Build a comprehensive medical coding analysis and healthcare analytics system"

WITH medical_code_analysis AS (
    SELECT 
        patient_id,
        patient_name,
        conditions,
        
        -- Parse medical codes (assuming space-separated)
        CASE 
            WHEN conditions IS NULL OR conditions = '' THEN ''
            ELSE conditions
        END as clean_conditions,
        
        -- Count total medical codes
        CASE 
            WHEN conditions IS NULL OR conditions = '' THEN 0
            ELSE (LENGTH(TRIM(conditions)) - LENGTH(REPLACE(TRIM(conditions), ' ', '')) + 1)
        END as total_codes,
        
        -- Primary condition (first code)
        CASE 
            WHEN conditions IS NULL OR conditions = '' THEN ''
            WHEN LOCATE(' ', conditions) > 0 THEN SUBSTRING(conditions, 1, LOCATE(' ', conditions) - 1)
            ELSE conditions
        END as primary_condition,
        
        -- Type 1 Diabetes analysis
        CASE WHEN conditions REGEXP '\\bDIAB1\\b' THEN 1 ELSE 0 END as has_diab1,
        
        -- Medical code categories
        CASE WHEN conditions REGEXP '\\bDIAB[0-9]*\\b' THEN 1 ELSE 0 END as has_diabetes_codes,
        CASE WHEN conditions REGEXP '\\bHYPT\\b|\\bBP\\b' THEN 1 ELSE 0 END as has_cardiovascular_codes,
        CASE WHEN conditions REGEXP '\\bCOUGH\\b|\\bFEVER\\b|\\bYFEV\\b|\\bFLU\\b|\\bASMA\\b' THEN 1 ELSE 0 END as has_respiratory_codes,
        CASE WHEN conditions REGEXP '\\bMYOP\\b|\\bVISION\\b' THEN 1 ELSE 0 END as has_vision_codes,
        CASE WHEN conditions REGEXP '\\bACNE\\b|\\bDERM\\b|\\bSKIN\\b' THEN 1 ELSE 0 END as has_dermatology_codes,
        CASE WHEN conditions REGEXP '\\bDIP\\b|\\bNEURO\\b' THEN 1 ELSE 0 END as has_neurological_codes,
        
        -- Code complexity analysis
        CASE 
            WHEN conditions REGEXP 'DIAB1[0-9]+' THEN 'Complex Diabetes Coding'
            WHEN conditions REGEXP '\\bDIAB1\\b' THEN 'Standard Diabetes Coding'
            ELSE 'Non-Diabetes Coding'
        END as diabetes_coding_complexity,
        
        -- Extract all unique medical codes
        conditions as all_conditions_raw
    FROM Patients
),
healthcare_system_analysis AS (
    SELECT 
        mca.*,
        
        -- Healthcare utilization prediction
        CASE 
            WHEN total_codes >= 4 THEN 'High Utilization - Multiple Specialists'
            WHEN total_codes = 3 THEN 'Medium Utilization - 2-3 Specialists'
            WHEN total_codes = 2 THEN 'Low Utilization - 1-2 Specialists'
            WHEN total_codes = 1 THEN 'Minimal Utilization - Primary Care'
            ELSE 'No Current Healthcare Needs'
        END as utilization_prediction,
        
        -- Care pathway assignment
        CASE 
            WHEN has_diab1 = 1 AND has_cardiovascular_codes = 1 
            THEN 'Diabetes-Cardiovascular Care Pathway'
            WHEN has_diab1 = 1 AND has_respiratory_codes = 1 
            THEN 'Diabetes-Respiratory Care Pathway'
            WHEN has_diab1 = 1 AND total_codes >= 3 
            THEN 'Complex Diabetes Care Pathway'
            WHEN has_diab1 = 1 
            THEN 'Standard Diabetes Care Pathway'
            WHEN has_cardiovascular_codes = 1 AND has_respiratory_codes = 1 
            THEN 'Cardio-Pulmonary Care Pathway'
            ELSE 'General Medical Care Pathway'
        END as care_pathway,
        
        -- Resource allocation scoring
        (has_diabetes_codes * 25) +
        (has_cardiovascular_codes * 20) +
        (has_respiratory_codes * 15) +
        (has_vision_codes * 10) +
        (has_dermatology_codes * 5) +
        (has_neurological_codes * 30) +
        (CASE WHEN total_codes >= 4 THEN 20 ELSE 0 END) as resource_allocation_score,
        
        -- Population health classification
        CASE 
            WHEN has_diab1 = 1 THEN 'Chronic Disease Management Population'
            WHEN total_codes >= 3 THEN 'High-Need Population'
            WHEN total_codes >= 2 THEN 'Moderate-Need Population'
            WHEN total_codes = 1 THEN 'Low-Need Population'
            ELSE 'Healthy Population'
        END as population_health_segment,
        
        -- Quality metrics target
        CASE 
            WHEN has_diab1 = 1 THEN 'HbA1c <7%, Blood Pressure <130/80, LDL <100'
            WHEN has_cardiovascular_codes = 1 THEN 'Blood Pressure <130/80, LDL <100'
            WHEN has_respiratory_codes = 1 THEN 'Peak Flow >80% predicted, Symptom Control'
            ELSE 'Annual Wellness Visit, Preventive Screenings'
        END as quality_metrics_targets
    FROM medical_code_analysis mca
),
clinical_decision_support AS (
    SELECT 
        hsa.*,
        
        -- Clinical alerts
        CASE 
            WHEN has_diab1 = 1 AND has_cardiovascular_codes = 1 
            THEN 'ALERT: High cardiovascular risk - Consider ACE inhibitor, statin therapy'
            WHEN has_diab1 = 1 AND has_respiratory_codes = 1 
            THEN 'ALERT: Respiratory complications - Monitor for diabetic pulmonary edema'
            WHEN has_diab1 = 1 AND has_vision_codes = 1 
            THEN 'ALERT: Diabetic retinopathy risk - Urgent ophthalmology referral'
            WHEN has_diab1 = 1 AND total_codes >= 4 
            THEN 'ALERT: Complex diabetes - Endocrinology consultation recommended'
            WHEN has_diab1 = 1 
            THEN 'REMINDER: Diabetes monitoring - A1C every 3 months'
            ELSE 'No specific clinical alerts'
        END as clinical_alerts,
        
        -- Medication interaction warnings
        CASE 
            WHEN has_diab1 = 1 AND has_cardiovascular_codes = 1 
            THEN 'Check: Insulin-ACE inhibitor interactions, Beta-blocker masking hypoglycemia'
            WHEN has_diab1 = 1 AND has_respiratory_codes = 1 
            THEN 'Check: Steroid effects on blood glucose, Beta-agonist interactions'
            WHEN has_diab1 = 1 
            THEN 'Check: All medications for glucose effects'
            ELSE 'Standard medication screening'
        END as medication_interaction_warnings,
        
        -- Preventive care recommendations
        CASE 
            WHEN has_diab1 = 1 THEN
                'Annual: Eye exam, Foot exam, Kidney function, Lipid panel; ' ||
                'Every 3 months: A1C testing; ' ||
                'Consider: Flu vaccine, Pneumonia vaccine'
            WHEN has_cardiovascular_codes = 1 THEN
                'Annual: Lipid panel, EKG; Every 6 months: Blood pressure check'
            WHEN has_respiratory_codes = 1 THEN
                'Annual: Chest X-ray, Pulmonary function; Consider: Flu vaccine'
            ELSE 'Standard preventive care: Annual physical, age-appropriate screenings'
        END as preventive_care_recommendations,
        
        -- Cost-effectiveness analysis
        CASE 
            WHEN resource_allocation_score >= 100 THEN 'High Cost, High Value - Intensive Management'
            WHEN resource_allocation_score >= 60 THEN 'Medium Cost, High Value - Standard Management'
            WHEN resource_allocation_score >= 30 THEN 'Low Cost, Medium Value - Preventive Focus'
            ELSE 'Minimal Cost, Preventive Value - Wellness Focus'
        END as cost_effectiveness_category
    FROM healthcare_system_analysis hsa
),
outcome_analytics AS (
    SELECT 
        cds.*,
        
        -- 30-day readmission risk
        CASE 
            WHEN resource_allocation_score >= 100 THEN '25-35% - Very High Risk'
            WHEN resource_allocation_score >= 80 THEN '15-25% - High Risk'
            WHEN resource_allocation_score >= 60 THEN '10-15% - Medium Risk'
            WHEN resource_allocation_score >= 40 THEN '5-10% - Low Risk'
            ELSE '<5% - Very Low Risk'
        END as readmission_risk_30day,
        
        -- Annual healthcare cost prediction
        CASE 
            WHEN has_diab1 = 1 AND total_codes >= 4 THEN '$15,000 - $25,000'
            WHEN has_diab1 = 1 AND total_codes >= 2 THEN '$8,000 - $15,000'
            WHEN has_diab1 = 1 THEN '$5,000 - $8,000'
            WHEN total_codes >= 3 THEN '$6,000 - $12,000'
            WHEN total_codes >= 2 THEN '$3,000 - $6,000'
            ELSE '$1,000 - $3,000'
        END as predicted_annual_cost,
        
        -- Patient satisfaction prediction
        CASE 
            WHEN care_pathway LIKE '%Complex%' THEN 'Coordination Challenges - Focus on Communication'
            WHEN has_diab1 = 1 THEN 'Chronic Disease - Emphasize Self-Management Support'
            WHEN total_codes >= 3 THEN 'Multiple Providers - Ensure Care Coordination'
            ELSE 'Standard Satisfaction Expected'
        END as satisfaction_prediction,
        
        -- Performance dashboard metrics
        CASE 
            WHEN has_diab1 = 1 THEN 'Track: A1C, Blood Pressure, LDL, Eye Exams, Foot Exams'
            WHEN has_cardiovascular_codes = 1 THEN 'Track: Blood Pressure, LDL, Medication Adherence'
            WHEN has_respiratory_codes = 1 THEN 'Track: Symptom Control, Medication Adherence, Exacerbations'
            ELSE 'Track: Preventive Care Completion, Annual Visits'
        END as performance_metrics
    FROM clinical_decision_support cds
)
SELECT 
    patient_id,
    patient_name,
    conditions,
    primary_condition,
    total_codes,
    population_health_segment,
    care_pathway,
    utilization_prediction,
    clinical_alerts,
    preventive_care_recommendations,
    resource_allocation_score,
    readmission_risk_30day,
    predicted_annual_cost,
    quality_metrics_targets,
    performance_metrics,
    cost_effectiveness_category
FROM outcome_analytics
WHERE has_diab1 = 1
ORDER BY 
    resource_allocation_score DESC,
    total_codes DESC,
    patient_id;
```

#### 3. **Healthcare Data Governance and Quality Management**
```sql
-- "Implement comprehensive healthcare data governance and quality assurance"

WITH data_quality_assessment AS (
    SELECT 
        patient_id,
        patient_name,
        conditions,
        
        -- Data completeness checks
        CASE 
            WHEN patient_id IS NULL THEN 'Missing Patient ID'
            WHEN patient_name IS NULL OR patient_name = '' THEN 'Missing Patient Name'
            WHEN conditions IS NULL THEN 'NULL Conditions'
            WHEN conditions = '' THEN 'Empty Conditions'
            ELSE 'Complete Record'
        END as data_completeness_status,
        
        -- Medical coding validation
        CASE 
            WHEN conditions IS NOT NULL AND conditions != '' THEN
                CASE 
                    WHEN conditions REGEXP '^[A-Z0-9 ]+$' THEN 'Valid Format'
                    WHEN conditions REGEXP '[a-z]' THEN 'Contains Lowercase - Standardization Needed'
                    WHEN conditions REGEXP '[^A-Z0-9 ]' THEN 'Contains Invalid Characters'
                    ELSE 'Unknown Format Issue'
                END
            ELSE 'No Conditions to Validate'
        END as coding_format_validation,
        
        -- DIAB1 validation and standardization
        CASE 
            WHEN conditions REGEXP '\\bDIAB1\\b' THEN 'Valid DIAB1 Code'
            WHEN conditions REGEXP '\\bdiab1\\b' THEN 'DIAB1 Format Issue - Lowercase'
            WHEN conditions REGEXP '\\bDIAB 1\\b' THEN 'DIAB1 Format Issue - Space'
            WHEN conditions REGEXP '\\bDIAB01\\b' THEN 'DIAB1 Format Issue - Leading Zero'
            WHEN conditions LIKE '%DIAB1%' THEN 'DIAB1 Partial Match - Review Needed'
            ELSE 'No DIAB1 Code'
        END as diab1_validation,
        
        -- Data consistency checks
        CASE 
            WHEN conditions IS NOT NULL AND conditions != '' THEN
                CASE 
                    WHEN conditions REGEXP '  +' THEN 'Multiple Spaces - Normalization Needed'
                    WHEN conditions LIKE ' %' OR conditions LIKE '% ' THEN 'Leading/Trailing Spaces'
                    ELSE 'Consistent Formatting'
                END
            ELSE 'No Conditions'
        END as format_consistency,
        
        -- Medical code standardization
        UPPER(TRIM(REGEXP_REPLACE(conditions, ' +', ' '))) as standardized_conditions,
        
        -- Audit trail simulation
        CONCAT('Patient ', patient_id, ' - Last updated: 2024-01-', 
               LPAD((patient_id % 30) + 1, 2, '0')) as audit_trail
    FROM Patients
),
healthcare_compliance_analysis AS (
    SELECT 
        dqa.*,
        
        -- HIPAA compliance assessment
        CASE 
            WHEN data_completeness_status = 'Complete Record' 
                 AND coding_format_validation = 'Valid Format'
            THEN 'HIPAA Compliant - Complete Protected Health Information'
            WHEN data_completeness_status IN ('Missing Patient Name', 'Missing Patient ID')
            THEN 'HIPAA Risk - Incomplete Patient Identification'
            WHEN coding_format_validation != 'Valid Format'
            THEN 'HIPAA Risk - Invalid Medical Coding Format'
            ELSE 'HIPAA Compliant with Data Quality Issues'
        END as hipaa_compliance_status,
        
        -- Clinical data governance scoring
        (CASE WHEN data_completeness_status = 'Complete Record' THEN 25 ELSE 0 END) +
        (CASE WHEN coding_format_validation = 'Valid Format' THEN 25 ELSE 0 END) +
        (CASE WHEN diab1_validation = 'Valid DIAB1 Code' THEN 20 ELSE 
              CASE WHEN diab1_validation != 'No DIAB1 Code' THEN 10 ELSE 0 END END) +
        (CASE WHEN format_consistency = 'Consistent Formatting' THEN 15 ELSE 0 END) +
        (CASE WHEN standardized_conditions IS NOT NULL THEN 15 ELSE 0 END) as data_governance_score,
        
        -- Data quality category
        CASE 
            WHEN (CASE WHEN data_completeness_status = 'Complete Record' THEN 25 ELSE 0 END) +
                 (CASE WHEN coding_format_validation = 'Valid Format' THEN 25 ELSE 0 END) +
                 (CASE WHEN diab1_validation = 'Valid DIAB1 Code' THEN 20 ELSE 
                       CASE WHEN diab1_validation != 'No DIAB1 Code' THEN 10 ELSE 0 END END) +
                 (CASE WHEN format_consistency = 'Consistent Formatting' THEN 15 ELSE 0 END) +
                 (CASE WHEN standardized_conditions IS NOT NULL THEN 15 ELSE 0 END) >= 85 
            THEN 'Excellent Data Quality'
            WHEN (CASE WHEN data_completeness_status = 'Complete Record' THEN 25 ELSE 0 END) +
                 (CASE WHEN coding_format_validation = 'Valid Format' THEN 25 ELSE 0 END) +
                 (CASE WHEN diab1_validation = 'Valid DIAB1 Code' THEN 20 ELSE 
                       CASE WHEN diab1_validation != 'No DIAB1 Code' THEN 10 ELSE 0 END END) +
                 (CASE WHEN format_consistency = 'Consistent Formatting' THEN 15 ELSE 0 END) +
                 (CASE WHEN standardized_conditions IS NOT NULL THEN 15 ELSE 0 END) >= 70 
            THEN 'Good Data Quality'
            WHEN (CASE WHEN data_completeness_status = 'Complete Record' THEN 25 ELSE 0 END) +
                 (CASE WHEN coding_format_validation = 'Valid Format' THEN 25 ELSE 0 END) +
                 (CASE WHEN diab1_validation = 'Valid DIAB1 Code' THEN 20 ELSE 
                       CASE WHEN diab1_validation != 'No DIAB1 Code' THEN 10 ELSE 0 END END) +
                 (CASE WHEN format_consistency = 'Consistent Formatting' THEN 15 ELSE 0 END) +
                 (CASE WHEN standardized_conditions IS NOT NULL THEN 15 ELSE 0 END) >= 50 
            THEN 'Acceptable Data Quality'
            ELSE 'Poor Data Quality - Immediate Attention Required'
        END as data_quality_category,
        
        -- Remediation recommendations
        CASE 
            WHEN data_completeness_status != 'Complete Record' 
            THEN 'Priority 1: Complete missing patient information'
            WHEN coding_format_validation != 'Valid Format' 
            THEN 'Priority 2: Standardize medical coding format'
            WHEN diab1_validation LIKE '%Format Issue%' 
            THEN 'Priority 3: Correct DIAB1 coding format'
            WHEN format_consistency != 'Consistent Formatting' 
            THEN 'Priority 4: Normalize condition formatting'
            ELSE 'No immediate remediation needed'
        END as remediation_priority
    FROM data_quality_assessment dqa
),
regulatory_reporting_analysis AS (
    SELECT 
        hca.*,
        
        -- CMS quality reporting readiness
        CASE 
            WHEN diab1_validation = 'Valid DIAB1 Code' AND data_quality_category IN ('Excellent Data Quality', 'Good Data Quality')
            THEN 'Ready for CMS Diabetes Quality Reporting'
            WHEN diab1_validation = 'Valid DIAB1 Code' 
            THEN 'Diabetes Code Valid - Improve Data Quality for Reporting'
            WHEN diab1_validation != 'No DIAB1 Code' 
            THEN 'Diabetes Code Issues - Fix Before Reporting'
            ELSE 'Not Applicable for Diabetes Quality Reporting'
        END as cms_reporting_readiness,
        
        -- Joint Commission compliance
        CASE 
            WHEN hipaa_compliance_status LIKE 'HIPAA Compliant%' AND data_governance_score >= 80
            THEN 'Joint Commission Ready - Meets Data Standards'
            WHEN hipaa_compliance_status LIKE 'HIPAA Compliant%'
            THEN 'Joint Commission Partial - Data Quality Improvements Needed'
            ELSE 'Joint Commission Risk - Address Compliance Issues'
        END as joint_commission_status,
        
        -- Medicare Advantage risk adjustment
        CASE 
            WHEN diab1_validation = 'Valid DIAB1 Code' AND data_quality_category = 'Excellent Data Quality'
            THEN 'Optimal RAF Score Capture - DIAB1 Documented'
            WHEN diab1_validation = 'Valid DIAB1 Code'
            THEN 'RAF Score Capture Possible - Improve Documentation'
            WHEN diab1_validation LIKE '%Format Issue%'
            THEN 'RAF Score Risk - Fix DIAB1 Format for Capture'
            ELSE 'No RAF Score Impact'
        END as risk_adjustment_impact,
        
        -- Quality measure eligibility
        CASE 
            WHEN diab1_validation = 'Valid DIAB1 Code' THEN
                'Eligible for: HbA1c Testing, Eye Exams, Nephropathy Screening, Blood Pressure Control'
            ELSE 'Not Eligible for Diabetes Quality Measures'
        END as quality_measure_eligibility
    FROM healthcare_compliance_analysis hca
),
organizational_metrics AS (
    SELECT 
        COUNT(*) as total_patients,
        SUM(CASE WHEN conditions REGEXP '\\bDIAB1\\b' THEN 1 ELSE 0 END) as diab1_patients,
        SUM(CASE WHEN data_quality_category = 'Excellent Data Quality' THEN 1 ELSE 0 END) as excellent_quality_records,
        SUM(CASE WHEN hipaa_compliance_status LIKE 'HIPAA Compliant%' THEN 1 ELSE 0 END) as hipaa_compliant_records,
        SUM(CASE WHEN cms_reporting_readiness LIKE 'Ready for%' THEN 1 ELSE 0 END) as cms_ready_records,
        
        ROUND(AVG(data_governance_score), 2) as avg_data_governance_score,
        ROUND(SUM(CASE WHEN conditions REGEXP '\\bDIAB1\\b' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) as diab1_prevalence_rate,
        ROUND(SUM(CASE WHEN data_quality_category = 'Excellent Data Quality' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) as excellent_quality_rate
    FROM regulatory_reporting_analysis
)
SELECT 
    rra.patient_id,
    rra.patient_name,
    rra.conditions,
    rra.standardized_conditions,
    rra.data_quality_category,
    rra.data_governance_score,
    rra.hipaa_compliance_status,
    rra.cms_reporting_readiness,
    rra.joint_commission_status,
    rra.risk_adjustment_impact,
    rra.quality_measure_eligibility,
    rra.remediation_priority,
    rra.audit_trail,
    
    -- Organizational context
    om.diab1_prevalence_rate as org_diab1_rate,
    om.excellent_quality_rate as org_quality_rate,
    CASE 
        WHEN rra.data_governance_score > om.avg_data_governance_score 
        THEN 'Above Average Data Quality'
        ELSE 'Below Average Data Quality'
    END as relative_data_quality
FROM regulatory_reporting_analysis rra
CROSS JOIN organizational_metrics om
WHERE rra.diab1_validation = 'Valid DIAB1 Code'
ORDER BY 
    rra.data_governance_score DESC,
    CASE rra.data_quality_category 
        WHEN 'Excellent Data Quality' THEN 1
        WHEN 'Good Data Quality' THEN 2
        WHEN 'Acceptable Data Quality' THEN 3
        ELSE 4
    END,
    rra.patient_id;
```

## 🔗 Related LeetCode Questions

1. **#1517 - Find Users With Valid E-Mails** (REGEXP pattern matching)
2. **#1667 - Fix Names in a Table** (String manipulation and pattern matching)
3. **#1484 - Group Sold Products By The Date** (String aggregation with LIKE patterns)
4. **#1873 - Calculate Special Bonus** (Conditional string pattern logic)
5. **#1965 - Employees With Missing Information** (Data validation with pattern matching)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Word Boundaries**: Ensuring exact word matches, not substrings
2. **Multiple Pattern Matching**: Different ways to achieve the same result
3. **Database Compatibility**: Different regex syntax across databases
4. **Edge Case Handling**: Empty strings, NULL values, single words

### 🚀 **Amazon Interview Tips**
1. **Explain word boundaries**: "DIAB1 must be a complete word, not part of DIAB11"
2. **Discuss alternatives**: "Can use REGEXP or multiple LIKE conditions"
3. **Consider performance**: "LIKE might be faster than REGEXP for simple patterns"
4. **Address compatibility**: "Different databases have different regex syntax"

### 🔧 **Common Patterns**
- Word boundary matching with `\\b` in MySQL
- Alternative LIKE patterns for exact word matching
- Handling leading/trailing spaces in conditions
- Case sensitivity considerations

### ⚠️ **Common Mistakes to Avoid**
1. **Substring matching** (finding DIAB1 within DIAB11)
2. **Missing edge cases** (single word conditions)
3. **Database incompatibility** (using MySQL syntax in PostgreSQL)
4. **Case sensitivity issues** (not considering lowercase variants)

### 🔍 **Performance Considerations**
- LIKE patterns with multiple OR conditions can be optimized
- REGEXP is powerful but potentially slower than simple LIKE
- Consider indexing strategies for condition codes
- Pattern matching on text fields can be expensive

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Accurate medical coding ensures proper patient care
- **Operational Excellence**: Systematic pattern matching for reliable diagnosis identification
- **Data-Driven Decisions**: Medical condition analysis supports clinical decision-making
- **Trust**: Precise medical coding builds confidence in healthcare systems

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find patients with multiple diabetes types (DIAB1 and DIAB2)**
2. **Extract all condition codes as separate rows**
3. **Find patients with condition codes starting with specific prefixes**
4. **Implement fuzzy matching for potentially misspelled conditions**

Remember: Medical condition analysis is critical for Amazon's healthcare initiatives, clinical decision support systems, population health management, and regulatory compliance across diverse healthcare platforms and patient care systems!