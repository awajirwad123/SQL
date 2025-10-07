# NOT EQUAL Operator

## Overview
The `NOT EQUAL` operator is a comparison operator used in a `WHERE` clause to filter for rows where a column's value does not match a specified value. Most database systems support two syntaxes for this operator: `<>` and `!=`.

## Core Concepts

The `NOT EQUAL` operator is straightforward: it returns `TRUE` if the two values being compared are not the same, and `FALSE` if they are.

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE column_name <> value;
```
or
```sql
SELECT column_list
FROM table_name
WHERE column_name != value;
```

**Example: Find all customers who are not from the USA**
```sql
SELECT customer_name, country
FROM customers
WHERE country <> 'USA';
```
This is functionally identical to:
```sql
SELECT customer_name, country
FROM customers
WHERE country != 'USA';
```

### `<>` vs. `!=`
-   **`<>`**: This is the traditional and **ANSI SQL standard** operator for not equal. It is guaranteed to work on all SQL database systems.
-   **`!=`**: This is a common, C-style alternative supported by most major databases (including PostgreSQL, SQL Server, and Oracle).

Because `<>` is the standard, it is often considered the more "correct" and portable operator to use. However, in practice, both are widely used and understood.

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you test for inequality in SQL?"**
    -   "I use the `NOT EQUAL` operator in the `WHERE` clause. The standard syntax is `<>`, but most databases also support `!=`."

2.  **"Is there a difference between `<>` and `!=`?"**
    -   "Functionally, they are identical in most database systems. However, `<>` is the official ANSI SQL standard, so it's the most portable and technically correct operator to use. `!=` is a common, non-standard alternative."

3.  **"Why does `WHERE my_column <> 'some_value'` not return rows where `my_column` is `NULL`?"**
    -   "This is because any comparison with `NULL`, including `<>`, results in `UNKNOWN` (or `NULL`), not `TRUE`. The `WHERE` clause only includes rows where the condition evaluates to `TRUE`. To include `NULL` rows, you would have to explicitly add `OR my_column IS NULL` to the condition."

### How to Explain in Interviews
"For checking inequality, I use the `<>` operator as it's the ANSI standard and works everywhere. It's a basic comparison operator, just like `=` or `>`. The most important thing to remember when using it is how it interacts with `NULL`s. A check like `column <> 'value'` will filter out `NULL`s, which may or may not be the desired behavior, so you have to be explicit if you want to include them."

## Quick Recall ✅

-   **Purpose**: Checks if two values are not equal.
-   **Standard Syntax**: `<>` (ANSI SQL).
-   **Common Syntax**: `!=`.
-   **Best Practice**: Prefer `<>` for maximum portability.
-   **`NULL` Handling**: `column <> value` will not match `NULL` values.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `NULL`s will be included.** This is the biggest trap. Developers often expect a condition to check all rows that don't match a value, but `NULL` is not a value; it's the absence of one. The comparison `NULL <> 'value'` is not `TRUE`, so the row is excluded.

2.  **Using `=` instead of `IS` for `NULL`.**
    -   `WHERE my_column <> NULL` is incorrect.
    -   `WHERE my_column IS NOT NULL` is correct.

### Tricky Interview Scenarios

-   **"You want to select all users except for 'John'. You write `WHERE name <> 'John'`. A user with a `NULL` name is not returned. Why, and how do you fix it?"**
    -   **Answer**: "The user with a `NULL` name is not returned because the expression `NULL <> 'John'` evaluates to `NULL`, not `TRUE`. To include this user, the query must be changed to `WHERE name <> 'John' OR name IS NULL`."

-   **"Can you use `<>` or `!=` to compare dates or numbers?"**
    -   **Answer**: "Yes, absolutely. The `NOT EQUAL` operator works on any data type that can be compared, including numbers, strings, and dates."
    ```sql
    -- Find sales that did not happen on Christmas
    WHERE sale_date <> '2025-12-25';

    -- Find products that do not have a price of 0
    WHERE price <> 0;
    ```

## Bonus

### Related Concepts
-   **`NOT` Operator**: `WHERE column <> 'value'` is equivalent to `WHERE NOT (column = 'value')`.
-   **Three-Valued Logic**: The behavior of `<>` with `NULL` is a direct consequence of SQL's `TRUE`/`FALSE`/`UNKNOWN` logic system.
-   **Comparison Operators**: `NOT EQUAL` is part of the standard set of comparison operators: `=`, `<>`, `<`, `>`, `<=`, `>=`.
-   **Portability**: Sticking to ANSI standard operators like `<>` ensures your code is more likely to run without modification on different database platforms.
