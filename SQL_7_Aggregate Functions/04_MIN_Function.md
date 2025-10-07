# MIN() Function

## Overview
The `MIN()` function is an aggregate function in SQL that returns the minimum (lowest) value from a set of values in a column. It can be used on numeric, string, and date data types.

## Core Concepts

The `MIN()` function scans a column and finds the smallest value according to the column's data type and collation.

**Syntax:**
```sql
SELECT MIN(column_name) FROM table_name;
```

-   **Data Types**: `MIN()` works with numbers, strings (alphabetical order), and dates (chronological order).
-   **`NULL` Handling**: The `MIN()` function **ignores `NULL` values**.

**Examples:**

1.  **Numeric Data: Find the lowest salary.**
    ```sql
    SELECT MIN(salary) FROM employees;
    ```

2.  **String Data: Find the first customer name alphabetically.**
    ```sql
    SELECT MIN(customer_name) FROM customers;
    ```
    This would return 'Aaron' if he is in the table.

3.  **Date Data: Find the earliest hire date.**
    ```sql
    SELECT MIN(hire_date) FROM employees;
    ```

### `MIN()` with `GROUP BY`
`MIN()` is very useful with the `GROUP BY` clause to find the minimum value within different categories.

**Example: Find the lowest price for a product in each category.**
```sql
SELECT category, MIN(price) AS lowest_price
FROM products
GROUP BY category;
```
This query returns a row for each category, showing its cheapest product's price.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `MIN()` function do?"**
    -   "`MIN()` is an aggregate function that returns the smallest value from a set of non-null values in a column. It works on numbers, strings, and dates."

2.  **"How does `MIN()` handle `NULL` values?"**
    -   "`MIN()` ignores them. If a column contains `(10, 2, NULL, 5)`, `MIN()` will return `2`."

3.  **"Can you use `MIN()` on a string column? What does it do?"**
    -   "Yes. When used on a string column, `MIN()` returns the first value in alphabetical order based on the database's collation. For example, `MIN(name)` would return the name that comes first alphabetically."

### How to Explain in Interviews
"`MIN` is a straightforward aggregate function for finding the lowest value in a dataset. I use it to find things like the earliest date, the lowest price, or the first item alphabetically. When combined with `GROUP BY`, it's great for summary reports, like finding the date of the first purchase for every customer."

## Quick Recall ✅

-   **Purpose**: Returns the minimum value in a set.
-   **Data Types**: Works on numbers, strings, and dates.
-   **`NULL` Handling**: Ignores `NULL` values.
-   **Empty Set**: Returns `NULL` if there are no rows to evaluate.
-   **`GROUP BY`**: Use to find the minimum value within each group.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `MIN()` returns something for an empty set.**
    -   Like other aggregate functions, if the query's `WHERE` clause filters out all rows, `MIN()` will return `NULL`, not an error or zero.
    -   `SELECT MIN(price) FROM products WHERE category = 'NonExistentCategory';` -> `NULL`

2.  **Forgetting to use `GROUP BY`.**
    -   `SELECT department, MIN(salary) FROM employees;` ❌
    -   This is invalid because it mixes a non-aggregate column with an aggregate function. You must group by the non-aggregate column.

3.  **String sorting confusion.**
    -   The behavior of `MIN()` on strings depends on the database's collation (rules for sorting characters). '10' is often less than '2' in string sorting because it starts with the character '1'. This can be a trap if you are using `MIN()` on a numeric column that is incorrectly stored as a string type.

### Tricky Interview Scenarios

-   **"How do you find the row(s) associated with the minimum value? For example, how do you find the name of the employee with the lowest salary?"**
    -   This is a classic question. You cannot simply write `SELECT employee_name, MIN(salary) FROM employees;`.
    -   **Answer**: "You need to use a subquery. First, you find what the minimum salary is, and then you select the employee(s) who have that salary."
    ```sql
    SELECT employee_name, salary
    FROM employees
    WHERE salary = (SELECT MIN(salary) FROM employees);
    ```
    "This correctly handles the case where multiple employees might share the same lowest salary."

-   **"What happens if you use `MIN()` on a column that has both positive and negative numbers?"**
    -   **Answer**: "`MIN()` works correctly with negative numbers. It will return the number with the lowest value, so if the values are `(-10, 0, 10)`, `MIN()` will return `-10`."

## Bonus

### Related Concepts
-   **`MAX()`**: The direct counterpart to `MIN()`, which finds the highest value.
-   **`LEAST()` / `GREATEST()`**: These are **not** aggregate functions. They are scalar functions that take a list of values *in the same row* and return the smallest or largest.
    -   `SELECT LEAST(col1, col2, col3) FROM my_table;` -> finds the min value across columns for each row.
    -   `SELECT MIN(col1) FROM my_table;` -> finds the min value down a single column for a group of rows.
-   **Window Functions**: `MIN()` can be used as a window function to find the minimum value within a specific "window" of rows, without collapsing the result.
    ```sql
    -- Show each employee's salary along with the minimum salary in their department
    SELECT
        employee_name,
        salary,
        department,
        MIN(salary) OVER (PARTITION BY department) AS min_dept_salary
    FROM employees;
    ```
