# LeetCode Easy #610: Triangle Judgement

## 📋 Problem Statement

A pupil Tim gets homework to identify whether three line segments could form a triangle.

However, this assignment is very heavy because there are hundreds of records to calculate.

Could you help Tim by writing a SQL query to judge whether these three sides can form a triangle?

## 🗄️ Table Schema

**Triangle Table:**
```
+-------------+------+
| Column Name | Type |
+-------------+------+
| x           | int  |
| y           | int  |
| z           | int  |
+-------------+------+
```
- Each row of this table contains the lengths of three line segments.

## 📊 Sample Data

**Triangle Table:**
| x  | y  | z  |
|----|----|----|
| 13 | 15 | 30 |
| 10 | 20 | 15 |

**Expected Output:**
| x  | y  | z  | triangle |
|----|----|----|----------|
| 13 | 15 | 30 | No       |
| 10 | 20 | 15 | Yes      |

**Explanation:**
- For (13, 15, 30): 13 + 15 = 28 < 30, so it cannot form a triangle
- For (10, 20, 15): All combinations satisfy the triangle inequality (10+20>15, 10+15>20, 20+15>10)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Need to apply triangle inequality theorem
- For three sides a, b, c to form a triangle: a + b > c, a + c > b, b + c > a
- All three conditions must be satisfied
- Use CASE statement to return "Yes" or "No"

### 2. **Key Mathematical Insight**
- Triangle Inequality: Sum of any two sides must be greater than the third side
- Need to check all three combinations
- If ANY condition fails, it's not a triangle

### 3. **Interview Discussion Points**
- "This is a mathematical validation problem using triangle inequality"
- "I need to check all three combinations of side sums"
- "CASE statement is perfect for conditional logic like this"

## 🔧 Step-by-Step Solution Logic

### Step 1: Understand Triangle Inequality
```sql
-- For sides x, y, z to form a triangle:
-- x + y > z  AND  x + z > y  AND  y + z > x
```

### Step 2: Apply Conditional Logic
```sql
-- Use CASE WHEN to check all three conditions
-- Return "Yes" if all conditions are true, "No" otherwise
```

### Step 3: Format Output
```sql
-- Select all original columns plus the triangle judgment
```

## ✅ Optimized SQL Solution

**Solution 1: CASE Statement with AND Logic**
```sql
SELECT 
    x, 
    y, 
    z,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END as triangle
FROM Triangle;
```

### Alternative Solutions

**Solution 2: Using Boolean Logic**
```sql
SELECT 
    x, 
    y, 
    z,
    IF(x + y > z AND x + z > y AND y + z > x, 'Yes', 'No') as triangle
FROM Triangle;
```

**Solution 3: Explicit Condition Check**
```sql
SELECT 
    x, 
    y, 
    z,
    CASE 
        WHEN (x + y <= z) OR (x + z <= y) OR (y + z <= x) THEN 'No'
        ELSE 'Yes'
    END as triangle
FROM Triangle;
```

**Solution 4: With Mathematical Validation**
```sql
SELECT 
    x, 
    y, 
    z,
    x + y as sum_xy,
    x + z as sum_xz,
    y + z as sum_yz,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END as triangle,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Valid Triangle'
        WHEN x + y <= z THEN 'x + y <= z'
        WHEN x + z <= y THEN 'x + z <= y'
        WHEN y + z <= x THEN 'y + z <= x'
    END as failure_reason
FROM Triangle;
```

**Solution 5: Using Greatest/Least Functions**
```sql
SELECT 
    x, 
    y, 
    z,
    CASE 
        WHEN GREATEST(x, y, z) < (x + y + z - GREATEST(x, y, z)) THEN 'Yes'
        ELSE 'No'
    END as triangle
FROM Triangle;
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Detailed Triangle Analysis**
```sql
-- "Provide comprehensive triangle analysis with calculations"

SELECT 
    x, 
    y, 
    z,
    x + y + z as perimeter,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END as triangle,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN
            CASE 
                WHEN x = y AND y = z THEN 'Equilateral'
                WHEN x = y OR y = z OR x = z THEN 'Isosceles'
                ELSE 'Scalene'
            END
        ELSE 'Not a Triangle'
    END as triangle_type,
    ROUND(
        CASE 
            WHEN x + y > z AND x + z > y AND y + z > x THEN
                SQRT((x + y + z) * (-x + y + z) * (x - y + z) * (x + y - z)) / 4
            ELSE 0
        END, 2
    ) as area_herons_formula
FROM Triangle;
```

#### 2. **Triangle Classification System**
```sql
-- "Classify triangles by both validity and type"

WITH triangle_analysis AS (
    SELECT 
        x, y, z,
        x + y + z as perimeter,
        CASE 
            WHEN x + y > z AND x + z > y AND y + z > x THEN 1
            ELSE 0
        END as is_valid_triangle,
        GREATEST(x, y, z) as longest_side,
        LEAST(x, y, z) as shortest_side
    FROM Triangle
)
SELECT 
    x, y, z,
    CASE 
        WHEN is_valid_triangle = 1 THEN 'Yes'
        ELSE 'No'
    END as triangle,
    CASE 
        WHEN is_valid_triangle = 0 THEN 'Invalid'
        WHEN x = y AND y = z THEN 'Equilateral'
        WHEN x = y OR y = z OR x = z THEN 'Isosceles'
        ELSE 'Scalene'
    END as triangle_type,
    CASE 
        WHEN is_valid_triangle = 0 THEN 'N/A'
        WHEN longest_side * longest_side = (perimeter - longest_side - longest_side) THEN 'Right Triangle'
        WHEN longest_side * longest_side > (x*x + y*y + z*z - longest_side*longest_side) THEN 'Obtuse Triangle'
        ELSE 'Acute Triangle'
    END as angle_classification,
    perimeter,
    ROUND(longest_side / shortest_side, 2) as aspect_ratio
FROM triangle_analysis
ORDER BY is_valid_triangle DESC, perimeter DESC;
```

#### 3. **Batch Triangle Validation**
```sql
-- "Analyze triangle validity statistics across the dataset"

WITH triangle_stats AS (
    SELECT 
        COUNT(*) as total_records,
        SUM(CASE WHEN x + y > z AND x + z > y AND y + z > x THEN 1 ELSE 0 END) as valid_triangles,
        SUM(CASE WHEN x + y <= z THEN 1 ELSE 0 END) as fail_xy_z,
        SUM(CASE WHEN x + z <= y THEN 1 ELSE 0 END) as fail_xz_y,
        SUM(CASE WHEN y + z <= x THEN 1 ELSE 0 END) as fail_yz_x,
        AVG(x + y + z) as avg_perimeter,
        MAX(x + y + z) as max_perimeter,
        MIN(x + y + z) as min_perimeter
    FROM Triangle
)
SELECT 
    total_records,
    valid_triangles,
    total_records - valid_triangles as invalid_triangles,
    ROUND(valid_triangles * 100.0 / total_records, 2) as validity_percentage,
    fail_xy_z,
    fail_xz_y,
    fail_yz_x,
    ROUND(avg_perimeter, 2) as avg_perimeter,
    max_perimeter,
    min_perimeter
FROM triangle_stats;
```

#### 4. **Triangle Quality Assessment**
```sql
-- "Assess triangle quality and characteristics"

SELECT 
    x, y, z,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END as triangle,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN
            CASE 
                WHEN ABS(x - y) <= 1 AND ABS(y - z) <= 1 AND ABS(x - z) <= 1 THEN 'Nearly Equilateral'
                WHEN GREATEST(x, y, z) / LEAST(x, y, z) <= 2 THEN 'Well Proportioned'
                WHEN GREATEST(x, y, z) / LEAST(x, y, z) <= 5 THEN 'Moderately Proportioned'
                ELSE 'Poorly Proportioned'
            END
        ELSE 'Invalid'
    END as quality_assessment,
    ROUND(
        CASE 
            WHEN LEAST(x, y, z) > 0 THEN GREATEST(x, y, z) / LEAST(x, y, z)
            ELSE 0
        END, 2
    ) as proportion_ratio,
    x + y + z as perimeter
FROM Triangle
ORDER BY 
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 1
        ELSE 0
    END DESC,
    proportion_ratio ASC;
```

#### 5. **Triangle Generation and Validation**
```sql
-- "Generate test triangles and validate them"

WITH generated_triangles AS (
    -- Generate various triangle test cases
    SELECT 3 as x, 4 as y, 5 as z, 'Classic Right Triangle' as test_case
    UNION ALL
    SELECT 5, 5, 5, 'Equilateral'
    UNION ALL
    SELECT 5, 5, 8, 'Isosceles'
    UNION ALL
    SELECT 3, 4, 8, 'Invalid - Sum Rule'
    UNION ALL
    SELECT 1, 1, 3, 'Invalid - Degenerate'
    UNION ALL
    SELECT 10, 24, 26, 'Right Triangle Large'
    UNION ALL
    SELECT 7, 24, 25, 'Right Triangle'
    UNION ALL
    SELECT 1, 2, 2, 'Small Isosceles'
)
SELECT 
    test_case,
    x, y, z,
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 'Yes'
        ELSE 'No'
    END as triangle,
    CASE 
        WHEN x + y <= z THEN CONCAT('Fails: ', x, ' + ', y, ' = ', x + y, ' <= ', z)
        WHEN x + z <= y THEN CONCAT('Fails: ', x, ' + ', z, ' = ', x + z, ' <= ', y)
        WHEN y + z <= x THEN CONCAT('Fails: ', y, ' + ', z, ' = ', y + z, ' <= ', x)
        ELSE 'All conditions satisfied'
    END as validation_details
FROM generated_triangles
ORDER BY 
    CASE 
        WHEN x + y > z AND x + z > y AND y + z > x THEN 1
        ELSE 0
    END DESC;
```

#### 6. **Performance and Edge Case Testing**
```sql
-- "Handle edge cases and performance considerations"

-- Test with extreme values and edge cases
WITH edge_cases AS (
    SELECT 
        x, y, z,
        CASE 
            WHEN x <= 0 OR y <= 0 OR z <= 0 THEN 'Invalid Input - Non-positive sides'
            WHEN x + y > z AND x + z > y AND y + z > x THEN 'Valid Triangle'
            ELSE 'Invalid Triangle'
        END as comprehensive_validation,
        CASE 
            WHEN x + y = z OR x + z = y OR y + z = x THEN 'Degenerate Triangle'
            WHEN x + y > z AND x + z > y AND y + z > x THEN 'Valid Triangle'
            ELSE 'Invalid Triangle'
        END as including_degenerate
    FROM Triangle
)
SELECT 
    x, y, z,
    comprehensive_validation,
    including_degenerate,
    CASE 
        WHEN comprehensive_validation = 'Valid Triangle' THEN 'Yes'
        ELSE 'No'
    END as simple_answer
FROM edge_cases
ORDER BY comprehensive_validation, x, y, z;
```

## 🔗 Related LeetCode Questions

1. **#596 - Classes More Than 5 Students** (Conditional aggregation)
2. **#620 - Not Boring Movies** (Conditional filtering)
3. **#1148 - Article Views I** (Basic conditional logic)
4. **#1633 - Percentage of Users Attended a Contest** (Mathematical calculations)
5. **#1873 - Calculate Special Bonus** (CASE statement usage)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **Triangle Inequality**: Mathematical rule for triangle validity
2. **CASE Statements**: Conditional logic in SQL
3. **Boolean Logic**: Combining multiple conditions with AND/OR
4. **Mathematical Validation**: Applying geometric principles in SQL

### 🚀 **Amazon Interview Tips**
1. **Explain the math**: "Triangle inequality states that sum of any two sides must exceed the third"
2. **Consider edge cases**: "What about negative values or zero-length sides?"
3. **Discuss performance**: "This is O(1) per row with simple arithmetic operations"
4. **Think applications**: "Useful for geometry validation, graphics, and engineering calculations"

### 🔧 **Common Patterns**
- CASE WHEN for conditional output
- Multiple condition checking with AND
- Mathematical formula implementation
- Data validation patterns

### ⚠️ **Common Mistakes to Avoid**
1. **Forgetting one condition** (only checking two of three triangle inequalities)
2. **Wrong logical operators** (using OR instead of AND)
3. **Edge case handling** (not considering zero or negative values)
4. **Mathematical errors** (incorrect inequality signs)

### 🔍 **Mathematical Applications**
- Geometry validation in CAD systems
- Graphics and 3D modeling validation
- Engineering stress analysis
- Physical simulation constraints

### 🎯 **Amazon Leadership Principles Applied**
- **Accuracy**: Mathematical precision in calculations
- **Customer Obsession**: Reliable geometric validation for applications
- **Operational Excellence**: Robust validation logic for critical systems
- **Innovation**: Creative application of mathematical concepts in data analysis

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Calculate triangle areas using Heron's formula**
2. **Classify triangles as acute, obtuse, or right triangles**
3. **Find the largest valid triangle in the dataset**
4. **Generate triangle test cases programmatically**

Remember: Mathematical validation and geometric calculations are essential for Amazon's logistics optimization, warehouse layout design, packaging efficiency, and delivery route planning!