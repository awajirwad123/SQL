# INTERSECT Operator

## Overview
The `INTERSECT` operator is a set operator in SQL that returns only the rows that are present in **both** of the result sets of the `SELECT` statements it connects. It finds the common ground, or intersection, between two sets of data.

## Core Concepts

To use `INTERSECT`, the `SELECT` statements must follow the same rules as other set operators like `UNION`:
1.  They must have the same number of columns.
2.  The data types of the corresponding columns must be compatible.

**Syntax:**
```sql
SELECT column1, column2, ... FROM table1
INTERSECT
SELECT column1, column2, ... FROM table2;
```

**How it works:**
1.  The database executes both `SELECT` statements.
2.  It then compares the result sets and returns only the rows that appear in **both** results.
3.  Like `UNION`, `INTERSECT` implicitly removes duplicates, returning only a `DISTINCT` set of rows.

**Example: Find all people who are both customers and employees.**
Assume `customers` and `employees` tables both have `first_name` and `last_name` columns.
```sql
SELECT first_name, last_name FROM customers
INTERSECT
SELECT first_name, last_name FROM employees;
```
This will return a list of names that appear in both the `customers` table and the `employees` table.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What does the `INTERSECT` operator do?"**
    -   "`INTERSECT` is a set operator that compares two result sets and returns only the rows that are common to both. It's the SQL equivalent of a mathematical set intersection. It also returns only unique rows."

2.  **"How is `INTERSECT` different from an `INNER JOIN`?"**
    -   "This is a great question. Both can be used to find common data, but they work differently. An `INNER JOIN` combines columns from two tables based on a join condition, creating a new, wider row. `INTERSECT` compares rows from two result sets (which could be from the same or different tables) and returns only the rows that are identical, without combining columns. Also, `INTERSECT` implicitly performs a `DISTINCT`, while an `INNER JOIN` will return multiple rows if the join condition matches multiple times."

3.  **"Is `INTERSECT` supported by all databases?"**
    -   "No, it's not. While it's part of the ANSI SQL standard and supported by systems like PostgreSQL, SQL Server, and Oracle, it is notably **not supported by MySQL**. In MySQL, you would have to emulate it using an `INNER JOIN` or `IN` with a subquery."

### How to Explain in Interviews
"`INTERSECT` is a very clear and readable way to find the common elements between two datasets that have the same structure. For example, to find which products from this year's catalog were also in last year's catalog, `INTERSECT` is perfect. I'm aware that it's not available in MySQL, where I would use an `INNER JOIN` on the relevant columns or an `IN` clause to achieve the same result."

## Quick Recall ✅

-   **Purpose**: Returns unique rows that are present in **both** result sets.
-   **Set Operation**: Set Intersection (A ∩ B).
-   **Duplicates**: `INTERSECT` returns a `DISTINCT` set of rows.
-   **Rules**: Same number of columns, compatible data types.
-   **RDBMS Support**:
    -   Supported: PostgreSQL, SQL Server, Oracle.
    -   Not Supported: MySQL.
-   **MySQL Alternative**: `INNER JOIN` or `IN`.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming it's supported in MySQL.** This is a common trip-up for developers who work across different database systems.

2.  **Confusing it with `INNER JOIN`**. They solve similar problems but are not the same. `INTERSECT` compares full rows; `INNER JOIN` connects rows based on a key and can combine columns from both tables.

**Example of `INTERSECT` vs `INNER JOIN`:**
```sql
-- Table A: (id, color) -> (1, 'red'), (2, 'blue')
-- Table B: (id, color) -> (1, 'red'), (3, 'green')

-- INTERSECT returns the common row
SELECT id, color FROM A INTERSECT SELECT id, color FROM B;
-- Result: (1, 'red')

-- INNER JOIN returns a combined row
SELECT A.*, B.* FROM A INNER JOIN B ON A.id = B.id;
-- Result: (1, 'red', 1, 'red')
```

### Tricky Interview Scenarios

-   **"Write a query to find all customers who have purchased both 'Product A' and 'Product B'."**
    -   This is a classic "set intersection" problem.
    ```sql
    SELECT customer_id
    FROM orders
    WHERE product_name = 'Product A'
    INTERSECT
    SELECT customer_id
    FROM orders
    WHERE product_name = 'Product B';
    ```
    -   **Alternative in MySQL:**
    ```sql
    SELECT customer_id
    FROM orders
    WHERE product_name IN ('Product A', 'Product B')
    GROUP BY customer_id
    HAVING COUNT(DISTINCT product_name) = 2;
    ```

-   **"Does `INTERSECT` treat `NULL`s as equal?"**
    -   **Answer**: "Yes. Unlike most other comparisons in SQL where `NULL` is not equal to `NULL`, set operators like `INTERSECT`, `UNION`, and `EXCEPT` treat `NULL`s as equal for the purpose of comparing rows. So, a row with a `NULL` value can find a match in the other result set if it also has a `NULL` in the same position."

## Bonus

### Related Concepts
-   **`UNION`**: Combines two sets (A ∪ B).
-   **`EXCEPT`**: Finds the difference between two sets (A - B).
-   **`INNER JOIN`**: The relational algebra equivalent. While `INTERSECT` is a set operation, `INNER JOIN` is a relational one.
-   **`INTERSECT ALL`**: A less common variant that does not eliminate duplicate rows from the result. Its support varies.
-   **Relational Division**: The problem of "finding customers who have bought *all* products" is a classic SQL challenge known as relational division, which is a more advanced version of the problem that `INTERSECT` helps solve.
