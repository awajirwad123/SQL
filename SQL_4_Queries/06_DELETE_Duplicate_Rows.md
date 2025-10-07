# DELETE Duplicate Rows

## Overview
Deleting duplicate rows is a common data cleaning task. Duplicates can occur due to application bugs, faulty data imports, or a lack of proper `UNIQUE` constraints. There is no single `DELETE DUPLICATES` command in SQL; instead, you must use more advanced techniques, often involving window functions, self-joins, or temporary tables.

## Core Concepts

The goal is to identify rows that are identical based on certain columns and then delete all but one of them. The "one" you keep is usually arbitrary or based on a tie-breaker column like a `timestamp` or an `id`.

Let's assume we have a table `employees` where `(first_name, last_name, email)` should be unique, but we have duplicates. We want to keep the one with the lowest `id`.

```sql
-- Sample duplicate data in 'employees' table
id | first_name | last_name | email
---|------------|-----------|--------------------
1  | John       | Smith     | john.smith@email.com
2  | Jane       | Doe       | jane.doe@email.com
3  | John       | Smith     | john.smith@email.com  -- Duplicate
4  | Peter      | Jones     | peter.jones@email.com
5  | John       | Smith     | john.smith@email.com  -- Duplicate
```
We want to delete rows with `id` 3 and 5, keeping `id` 1.

### Method 1: Using Window Functions (`ROW_NUMBER`)
This is the most modern and often the most efficient method. It works in PostgreSQL, SQL Server, Oracle, and recent versions of MySQL.

**Concept:**
1.  Use the `ROW_NUMBER()` window function to assign a unique, sequential integer to rows partitioned by the columns that define a duplicate.
2.  The numbering restarts for each unique group of `(first_name, last_name, email)`.
3.  We order within the partition by a tie-breaker column (like `id` or a timestamp) to decide which row to keep (the one that gets `row_number = 1`).
4.  Delete any row where the `row_number` is greater than 1.

**Query using a Common Table Expression (CTE):**
```sql
WITH NumberedRows AS (
    SELECT
        id,
        ROW_NUMBER() OVER(
            PARTITION BY first_name, last_name, email
            ORDER BY id ASC -- Keep the row with the smallest id
        ) AS rn
    FROM
        employees
)
DELETE FROM employees
WHERE id IN (
    SELECT id FROM NumberedRows WHERE rn > 1
);
```
*Note: The final `DELETE` syntax might vary slightly. In SQL Server, you can `DELETE FROM NumberedRows WHERE rn > 1;` directly.*

### Method 2: Using a Self-Join
This method works in older versions of databases that don't support window functions (like older MySQL).

**Concept:**
1.  Join the table to itself (`t1` and `t2`).
2.  The join condition matches rows that are duplicates (`t1.email = t2.email`, etc.).
3.  A second condition (`t1.id > t2.id`) ensures that for any pair of duplicates, we are only selecting the one with the higher `id` to be deleted.

**Query:**
```sql
DELETE t1
FROM employees t1
JOIN employees t2
WHERE
    t1.id > t2.id AND
    t1.first_name = t2.first_name AND
    t1.last_name = t2.last_name AND
    t1.email = t2.email;
```
This query effectively says, "Delete any row from `employees` (`t1`) for which another row exists (`t2`) with the same content but a lower `id`."

### Method 3: Using a Temporary Table
This is a safe and straightforward, if sometimes slower, approach.

**Concept:**
1.  Create a temporary table containing only the unique rows you want to *keep*.
2.  Empty the original table using `TRUNCATE`.
3.  Copy the unique rows from the temporary table back into the original table.

**Query:**
```sql
-- Step 1: Create a new table with the distinct rows you want to keep
CREATE TABLE employees_temp AS
SELECT MIN(id) as id, first_name, last_name, email
FROM employees
GROUP BY first_name, last_name, email;

-- Step 2: Empty the original table
TRUNCATE TABLE employees;

-- Step 3: Re-insert the unique rows
-- (You may need to disable/re-enable identity insert in some systems)
INSERT INTO employees (id, first_name, last_name, email)
SELECT id, first_name, last_name, email
FROM employees_temp;

-- Step 4: Clean up
DROP TABLE employees_temp;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How would you delete duplicate rows from a table?"**
    -   "My preferred method is to use the `ROW_NUMBER()` window function within a Common Table Expression (CTE). I would partition by the columns that define the duplicate and order by a unique key like an `id` or `timestamp` to select which row to keep. Then, I'd delete all rows where the row number is greater than 1."

2.  **"What if your database doesn't support window functions?"**
    -   "In that case, I would use a self-join. I'd join the table to itself on the duplicate columns and use a condition like `t1.id > t2.id` to identify and delete the rows with the higher IDs, keeping the one with the lowest ID."

3.  **"Which row do you typically keep when deleting duplicates?"**
    -   "It depends on the business requirement. Usually, you want to keep either the oldest record (e.g., `ORDER BY created_at ASC`) or the newest record (`ORDER BY created_at DESC`). The `ORDER BY` clause within the `ROW_NUMBER()` function gives you precise control over this."

### How to Explain in Interviews
"Deleting duplicates is a classic data cleaning task. The most robust solution in modern SQL is using the `ROW_NUMBER()` window function. It allows me to define what constitutes a duplicate via the `PARTITION BY` clause and then precisely control which one to keep using the `ORDER BY` clause. This method is generally more readable and performant than older techniques like self-joins, especially on large datasets."

## Quick Recall ✅

-   **Modern Method**: `ROW_NUMBER()` with a CTE.
    -   `PARTITION BY`: The columns that define a duplicate.
    -   `ORDER BY`: The column that decides which one to keep (the first one).
    -   `DELETE WHERE rn > 1`.
-   **Classic Method**: Self-join.
    -   `JOIN` on duplicate columns.
    -   `DELETE` based on a tie-breaker like `t1.id > t2.id`.
-   **Safe Method**: Temporary table.
    -   Copy unique rows to a temp table, `TRUNCATE` the original, and re-insert.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Not having a tie-breaker.**
    -   If you don't have a unique column like an `id` or a `timestamp`, it can be impossible to distinguish between two perfectly identical rows, and some methods might fail or delete unpredictably.

2.  **Incorrect `PARTITION BY` or `JOIN` columns.**
    -   If you miss a column in your definition of a duplicate, you might fail to delete actual duplicates or, worse, delete non-duplicate rows.

3.  **Performance on large tables.**
    -   A self-join on a very large table can be extremely slow. The `ROW_NUMBER()` approach is generally better optimized. The temporary table method can also be slow due to the large data movement.

### Tricky Interview Scenarios

-   **"How would you *prevent* duplicates from being inserted in the first place?"**
    -   This is the best question because it addresses the root cause.
    -   **Answer**: "The best way to handle duplicates is to prevent them with database constraints. I would add a `UNIQUE` constraint or a `PRIMARY KEY` on the set of columns that should be unique. For example: `ALTER TABLE employees ADD CONSTRAINT uq_email UNIQUE (email);` or `ADD UNIQUE (first_name, last_name, email);`. This way, the database enforces uniqueness, and an `INSERT` of a duplicate will fail."

-   **"You need to delete duplicates from a massive table (billions of rows). The CTE or self-join methods are too slow. What's your strategy?"**
    -   This is an advanced, big-data scenario.
    -   **Answer**: "For a table that large, DML operations are too slow. The most performant method would be the temporary table approach, but on a larger scale. I would create a new, permanent table (`employees_unique`) with the correct `UNIQUE` constraint. Then, I'd use an `INSERT ... SELECT` with `GROUP BY` or `DISTINCT ON` to copy the unique rows from the old table to the new one. After verifying the data, I would drop the old table and rename the new one. This minimizes transaction logging and is much faster than a `DELETE`."

## Bonus

### Related Concepts
-   **Window Functions**: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` are essential for advanced analytical queries. Deleting duplicates is a perfect practical application.
-   **CTEs (Common Table Expressions)**: The `WITH` clause makes complex queries like this much more readable.
-   **`UNIQUE` Constraints**: The preventative measure for this entire problem.
-   **Idempotency**: Data cleaning scripts should be idempotent, meaning they can be run multiple times without changing the result after the first run. The `DELETE WHERE rn > 1` approach is naturally idempotent.
