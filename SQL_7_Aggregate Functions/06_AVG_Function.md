# AVG() Function

## Overview
The `AVG()` function is an aggregate function in SQL that calculates the average value of a set of non-null numeric values in a column. It is widely used in statistical and data analysis to find the mean of a dataset.

## Core Concepts

The `AVG()` function computes the sum of the values in a column and divides it by the count of non-null values.

**Syntax:**
```sql
SELECT AVG(column_name) FROM table_name;
```

-   **Data Types**: `AVG()` can only be used on columns with numeric data types (e.g., `INT`, `DECIMAL`, `FLOAT`).
-   **`NULL` Handling**: The `AVG()` function **ignores `NULL` values** in its calculation. This is a critical point.

**Example: Calculate the average salary of all employees.**
```sql
-- Table: employees
-- employee_id | salary
-- ------------|-------
-- 1           | 60000
-- 2           | 80000
-- 3           | NULL
-- 4           | 70000

SELECT AVG(salary) FROM employees;
```
**Calculation:**
The function ignores the `NULL` value.
-   `SUM(salary)` = 60000 + 80000 + 70000 = 210000
-   `COUNT(salary)` = 3 (since there are 3 non-null values)
-   `Result` = 210000 / 3 = 70000

### `AVG()` with `GROUP BY`
`AVG()` is commonly used with the `GROUP BY` clause to calculate the average for different categories.

**Example: Calculate the average salary for each department.**
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

### `AVG()` with `DISTINCT`
`AVG()` can be used with the `DISTINCT` keyword to average only the unique values in a column.

**Example: Average of unique salary values.**
If a `salaries` table has values `(50000, 60000, 50000)`, `AVG(DISTINCT salary)` would be `(50000 + 60000) / 2 = 55000`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `AVG()` function do?"**
    -   "`AVG()` is an aggregate function that computes the arithmetic mean (the average) of all non-null values in a numeric column."

2.  **"How does `AVG()` handle `NULL` values? Give an example."**
    -   "`AVG()` completely ignores `NULL` values from its calculation. For a column with values `(10, 20, NULL)`, the average is `(10 + 20) / 2`, which is `15`. It does not treat `NULL` as zero and divide by 3."

3.  **"What is the difference between `AVG(col)` and `SUM(col) / COUNT(*)`?"**
    -   "This is a great question that tests `NULL` handling. `AVG(col)` is equivalent to `SUM(col) / COUNT(col)`, where both `SUM` and `COUNT` ignore `NULL`s. In contrast, `SUM(col) / COUNT(*)` would be incorrect if the column contains `NULL`s, because `COUNT(*)` includes all rows, leading to a smaller, incorrect average. This is a classic data analysis mistake."

### How to Explain in Interviews
"`AVG` is the standard function for calculating the mean of a dataset. I use it with `GROUP BY` to compare average metrics across different segments, like the average purchase value per customer type. The most critical detail I always remember is its `NULL` handling. It ignores `NULL`s, which is usually the desired behavior, but if `NULL`s should be treated as zero, I know to use `AVG(COALESCE(column, 0))` to make that explicit."

## Quick Recall ✅

-   **Purpose**: Calculates the average (mean) of a set of numeric values.
-   **Data Types**: Numeric columns only.
-   **`NULL` Handling**: **Ignores `NULL` values**.
-   **Formula**: `SUM(non-null-values) / COUNT(non-null-values)`.
-   **Empty Set**: Returns `NULL` if there are no rows to average.
-   **`GROUP BY`**: Use to calculate the average for each group.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Misunderstanding `NULL` handling.** This is the most common trap. Assuming `NULL` is treated as `0` will lead to incorrect analysis.

2.  **Integer Division.** In some database systems (like older versions of SQL Server), if you take the average of a column of integers, the result might be truncated to an integer.
    -   `AVG` of `(2, 2, 3)` would be `2`, not `2.33`.
    -   **Fix**: Cast the column to a decimal or float type before averaging: `AVG(CAST(my_column AS DECIMAL(10, 2)))`. Most modern databases handle this more gracefully.

3.  **Assuming `AVG()` returns 0 for an empty set.**
    -   Like `SUM()`, `AVG()` returns `NULL` if its input set is empty. Use `COALESCE(AVG(col), 0)` if you need a zero.

### Tricky Interview Scenarios

-   **"A `scores` column has the values `(100, 50, NULL)`. Your manager wants the average score, assuming anyone with a `NULL` score got a `0`. How do you write the query?"**
    -   **Answer**: "Since `AVG()` ignores `NULL`s by default, I would need to explicitly convert the `NULL` to a `0` using the `COALESCE` function before the aggregation."
    ```sql
    SELECT AVG(COALESCE(score, 0)) FROM test_results;
    ```
    "This would calculate `(100 + 50 + 0) / 3`, which is `50`."

-   **"How can you calculate a moving average?"**
    -   This is an advanced question that requires window functions.
    -   **Answer**: "A standard `AVG()` calculates the average for an entire group. To get a moving average, I would use `AVG()` as a window function, specifying the window frame (e.g., the preceding 7 rows)."
    ```sql
    SELECT
        sale_date,
        daily_revenue,
        AVG(daily_revenue) OVER (ORDER BY sale_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS seven_day_moving_avg
    FROM daily_sales;
    ```

## Bonus

### Related Concepts
-   **`SUM()` and `COUNT()`**: `AVG(col)` is conceptually `SUM(col) / COUNT(col)`.
-   **`COALESCE()`**: Essential for handling `NULL`s before aggregation if they need to be treated as a specific value like zero.
-   **Window Functions**: Using `AVG() OVER (...)` allows for powerful calculations like moving averages without collapsing the rows.
-   **Median**: `AVG()` calculates the mean. The median (the middle value) is another important statistical measure. Some databases (like PostgreSQL and Oracle) provide a `MEDIAN()` or `PERCENTILE_CONT(0.5)` function, but there is no standard SQL way to calculate it.
