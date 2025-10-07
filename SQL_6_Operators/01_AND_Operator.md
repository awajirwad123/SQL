# AND Operator

## Overview
The `AND` operator is a logical operator in SQL used within a `WHERE` or `HAVING` clause to filter records based on more than one condition. It combines multiple boolean expressions and returns `TRUE` only if **all** of the individual expressions are `TRUE`.

## Core Concepts

The `AND` operator is used to create a more specific filter by ensuring that a row must meet several criteria simultaneously to be included in the result set.

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE condition1 AND condition2 AND condition3 ...;
```

**How it works:**
For a row to be selected, the entire `WHERE` clause must evaluate to `TRUE`.
-   If `condition1` is `TRUE` **and** `condition2` is `TRUE`, the combined result is `TRUE`.
-   If `condition1` is `TRUE` **and** `condition2` is `FALSE`, the combined result is `FALSE`.
-   If `condition1` is `FALSE` **and** `condition2` is `TRUE`, the combined result is `FALSE`.
-   If `condition1` is `FALSE` **and** `condition2` is `FALSE`, the combined result is `FALSE`.

**Example: Find all electronics that are in stock and cost more than $500.**
```sql
SELECT product_name, price
FROM products
WHERE category = 'Electronics'
  AND in_stock = TRUE
  AND price > 500;
```
A product will only be listed if it meets all three of these conditions.

### Operator Precedence
In SQL, `AND` has a higher precedence than `OR`. This means that `AND` conditions are evaluated before `OR` conditions. It's a common source of bugs, so it's best practice to use parentheses `()` to make the order of evaluation explicit and clear.

**Example of precedence:**
```sql
-- Find products that are 'Electronics' AND 'in_stock', OR are 'Books'
WHERE category = 'Electronics' AND in_stock = TRUE OR category = 'Books'
```
Due to precedence, this is evaluated as:
`WHERE (category = 'Electronics' AND in_stock = TRUE) OR category = 'Books'`

If you meant to find electronics or books that are in stock, you must use parentheses:
```sql
WHERE (category = 'Electronics' OR category = 'Books') AND in_stock = TRUE
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `AND` operator do in SQL?"**
    -   "The `AND` operator is a logical operator used in a `WHERE` or `HAVING` clause to combine multiple conditions. It ensures that a row is returned only if all the specified conditions are met."

2.  **"What is the operator precedence between `AND` and `OR`?"**
    -   "`AND` has a higher precedence than `OR`. This means `AND` operations are performed before `OR` operations in a complex `WHERE` clause. To avoid ambiguity and bugs, I always use parentheses to explicitly define the order of evaluation."

3.  **"Provide an example of a query using `AND`."**
    -   "Sure. To find all employees in the 'Sales' department who were hired after January 1st, 2020, I would write: `SELECT * FROM employees WHERE department = 'Sales' AND hire_date > '2020-01-01';`"

### How to Explain in Interviews
"The `AND` operator is fundamental for creating precise filters in SQL. I think of it as making a filter progressively narrower. Each `AND` condition I add must also be true for a row to be included. I'm also very careful about its interaction with the `OR` operator; I make it a habit to use parentheses to group my conditions logically, which makes the query's intent clear and prevents precedence-related bugs."

## Quick Recall ✅

-   **Purpose**: Combines multiple conditions.
-   **Rule**: Returns `TRUE` only if **all** conditions are `TRUE`.
-   **Precedence**: Higher than `OR`.
-   **Best Practice**: Use parentheses `()` to clarify logic when mixing `AND` and `OR`.
-   **Used in**: `WHERE` and `HAVING` clauses.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting Parentheses:** This is the most common error when mixing `AND` and `OR`. The query runs but returns an incorrect result because the default precedence was not what the developer intended.

2.  **`NULL` Handling:** The interaction with `NULL` can be tricky.
    -   `TRUE AND NULL` evaluates to `NULL`.
    -   `FALSE AND NULL` evaluates to `FALSE`.
    -   A `WHERE` clause only accepts rows where the final evaluation is `TRUE`. Rows that evaluate to `FALSE` or `NULL` are rejected.

**Example:**
`WHERE price > 100 AND color = NULL` will never return any rows, because `color = NULL` is always `NULL`, and `TRUE AND NULL` is `NULL`. The correct way to check for `NULL` is `WHERE price > 100 AND color IS NULL`.

### Tricky Interview Scenarios

-   **"Write a query to find users who signed up in 2021 and live in either 'California' or 'Nevada'."**
    -   This tests the combination of `AND` and `OR` with parentheses.
    ```sql
    SELECT user_id, user_name, state
    FROM users
    WHERE registration_date BETWEEN '2021-01-01' AND '2021-12-31'
      AND (state = 'California' OR state = 'Nevada');
    ```
    The parentheses around the `OR` condition are crucial.

## Bonus

### Related Concepts
-   **`OR` Operator**: The logical counterpart to `AND`. Returns `TRUE` if *any* condition is `TRUE`.
-   **`NOT` Operator**: Negates a condition. `WHERE NOT (country = 'USA')`.
-   **Boolean Logic**: The foundation of how `AND`, `OR`, and `NOT` work, based on truth tables.
-   **Short-Circuit Evaluation**: Some database engines use short-circuiting. In `condition1 AND condition2`, if `condition1` is `FALSE`, the engine won't even bother to evaluate `condition2` because the overall result must be `FALSE`. This can have performance implications.
