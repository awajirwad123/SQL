# DISTINCT Clause

## Overview
The `DISTINCT` keyword is used in a `SELECT` statement to eliminate all duplicate rows from the result set and return only unique rows. It operates on the entire row, not just a single column.

## Core Concepts

When `DISTINCT` is specified, the database looks at the combination of all values in each row of the result set. If a combination of values is identical to one that has already been processed, it is discarded.

**Syntax:**
```sql
SELECT DISTINCT column1, column2, ...
FROM table_name;
```

**Example: Get the list of unique countries where customers are from**
```sql
-- Table: customers
-- customer_id | name  | country
-- ----------------------------
-- 1           | John  | USA
-- 2           | David | UK
-- 3           | Maria | USA

SELECT DISTINCT country
FROM customers;

-- Result:
-- country
-- -------
-- USA
-- UK
```

**`DISTINCT` on Multiple Columns:**
When used with multiple columns, `DISTINCT` returns the unique *combinations* of those columns.

```sql
-- Table: orders
-- order_id | customer_id | product_category
-- -----------------------------------------
-- 101      | 1           | Electronics
-- 102      | 2           | Books
-- 103      | 1           | Books
-- 104      | 1           | Electronics

SELECT DISTINCT customer_id, product_category
FROM orders;

-- Result:
-- customer_id | product_category
-- -----------------------------
-- 1           | Electronics
-- 2           | Books
-- 1           | Books
```
The combination `(1, Electronics)` is returned only once, even though it appears twice in the source table.

### `COUNT(DISTINCT ...)`
`DISTINCT` can also be used inside an aggregate function, most commonly `COUNT`, to count the number of unique non-null values in a column.

**Example: Count the number of unique customers who made a purchase**
```sql
SELECT COUNT(DISTINCT customer_id) AS unique_customers
FROM orders;
```
This is different from `COUNT(customer_id)`, which would count all orders, including multiple orders from the same customer.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `DISTINCT` keyword do?"**
    -   "`DISTINCT` is used with a `SELECT` statement to remove duplicate rows from the result set. It ensures that every row returned is unique."

2.  **"How does `DISTINCT` work with multiple columns?"**
    -   "When you use `DISTINCT` with multiple columns, it returns the unique combinations of values across those columns. It doesn't look at each column in isolation; it looks at the entire row."

3.  **"What is the difference between `COUNT(column)` and `COUNT(DISTINCT column)`?"**
    -   "`COUNT(column)` counts all non-null values in that column. `COUNT(DISTINCT column)` counts only the unique non-null values in that column."

4.  **"Is `DISTINCT` a function?"**
    -   "No, it's a keyword or a clause modifier. It applies to the entire `SELECT` statement's result set, not just a single column (unless it's the only column selected). The syntax `SELECT DISTINCT(col)` is misleading; it's the same as `SELECT DISTINCT col`."

### How to Explain in Interviews
"`DISTINCT` is a simple but important tool for data exploration and cleaning. I use it to get a quick overview of the unique values in a column, like finding all the unique product categories we sell. I also frequently use `COUNT(DISTINCT ...)` for analytics, for example, to count the number of unique daily active users. It's important to remember that `DISTINCT` can have a performance cost, as the database needs to sort or hash the data to find the duplicates."

## Quick Recall ✅

-   **Purpose**: Return only unique rows.
-   **Scope**: Applies to the entire row, not just one column.
-   **Multi-column**: Returns unique combinations of the selected columns.
-   **`COUNT(DISTINCT ...)`**: Counts unique non-null values.
-   **Performance**: Can be slow on large datasets as it requires a sort or hash operation.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Thinking `DISTINCT` applies only to the first column.**
    ```sql
    -- A common misconception
    SELECT DISTINCT customer_id, order_date FROM orders;
    ```
    This does **not** return one row for each distinct `customer_id`. It returns one row for each distinct `(customer_id, order_date)` pair.

2.  **Using `DISTINCT` when `GROUP BY` is more appropriate.**
    -   `SELECT DISTINCT department FROM employees;`
    -   `SELECT department FROM employees GROUP BY department;`
    -   Both queries return the same list of unique departments. However, if you intend to do any aggregation (like counting employees in each department), you should use `GROUP BY` from the start. Use `DISTINCT` for simple uniqueness checks.

3.  **Expecting a specific order.**
    -   `DISTINCT` does not guarantee any order in the final result. If you need the results sorted, you must add an `ORDER BY` clause.
    ```sql
    SELECT DISTINCT country FROM customers ORDER BY country;
    ```

### Tricky Interview Scenarios

-   **"How does `DISTINCT` affect performance?"**
    -   **Answer**: "`DISTINCT` requires the database to find and remove duplicate rows. To do this, it typically performs a sort or a hash operation on the result set, which can be memory and CPU intensive, especially with large amounts of data. If the result set is large, a query with `DISTINCT` will be significantly slower than one without it."

-   **"Your query `SELECT DISTINCT a, b, c FROM my_table` is slow. How might you optimize it?"**
    -   **Answer**: "First, I'd check if `DISTINCT` is truly necessary or if the logic can be rewritten. If it is, an index on the columns `(a, b, c)` can sometimes help. An index stores the data in a sorted order, so the database might be able to use the index to retrieve the unique rows more efficiently without a separate sort step. Another option is to see if `GROUP BY a, b, c` performs better, as some database optimizers can handle it more efficiently."

## Bonus

### Related Concepts
-   **`GROUP BY` Clause**: `GROUP BY` provides a more powerful way to handle unique data because it allows for aggregation on each unique group. `SELECT DISTINCT a, b` is semantically equivalent to `SELECT a, b GROUP BY a, b`.
-   **`UNION` vs. `UNION ALL`**: The `UNION` operator combines the result sets of two queries and, by default, behaves like `DISTINCT`, removing duplicate rows. `UNION ALL` is much faster because it includes all rows, including duplicates.
-   **Window Functions**: You can achieve the same result as `DISTINCT` using window functions, which can be more flexible.
    ```sql
    -- Get distinct rows using ROW_NUMBER()
    WITH NumberedRows AS (
        SELECT
            col1, col2,
            ROW_NUMBER() OVER(PARTITION BY col1, col2 ORDER BY col1) as rn
        FROM my_table
    )
    SELECT col1, col2 FROM NumberedRows WHERE rn = 1;
    ```
    This is more verbose for a simple distinct operation but can be extended for more complex "get the first/best row per group" scenarios.
