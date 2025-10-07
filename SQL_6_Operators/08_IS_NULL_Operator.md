# IS NULL Operator

## Overview
The `IS NULL` operator is a special operator in SQL used to check if a column's value is `NULL`. It is the **only** correct way to test for `NULL`, as `NULL` represents an unknown or missing value and cannot be compared using standard comparison operators like `=` or `<>`.

## Core Concepts

`NULL` is not a value; it is a marker for the absence of a value. Therefore, it cannot be equal or not equal to anything, including itself.
-   `some_column = NULL` will always evaluate to `UNKNOWN` (or `NULL`), not `TRUE`.
-   `some_column <> NULL` will also always evaluate to `UNKNOWN`.
-   `NULL = NULL` also evaluates to `UNKNOWN`.

The `IS NULL` operator is specifically designed to handle this.

**Syntax:**
```sql
-- To find rows where a value is missing
SELECT column_list
FROM table_name
WHERE column_name IS NULL;

-- To find rows where a value is present
SELECT column_list
FROM table_name
WHERE column_name IS NOT NULL;
```

**Example: Find all customers who have not yet been assigned a sales representative.**
```sql
SELECT customer_name
FROM customers
WHERE sales_rep_id IS NULL;
```

**Example: Find all products that have a description.**
```sql
SELECT product_name
FROM products
WHERE description IS NOT NULL;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you check for `NULL` values in SQL?"**
    -   "You must use the `IS NULL` operator. For example, `WHERE my_column IS NULL`. You cannot use `WHERE my_column = NULL`, as any comparison with `NULL` using standard operators results in `UNKNOWN`."

2.  **"Why doesn't `column = NULL` work?"**
    -   "`NULL` represents a missing or unknown state, not a specific value. Because its value is unknown, it can't be said to be equal (or not equal) to anything. The `IS NULL` operator is specifically designed as the correct syntax for this check."

3.  **"What is the difference between `IS NULL` and `IS NOT NULL`?"**
    -   "`IS NULL` checks for the presence of a `NULL` marker, meaning the value is missing. `IS NOT NULL` checks for the absence of a `NULL` marker, meaning a value of some kind exists in the column."

### How to Explain in Interviews
"Checking for `NULL` is a fundamental concept in SQL that hinges on understanding that `NULL` isn't a value. I always use `IS NULL` or `IS NOT NULL` for this check. Using `= NULL` is a classic beginner's mistake that I know to avoid. This concept is crucial for writing correct filters, especially in `LEFT JOIN`s, where `IS NULL` is used to find rows in one table that don't have a match in another."

## Quick Recall ✅

-   **Purpose**: Test if a value is `NULL`.
-   **Correct Syntax**: `IS NULL`.
-   **Incorrect Syntax**: `= NULL` or `<> NULL`.
-   **Negation**: `IS NOT NULL`.
-   **Reason**: `NULL` represents an unknown state and cannot be compared with standard operators.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Using `=` or `<>` with `NULL`:** This is the most common error. The query will run but will not return the expected rows because the `WHERE` clause condition never evaluates to `TRUE`.

2.  **Confusing `NULL` with an empty string or zero.**
    -   `NULL` is the absence of a value.
    -   An empty string (`''`) is a valid string value of zero length.
    -   `0` is a valid numeric value.
    -   `WHERE my_column IS NULL` will **not** match rows where `my_column` is `''` or `0`.

### Tricky Interview Scenarios

-   **"A `LEFT JOIN` is returning more rows than you expect. How might `NULL`s be involved?"**
    -   **Answer**: "This can happen if the right table has multiple `NULL`s in the join key. If the left table's join key is also `NULL`, a standard join `ON a.key = b.key` won't match them. However, if there are other conditions, it's important to remember that `NULL`s in data can have unexpected effects on joins and filters." (This is a complex topic, but acknowledging `NULL`'s role is key).

-   **"How do aggregate functions like `COUNT` handle `NULL`?"**
    -   **Answer**: "This is an important distinction. `COUNT(*)` counts all rows. `COUNT(column_name)` counts all rows where `column_name` **is not null**. Other aggregate functions like `SUM`, `AVG`, `MIN`, and `MAX` also ignore `NULL` values in their calculations."

-   **"How can you treat `NULL` as a specific value, for example, 'N/A', in a query result?"**
    -   **Answer**: "I would use the `COALESCE` function. `COALESCE(my_column, 'N/A')` will return the value of `my_column` if it's not `NULL`, otherwise it will return the string 'N/A'. This is very useful for presentation purposes." Other similar functions include `ISNULL` (SQL Server) and `IFNULL` (MySQL).

## Bonus

### Related Concepts
-   **Three-Valued Logic**: The `TRUE`/`FALSE`/`UNKNOWN` system that governs all logical operations involving `NULL`.
-   **`COALESCE` function**: Returns the first non-null value in a list. `COALESCE(col1, col2, 'default')`.
-   **`LEFT JOIN ... WHERE key IS NULL`**: A very common and efficient pattern for finding records that exist in one table but not another.
    ```sql
    -- Find customers who have never placed an order
    SELECT c.*
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    WHERE o.order_id IS NULL;
    ```
-   **ANSI `NULL` Settings**: Some database systems have session-level settings (like `ANSI_NULLS` in SQL Server) that can change the behavior of `NULL` comparisons, but relying on these non-standard settings is extremely bad practice. The standard behavior (`= NULL` is `UNKNOWN`) should always be assumed.
