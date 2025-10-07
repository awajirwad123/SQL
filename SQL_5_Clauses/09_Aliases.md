# SQL Aliases

## Overview
SQL aliases are used to give a temporary, more readable name to a table or a column in a query. Aliases are essential for improving the readability of complex queries and are required in certain types of joins and subqueries.

## Core Concepts

There are two types of aliases: column aliases and table aliases.

### 1. Column Aliases
A column alias is used to rename a column in the result set. This is useful for making output more user-friendly or for giving a name to a calculated column.

**Syntax:**
```sql
SELECT column_name AS alias_name
FROM table_name;
```
The `AS` keyword is optional in most database systems, but it's good practice to include it for clarity.

**Example: Renaming a column**
```sql
SELECT first_name AS name, country AS location
FROM customers;
```

**Example: Naming a calculated column**
```sql
SELECT
    product_name,
    price * (1 - discount_rate) AS final_price
FROM products;
```
Without the alias, the calculated column would have a generic or system-defined name (like `?column?`), which is not useful.

### 2. Table Aliases
A table alias is a temporary name given to a table in the `FROM` or `JOIN` clause. They are crucial for:
-   **Brevity**: Shortening long table names.
-   **Clarity**: Distinguishing between multiple instances of the same table (in a self-join).
-   **Qualification**: Avoiding ambiguity when tables in a `JOIN` have columns with the same name.

**Syntax:**
```sql
SELECT t.column_name
FROM long_table_name AS t;
```

**Example: Brevity and Qualification in a `JOIN`**
```sql
-- Without aliases (verbose and ambiguous if columns have same name)
SELECT customers.customer_name, orders.order_date
FROM customers
JOIN orders ON customers.customer_id = orders.customer_id;

-- With aliases (concise and clear)
SELECT c.customer_name, o.order_date
FROM customers AS c
JOIN orders AS o ON c.customer_id = o.customer_id;
```

**Example: Required for a self-join**
A self-join is when you join a table to itself. Aliases are mandatory here to let the database distinguish between the two instances of the table.
```sql
-- Find employees who are managed by another employee
SELECT
    e.employee_name AS employee,
    m.employee_name AS manager
FROM employees AS e
JOIN employees AS m ON e.manager_id = m.employee_id;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What are SQL aliases and why are they used?"**
    -   "SQL aliases provide temporary names for columns or tables within a query. Column aliases are used to make output headers more readable and to name calculated columns. Table aliases are used to shorten long table names and, more importantly, to resolve ambiguity in joins, especially in self-joins where they are required."

2.  **"When is it mandatory to use a table alias?"**
    -   "It's mandatory in a self-join, where you join a table to itself. You need to create at least two different aliases for the table so you can refer to each instance independently."

3.  **"Can you use a column alias in a `WHERE` clause?"**
    -   "No, you generally cannot. This is because of the logical order of query execution. The `WHERE` clause is processed before the `SELECT` list, so the column alias hasn't been created yet. You have to repeat the expression in the `WHERE` clause."

4.  **"Can you use a column alias in an `ORDER BY` clause?"**
    -   "Yes, you can. The `ORDER BY` clause is one of the last clauses to be processed, occurring after the `SELECT` list is evaluated. Therefore, it can see and use the column aliases."

### How to Explain in Interviews
"Aliases are a fundamental part of writing clean, readable SQL. I use table aliases (`c` for `customers`, `p` for `products`) in every `JOIN` I write to keep the syntax concise and to clearly qualify which table each column comes from. For columns, I use aliases to provide meaningful names for expressions and calculations, like `price * quantity AS total_sale`. I'm also mindful of the logical query processing order, which dictates where I can and cannot use a column alias—for instance, they are fine in `ORDER BY` but not in `WHERE`."

## Quick Recall ✅

-   **Column Alias**: Renames a column in the output. `SELECT col AS new_name`.
-   **Table Alias**: Renames a table in the query. `FROM my_table AS t`.
-   **Readability**: The primary benefit.
-   **Self-Joins**: Table aliases are mandatory.
-   **`WHERE` Clause**: Cannot use a column alias.
-   **`ORDER BY` Clause**: Can use a column alias.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Using a column alias in `WHERE` or `GROUP BY` or `HAVING`.**
    -   This is the most common trap. Because of the logical execution order, these clauses are processed before the `SELECT` list where the alias is defined.
    ```sql
    SELECT YEAR(order_date) as order_year, COUNT(*)
    FROM orders
    GROUP BY order_year; -- ❌ Fails in many DBs (must use GROUP BY YEAR(order_date))
    ```
    *Note: Some modern databases like PostgreSQL are smart enough to allow this, but it's not standard SQL and is not portable.*

2.  **Using quotes incorrectly.**
    -   Use double quotes (`"`) or square brackets (`[]`) for aliases that contain spaces or are reserved keywords. Single quotes (`'`) are for string literals.
    ```sql
    SELECT price AS "Final Price" -- ✅ Correct
    FROM products;

    SELECT price AS 'Final Price' -- ❌ Incorrect (this is a string, not an identifier)
    FROM products;
    ```

### Tricky Interview Scenarios

-   **"You have a query with a complex calculation that you need to use in both the `SELECT` list and the `WHERE` clause. How do you write this without repeating the calculation?"**
    -   This tests knowledge of how to work around the alias limitation.
    -   **Answer**: "Since I can't define an alias in `SELECT` and use it in `WHERE`, I would use a Common Table Expression (CTE) or a derived table. I'd perform the calculation inside the CTE, giving it an alias there. The outer query can then reference that alias freely in both its `SELECT` and `WHERE` clauses."
    ```sql
    WITH CalculatedData AS (
        SELECT
            order_id,
            (price * quantity * (1 - discount)) AS final_price
        FROM order_details
    )
    SELECT order_id, final_price
    FROM CalculatedData
    WHERE final_price > 100.00;
    ```

## Bonus

### Related Concepts
-   **Logical Query Processing Order**: The reason aliases can be used in some clauses but not others. The typical order is `FROM`, `WHERE`, `GROUP BY`, `HAVING`, `SELECT`, `ORDER BY`.
-   **Self-Join**: A join of a table to itself, where table aliases are not just good practice but a requirement.
-   **Common Table Expressions (CTEs)**: A great way to define calculations and give them aliases, which can then be used in the main query, solving the `WHERE` clause limitation.
-   **Readability**: While a technical concept, the primary driver for using aliases is to make code maintainable and understandable for other developers (or your future self).
