# WHERE Clause

## Overview
The `WHERE` clause is a fundamental part of SQL used to filter records. It extracts only those records that fulfill a specified condition. It can be used with `SELECT`, `UPDATE`, and `DELETE` statements to control which rows are retrieved, modified, or removed.

## Core Concepts

The `WHERE` clause is applied to each row of the data source (table or view) specified in the `FROM` clause. If the condition evaluates to `TRUE` for a given row, that row is included in the operation.

**Logical Execution Order:**
In a `SELECT` statement, the `WHERE` clause is processed *after* the `FROM` clause but *before* `GROUP BY`, `HAVING`, `SELECT`, and `ORDER BY`. This is critical to remember.

**Common Operators Used with `WHERE`:**

| Operator | Description | Example |
| :--- | :--- | :--- |
| `=` | Equal to | `WHERE price = 100` |
| `<>`, `!=` | Not equal to | `WHERE country <> 'USA'` |
| `>` | Greater than | `WHERE price > 50` |
| `<` | Less than | `WHERE price < 50` |
| `>=` | Greater than or equal to | `WHERE price >= 50` |
| `<=` | Less than or equal to | `WHERE price <= 50` |
| `BETWEEN` | Within a certain range (inclusive) | `WHERE price BETWEEN 50 AND 100` |
| `LIKE` | Search for a pattern | `WHERE name LIKE 'J%'` |
| `IN` | To specify multiple possible values | `WHERE country IN ('USA', 'UK', 'Canada')` |
| `IS NULL` | Check for null values | `WHERE address IS NULL` |
| `IS NOT NULL`| Check for non-null values | `WHERE address IS NOT NULL` |

**Combining Conditions with `AND`, `OR`, `NOT`:**
-   **`AND`**: Displays a record if all conditions separated by `AND` are TRUE.
-   **`OR`**: Displays a record if any of the conditions separated by `OR` is TRUE.
-   **`NOT`**: Displays a record if the condition(s) is NOT TRUE.

**Example:**
```sql
SELECT product_name, category, price
FROM products
WHERE
    (category = 'Electronics' OR category = 'Appliances')
    AND price < 500
    AND product_name NOT LIKE 'Refurbished%';
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the purpose of the `WHERE` clause?"**
    -   "The `WHERE` clause is used to filter rows from a result set based on a specific condition. It's used in `SELECT`, `UPDATE`, and `DELETE` statements to specify exactly which rows to act upon."

2.  **"What is the difference between the `WHERE` clause and the `HAVING` clause?"**
    -   "This is a key distinction. The `WHERE` clause filters rows *before* any aggregations are performed (i.e., before `GROUP BY`). The `HAVING` clause filters groups of rows *after* aggregations are performed. So, you use `WHERE` for conditions on individual row data and `HAVING` for conditions on aggregate function results, like `COUNT(*)` or `SUM(price)`."

3.  **"In what order is the `WHERE` clause executed in a `SELECT` query?"**
    -   "It's executed after `FROM` but before `GROUP BY`, `HAVING`, `SELECT`, and `ORDER BY`. This is why you can't use a column alias defined in the `SELECT` list within the `WHERE` clause."

### How to Explain in Interviews
"The `WHERE` clause is the primary mechanism for filtering data at the row level. I think of it as the gatekeeper that decides which rows from the source table get to move on to the next stage of processing, like grouping or ordering. Understanding its place in the logical query execution order is crucial for writing correct and efficient queries, especially when distinguishing it from the `HAVING` clause."

## Quick Recall ✅

-   **Purpose**: Filter **rows**.
-   **Used with**: `SELECT`, `UPDATE`, `DELETE`.
-   **Execution Order**: After `FROM`, before `GROUP BY`.
-   **Key Operators**: `=`, `>`, `<`, `IN`, `LIKE`, `BETWEEN`, `IS NULL`.
-   **Logic**: `AND`, `OR`, `NOT`.
-   **`WHERE` vs. `HAVING`**: `WHERE` filters rows, `HAVING` filters groups.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Using an aggregate function in `WHERE`**.
    ```sql
    -- ❌ INCORRECT
    SELECT category, COUNT(*)
    FROM products
    GROUP BY category
    WHERE COUNT(*) > 10;
    ```
    This is wrong because `WHERE` operates on individual rows, not aggregate results.
    **Correct way (use `HAVING`)**:
    ```sql
    SELECT category, COUNT(*)
    FROM products
    GROUP BY category
    HAVING COUNT(*) > 10;
    ```

2.  **Using a column alias from the `SELECT` list in `WHERE`**.
    ```sql
    -- ❌ INCORRECT
    SELECT price * 0.9 AS discounted_price
    FROM products
    WHERE discounted_price < 100;
    ```
    This fails because `WHERE` is evaluated *before* the `SELECT` list, so the alias `discounted_price` doesn't exist yet.
    **Correct way**:
    ```sql
    SELECT price * 0.9 AS discounted_price
    FROM products
    WHERE (price * 0.9) < 100;
    ```

3.  **Confusing `NULL` handling.**
    -   `NULL` is not equal to anything, not even itself. You cannot use `WHERE my_column = NULL`. You must use `WHERE my_column IS NULL`.

### Tricky Interview Scenarios

-   **"How can you optimize a query that is slow because of its `WHERE` clause?"**
    -   This tests knowledge of performance tuning.
    -   **Answer**: "The most effective way is to ensure the columns used in the `WHERE` clause are indexed. An index allows the database to quickly find the matching rows without scanning the entire table. It's also important to use 'sargable' predicates. This means writing conditions in a way that allows the database to use an index, for example, using `date_col >= '2022-01-01'` instead of `YEAR(date_col) = 2022`, as the function `YEAR()` can prevent the index from being used."

## Bonus

### Related Concepts
-   **SARGable Predicates**: A predicate (a condition in the `WHERE` clause) is "SARGable" (Search ARGument-able) if the database engine can use an index to speed up the execution of the query.
    -   **SARGable**: `age = 25`, `name LIKE 'J%'`, `date BETWEEN '2021' AND '2022'`
    -   **Non-SARGable**: `YEAR(order_date) = 2022`, `SUBSTR(name, 1, 1) = 'J'`, `age + 5 = 30`
-   **Indexing**: The primary database feature used to speed up `WHERE` clause performance. A B-Tree index is the most common type and works perfectly for the operators listed above.
-   **Query Optimizer**: The component of the database that analyzes a query and decides the most efficient way to execute it, heavily relying on the `WHERE` clause and available indexes to create its execution plan.
