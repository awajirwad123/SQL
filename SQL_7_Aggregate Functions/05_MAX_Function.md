# MAX() Function

## Overview
The `MAX()` function is an aggregate function in SQL that returns the maximum (highest) value from a set of values in a column. It is the direct opposite of the `MIN()` function and can be used on numeric, string, and date data types.

## Core Concepts

The `MAX()` function scans a column and finds the largest value according to the column's data type and collation.

**Syntax:**
```sql
SELECT MAX(column_name) FROM table_name;
```

-   **Data Types**: `MAX()` works with numbers, strings (alphabetical order), and dates (chronological order).
-   **`NULL` Handling**: The `MAX()` function **ignores `NULL` values**.

**Examples:**

1.  **Numeric Data: Find the highest salary.**
    ```sql
    SELECT MAX(salary) FROM employees;
    ```

2.  **String Data: Find the last customer name alphabetically.**
    ```sql
    SELECT MAX(customer_name) FROM customers;
    ```
    This would return 'Zoe' if she is in the table.

3.  **Date Data: Find the most recent hire date.**
    ```sql
    SELECT MAX(hire_date) FROM employees;
    ```

### `MAX()` with `GROUP BY`
`MAX()` is very useful with the `GROUP BY` clause to find the maximum value within different categories.

**Example: Find the highest price for a product in each category.**
```sql
SELECT category, MAX(price) AS highest_price
FROM products
GROUP BY category;
```
This query returns a row for each category, showing its most expensive product's price.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `MAX()` function do?"**
    -   "`MAX()` is an aggregate function that returns the largest value from a set of non-null values in a column. It works on numbers, strings, and dates."

2.  **"How does `MAX()` handle `NULL` values?"**
    -   "`MAX()` ignores them. If a column contains `(10, 100, NULL, 50)`, `MAX()` will return `100`."

3.  **"Can you use `MAX()` on a string column? What does it do?"**
    -   "Yes. When used on a string column, `MAX()` returns the last value in alphabetical order based on the database's collation. For example, `MAX(name)` would return the name that comes last alphabetically."

### How to Explain in Interviews
"`MAX` is a fundamental aggregate function for finding the highest value. I use it to find things like the most recent date, the highest price, or the last item alphabetically. When combined with `GROUP BY`, it's essential for summary reports, like finding the date of the most recent purchase for every customer, which is useful for recency analysis."

## Quick Recall ✅

-   **Purpose**: Returns the maximum value in a set.
-   **Data Types**: Works on numbers, strings, and dates.
-   **`NULL` Handling**: Ignores `NULL` values.
-   **Empty Set**: Returns `NULL` if there are no rows to evaluate.
-   **`GROUP BY`**: Use to find the maximum value within each group.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `MAX()` returns something for an empty set.**
    -   Like other aggregate functions, if a query's `WHERE` clause filters out all rows, `MAX()` will return `NULL`.
    -   `SELECT MAX(price) FROM products WHERE category = 'NonExistentCategory';` -> `NULL`

2.  **Forgetting to use `GROUP BY`.**
    -   `SELECT department, MAX(salary) FROM employees;` ❌
    -   This is invalid because it mixes a non-aggregate column with an aggregate function. You must group by the non-aggregate column.

3.  **String sorting confusion.**
    -   The behavior of `MAX()` on strings depends on the database's collation. '2' is often greater than '10' in string sorting. This can be a trap if you are using `MAX()` on a numeric column that is incorrectly stored as a string type.

### Tricky Interview Scenarios

-   **"How do you find the row(s) associated with the maximum value? For example, how do you find the name of the employee with the highest salary?"**
    -   This is a classic question. You cannot simply write `SELECT employee_name, MAX(salary) FROM employees;`.
    -   **Answer**: "You need to use a subquery. First, you find what the maximum salary is, and then you select the employee(s) who have that salary."
    ```sql
    SELECT employee_name, salary
    FROM employees
    WHERE salary = (SELECT MAX(salary) FROM employees);
    ```
    "This is a common and effective pattern. Another way is to sort the results in descending order and take the top row, using `ORDER BY salary DESC LIMIT 1`."

-   **"How do you find the Nth highest value, for example, the 2nd highest salary?"**
    -   **Answer**: "A common method is to use a subquery to find the max salary that is less than the overall max salary: `SELECT MAX(salary) FROM employees WHERE salary < (SELECT MAX(salary) FROM employees);`. However, a more robust method for this problem is to use window functions like `DENSE_RANK` or `LIMIT`/`OFFSET`."

## Bonus

### Related Concepts
-   **`MIN()`**: The direct counterpart to `MAX()`, which finds the lowest value.
-   **`GREATEST()` / `LEAST()`**: These are **not** aggregate functions. They are scalar functions that take a list of values *in the same row* and return the largest or smallest.
    -   `SELECT GREATEST(col1, col2, col3) FROM my_table;` -> finds the max value across columns for each row.
    -   `SELECT MAX(col1) FROM my_table;` -> finds the max value down a single column for a group of rows.
-   **Window Functions**: `MAX()` can be used as a window function to find the maximum value within a specific "window" of rows, without collapsing the result.
    ```sql
    -- Show each employee's salary along with the maximum salary in their department
    SELECT
        employee_name,
        salary,
        department,
        MAX(salary) OVER (PARTITION BY department) AS max_dept_salary
    FROM employees;
    ```
