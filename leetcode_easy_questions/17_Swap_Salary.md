# LeetCode Easy #627: Swap Salary

## 📋 Problem Statement

Write a SQL query to **swap all 'f' and 'm' values** (i.e., change all 'f' values to 'm' and vice versa) in the **sex** column using a **single update statement** and **no intermediate temporary tables**.

Note that you must write a single update statement. **Do not write any select statement** for this problem.

## 🗄️ Table Schema

**Salary Table:**
```
+-------------+----------+
| Column Name | Type     |
+-------------+----------+
| id          | int      |
| name        | varchar  |
| sex         | ENUM     |
| salary      | int      |
+-------------+----------+
```
- `id` is the primary key for this table.
- The sex column is ENUM value of type ('m', 'f').
- The table contains information about an employee.

## 📊 Sample Data

**Before Update:**

**Salary Table:**
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |

**After Update:**

**Salary Table:**
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |

**Explanation:** 
- All 'm' values changed to 'f'
- All 'f' values changed to 'm'
- Other columns remain unchanged

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to swap values in a single UPDATE statement
- No SELECT statements allowed
- No temporary tables allowed
- Must handle bidirectional swap (m↔f)

### 2. **Key Insights**
- CASE statement can handle conditional updates
- Need to swap both directions simultaneously
- UPDATE with CASE WHEN for conditional logic
- Single statement requirement rules out multi-step approaches

### 3. **Interview Discussion Points**
- "I need to use CASE statement to handle both swap directions"
- "Single UPDATE with conditional logic is the key"
- "Must avoid creating intermediate states during the swap"

## 🔧 Step-by-Step Solution Logic

### Step 1: Understand the Swap Logic
```sql
-- If sex = 'm' then change to 'f'
-- If sex = 'f' then change to 'm'
-- Use CASE statement for conditional logic
```

### Step 2: Construct the UPDATE Statement
```sql
-- UPDATE table_name SET column = CASE WHEN condition THEN value END
-- Handle both 'm' and 'f' cases in single CASE statement
```

### Step 3: Apply to All Rows
```sql
-- No WHERE clause needed - apply to all rows
-- CASE statement handles the logic automatically
```

## ✅ Optimized SQL Solution

**Solution 1: CASE Statement Approach**
```sql
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
END;
```

### Alternative Solutions

**Solution 2: IF Function (MySQL)**
```sql
UPDATE Salary
SET sex = IF(sex = 'm', 'f', 'm');
```

**Solution 3: Using CHAR and ASCII Manipulation**
```sql
UPDATE Salary
SET sex = CHAR(ASCII('f') + ASCII('m') - ASCII(sex));
```

**Solution 4: Using Boolean Logic**
```sql
UPDATE Salary
SET sex = CASE sex
    WHEN 'm' THEN 'f'
    ELSE 'm'
END;
```

**Solution 5: Mathematical Approach with ENUM**
```sql
-- Note: This approach works because ENUM values have numeric positions
UPDATE Salary
SET sex = ELT(3 - FIELD(sex, 'm', 'f'), 'm', 'f');
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Audit Trail and Change Tracking**
```sql
-- "How would you track this change for audit purposes?"

-- First, let's create an audit table
-- CREATE TABLE salary_audit (
--     audit_id INT AUTO_INCREMENT PRIMARY KEY,
--     employee_id INT,
--     old_sex ENUM('m', 'f'),
--     new_sex ENUM('m', 'f'),
--     change_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
--     changed_by VARCHAR(50)
-- );

-- Before the swap, create audit trail using a trigger or separate process
-- For demonstration, here's how you might log changes:

-- Step 1: Create backup of current state
CREATE TEMPORARY TABLE salary_backup AS 
SELECT id, sex FROM Salary;

-- Step 2: Perform the swap
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
END;

-- Step 3: Log the changes
INSERT INTO salary_audit (employee_id, old_sex, new_sex, changed_by)
SELECT 
    s.id,
    sb.sex as old_sex,
    s.sex as new_sex,
    'system_swap_operation' as changed_by
FROM Salary s
JOIN salary_backup sb ON s.id = sb.id
WHERE s.sex != sb.sex;

-- Clean up
DROP TEMPORARY TABLE salary_backup;
```

#### 2. **Safe Swap with Transaction Management**
```sql
-- "Implement the swap with proper transaction safety"

-- Begin transaction for atomicity
START TRANSACTION;

-- Check current state before swap
SELECT 
    'Pre-Swap Analysis' as check_point,
    COUNT(*) as total_employees,
    COUNT(CASE WHEN sex = 'm' THEN 1 END) as male_count,
    COUNT(CASE WHEN sex = 'f' THEN 1 END) as female_count
FROM Salary;

-- Perform the swap
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
END;

-- Verify the swap worked correctly
SELECT 
    'Post-Swap Verification' as check_point,
    COUNT(*) as total_employees,
    COUNT(CASE WHEN sex = 'm' THEN 1 END) as male_count_after,
    COUNT(CASE WHEN sex = 'f' THEN 1 END) as female_count_after
FROM Salary;

-- If verification passes, commit; otherwise rollback
-- COMMIT; or ROLLBACK;
```

#### 3. **Batch Processing for Large Datasets**
```sql
-- "How would you handle this swap for millions of records?"

-- For very large tables, consider batch processing
-- Step 1: Create procedure for batch updates

DELIMITER //
CREATE PROCEDURE SwapSalaryBatch(IN batch_size INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE batch_start INT DEFAULT 1;
    DECLARE batch_end INT;
    DECLARE total_rows INT;
    
    -- Get total number of rows
    SELECT COUNT(*) INTO total_rows FROM Salary;
    
    WHILE batch_start <= total_rows DO
        SET batch_end = batch_start + batch_size - 1;
        
        -- Update batch
        UPDATE Salary
        SET sex = CASE 
            WHEN sex = 'm' THEN 'f'
            WHEN sex = 'f' THEN 'm'
        END
        WHERE id >= batch_start AND id <= batch_end;
        
        -- Log progress
        SELECT CONCAT('Processed batch: ', batch_start, ' to ', batch_end) as progress;
        
        SET batch_start = batch_end + 1;
        
        -- Small delay to prevent overwhelming the system
        SELECT SLEEP(0.1);
    END WHILE;
END //
DELIMITER ;

-- Execute batch swap
-- CALL SwapSalaryBatch(1000);
```

#### 4. **Data Validation and Quality Checks**
```sql
-- "Add comprehensive data validation for the swap operation"

-- Pre-swap validation
SELECT 
    'Data Quality Check' as validation_type,
    COUNT(*) as total_records,
    COUNT(CASE WHEN sex IS NULL THEN 1 END) as null_sex_count,
    COUNT(CASE WHEN sex NOT IN ('m', 'f') THEN 1 END) as invalid_sex_count,
    COUNT(DISTINCT sex) as unique_sex_values,
    GROUP_CONCAT(DISTINCT sex) as sex_values_found
FROM Salary;

-- Perform swap with validation
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
    ELSE sex  -- Keep unchanged if not 'm' or 'f'
END
WHERE sex IN ('m', 'f');  -- Only update valid values

-- Post-swap validation
WITH validation_results AS (
    SELECT 
        COUNT(*) as total_after_swap,
        COUNT(CASE WHEN sex = 'm' THEN 1 END) as male_count,
        COUNT(CASE WHEN sex = 'f' THEN 1 END) as female_count,
        COUNT(CASE WHEN sex NOT IN ('m', 'f') THEN 1 END) as invalid_count
    FROM Salary
)
SELECT 
    *,
    CASE 
        WHEN invalid_count = 0 THEN 'Swap Successful'
        ELSE 'Data Quality Issues Detected'
    END as swap_status
FROM validation_results;
```

#### 5. **Performance Optimization and Monitoring**
```sql
-- "Optimize the swap operation for performance"

-- Add index if not exists (though primary key likely exists)
-- CREATE INDEX idx_salary_sex ON Salary(sex);

-- Monitor query execution
EXPLAIN UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
END;

-- Performance-optimized version with specific targeting
-- Option 1: Two separate updates (if single statement not required)
-- UPDATE Salary SET sex = 'temp_m' WHERE sex = 'm';
-- UPDATE Salary SET sex = 'f' WHERE sex = 'temp_m';
-- UPDATE Salary SET sex = 'm' WHERE sex = 'f';

-- Option 2: Optimized single statement
UPDATE Salary
SET sex = IF(sex = 'm', 'f', 'm')
WHERE sex IN ('m', 'f');

-- Performance monitoring
SELECT 
    TABLE_NAME,
    TABLE_ROWS,
    AVG_ROW_LENGTH,
    DATA_LENGTH,
    INDEX_LENGTH
FROM information_schema.TABLES 
WHERE TABLE_NAME = 'Salary';
```

#### 6. **Advanced Scenario Handling**
```sql
-- "Handle complex scenarios and edge cases"

-- Scenario 1: Multiple gender values
ALTER TABLE Salary MODIFY sex ENUM('m', 'f', 'o', 'n');  -- Add 'other', 'not specified'

-- Complex swap with additional logic
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
    WHEN sex = 'o' THEN 'o'  -- Keep 'other' unchanged
    WHEN sex = 'n' THEN 'n'  -- Keep 'not specified' unchanged
    ELSE sex  -- Handle any unexpected values
END;

-- Scenario 2: Conditional swap based on other criteria
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' AND salary > 3000 THEN 'f'  -- Only swap high earners
    WHEN sex = 'f' AND salary > 3000 THEN 'm'
    ELSE sex  -- Keep others unchanged
END;

-- Scenario 3: Swap with statistical tracking
WITH pre_swap_stats AS (
    SELECT 
        sex,
        COUNT(*) as count,
        AVG(salary) as avg_salary,
        MAX(salary) as max_salary,
        MIN(salary) as min_salary
    FROM Salary
    GROUP BY sex
)
SELECT 
    'Pre-Swap Statistics' as phase,
    sex,
    count,
    ROUND(avg_salary, 2) as avg_salary,
    max_salary,
    min_salary
FROM pre_swap_stats;

-- Perform swap
UPDATE Salary
SET sex = CASE 
    WHEN sex = 'm' THEN 'f'
    WHEN sex = 'f' THEN 'm'
END;

-- Post-swap analysis
WITH post_swap_stats AS (
    SELECT 
        sex,
        COUNT(*) as count,
        AVG(salary) as avg_salary,
        MAX(salary) as max_salary,
        MIN(salary) as min_salary
    FROM Salary
    GROUP BY sex
)
SELECT 
    'Post-Swap Statistics' as phase,
    sex,
    count,
    ROUND(avg_salary, 2) as avg_salary,
    max_salary,
    min_salary
FROM post_swap_stats;
```

## 🔗 Related LeetCode Questions

1. **#196 - Delete Duplicate Emails** (Single DML statement operations)
2. **#610 - Triangle Judgement** (CASE statement usage)
3. **#1873 - Calculate Special Bonus** (Conditional updates)
4. **#1667 - Fix Names in a Table** (String manipulation in updates)
5. **#1484 - Group Sold Products By The Date** (Data transformation)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **UPDATE with CASE**: Conditional data modification
2. **Single Statement Requirement**: Avoiding temporary tables
3. **ENUM Handling**: Working with enumerated values
4. **Bidirectional Swaps**: Simultaneous value exchanges

### 🚀 **Amazon Interview Tips**
1. **Clarify constraints**: "Single statement means no temporary tables, right?"
2. **Discuss alternatives**: "IF function vs CASE statement trade-offs"
3. **Consider performance**: "Index considerations for large table updates"
4. **Think safety**: "Transaction management for critical data changes"

### 🔧 **Common Patterns**
- UPDATE table SET column = CASE WHEN condition THEN value END
- IF(condition, true_value, false_value) for simple swaps
- Mathematical approaches for numeric enum positions
- Transaction wrapping for data safety

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting CASE END clause** (incomplete CASE statements)
2. **Using multiple statements** when single statement required
3. **Not handling NULL values** properly in CASE logic
4. **Performance issues** on large tables without proper indexing

### 🔍 **Performance Considerations**
- Single UPDATE is generally efficient for most table sizes
- Consider batch processing for very large tables (millions of rows)
- Index on the updated column for better performance
- Transaction log considerations for large updates

### 🎯 **Amazon Leadership Principles Applied**
- **Operational Excellence**: Safe and efficient data operations
- **Customer Obsession**: Maintaining data integrity for customers
- **Frugality**: Efficient single-statement approach saves resources
- **Ownership**: Taking responsibility for data consistency

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Swap different enum values (e.g., 'active'/'inactive')**
2. **Conditional swap based on salary ranges**
3. **Swap with additional audit logging**
4. **Batch swap processing for performance**

Remember: Data transformation and integrity operations are fundamental to Amazon's data processing pipelines, ETL operations, and maintaining consistent data across distributed systems!