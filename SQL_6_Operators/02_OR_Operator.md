# OR Operator

## Overview
The `OR` operator is a logical operator in SQL used within a `WHERE` or `HAVING` clause to filter records based on more than one condition. It combines multiple boolean expressions and returns `TRUE` if **any** of the individual expressions are `TRUE`.

## Core Concepts

The `OR` operator is used to create a broader filter, allowing a row to be included if it meets at least one of several criteria.

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE condition1 OR condition2 OR condition3 ...;
```

**How it works:**
For a row to be selected, the entire `WHERE` clause must evaluate to `TRUE`.
-   If `condition1` is `TRUE` **or** `condition2` is `TRUE`, the combined result is `TRUE`.
-   If `condition1` is `TRUE` **or** `condition2` is `FALSE`, the combined result is `TRUE`.
-   If `condition1` is `FALSE` **or** `condition2` is `TRUE`, the combined result is `TRUE`.
-   If `condition1` is `FALSE` **or** `condition2` is `FALSE`, the combined result is `FALSE`.

**Example: Find all products that are in the 'Electronics' or 'Appliances' category.**
```sql
SELECT product_name, category
FROM products
WHERE category = 'Electronics' OR category = 'Appliances';
```
A product will be listed if its category is 'Electronics' or if its category is 'Appliances'. This is a common use case that can also be written more concisely with the `IN` operator: `WHERE category IN ('Electronics', 'Appliances')`.

### Operator Precedence
In SQL, `AND` has a higher precedence than `OR`. When you mix them, `AND` conditions are evaluated first. This is a frequent source of bugs. Always use parentheses `()` to make the order of evaluation explicit.

**Example of precedence:**
```sql
-- Find products that are 'Books', OR are 'Electronics' that are also 'on_sale'
WHERE category = 'Books' OR category = 'Electronics' AND on_sale = TRUE
```
Due to precedence, this is evaluated as:
`WHERE category = 'Books' OR (category = 'Electronics' AND on_sale = TRUE)`

If you meant to find books or electronics that are on sale, you must use parentheses:
```sql
WHERE (category = 'Books' OR category = 'Electronics') AND on_sale = TRUE
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `OR` operator do in SQL?"**
    -   "The `OR` operator is a logical operator used in a `WHERE` or `HAVING` clause to combine multiple conditions. It allows a row to be returned if it meets at least one of the specified conditions."

2.  **"When would you use `OR` instead of `IN`?"**
    -   "`OR` and `IN` can often be used for the same purpose. `WHERE category = 'A' OR category = 'B'` is the same as `WHERE category IN ('A', 'B')`. The `IN` operator is generally more concise and readable when you have a list of values to check against a single column. I would use `OR` when the conditions involve different columns, for example, `WHERE category = 'Books' OR price < 10`."

3.  **"Explain the precedence of `AND` and `OR`."**
    -   "`AND` is evaluated before `OR`. Because this can lead to unexpected results, I always use parentheses to group my conditions explicitly whenever I mix `AND` and `OR` in the same `WHERE` clause."

### How to Explain in Interviews
"The `OR` operator is used to widen a search filter. While an `AND` condition narrows the results, an `OR` condition expands them by providing alternative criteria for a match. I often use it for simple alternative checks, but for checking a single column against a list of values, I prefer the `IN` operator for readability. The most critical thing to remember when using `OR` is to use parentheses when combining it with `AND` to ensure the logic is evaluated correctly."

## Quick Recall ✅

-   **Purpose**: Combines multiple conditions.
-   **Rule**: Returns `TRUE` if **any** condition is `TRUE`.
-   **Precedence**: Lower than `AND`.
-   **Best Practice**: Use parentheses `()` to clarify logic when mixing `AND` and `OR`.
-   **Alternative**: For checking one column against multiple values, `IN` is often better.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting Parentheses:** This is the number one mistake. The query runs without error but produces the wrong data because the default `AND`-before-`OR` precedence was not what the developer intended.

2.  **`NULL` Handling:** The interaction with `NULL` can be non-obvious.
    -   `TRUE OR NULL` evaluates to `TRUE`.
    -   `FALSE OR NULL` evaluates to `NULL`.
    -   A `WHERE` clause only accepts rows where the final evaluation is `TRUE`. Rows that evaluate to `FALSE` or `NULL` are rejected.

**Example:**
`WHERE price > 100 OR color IS NULL` will return rows where the price is over 100, and also rows where the color is `NULL` (regardless of price).

### Tricky Interview Scenarios

-   **"Write a query to find users who either live in 'California' or have made a purchase over $1000."**
    -   This tests the use of `OR` with conditions on different columns/tables.
    ```sql
    SELECT DISTINCT u.user_id, u.user_name, u.state
    FROM users AS u
    LEFT JOIN orders AS o ON u.user_id = o.user_id
    WHERE u.state = 'California' OR o.purchase_amount > 1000;
    ```
    The `DISTINCT` is important because a user from California with a large purchase would otherwise appear multiple times.

## Bonus

### Related Concepts
-   **`AND` Operator**: The logical counterpart to `OR`.
-   **`IN` Operator**: Often a more readable shorthand for a series of `OR` conditions on the same column.
-   **Boolean Logic**: The foundation of how `AND`, `OR`, and `NOT` work.
-   **Short-Circuit Evaluation**: Some database engines use short-circuiting. In `condition1 OR condition2`, if `condition1` is `TRUE`, the engine won't evaluate `condition2` because the overall result must be `TRUE`. This can be a micro-optimization.
