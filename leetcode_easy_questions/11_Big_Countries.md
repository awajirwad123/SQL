# LeetCode Easy #595: Big Countries

## 📋 Problem Statement

A country is **big** if:
- it has an area of at least 3 million (i.e., `3000000 km²`), **or**
- it has a population of at least 25 million (i.e., `25000000`)

Write a SQL query to report the **name**, **population**, and **area** of the big countries.

Return the result table in **any order**.

## 🗄️ Table Schema

**World Table:**
```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| name        | varchar |
| continent   | varchar |
| area        | int     |
| population  | int     |
| gdp         | bigint  |
+-------------+---------+
```
- `name` is the primary key column for this table.
- This table gives information about the name of the country, the continent to which it belongs, its area, the population, and its GDP value.

## 📊 Sample Data

**World Table:**
| name        | continent | area    | population | gdp          |
|-------------|-----------|---------|------------|--------------|
| Afghanistan | Asia      | 652230  | 25500100   | 20343000000  |
| Albania     | Europe    | 28748   | 2831741    | 12960000000  |
| Algeria     | Africa    | 2381741 | 37100000   | 188681000000 |
| Andorra     | Europe    | 468     | 78115      | 3712000000   |
| Angola      | Africa    | 1246700 | 20609294   | 100990000000 |

**Expected Output:**
| name        | population | area    |
|-------------|------------|---------|
| Afghanistan | 25500100   | 652230  |
| Algeria     | 37100000   | 2381741 |

**Explanation:** 
- Afghanistan has a population of 25,500,100 (≥ 25,000,000)
- Algeria has an area of 2,381,741 km² (≥ 3,000,000) and population of 37,100,000 (≥ 25,000,000)

## 🤔 Interview Thought Process

### 1. **Problem Analysis**
- Simple filtering problem with OR condition
- Need to check two criteria: area ≥ 3,000,000 OR population ≥ 25,000,000
- Return name, population, and area columns
- No aggregation or joining required

### 2. **Key Insights**
- Use WHERE clause with OR condition
- Two independent criteria, either one qualifies the country
- Straightforward SELECT with filtering
- Consider performance implications of OR conditions

### 3. **Interview Discussion Points**
- "This is a basic filtering problem with OR logic"
- "I need to check if area is large OR population is large"
- "The OR condition means either criteria can qualify a country"

## 🔧 Step-by-Step Solution Logic

### Step 1: Identify Required Columns
```sql
-- SELECT name, population, area
-- These are the specific columns requested
```

### Step 2: Apply Filtering Criteria
```sql
-- WHERE area >= 3000000 OR population >= 25000000
-- OR means either condition satisfies the requirement
```

### Step 3: Return Results
```sql
-- No specific ordering required, return all qualifying countries
```

## ✅ Optimized SQL Solution

**Solution 1: Basic OR Condition**
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

### Alternative Solutions

**Solution 2: Using UNION (Less Efficient)**
```sql
SELECT name, population, area
FROM World
WHERE area >= 3000000

UNION

SELECT name, population, area
FROM World
WHERE population >= 25000000;
```

**Solution 3: Using CASE for Analysis**
```sql
SELECT 
    name, 
    population, 
    area,
    CASE 
        WHEN area >= 3000000 AND population >= 25000000 THEN 'Both Criteria'
        WHEN area >= 3000000 THEN 'Large Area'
        WHEN population >= 25000000 THEN 'Large Population'
    END as big_country_reason
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

**Solution 4: With Additional Statistics**
```sql
SELECT 
    name, 
    population, 
    area,
    ROUND(population / area, 2) as population_density
FROM World
WHERE area >= 3000000 OR population >= 25000000
ORDER BY area DESC, population DESC;
```

**Solution 5: Using IN with Subqueries (Over-engineered)**
```sql
SELECT name, population, area
FROM World
WHERE name IN (
    SELECT name FROM World WHERE area >= 3000000
    UNION
    SELECT name FROM World WHERE population >= 25000000
);
```

## 🎯 Amazon Interview Extensions

### Follow-up Questions You Might Get:

#### 1. **Performance Optimization**
```sql
-- "How would you optimize this query for millions of countries?"

-- Create composite index for OR conditions
CREATE INDEX idx_world_big_countries ON World(area, population);

-- Or separate indexes
CREATE INDEX idx_world_area ON World(area);
CREATE INDEX idx_world_population ON World(population);

-- Original query remains the same - database optimizer handles the rest
SELECT name, population, area
FROM World
WHERE area >= 3000000 OR population >= 25000000;
```

#### 2. **Advanced Filtering**
```sql
-- "Find big countries by continent"

SELECT 
    continent,
    COUNT(*) as big_countries_count,
    AVG(area) as avg_area,
    AVG(population) as avg_population
FROM World
WHERE area >= 3000000 OR population >= 25000000
GROUP BY continent
ORDER BY big_countries_count DESC;
```

#### 3. **Economic Analysis**
```sql
-- "Include economic metrics for big countries"

SELECT 
    name, 
    population, 
    area,
    gdp,
    ROUND(gdp / population, 2) as gdp_per_capita,
    ROUND(gdp / area, 2) as gdp_per_km2,
    CASE 
        WHEN gdp / population > 50000 THEN 'High Income'
        WHEN gdp / population > 20000 THEN 'Upper Middle Income'
        WHEN gdp / population > 5000 THEN 'Lower Middle Income'
        ELSE 'Low Income'
    END as income_category
FROM World
WHERE area >= 3000000 OR population >= 25000000
ORDER BY gdp_per_capita DESC;
```

#### 4. **Threshold Analysis**
```sql
-- "What if we adjust the thresholds? How many countries qualify?"

WITH threshold_analysis AS (
    SELECT 
        'Original (3M area OR 25M pop)' as criteria,
        COUNT(*) as qualifying_countries
    FROM World
    WHERE area >= 3000000 OR population >= 25000000
    
    UNION ALL
    
    SELECT 
        'Strict (5M area OR 50M pop)' as criteria,
        COUNT(*) as qualifying_countries
    FROM World
    WHERE area >= 5000000 OR population >= 50000000
    
    UNION ALL
    
    SELECT 
        'Relaxed (1M area OR 10M pop)' as criteria,
        COUNT(*) as qualifying_countries
    FROM World
    WHERE area >= 1000000 OR population >= 10000000
)
SELECT * FROM threshold_analysis;
```

#### 5. **Data Quality Validation**
```sql
-- "Check for data quality issues in the dataset"

-- Find countries with potential data issues
SELECT 
    name,
    area,
    population,
    gdp,
    CASE 
        WHEN area IS NULL OR area <= 0 THEN 'Invalid Area'
        WHEN population IS NULL OR population <= 0 THEN 'Invalid Population'
        WHEN gdp IS NULL OR gdp < 0 THEN 'Invalid GDP'
        WHEN population / area > 100000 THEN 'Extremely High Density'
        WHEN population / area < 0.1 THEN 'Extremely Low Density'
        ELSE 'Data OK'
    END as data_quality_check
FROM World
WHERE area >= 3000000 OR population >= 25000000
ORDER BY 
    CASE 
        WHEN area IS NULL OR area <= 0 THEN 1
        WHEN population IS NULL OR population <= 0 THEN 2
        WHEN gdp IS NULL OR gdp < 0 THEN 3
        ELSE 4
    END;
```

#### 6. **Comprehensive Country Analysis**
```sql
-- "Provide comprehensive analysis of big countries"

WITH big_countries AS (
    SELECT 
        name, 
        continent,
        population, 
        area,
        gdp,
        ROUND(population / area, 2) as population_density,
        ROUND(gdp / population, 2) as gdp_per_capita
    FROM World
    WHERE area >= 3000000 OR population >= 25000000
),
continent_stats AS (
    SELECT 
        continent,
        COUNT(*) as countries_count,
        AVG(population_density) as avg_density,
        AVG(gdp_per_capita) as avg_gdp_per_capita
    FROM big_countries
    GROUP BY continent
)
SELECT 
    bc.*,
    cs.avg_density as continent_avg_density,
    cs.avg_gdp_per_capita as continent_avg_gdp_per_capita,
    CASE 
        WHEN bc.population_density > cs.avg_density THEN 'Above Average Density'
        ELSE 'Below Average Density'
    END as density_comparison,
    CASE 
        WHEN bc.gdp_per_capita > cs.avg_gdp_per_capita THEN 'Above Average GDP/Capita'
        ELSE 'Below Average GDP/Capita'
    END as gdp_comparison
FROM big_countries bc
JOIN continent_stats cs ON bc.continent = cs.continent
ORDER BY bc.continent, bc.gdp_per_capita DESC;
```

#### 7. **Time Series Analysis (with date dimension)**
```sql
-- "Track how the list of big countries changes over time"

-- Assuming we have historical data with a year column
SELECT 
    year,
    COUNT(*) as big_countries_count,
    COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY year) as year_over_year_change
FROM world_historical
WHERE area >= 3000000 OR population >= 25000000
GROUP BY year
ORDER BY year;
```

## 🔗 Related LeetCode Questions

1. **#183 - Customers Who Never Order** (Basic filtering with NULL)
2. **#584 - Find Customer Referee** (NULL handling in conditions)
3. **#596 - Classes More Than 5 Students** (GROUP BY with HAVING)
4. **#620 - Not Boring Movies** (WHERE conditions with AND/OR)
5. **#1757 - Recyclable and Low Fat Products** (Multiple AND conditions)

## 📚 Key Takeaways & Best Practices

### 💡 **Core Concepts**
1. **OR Logic**: Either condition can satisfy the requirement
2. **Threshold Filtering**: Using comparison operators with constants
3. **Multi-Column Selection**: Returning specific columns from filtered data
4. **Index Strategy**: Optimizing OR conditions with proper indexing

### 🚀 **Amazon Interview Tips**
1. **Clarify requirements**: "Are these thresholds fixed or configurable?"
2. **Discuss performance**: "OR conditions may require special indexing considerations"
3. **Consider edge cases**: "What about countries with NULL values?"
4. **Think business context**: "These thresholds might represent market size criteria"

### 🔧 **Common Patterns**
- Simple WHERE clause with OR conditions
- Comparison operators (>=, >, <, <=) for numerical thresholds
- CASE statements for categorization
- Index design for OR conditions

### ⚠️ **Common Mistakes to Avoid**
1. **Using AND instead of OR** when either condition should qualify
2. **Incorrect threshold values** (mixing millions with actual numbers)
3. **Missing NULL handling** for incomplete data
4. **Poor indexing** for OR conditions (may need multiple indexes)

### 🔍 **Performance Considerations**
- OR conditions can be challenging for query optimizers
- Consider separate indexes on both area and population
- Monitor query execution plans for large datasets
- Use UNION if OR performance is poor (though usually less efficient)

### 🎯 **Amazon Leadership Principles Applied**
- **Customer Obsession**: Understanding market sizes for customer reach
- **Data-Driven Decisions**: Using quantitative criteria for market entry
- **Think Big**: Analyzing global markets and opportunities
- **Frugality**: Efficient queries for large-scale geographic analysis

---

## 🔄 Practice Variations

Try solving these variations to master the concept:

1. **Find countries that are big in BOTH area AND population**
2. **Find countries with high GDP but not meeting big country criteria**
3. **Calculate what percentage of countries are considered "big"**
4. **Find the smallest "big" country by each criterion**

Remember: Geographic and demographic analysis is crucial for Amazon's global expansion strategy, logistics planning, and market opportunity assessment!