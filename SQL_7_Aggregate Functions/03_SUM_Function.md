# SUM() Function

## Overview
The `SUM()` function is an aggregate function in SQL that calculates the sum of all non-null numeric values in a column. It is a fundamental function for financial, sales, and inventory analysis.

## Core Concepts

The `SUM()` function operates on a set of rows (either the entire table or groups defined by `GROUP BY`) and returns a single value representing the total.

**Syntax:**
```sql
SELECT SUM(column_name) FROM table_name;
```

-   **Data Types**: `SUM()` can only be used on columns with numeric data types (e.g., `INT`, `DECIMAL`, `FLOAT`, `NUMERIC`).
-   **`NULL` Handling**: The `SUM()` function **ignores `NULL` values** in its calculation.

**Example: Calculate the total sales from all orders.**
```sql
-- Table: orders
-- order_id | amount
-- ---------|-------
-- 1        | 100.50
-- 2        | 75.00
-- 3        | NULL
-- 4        | 25.00

SELECT SUM(amount) FROM orders;
-- Result: 100.50 + 75.00 + 25.00 = 200.50
```
The `NULL` value is completely ignored.

### `SUM()` with `GROUP BY`
`SUM()` is most powerful when used with the `GROUP BY` clause to calculate subtotals for different categories.

**Example: Calculate the total sales for each product category.**
```sql
SELECT category, SUM(sales_amount) AS total_sales
FROM sales
GROUP BY category;
```
This query would return a row for each category with the sum of its sales.

### `SUM()` with `DISTINCT`
While less common, `SUM()` can be used with the `DISTINCT` keyword to sum only the unique values in a column.

**Example: Sum of unique salary values.**
If a `salaries` table has values `(50000, 60000, 50000)`, `SUM(DISTINCT salary)` would be `50000 + 60000 = 110000`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `SUM()` function do?"**
    -   "`SUM()` is an aggregate function that computes the total of all non-null values in a numeric column. It's essential for calculating totals like revenue, inventory counts, or scores."

2.  **"How does `SUM()` handle `NULL` values?"**
    -   "`SUM()` ignores them. If you are summing a column with values `(10, 20, NULL)`, the result is `30`. It does not treat `NULL` as zero."

3.  **"What happens if you use `SUM()` on a table with no rows?"**
    -   "If the query returns no rows to aggregate, `SUM()` returns `NULL`, not zero. This is an important distinction. For example, `SELECT SUM(sales) FROM orders WHERE 1=0;` will result in `NULL`."

### How to Explain in Interviews
"`SUM` is the primary function for calculating totals in SQL. I use it frequently with `GROUP BY` to generate summary reports, like total sales per region or total hours worked per project. I'm careful to remember that it ignores `NULL`s and returns `NULL` (not zero) for an empty set, and I use `COALESCE` to handle these cases if a zero is required for reporting or further calculations."

## Quick Recall ✅

-   **Purpose**: Calculates the total of a set of numeric values.
-   **Data Types**: Numeric columns only.
-   **`NULL` Handling**: Ignores `NULL` values.
-   **Empty Set**: Returns `NULL` if there are no rows to sum.
-   **`GROUP BY`**: Use to calculate subtotals for groups.
-   **`DISTINCT`**: Can be used (`SUM(DISTINCT ...)`), but it's less common.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `SUM()` returns 0 for an empty set.**
    -   This is a frequent bug. If a query with a `SUM()` has a `WHERE` clause that results in no rows being selected, the result is `NULL`.
    -   **Fix**: Use `COALESCE` to convert a `NULL` result to `0`.
    ```sql
    SELECT COALESCE(SUM(sales), 0) FROM daily_sales WHERE sale_date = '2025-10-07';
    ```

2.  **Using `SUM()` on non-numeric data.**
    -   This will throw an error in most database systems.

3.  **Forgetting to use `GROUP BY`.**
    -   `SELECT department, SUM(salary) FROM employees;` ❌
    -   This is invalid because it mixes a non-aggregate column (`department`) with an aggregate function (`SUM`). You must group by the non-aggregate column.

### Tricky Interview Scenarios

-   **"You need to calculate the total value of all items in an inventory, but some items have a `NULL` quantity. How do you ensure these are treated as zero?"**
    -   **Answer**: "The `SUM` function naturally ignores `NULL`s, so if you just want to sum the items with known quantities, `SUM(quantity)` is fine. If the business rule is to treat a `NULL` quantity as `0`, I would use `SUM(COALESCE(quantity, 0))`. This ensures that `NULL`s are explicitly converted to zero before the summation."

-   **"How can you calculate a cumulative sum (or running total)?"**
    -   This is an advanced question that requires window functions.
    -   **Answer**: "A standard `SUM()` calculates a total for the entire group. To get a running total, I would use `SUM()` as a window function with an `OVER` clause."
    ```sql
    SELECT
        sale_date,
        daily_revenue,
        SUM(daily_revenue) OVER (ORDER BY sale_date) AS running_total_revenue
    FROM daily_sales;
    ```

## Bonus

### Related Concepts
-   **`AVG()`**: The average function, which can be thought of as `SUM(col) / COUNT(col)`.
-   **`COALESCE()`**: A crucial function for handling `NULL` results from `SUM()` on empty sets, converting them to `0`.
-   **Window Functions**: Using `SUM() OVER (...)` allows for powerful calculations like running totals and moving averages without collapsing the rows.
-   **Floating-Point Inaccuracy**: When summing `FLOAT` or `REAL` data types, be aware of potential for small precision errors. For financial calculations, always use `DECIMAL` or `NUMERIC` data types.
