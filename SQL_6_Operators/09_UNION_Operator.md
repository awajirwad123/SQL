# UNION Operator

## Overview
The `UNION` operator is a set operator used to combine the result sets of two or more `SELECT` statements into a single result set. By default, `UNION` removes duplicate rows from the combined result.

## Core Concepts

To use `UNION`, the `SELECT` statements being combined must follow two rules:
1.  They must have the same number of columns.
2.  The data types of the corresponding columns must be compatible (e.g., you can union a `VARCHAR(10)` with a `VARCHAR(20)`, but not a `VARCHAR` with an `INT`).

**Syntax:**
```sql
SELECT column1, column2, ... FROM table1
UNION
SELECT column1, column2, ... FROM table2;
```

**Key Behavior: Duplicate Removal**
`UNION` implicitly performs a `DISTINCT` operation on the combined result set. If a row appears in the first result set and also in the second, it will only appear once in the final output.

**Example: Get a single list of all unique cities where suppliers and customers are located.**
```sql
-- Table: suppliers
-- city
-- ------
-- London
-- Tokyo

-- Table: customers
-- city
-- ------
-- London
-- Paris

SELECT city FROM suppliers
UNION
SELECT city FROM customers;

-- Result:
-- city
-- ------
-- London  (Appears only once)
-- Tokyo
-- Paris
```

The column names in the final result set are usually taken from the *first* `SELECT` statement.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `UNION` operator do?"**
    -   "`UNION` is a set operator that combines the results of two or more `SELECT` statements into a single result set. Crucially, it also removes any duplicate rows from the combined results."

2.  **"What is the difference between `UNION` and `UNION ALL`?"**
    -   "This is a classic interview question. `UNION` removes duplicate rows, while `UNION ALL` includes all rows from both `SELECT` statements, including duplicates. Because `UNION` has to do the extra work of finding and removing duplicates, `UNION ALL` is significantly faster and should be preferred if you know the results are already unique or if duplicates are acceptable."

3.  **"What are the rules for using `UNION`?"**
    -   "The `SELECT` statements involved must have the same number of columns, and the data types of the corresponding columns must be compatible."

4.  **"If you `UNION` two tables and want to sort the final result, where do you put the `ORDER BY` clause?"**
    -   "The `ORDER BY` clause is placed only once, at the very end of the entire `UNION` statement. It sorts the final, combined result set."
    ```sql
    SELECT city FROM suppliers
    UNION
    SELECT city FROM customers
    ORDER BY city;
    ```

### How to Explain in Interviews
"`UNION` is the SQL operator for set union. I use it when I need to merge rows from different tables or queries that share a similar structure, for example, to create a consolidated list of all contacts from separate `leads` and `customers` tables. I'm always mindful of the key difference between `UNION` and `UNION ALL`. I use `UNION` when I specifically need a unique list, but I default to `UNION ALL` for better performance if duplicates aren't an issue."

## Quick Recall ✅

-   **Purpose**: Combines result sets of two or more `SELECT` statements.
-   **Key Feature**: **Removes duplicate rows** (implicitly `DISTINCT`).
-   **Rules**: Same number of columns, compatible data types.
-   **Performance**: Slower than `UNION ALL` due to the duplicate removal step.
-   **`ORDER BY`**: Placed at the end of the entire statement.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Using `UNION` when `UNION ALL` would suffice.**
    -   If you don't need duplicates removed, using `UNION` adds unnecessary overhead. This is a common performance mistake.

2.  **Mismatching columns.**
    -   `SELECT name, email FROM t1 UNION SELECT id, name FROM t2;` ❌
    -   This will fail because the number of columns is the same, but the data types (`email` vs. `id`, `name` vs. `name`) might not be compatible, and the meaning is confused. The columns should correspond semantically.

3.  **Placing `ORDER BY` in the wrong spot.**
    -   `SELECT ... FROM t1 ORDER BY col1 UNION SELECT ... FROM t2;` ❌
    -   An `ORDER BY` in the individual `SELECT` statements is usually not allowed or doesn't affect the final order (unless combined with `LIMIT`/`TOP`). The final sort must happen at the end.

### Tricky Interview Scenarios

-   **"You need to combine results from three tables. How would you do it?"**
    -   **Answer**: "You can chain `UNION` operators."
    ```sql
    SELECT column FROM table1
    UNION
    SELECT column FROM table2
    UNION
    SELECT column FROM table3;
    ```

-   **"How does `UNION` handle different column names in the `SELECT` statements?"**
    -   **Answer**: "The `UNION` operation itself only cares about the number of columns and their data types, not their names. The final result set will adopt the column names from the *first* `SELECT` statement."
    ```sql
    SELECT customer_name AS name FROM customers
    UNION
    SELECT supplier_name FROM suppliers;
    -- The final column will be named 'name'.
    ```

## Bonus

### Related Concepts
-   **`UNION ALL`**: The faster, non-deduplicating version of `UNION`.
-   **`INTERSECT`**: A set operator that returns only the rows that are present in **both** result sets.
-   **`EXCEPT`** (or `MINUS` in Oracle): A set operator that returns rows from the first result set that are **not** present in the second result set.
-   **Set Theory**: `UNION`, `INTERSECT`, and `EXCEPT` are direct implementations of mathematical set operations.
-   **`JOIN`**: `JOIN`s combine columns from different tables based on a related key. `UNION` combines rows. They solve different problems.
