# Aggregate Functions

## Overview
Aggregate functions in SQL perform a calculation on a set of rows and return a single, summary value. They are essential for data analysis, allowing you to compute statistics like sums, averages, counts, and more from your data. Aggregate functions are most commonly used with the `SELECT` statement and are often paired with the `GROUP BY` clause to calculate metrics for different categories or groups.

## Core Concepts

An aggregate function takes multiple values (typically from a column) and reduces them to a single value.

**Common Aggregate Functions:**
-   **`COUNT()`**: Counts the number of rows.
-   **`SUM()`**: Calculates the sum of a set of values.
-   **`AVG()`**: Calculates the average of a set of values.
-   **`MIN()`**: Finds the minimum value in a set.
-   **`MAX()`**: Finds the maximum value in a set.

**How they work:**
-   **Without `GROUP BY`**: When used without a `GROUP BY` clause, an aggregate function operates on all rows in the result set and returns a single row.
    ```sql
    -- Get the total number of employees
    SELECT COUNT(*) FROM employees;

    -- Get the average salary of all employees
    SELECT AVG(salary) FROM employees;
    ```
-   **With `GROUP BY`**: When used with a `GROUP BY` clause, the function calculates a result for each group.
    ```sql
    -- Get the number of employees in each department
    SELECT department, COUNT(*)
    FROM employees
    GROUP BY department;
    ```

### `NULL` Handling
A critical feature of aggregate functions is how they handle `NULL` values.
-   All aggregate functions **ignore `NULL`s** in their calculations, except for `COUNT(*)`.
-   `COUNT(column_name)` counts non-null values in that column.
-   `COUNT(*)` counts all rows, regardless of `NULL`s.
-   `SUM(column_name)`, `AVG(column_name)`, `MIN(column_name)`, `MAX(column_name)` all compute their result after discarding any `NULL` values.

**Example of `AVG` handling `NULL`:**
If a `salary` column has the values `(100, 200, NULL)`, `AVG(salary)` will be `(100 + 200) / 2 = 150`, not `(100 + 200 + 0) / 3`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What are aggregate functions in SQL?"**
    -   "Aggregate functions perform a calculation on a set of rows and return a single summary value. They are used to compute statistics like counts, sums, and averages. The most common ones are `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX`."

2.  **"How do aggregate functions handle `NULL` values?"**
    -   "This is a key point. All aggregate functions ignore `NULL` values in their input, with the one exception being `COUNT(*)`, which counts all rows. For example, `AVG(column)` will sum the non-null values and divide by the count of non-null values."

3.  **"What is the difference between `COUNT(*)` and `COUNT(column_name)`?"**
    -   "`COUNT(*)` counts every row in the table or group. `COUNT(column_name)` counts only the rows where `column_name` is not `NULL`. If the column has no `NULL`s, their results are the same."

### How to Explain in Interviews
"Aggregate functions are the foundation of data analysis in SQL. I use them to summarize data, for example, calculating the total sales (`SUM`), average order value (`AVG`), or number of unique visitors (`COUNT(DISTINCT user_id)`). I always pair them with the `GROUP BY` clause to break down these metrics by different dimensions like department, region, or time period. Understanding how they handle `NULL`s is crucial for accurate calculations, especially with `AVG` and `COUNT`."

## Quick Recall ✅

-   **Purpose**: Calculate a single summary value from a set of rows.
-   **Key Functions**: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`.
-   **`GROUP BY`**: Use with `GROUP BY` to calculate aggregates for each group.
-   **`NULL` Handling**: All functions ignore `NULL`s, except `COUNT(*)`.
-   **`DISTINCT`**: Can be used inside most aggregate functions (e.g., `COUNT(DISTINCT country)`) to operate only on unique values.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Mixing aggregate and non-aggregate columns without `GROUP BY`.**
    -   `SELECT department, COUNT(*) FROM employees;` ❌
    -   This will cause an error in most databases. If you are selecting a regular column (`department`) alongside an aggregate function (`COUNT(*)`), you **must** include that column in a `GROUP BY` clause.

2.  **Using `WHERE` to filter aggregate results.**
    -   `WHERE` filters rows *before* they are aggregated.
    -   `HAVING` filters groups *after* they are aggregated.
    -   `SELECT department, COUNT(*) FROM employees GROUP BY department WHERE COUNT(*) > 10;` ❌
    -   `SELECT department, COUNT(*) FROM employees GROUP BY department HAVING COUNT(*) > 10;` ✅

3.  **Incorrectly calculating averages.**
    -   Forgetting that `AVG` ignores `NULL`s can lead to misinterpretation of the result. If you want to treat `NULL`s as zero, you must do so explicitly with `COALESCE`: `AVG(COALESCE(score, 0))`.

### Tricky Interview Scenarios

-   **"You have a table of `scores` with values `(10, 20, 30, NULL)`. What is the result of `AVG(scores)`?"**
    -   **Answer**: "The `AVG` function will ignore the `NULL` value. The calculation will be `(10 + 20 + 30) / 3`, which is `20`."

-   **"How would you get the second highest salary from the `employees` table?"**
    -   This is a classic question that can be solved with aggregates.
    -   **Answer**: "A common way is to find the maximum salary among all salaries that are not the overall maximum salary."
    ```sql
    SELECT MAX(salary)
    FROM employees
    WHERE salary < (SELECT MAX(salary) FROM employees);
    ```
    "Alternatively, using window functions like `DENSE_RANK` or `LIMIT`/`OFFSET` is often a more robust solution."

## Bonus

### Advanced Aggregates
-   **`STRING_AGG()` (PostgreSQL, SQL Server) / `GROUP_CONCAT()` (MySQL)**: Concatenates strings from a group into a single string.
-   **`APPROX_COUNT_DISTINCT()`**: In some modern data warehouses (like BigQuery, Snowflake), this provides a very fast but approximate count of distinct values, useful for massive datasets.
-   **Statistical Functions**: Many databases offer more advanced statistical aggregate functions like `STDDEV()` (standard deviation) and `VARIANCE()`.
-   **Window Functions**: While not strictly aggregate functions, window functions (`ROW_NUMBER`, `RANK`, `LEAD`, `LAG`) perform calculations across a set of table rows which are somehow related to the current row. They are often used alongside aggregates.
