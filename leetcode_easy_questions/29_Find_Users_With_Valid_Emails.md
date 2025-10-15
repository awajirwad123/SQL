# LeetCode Easy #1517: Find Users With Valid E-Mails

## 📋 Problem Statement

Write a SQL query to find the users who have **valid emails**.

A valid e-mail has a **prefix name** and a **domain** where:

- The **prefix name** is a string that may contain letters (upper or lower case), digits, underscore `'_'`, period `'.'`, and/or dash `'-'`. The prefix name **must** start with a letter.
- The **domain** is `'@leetcode.com'`.

Return the result table in **any order**.

## 🗄️ Table Schema

**Users Table:**
```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| user_id       | int     |
| name          | varchar |
| mail          | varchar |
+---------------+---------+
```
- user_id is the primary key for this table.
- This table contains information of the users signed up on a website. Some e-mails are invalid.

## 📊 Sample Data

**Users Table:**
| user_id | name      | mail                    |
|---------|-----------|-------------------------|
| 1       | Winston   | winston@leetcode.com    |
| 2       | Jonathan  | jonathanisgreat         |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |
| 5       | Marwan    | quarz#2020@leetcode.com |
| 6       | David     | david69@gmail.com       |
| 7       | Shapiro   | .shapo@leetcode.com     |

**Expected Output:**
| user_id | name      | mail                    |
|---------|-----------|-------------------------|
| 1       | Winston   | winston@leetcode.com    |
| 3       | Annabelle | bella-@leetcode.com     |
| 4       | Sally     | sally.come@leetcode.com |

**Explanation:**
- **User 1 (Winston)**: ✅ Valid - starts with letter, contains only allowed characters, correct domain
- **User 2 (Jonathan)**: ❌ Invalid - missing '@leetcode.com' domain
- **User 3 (Annabelle)**: ✅ Valid - starts with letter, dash is allowed, correct domain
- **User 4 (Sally)**: ✅ Valid - starts with letter, period is allowed, correct domain
- **User 5 (Marwan)**: ❌ Invalid - contains '#' which is not allowed
- **User 6 (David)**: ❌ Invalid - wrong domain (gmail.com instead of leetcode.com)
- **User 7 (Shapiro)**: ❌ Invalid - starts with '.' instead of a letter

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Email must end with '@leetcode.com'
- Prefix must start with a letter (a-z, A-Z)
- Prefix can contain letters, digits, underscore, period, dash
- Need to use REGEXP/REGEX pattern matching

### 2. **Key Insights**
- Regular expression for pattern matching
- Domain validation is straightforward string matching
- Prefix validation requires character class matching
- Different databases have different regex syntax

### 3. **Interview Discussion Points**
- "This requires regular expression pattern matching"
- "Need to validate both prefix format and domain exactly"
- "Different SQL databases have different regex syntax"

## 🔧 Step-by-Step Solution Logic

### Step 1: Domain Validation
```sql
-- mail LIKE '%@leetcode.com'
-- Ensures email ends with correct domain
```

### Step 2: Prefix Start Validation
```sql
-- mail REGEXP '^[a-zA-Z]'
-- Ensures email starts with a letter
```

### Step 3: Character Set Validation
```sql
-- mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\.com$'
-- Complete pattern for valid email format
```

### Step 4: Combine Conditions
```sql
-- WHERE clause with regex pattern
-- Returns only users with valid email addresses
```

## ✅ Optimized SQL Solution

**Solution 1: MySQL REGEXP**
```sql
SELECT 
    user_id,
    name,
    mail
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$';
```

### Alternative Solutions

**Solution 2: Using Multiple Conditions (More Readable)**
```sql
SELECT 
    user_id,
    name,
    mail
FROM Users
WHERE mail LIKE '%@leetcode.com'
    AND mail REGEXP '^[a-zA-Z]'
    AND mail REGEXP '^[a-zA-Z0-9._-]+@leetcode\\.com$';
```

**Solution 3: PostgreSQL Compatible**
```sql
SELECT 
    user_id,
    name,
    mail
FROM Users
WHERE mail ~ '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\.com$';
```

**Solution 4: SQL Server Compatible**
```sql
SELECT 
    user_id,
    name,
    mail
FROM Users
WHERE mail LIKE '%@leetcode.com'
    AND LEFT(mail, 1) LIKE '[a-zA-Z]'
    AND mail NOT LIKE '%[^a-zA-Z0-9._-@]%'
    AND CHARINDEX('@', mail) > 1;
```

**Solution 5: With Detailed Validation Breakdown**
```sql
SELECT 
    user_id,
    name,
    mail,
    CASE 
        WHEN mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$' THEN 'Valid Email'
        WHEN mail NOT LIKE '%@leetcode.com' THEN 'Invalid Domain'
        WHEN mail NOT REGEXP '^[a-zA-Z]' THEN 'Invalid Prefix Start'
        WHEN mail REGEXP '[^a-zA-Z0-9._-@]' THEN 'Invalid Characters'
        ELSE 'Other Validation Error'
    END as email_validation_status
FROM Users
WHERE mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$';
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Comprehensive Email Validation and User Management System**
```sql
-- "Create a complete user email validation and management system"

WITH email_validation_analysis AS (
    SELECT 
        user_id,
        name,
        mail,
        
        -- Basic validation checks
        CASE 
            WHEN mail IS NULL OR mail = '' THEN 0
            ELSE 1
        END as has_email,
        
        CASE 
            WHEN mail LIKE '%@leetcode.com' THEN 1
            ELSE 0
        END as correct_domain,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z]' THEN 1
            ELSE 0
        END as starts_with_letter,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z0-9._-]+@' THEN 1
            ELSE 0
        END as valid_prefix_characters,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$' THEN 1
            ELSE 0
        END as fully_valid,
        
        -- Email analysis metrics
        LENGTH(mail) as email_length,
        LOCATE('@', mail) as at_position,
        LENGTH(mail) - LOCATE('@', mail) as domain_length,
        LOCATE('@', mail) - 1 as prefix_length,
        
        -- Extract components
        SUBSTRING(mail, 1, LOCATE('@', mail) - 1) as email_prefix,
        SUBSTRING(mail, LOCATE('@', mail)) as email_domain,
        
        -- Advanced validation flags
        CASE 
            WHEN mail LIKE '%.%@leetcode.com' THEN 1
            ELSE 0
        END as contains_period,
        
        CASE 
            WHEN mail LIKE '%_%@leetcode.com' THEN 1
            ELSE 0
        END as contains_underscore,
        
        CASE 
            WHEN mail LIKE '%-%@leetcode.com' THEN 1
            ELSE 0
        END as contains_dash,
        
        CASE 
            WHEN mail REGEXP '[0-9]' THEN 1
            ELSE 0
        END as contains_numbers
    FROM Users
),
validation_categorization AS (
    SELECT 
        eva.*,
        
        -- Comprehensive validation status
        CASE 
            WHEN fully_valid = 1 THEN 'Valid Email'
            WHEN has_email = 0 THEN 'Missing Email'
            WHEN correct_domain = 0 AND starts_with_letter = 0 THEN 'Invalid Domain and Prefix'
            WHEN correct_domain = 0 THEN 'Invalid Domain'
            WHEN starts_with_letter = 0 THEN 'Invalid Prefix Start'
            WHEN valid_prefix_characters = 0 THEN 'Invalid Characters in Prefix'
            ELSE 'Other Validation Issue'
        END as validation_status,
        
        -- Email complexity scoring
        (contains_period + contains_underscore + contains_dash + contains_numbers) as complexity_score,
        
        CASE 
            WHEN prefix_length <= 5 THEN 'Short Prefix'
            WHEN prefix_length <= 15 THEN 'Medium Prefix'
            WHEN prefix_length <= 25 THEN 'Long Prefix'
            ELSE 'Very Long Prefix'
        END as prefix_length_category,
        
        -- Security risk assessment
        CASE 
            WHEN fully_valid = 0 THEN 'High Risk - Invalid Email'
            WHEN prefix_length < 3 THEN 'Medium Risk - Very Short Prefix'
            WHEN complexity_score = 0 THEN 'Low Risk - Simple but Valid'
            WHEN complexity_score >= 3 THEN 'Very Low Risk - Complex Valid Email'
            ELSE 'Low Risk - Standard Valid Email'
        END as security_risk_level
    FROM email_validation_analysis eva
),
user_management_insights AS (
    SELECT 
        vc.*,
        
        -- User engagement potential based on email quality
        CASE 
            WHEN validation_status = 'Valid Email' AND complexity_score >= 2 
            THEN 'High Engagement Potential'
            WHEN validation_status = 'Valid Email' 
            THEN 'Standard Engagement Potential'
            WHEN validation_status LIKE '%Invalid Domain%' 
            THEN 'Re-engagement Required'
            WHEN validation_status = 'Missing Email' 
            THEN 'Email Collection Required'
            ELSE 'Account Verification Required'
        END as engagement_strategy,
        
        -- Communication recommendations
        CASE 
            WHEN validation_status = 'Valid Email' 
            THEN 'Enable all email communications'
            WHEN validation_status LIKE '%Invalid Domain%' 
            THEN 'Request email update before communications'
            WHEN validation_status = 'Missing Email' 
            THEN 'Collect email for account verification'
            ELSE 'Manual review required before email communication'
        END as communication_recommendation,
        
        -- Account status implications
        CASE 
            WHEN validation_status = 'Valid Email' 
            THEN 'Active Account'
            WHEN validation_status IN ('Missing Email', 'Invalid Domain') 
            THEN 'Verification Required'
            ELSE 'Suspended Pending Email Fix'
        END as recommended_account_status
    FROM validation_categorization vc
),
organizational_metrics AS (
    SELECT 
        COUNT(*) as total_users,
        SUM(fully_valid) as users_with_valid_emails,
        SUM(CASE WHEN validation_status = 'Missing Email' THEN 1 ELSE 0 END) as users_missing_emails,
        SUM(CASE WHEN validation_status LIKE '%Invalid Domain%' THEN 1 ELSE 0 END) as users_invalid_domain,
        SUM(CASE WHEN validation_status LIKE '%Invalid%' THEN 1 ELSE 0 END) as users_invalid_emails,
        
        ROUND(SUM(fully_valid) * 100.0 / COUNT(*), 2) as email_validity_rate,
        ROUND(AVG(complexity_score), 2) as avg_email_complexity,
        ROUND(AVG(prefix_length), 2) as avg_prefix_length,
        
        COUNT(CASE WHEN security_risk_level = 'High Risk - Invalid Email' THEN 1 END) as high_risk_users,
        COUNT(CASE WHEN engagement_strategy = 'High Engagement Potential' THEN 1 END) as high_engagement_users
    FROM user_management_insights
)
SELECT 
    umi.user_id,
    umi.name,
    umi.mail,
    umi.validation_status,
    umi.email_prefix,
    umi.email_domain,
    umi.prefix_length_category,
    umi.complexity_score,
    umi.security_risk_level,
    umi.engagement_strategy,
    umi.communication_recommendation,
    umi.recommended_account_status,
    
    -- Organizational context
    om.email_validity_rate as org_validity_rate,
    CASE 
        WHEN umi.fully_valid = 1 AND om.email_validity_rate < 50 
        THEN 'Above org average'
        WHEN umi.fully_valid = 0 AND om.email_validity_rate > 80 
        THEN 'Below org standard'
        ELSE 'Standard for organization'
    END as relative_email_quality
FROM user_management_insights umi
CROSS JOIN organizational_metrics om
ORDER BY 
    CASE umi.validation_status 
        WHEN 'Valid Email' THEN 1 
        WHEN 'Invalid Domain' THEN 2
        WHEN 'Invalid Prefix Start' THEN 3
        ELSE 4 
    END,
    umi.complexity_score DESC,
    umi.user_id;
```

#### 2. **Email Security and Fraud Detection System**
```sql
-- "Implement comprehensive email security analysis and fraud detection"

WITH advanced_email_analysis AS (
    SELECT 
        user_id,
        name,
        mail,
        
        -- Basic validation
        CASE 
            WHEN mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$' THEN 1
            ELSE 0
        END as is_valid_email,
        
        -- Security pattern analysis
        CASE 
            WHEN mail REGEXP '\\.\\.+' THEN 1  -- Consecutive periods
            ELSE 0
        END as has_consecutive_periods,
        
        CASE 
            WHEN mail REGEXP '__+' THEN 1  -- Consecutive underscores
            ELSE 0
        END as has_consecutive_underscores,
        
        CASE 
            WHEN mail REGEXP '--+' THEN 1  -- Consecutive dashes
            ELSE 0
        END as has_consecutive_dashes,
        
        CASE 
            WHEN mail REGEXP '[0-9]{4,}' THEN 1  -- 4+ consecutive numbers
            ELSE 0
        END as has_long_number_sequence,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z]{1,2}[0-9]+@' THEN 1  -- Very short prefix with numbers
            ELSE 0
        END as suspicious_short_pattern,
        
        -- Character frequency analysis
        (LENGTH(mail) - LENGTH(REPLACE(mail, '.', ''))) as period_count,
        (LENGTH(mail) - LENGTH(REPLACE(mail, '_', ''))) as underscore_count,
        (LENGTH(mail) - LENGTH(REPLACE(mail, '-', ''))) as dash_count,
        (LENGTH(mail) - LENGTH(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(REPLACE(mail, '0', ''), '1', ''), '2', ''), '3', ''), '4', ''), '5', ''), '6', ''), '7', ''), '8', ''), '9', ''))) as number_count,
        
        -- Email structure analysis
        SUBSTRING(mail, 1, LOCATE('@', mail) - 1) as prefix,
        LENGTH(SUBSTRING(mail, 1, LOCATE('@', mail) - 1)) as prefix_length,
        
        -- Pattern recognition for common attack vectors
        CASE 
            WHEN mail LIKE '%admin%' OR mail LIKE '%test%' OR mail LIKE '%demo%' THEN 1
            ELSE 0
        END as contains_system_words,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z]+[0-9]+$' THEN 1  -- Simple name+number pattern
            ELSE 0
        END as simple_enumeration_pattern,
        
        -- Registration timestamp simulation for temporal analysis
        CASE 
            WHEN user_id % 10 = 0 THEN '2024-01-15'
            WHEN user_id % 10 = 1 THEN '2024-01-15'
            WHEN user_id % 10 = 2 THEN '2024-01-16'
            ELSE '2024-01-17'
        END as simulated_registration_date
    FROM Users
),
fraud_risk_scoring AS (
    SELECT 
        aea.*,
        
        -- Risk scoring (0-100, higher = more suspicious)
        (has_consecutive_periods * 15) +
        (has_consecutive_underscores * 10) +
        (has_consecutive_dashes * 10) +
        (has_long_number_sequence * 20) +
        (suspicious_short_pattern * 25) +
        (contains_system_words * 15) +
        (simple_enumeration_pattern * 10) +
        (CASE WHEN period_count > 3 THEN 10 ELSE 0 END) +
        (CASE WHEN underscore_count > 2 THEN 8 ELSE 0 END) +
        (CASE WHEN number_count > 5 THEN 12 ELSE 0 END) +
        (CASE WHEN prefix_length < 3 THEN 15 ELSE 0 END) +
        (CASE WHEN prefix_length > 20 THEN 8 ELSE 0 END) as fraud_risk_score,
        
        -- Legitimacy indicators
        CASE 
            WHEN prefix_length BETWEEN 4 AND 15 
                AND period_count <= 2 
                AND underscore_count <= 1 
                AND number_count <= 3 
            THEN 'High Legitimacy'
            WHEN prefix_length BETWEEN 3 AND 20 
                AND period_count <= 3 
                AND number_count <= 5 
            THEN 'Medium Legitimacy'
            ELSE 'Low Legitimacy'
        END as legitimacy_assessment,
        
        -- Automated action recommendations
        CASE 
            WHEN is_valid_email = 0 THEN 'Reject - Invalid Format'
            WHEN has_consecutive_periods = 1 OR has_consecutive_underscores = 1 OR has_consecutive_dashes = 1 
            THEN 'Manual Review - Suspicious Patterns'
            WHEN suspicious_short_pattern = 1 AND contains_system_words = 1 
            THEN 'Block - High Fraud Risk'
            WHEN simple_enumeration_pattern = 1 AND has_long_number_sequence = 1 
            THEN 'Flag - Potential Bot Registration'
            ELSE 'Approve - Standard Processing'
        END as automated_action
    FROM advanced_email_analysis aea
),
temporal_fraud_detection AS (
    SELECT 
        frs.*,
        
        -- Simulate rapid registration detection
        COUNT(*) OVER (
            PARTITION BY simulated_registration_date 
            ORDER BY user_id 
            ROWS BETWEEN 4 PRECEDING AND CURRENT ROW
        ) as registrations_in_window,
        
        -- Similar email pattern detection
        COUNT(*) OVER (
            PARTITION BY 
                SUBSTRING(prefix, 1, 3),  -- First 3 characters of prefix
                simulated_registration_date
        ) as similar_prefix_same_day,
        
        -- Enumeration pattern detection
        CASE 
            WHEN simple_enumeration_pattern = 1 AND 
                 EXISTS (
                     SELECT 1 FROM Users u2 
                     WHERE u2.user_id != frs.user_id 
                     AND u2.mail LIKE CONCAT(REGEXP_REPLACE(frs.prefix, '[0-9]+$', ''), '%@leetcode.com')
                 )
            THEN 1
            ELSE 0
        END as part_of_enumeration_attack
    FROM fraud_risk_scoring frs
),
comprehensive_security_assessment AS (
    SELECT 
        tfd.*,
        
        -- Enhanced risk scoring with temporal factors
        fraud_risk_score +
        (CASE WHEN registrations_in_window >= 5 THEN 20 ELSE 0 END) +
        (CASE WHEN similar_prefix_same_day >= 3 THEN 15 ELSE 0 END) +
        (CASE WHEN part_of_enumeration_attack = 1 THEN 25 ELSE 0 END) as total_risk_score,
        
        -- Final security classification
        CASE 
            WHEN is_valid_email = 0 THEN 'Invalid Email'
            WHEN fraud_risk_score + 
                 (CASE WHEN registrations_in_window >= 5 THEN 20 ELSE 0 END) +
                 (CASE WHEN similar_prefix_same_day >= 3 THEN 15 ELSE 0 END) +
                 (CASE WHEN part_of_enumeration_attack = 1 THEN 25 ELSE 0 END) >= 60 
            THEN 'High Risk - Fraudulent'
            WHEN fraud_risk_score + 
                 (CASE WHEN registrations_in_window >= 5 THEN 20 ELSE 0 END) +
                 (CASE WHEN similar_prefix_same_day >= 3 THEN 15 ELSE 0 END) +
                 (CASE WHEN part_of_enumeration_attack = 1 THEN 25 ELSE 0 END) >= 35 
            THEN 'Medium Risk - Suspicious'
            WHEN fraud_risk_score + 
                 (CASE WHEN registrations_in_window >= 5 THEN 20 ELSE 0 END) +
                 (CASE WHEN similar_prefix_same_day >= 3 THEN 15 ELSE 0 END) +
                 (CASE WHEN part_of_enumeration_attack = 1 THEN 25 ELSE 0 END) >= 15 
            THEN 'Low Risk - Monitor'
            ELSE 'Legitimate - Approve'
        END as security_classification,
        
        -- Remediation actions
        CASE 
            WHEN automated_action = 'Reject - Invalid Format' 
            THEN 'Require email correction during registration'
            WHEN automated_action = 'Block - High Fraud Risk' 
            THEN 'Block user and flag IP for investigation'
            WHEN automated_action = 'Flag - Potential Bot Registration' 
            THEN 'Require CAPTCHA verification and manual review'
            WHEN automated_action = 'Manual Review - Suspicious Patterns' 
            THEN 'Queue for manual verification within 24 hours'
            WHEN part_of_enumeration_attack = 1 
            THEN 'Implement rate limiting and require phone verification'
            ELSE 'Standard account activation process'
        END as remediation_action
    FROM temporal_fraud_detection tfd
)
SELECT 
    user_id,
    name,
    mail,
    is_valid_email,
    legitimacy_assessment,
    fraud_risk_score,
    total_risk_score,
    security_classification,
    automated_action,
    remediation_action,
    
    -- Key risk indicators
    CASE WHEN has_consecutive_periods = 1 THEN 'Consecutive Periods; ' ELSE '' END ||
    CASE WHEN suspicious_short_pattern = 1 THEN 'Short Pattern; ' ELSE '' END ||
    CASE WHEN contains_system_words = 1 THEN 'System Words; ' ELSE '' END ||
    CASE WHEN part_of_enumeration_attack = 1 THEN 'Enumeration Attack; ' ELSE '' END ||
    CASE WHEN registrations_in_window >= 5 THEN 'Rapid Registration; ' ELSE '' END as risk_indicators,
    
    registrations_in_window,
    similar_prefix_same_day,
    simulated_registration_date
FROM comprehensive_security_assessment
ORDER BY 
    total_risk_score DESC,
    CASE security_classification 
        WHEN 'High Risk - Fraudulent' THEN 1 
        WHEN 'Medium Risk - Suspicious' THEN 2 
        WHEN 'Low Risk - Monitor' THEN 3 
        ELSE 4 
    END,
    user_id;
```

#### 3. **Email Deliverability and Communication Optimization**
```sql
-- "Optimize email communication strategies based on email patterns and user behavior"

WITH email_deliverability_analysis AS (
    SELECT 
        user_id,
        name,
        mail,
        
        -- Basic validation
        CASE 
            WHEN mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$' THEN 1
            ELSE 0
        END as is_deliverable,
        
        -- Email reputation factors
        SUBSTRING(mail, 1, LOCATE('@', mail) - 1) as local_part,
        LENGTH(SUBSTRING(mail, 1, LOCATE('@', mail) - 1)) as local_part_length,
        
        -- Spam risk indicators
        CASE 
            WHEN mail REGEXP '[0-9]{3,}' THEN 1  -- 3+ consecutive numbers
            ELSE 0
        END as has_number_sequence,
        
        CASE 
            WHEN mail LIKE '%-%-%' OR mail LIKE '%_.%' OR mail LIKE '%._%' THEN 1
            ELSE 0
        END as has_mixed_separators,
        
        CASE 
            WHEN mail REGEXP '^[a-zA-Z]{1,3}[0-9]+@' THEN 1  -- Short letters + numbers
            ELSE 0
        END as auto_generated_pattern,
        
        -- Professional vs casual email assessment
        CASE 
            WHEN mail REGEXP '^[a-zA-Z]+\\.[a-zA-Z]+@' THEN 'Professional (firstname.lastname)'
            WHEN mail REGEXP '^[a-zA-Z]+_[a-zA-Z]+@' THEN 'Professional (firstname_lastname)'
            WHEN mail REGEXP '^[a-zA-Z]+[a-zA-Z0-9]*@' THEN 'Personal'
            WHEN mail REGEXP '^[a-zA-Z]+[0-9]+@' THEN 'Personal with numbers'
            ELSE 'Other pattern'
        END as email_style,
        
        -- Communication preference prediction
        (LENGTH(mail) - LENGTH(REPLACE(mail, '.', ''))) as period_count,
        (LENGTH(mail) - LENGTH(REPLACE(mail, '_', ''))) as underscore_count,
        (LENGTH(mail) - LENGTH(REPLACE(mail, '-', ''))) as dash_count,
        
        -- Simulate user engagement metrics
        CASE 
            WHEN user_id % 5 = 0 THEN 85  -- High engagement
            WHEN user_id % 5 = 1 THEN 65  -- Medium engagement
            WHEN user_id % 5 = 2 THEN 45  -- Low engagement
            WHEN user_id % 5 = 3 THEN 25  -- Very low engagement
            ELSE 15  -- Minimal engagement
        END as simulated_engagement_score,
        
        -- Simulate email preferences
        CASE 
            WHEN user_id % 4 = 0 THEN 'Weekly Digest'
            WHEN user_id % 4 = 1 THEN 'Daily Updates'
            WHEN user_id % 4 = 2 THEN 'Important Only'
            ELSE 'No Email Preference Set'
        END as simulated_email_preference
    FROM Users
),
communication_strategy_analysis AS (
    SELECT 
        eda.*,
        
        -- Spam risk scoring
        (has_number_sequence * 15) +
        (has_mixed_separators * 10) +
        (auto_generated_pattern * 20) +
        (CASE WHEN local_part_length < 4 THEN 15 ELSE 0 END) +
        (CASE WHEN local_part_length > 20 THEN 10 ELSE 0 END) as spam_risk_score,
        
        -- Professional communication suitability
        CASE 
            WHEN email_style LIKE 'Professional%' THEN 'High Professional Suitability'
            WHEN email_style = 'Personal' AND local_part_length >= 6 THEN 'Medium Professional Suitability'
            WHEN email_style = 'Personal with numbers' AND local_part_length >= 5 THEN 'Low Professional Suitability'
            ELSE 'Casual Communication Only'
        END as professional_suitability,
        
        -- Deliverability prediction
        CASE 
            WHEN is_deliverable = 1 AND spam_risk_score <= 15 THEN 'High Deliverability'
            WHEN is_deliverable = 1 AND spam_risk_score <= 35 THEN 'Medium Deliverability'
            WHEN is_deliverable = 1 AND spam_risk_score <= 50 THEN 'Low Deliverability'
            WHEN is_deliverable = 1 THEN 'Risk of Spam Filtering'
            ELSE 'Non-Deliverable'
        END as deliverability_prediction,
        
        -- Engagement likelihood based on email pattern
        CASE 
            WHEN email_style LIKE 'Professional%' AND simulated_engagement_score >= 60 THEN 'High Engagement Likely'
            WHEN email_style = 'Personal' AND simulated_engagement_score >= 40 THEN 'Medium Engagement Likely'
            WHEN auto_generated_pattern = 1 THEN 'Low Engagement Expected'
            ELSE 'Variable Engagement'
        END as engagement_likelihood
    FROM email_deliverability_analysis eda
),
personalized_communication_strategy AS (
    SELECT 
        csa.*,
        
        -- Optimal communication frequency
        CASE 
            WHEN engagement_likelihood = 'High Engagement Likely' AND professional_suitability = 'High Professional Suitability'
            THEN 'Daily professional updates + Weekly newsletter'
            WHEN engagement_likelihood = 'High Engagement Likely'
            THEN 'Bi-daily updates + Weekly digest'
            WHEN engagement_likelihood = 'Medium Engagement Likely'
            THEN 'Weekly digest + Monthly newsletter'
            WHEN engagement_likelihood = 'Low Engagement Expected'
            THEN 'Monthly highlights only'
            ELSE 'Quarterly important updates only'
        END as optimal_frequency,
        
        -- Content personalization strategy
        CASE 
            WHEN professional_suitability = 'High Professional Suitability' AND simulated_engagement_score >= 70
            THEN 'Professional tone, detailed content, industry insights'
            WHEN professional_suitability = 'High Professional Suitability'
            THEN 'Professional tone, concise content, key updates'
            WHEN email_style = 'Personal' AND simulated_engagement_score >= 50
            THEN 'Friendly tone, engaging content, visual elements'
            WHEN auto_generated_pattern = 1
            THEN 'Simple format, essential information only'
            ELSE 'Standard template, balanced approach'
        END as content_strategy,
        
        -- Delivery time optimization
        CASE 
            WHEN professional_suitability = 'High Professional Suitability'
            THEN 'Business hours (9 AM - 5 PM weekdays)'
            WHEN email_style = 'Personal' AND simulated_engagement_score >= 60
            THEN 'Evening hours (6 PM - 8 PM) or weekend mornings'
            WHEN engagement_likelihood = 'Low Engagement Expected'
            THEN 'Mid-week, mid-day for maximum visibility'
            ELSE 'Standard schedule (Tuesday/Thursday 10 AM)'
        END as optimal_delivery_time,
        
        -- A/B testing recommendations
        CASE 
            WHEN deliverability_prediction = 'High Deliverability' AND engagement_likelihood = 'High Engagement Likely'
            THEN 'Test advanced features: interactive content, personalization tokens'
            WHEN deliverability_prediction = 'Medium Deliverability'
            THEN 'Test subject line optimization and send time'
            WHEN deliverability_prediction = 'Low Deliverability'
            THEN 'Focus on deliverability: shorter subject lines, text-heavy content'
            ELSE 'Basic A/B testing on frequency and content length'
        END as ab_testing_strategy
    FROM communication_strategy_analysis csa
),
campaign_optimization AS (
    SELECT 
        pcs.*,
        
        -- Email campaign assignment
        CASE 
            WHEN is_deliverable = 1 AND engagement_likelihood = 'High Engagement Likely'
            THEN 'Premium Engagement Campaign'
            WHEN is_deliverable = 1 AND professional_suitability = 'High Professional Suitability'
            THEN 'Professional Development Campaign'
            WHEN is_deliverable = 1 AND engagement_likelihood = 'Medium Engagement Likely'
            THEN 'Standard Engagement Campaign'
            WHEN is_deliverable = 1 AND deliverability_prediction IN ('Low Deliverability', 'Risk of Spam Filtering')
            THEN 'Careful Delivery Campaign'
            WHEN is_deliverable = 1
            THEN 'Basic Notification Campaign'
            ELSE 'Email Update Required Campaign'
        END as campaign_assignment,
        
        -- Budget allocation recommendation
        CASE 
            WHEN engagement_likelihood = 'High Engagement Likely' AND deliverability_prediction = 'High Deliverability'
            THEN 'High Budget - Premium content and personalization'
            WHEN professional_suitability = 'High Professional Suitability'
            THEN 'Medium-High Budget - Quality professional content'
            WHEN engagement_likelihood = 'Medium Engagement Likely'
            THEN 'Medium Budget - Standard content with some personalization'
            WHEN auto_generated_pattern = 1 OR spam_risk_score > 35
            THEN 'Low Budget - Basic templates and minimal frequency'
            ELSE 'Standard Budget - Balanced approach'
        END as budget_recommendation,
        
        -- Success metrics to track
        CASE 
            WHEN engagement_likelihood = 'High Engagement Likely'
            THEN 'Open rate >40%, Click rate >8%, Conversion rate >3%'
            WHEN engagement_likelihood = 'Medium Engagement Likely'
            THEN 'Open rate >25%, Click rate >4%, Conversion rate >1.5%'
            WHEN deliverability_prediction = 'Low Deliverability'
            THEN 'Delivery rate >90%, Open rate >15%, Spam complaints <0.1%'
            ELSE 'Open rate >20%, Click rate >2%, Unsubscribe rate <2%'
        END as success_metrics_targets
    FROM personalized_communication_strategy pcs
)
SELECT 
    user_id,
    name,
    mail,
    is_deliverable,
    email_style,
    professional_suitability,
    deliverability_prediction,
    engagement_likelihood,
    optimal_frequency,
    content_strategy,
    optimal_delivery_time,
    campaign_assignment,
    budget_recommendation,
    success_metrics_targets,
    ab_testing_strategy,
    spam_risk_score,
    simulated_engagement_score
FROM campaign_optimization
ORDER BY 
    CASE campaign_assignment 
        WHEN 'Premium Engagement Campaign' THEN 1
        WHEN 'Professional Development Campaign' THEN 2
        WHEN 'Standard Engagement Campaign' THEN 3
        WHEN 'Basic Notification Campaign' THEN 4
        WHEN 'Careful Delivery Campaign' THEN 5
        ELSE 6
    END,
    simulated_engagement_score DESC,
    user_id;
```

#### 4. **Regulatory Compliance and Data Privacy Analysis**
```sql
-- "Ensure email data compliance with privacy regulations and data protection laws"

WITH privacy_compliance_analysis AS (
    SELECT 
        user_id,
        name,
        mail,
        
        -- Basic email validation
        CASE 
            WHEN mail REGEXP '^[a-zA-Z][a-zA-Z0-9._-]*@leetcode\\.com$' THEN 1
            ELSE 0
        END as is_valid_format,
        
        -- Data minimization assessment
        SUBSTRING(mail, 1, LOCATE('@', mail) - 1) as email_prefix,
        LENGTH(SUBSTRING(mail, 1, LOCATE('@', mail) - 1)) as prefix_length,
        
        -- PII (Personally Identifiable Information) risk assessment
        CASE 
            WHEN mail REGEXP '(first|last|name|birth|ssn|social)' THEN 'High PII Risk'
            WHEN mail REGEXP '[0-9]{4,}' THEN 'Potential PII in Numbers'
            WHEN mail REGEXP '^[a-zA-Z]+\\.[a-zA-Z]+@' THEN 'Likely Contains Name'
            ELSE 'Low PII Risk'
        END as pii_risk_level,
        
        -- Consent and purpose limitation
        CASE 
            WHEN is_valid_format = 1 THEN 'Valid for Email Communication'
            ELSE 'Cannot Use for Email Communication'
        END as communication_consent_status,
        
        -- Data retention considerations
        CASE 
            WHEN mail IS NULL OR mail = '' THEN 'No Email Data to Retain'
            WHEN is_valid_format = 0 THEN 'Invalid Data - Consider Deletion'
            ELSE 'Valid Data - Apply Retention Policy'
        END as retention_recommendation,
        
        -- Cross-border transfer assessment
        CASE 
            WHEN mail LIKE '%@leetcode.com' THEN 'Internal Domain - Domestic Processing'
            ELSE 'External Domain - International Transfer Rules Apply'
        END as transfer_classification,
        
        -- Anonymization potential
        CASE 
            WHEN LENGTH(SUBSTRING(mail, 1, LOCATE('@', mail) - 1)) <= 5 
            THEN 'Easy to Anonymize'
            WHEN mail REGEXP '^[a-zA-Z]+[0-9]+@' 
            THEN 'Moderate Anonymization Needed'
            WHEN mail REGEXP '^[a-zA-Z]+\\.[a-zA-Z]+@' 
            THEN 'Difficult to Anonymize (Contains Name)'
            ELSE 'Standard Anonymization Process'
        END as anonymization_complexity
    FROM Users
),
gdpr_compliance_assessment AS (
    SELECT 
        pca.*,
        
        -- GDPR Article 5 - Lawfulness, fairness, transparency
        CASE 
            WHEN communication_consent_status = 'Valid for Email Communication' 
            THEN 'Compliant - Valid for Processing'
            ELSE 'Non-Compliant - Invalid Email'
        END as lawfulness_assessment,
        
        -- GDPR Article 6 - Lawful basis for processing
        CASE 
            WHEN is_valid_format = 1 
            THEN 'Legitimate Interest for Service Communication'
            ELSE 'No Lawful Basis - Cannot Process'
        END as lawful_basis,
        
        -- GDPR Article 17 - Right to erasure (Right to be forgotten)
        CASE 
            WHEN retention_recommendation = 'Invalid Data - Consider Deletion' 
            THEN 'Immediate Deletion Candidate'
            WHEN pii_risk_level = 'High PII Risk' 
            THEN 'High Priority for Erasure Requests'
            WHEN anonymization_complexity = 'Easy to Anonymize' 
            THEN 'Can be Anonymized Instead of Deleted'
            ELSE 'Standard Retention and Erasure Policy'
        END as erasure_considerations,
        
        -- GDPR Article 20 - Right to data portability
        CASE 
            WHEN is_valid_format = 1 
            THEN 'Portable Data - Include in Export'
            ELSE 'Invalid Data - Exclude from Export'
        END as portability_status,
        
        -- GDPR Article 32 - Security of processing
        CASE 
            WHEN pii_risk_level = 'High PII Risk' 
            THEN 'High Security Measures Required'
            WHEN pii_risk_level = 'Likely Contains Name' 
            THEN 'Standard Security with Name Protection'
            ELSE 'Standard Security Measures'
        END as security_requirements
    FROM privacy_compliance_analysis pca
),
ccpa_compliance_assessment AS (
    SELECT 
        gca.*,
        
        -- CCPA - Personal Information Categories
        CASE 
            WHEN pii_risk_level IN ('High PII Risk', 'Likely Contains Name') 
            THEN 'Category B: Personal Information (Identifiers)'
            WHEN pii_risk_level = 'Potential PII in Numbers' 
            THEN 'Category F: Professional or Employment Information'
            ELSE 'Category K: Inferences (Communication Preferences)'
        END as ccpa_category,
        
        -- CCPA Right to Know
        CASE 
            WHEN is_valid_format = 1 
            THEN 'Must Disclose: Email address for service communication'
            ELSE 'Must Disclose: Invalid email data collected'
        END as right_to_know_disclosure,
        
        -- CCPA Right to Delete
        CASE 
            WHEN retention_recommendation = 'Invalid Data - Consider Deletion' 
            THEN 'Must Delete: No Business Purpose'
            WHEN pii_risk_level = 'High PII Risk' 
            THEN 'Must Delete Unless Exception Applies'
            ELSE 'Can Retain: Ongoing Business Relationship'
        END as right_to_delete_assessment,
        
        -- CCPA Do Not Sell
        'Email addresses not sold to third parties' as do_not_sell_compliance
    FROM gdpr_compliance_assessment gca
),
comprehensive_compliance_report AS (
    SELECT 
        cca.*,
        
        -- Overall compliance status
        CASE 
            WHEN lawfulness_assessment = 'Compliant - Valid for Processing' 
                 AND lawful_basis != 'No Lawful Basis - Cannot Process'
                 AND portability_status = 'Portable Data - Include in Export'
            THEN 'Fully Compliant'
            WHEN is_valid_format = 0 
            THEN 'Non-Compliant - Invalid Data'
            WHEN pii_risk_level = 'High PII Risk' 
            THEN 'Compliant with Enhanced Monitoring Required'
            ELSE 'Compliant with Standard Monitoring'
        END as overall_compliance_status,
        
        -- Automated compliance actions
        CASE 
            WHEN retention_recommendation = 'Invalid Data - Consider Deletion' 
            THEN 'Schedule for automated deletion in 30 days'
            WHEN pii_risk_level = 'High PII Risk' 
            THEN 'Flag for manual privacy review'
            WHEN anonymization_complexity = 'Easy to Anonymize' 
            THEN 'Add to anonymization queue'
            ELSE 'Continue standard processing'
        END as automated_compliance_action,
        
        -- Privacy officer review requirements
        CASE 
            WHEN pii_risk_level = 'High PII Risk' OR security_requirements = 'High Security Measures Required'
            THEN 'Required - High Risk Data'
            WHEN ccpa_category = 'Category B: Personal Information (Identifiers)'
            THEN 'Required - Personal Identifiers Present'
            WHEN anonymization_complexity = 'Difficult to Anonymize (Contains Name)'
            THEN 'Recommended - Name Data Present'
            ELSE 'Not Required - Standard Risk Level'
        END as privacy_officer_review,
        
        -- Data subject rights automation
        CASE 
            WHEN portability_status = 'Portable Data - Include in Export' 
            THEN 'Include in automated data export functionality'
            ELSE 'Exclude from automated data export'
        END as data_export_automation,
        
        CASE 
            WHEN erasure_considerations = 'Immediate Deletion Candidate' 
            THEN 'Enable immediate deletion in self-service portal'
            WHEN erasure_considerations = 'High Priority for Erasure Requests' 
            THEN 'Prioritize deletion requests for this data'
            ELSE 'Standard deletion request processing'
        END as deletion_automation
    FROM ccpa_compliance_assessment cca
)
SELECT 
    user_id,
    name,
    mail,
    is_valid_format,
    pii_risk_level,
    overall_compliance_status,
    lawful_basis,
    ccpa_category,
    security_requirements,
    automated_compliance_action,
    privacy_officer_review,
    erasure_considerations,
    anonymization_complexity,
    data_export_automation,
    deletion_automation,
    right_to_know_disclosure,
    right_to_delete_assessment
FROM comprehensive_compliance_report
ORDER BY 
    CASE overall_compliance_status 
        WHEN 'Non-Compliant - Invalid Data' THEN 1
        WHEN 'Compliant with Enhanced Monitoring Required' THEN 2
        WHEN 'Compliant with Standard Monitoring' THEN 3
        WHEN 'Fully Compliant' THEN 4
    END,
    CASE privacy_officer_review 
        WHEN 'Required - High Risk Data' THEN 1
        WHEN 'Required - Personal Identifiers Present' THEN 2
        WHEN 'Recommended - Name Data Present' THEN 3
        ELSE 4
    END,
    user_id;
```

## 🔗 Related LeetCode Questions

1. **#1527 - Patients With a Condition** (REGEXP pattern matching)
2. **#1484 - Group Sold Products By The Date** (String manipulation and aggregation)
3. **#1667 - Fix Names in a Table** (String functions and formatting)
4. **#1873 - Calculate Special Bonus** (Conditional logic with string operations)
5. **#1965 - Employees With Missing Information** (Data validation scenarios)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Regular Expressions**: Pattern matching for complex string validation
2. **Domain Validation**: Exact string matching for specific requirements
3. **Character Classes**: [a-zA-Z0-9._-] for allowed character sets
4. **Anchor Characters**: ^ for start, $ for end of string

### 🚀 **Amazon Interview Tips**
1. **Know database differences**: "MySQL uses REGEXP, PostgreSQL uses ~"
2. **Explain regex components**: "^ means start, $ means end, [a-zA-Z] means any letter"
3. **Consider performance**: "Regex can be expensive, consider indexing strategies"
4. **Discuss edge cases**: "What about international characters or special domains?"

### 🔧 **Common Patterns**
- Start anchor (^) for prefix validation
- Character classes for allowed characters
- Domain suffix validation with exact matching
- Combining multiple validation conditions

### ⚠️ **Common Mistakes to Avoid**
1. **Wrong regex syntax** (different databases use different syntax)
2. **Missing anchors** (not using ^ and $ for exact matching)
3. **Escaping issues** (not properly escaping special characters like .)
4. **Performance problems** (not considering regex optimization)

### 🔍 **Performance Considerations**
- Regex operations can be CPU-intensive
- Consider creating computed columns for frequently checked patterns
- Index on domain suffix for faster filtering
- Use simple string operations when possible before regex

### 🎯 **Amazon Leadership Principles Applied**
- **Security First**: Proper email validation prevents security issues
- **Customer Obsession**: Valid emails ensure reliable customer communication
- **Operational Excellence**: Systematic data validation improves system reliability
- **Innovation**: Advanced email analytics for personalized customer experience

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Validate emails with multiple domain options**
2. **Extract and analyze email prefix patterns**
3. **Find emails with specific character combinations**
4. **Implement international email validation**

Remember: Email validation is crucial for Amazon's customer communication systems, marketing platforms, account security, and regulatory compliance across global operations and diverse customer bases!