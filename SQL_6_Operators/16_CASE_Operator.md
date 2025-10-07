# CASE Expression

## Overview
The `CASE` expression is SQL's way of handling `if/then/else` logic. It is not an operator, but a powerful expression that allows you to return different values based on a set of conditions. It can be used in `SELECT` lists, `WHERE` clauses, `GROUP BY` clauses, and `ORDER BY` clauses.

## Core Concepts

There are two forms of the `CASE` expression: Simple `CASE` and Searched `CASE`.

### 1. Simple `CASE` Expression
This form compares a single expression against a set of specific values. It's like a `switch` statement in many programming languages.

**Syntax:**
```sql
CASE input_expression
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ...
    [ELSE else_result]
END
```

**Example: Translate a status code into a human-readable string.**
```sql
SELECT
    order_id,
    CASE status
        WHEN 1 THEN 'Processing'
        WHEN 2 THEN 'Shipped'
        WHEN 3 THEN 'Delivered'
        ELSE 'Unknown'
    END AS order_status
FROM orders;
```

### 2. Searched `CASE` Expression
This is the more flexible and common form. It evaluates a list of independent boolean conditions. The first condition that evaluates to `TRUE` determines the result.

**Syntax:**
```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    [ELSE else_result]
END
```

**Example: Categorize products by price range.**
```sql
SELECT
    product_name,
    price,
    CASE
        WHEN price < 50 THEN 'Cheap'
        WHEN price BETWEEN 50 AND 200 THEN 'Moderate'
        WHEN price > 200 THEN 'Expensive'
        ELSE 'Not Priced'
    END AS price_category
FROM products;
```
**Important:** The conditions are evaluated in order. The first one to be met wins. If a product costs $30, it matches `price < 50` and the expression stops; it will not be evaluated against the other `WHEN` clauses.

### `ELSE` Clause
The `ELSE` clause is optional. If it is omitted and none of the `WHEN` conditions are met, the `CASE` expression returns `NULL`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the `CASE` expression used for?"**
    -   "The `CASE` expression provides `if/then/else` conditional logic within a SQL query. It allows me to return different values based on specified conditions. It's incredibly versatile and can be used in `SELECT` lists to create derived columns, in `ORDER BY` for custom sorting, or in `WHERE` and `GROUP BY` clauses."

2.  **"What are the two types of `CASE` expressions?"**
    -   "There's the 'Simple `CASE`,' which compares one expression against several possible values, much like a switch statement. Then there's the 'Searched `CASE`,' which is more powerful and evaluates a series of independent boolean conditions."

3.  **"What happens if no conditions in a `CASE` expression are met and there is no `ELSE` clause?"**
    -   "In that situation, the `CASE` expression returns `NULL`."

### How to Explain in Interviews
"`CASE` is one of the most powerful tools in SQL for data transformation and analysis. I use it all the time to create new, meaningful columns on the fly. For example, I can use it to segment customers into tiers based on their spending, or to pivot data by counting records that meet certain criteria. It's the standard way to embed conditional logic directly into a query."

## Quick Recall ✅

-   **Purpose**: Provides `if/then/else` logic in SQL.
-   **Simple `CASE`**: `CASE input WHEN val1 THEN res1 END`.
-   **Searched `CASE`**: `CASE WHEN cond1 THEN res1 END`.
-   **Order**: Conditions are evaluated in the order they are written. The first `TRUE` condition wins.
-   **`ELSE`**: Optional. If omitted, returns `NULL` if no conditions are met.
-   **Versatility**: Can be used in `SELECT`, `WHERE`, `GROUP BY`, and `ORDER BY`.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting the `END` keyword.** Every `CASE` expression must be terminated with `END`.
2.  **Mixing data types.** All `result` expressions (in the `THEN` and `ELSE` parts) must have compatible data types. You can't have one `THEN` return a number and another return a string.
3.  **Order of evaluation.** In a searched `CASE`, putting a more general condition before a more specific one can lead to bugs.
    ```sql
    -- Buggy logic
    CASE
        WHEN price > 50 THEN 'Over 50'
        WHEN price > 200 THEN 'Over 200' -- This condition will never be met!
    END
    ```
    A price of 300 is greater than 50, so the first condition is met and the expression returns 'Over 50'. The more specific condition should come first.

### Tricky Interview Scenarios

-   **"How can you use `CASE` to perform a pivot, for example, to count male and female employees in a single row for each department?"**
    -   This is an advanced use of `CASE` with `GROUP BY` for conditional aggregation.
    ```sql
    SELECT
        department,
        COUNT(CASE WHEN gender = 'Male' THEN 1 END) AS male_count,
        COUNT(CASE WHEN gender = 'Female' THEN 1 END) AS female_count
    FROM employees
    GROUP BY department;
    ```
    **How it works**: `COUNT` only counts non-null values. The `CASE` expression returns `1` (a non-null value) if the condition is met, and `NULL` otherwise (because there is no `ELSE`). So, `COUNT` effectively only counts the rows that match the `WHEN` condition.

-   **"How would you implement a custom sort order, for example, to always show 'Critical' items first, then 'High', then 'Low'?"**
    -   This demonstrates using `CASE` in an `ORDER BY` clause.
    ```sql
    SELECT request_name, priority
    FROM support_tickets
    ORDER BY
        CASE priority
            WHEN 'Critical' THEN 1
            WHEN 'High' THEN 2
            WHEN 'Medium' THEN 3
            WHEN 'Low' THEN 4
            ELSE 5
        END;
    ```

## Bonus

### Related Concepts
-   **`COALESCE` and `NULLIF`**: These are specialized functions that can sometimes be used as a shorthand for specific `CASE` expressions.
    -   `COALESCE(a, b)` is equivalent to `CASE WHEN a IS NOT NULL THEN a ELSE b END`.
    -   `NULLIF(a, b)` is equivalent to `CASE WHEN a = b THEN NULL ELSE a END`.
-   **Pivot/Unpivot**: `CASE` with `GROUP BY` is the manual way to pivot data (turning rows into columns). Dedicated `PIVOT` and `UNPIVOT` operators exist in some databases like SQL Server and Oracle for this purpose.
-   **Conditional Logic**: `CASE` is the heart of conditional logic within SQL queries.
