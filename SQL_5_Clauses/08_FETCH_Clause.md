# FETCH Clause

## Overview
The `FETCH` clause is the ANSI SQL standard way to limit the number of rows returned by a query. It is used in conjunction with the `OFFSET` clause for pagination and "top-N" queries. It provides a more readable and standardized alternative to the `LIMIT` clause found in MySQL and PostgreSQL or the `TOP` clause in SQL Server.

## Core Concepts

The `FETCH` clause is always used with `OFFSET` and comes after the `ORDER BY` clause.

**Syntax:**
```sql
SELECT column_list
FROM table_name
[WHERE condition]
ORDER BY column_list
OFFSET rows_to_skip ROWS
FETCH {FIRST | NEXT} rows_to_fetch ROWS ONLY;
```

-   **`OFFSET rows_to_skip ROWS`**: Specifies how many rows to skip from the beginning of the sorted result set. `OFFSET 0 ROWS` is the default if omitted.
-   **`FETCH {FIRST | NEXT} rows_to_fetch ROWS ONLY`**: Specifies how many rows to return after the offset. `FIRST` and `NEXT` are interchangeable keywords. You can use `ROW` or `ROWS` for singular/plural.

**Example: Get the top 5 highest-paid employees**
```sql
SELECT employee_name, salary
FROM employees
ORDER BY salary DESC
OFFSET 0 ROWS
FETCH FIRST 5 ROWS ONLY;
```
This is equivalent to `LIMIT 5` in MySQL.

**Example: Implementing pagination (Page 3, with 10 items per page)**
This means we need to skip the first 20 rows (`(3-1) * 10`) and fetch the next 10.
```sql
SELECT product_name, price
FROM products
ORDER BY product_name
OFFSET 20 ROWS
FETCH NEXT 10 ROWS ONLY;
```
This is equivalent to `LIMIT 10 OFFSET 20` in MySQL.

### `FETCH` with Percentage
Some databases (like Oracle and SQL Server) support fetching a percentage of rows.
```sql
-- Get the top 10% of employees by salary
FETCH FIRST 10 PERCENT ROWS ONLY
```

### `WITH TIES`
This powerful option, used with `FETCH`, includes any additional rows that have the same value in the `ORDER BY` columns as the last row fetched.

**Example:**
Imagine the 5th and 6th highest salaries are both $90,000.
```sql
SELECT employee_name, salary
FROM employees
ORDER BY salary DESC
OFFSET 0 ROWS
FETCH FIRST 5 ROWS WITH TIES;
```
This query will return **6 rows**: the top 4, plus *both* employees who are tied for 5th place with a salary of $90,000. `LIMIT` cannot do this.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the `FETCH` clause?"**
    -   "`FETCH` is the ANSI SQL standard clause for limiting the number of rows returned by a query. It's used with `OFFSET` for pagination and is the standard equivalent of the non-standard `LIMIT` or `TOP` clauses."

2.  **"How is `FETCH` different from `LIMIT`?"**
    -   "Functionally, `OFFSET ... FETCH` is very similar to `LIMIT ... OFFSET`. The main difference is that `FETCH` is the SQL standard and has a more verbose but arguably more readable syntax. Also, `FETCH` has advanced options like `WITH TIES`, which `LIMIT` does not."

3.  **"Explain what `WITH TIES` does."**
    -   "`WITH TIES` is an option for the `FETCH` clause that includes any extra rows that have the same sorting value as the last row in the limit. For example, if you ask for the top 5 products by sales, and the 5th and 6th products are tied, `WITH TIES` will return both, giving you 6 rows in total."

### How to Explain in Interviews
"The `FETCH` clause is the standard SQL way to handle row limiting and pagination. I prefer it when working with databases that support it, like PostgreSQL or Oracle, because it's part of the official standard. Its syntax, `OFFSET X ROWS FETCH NEXT Y ROWS ONLY`, is very explicit. I particularly appreciate the `WITH TIES` option, which is very useful for ranking scenarios where you need to handle ties gracefully without arbitrarily cutting off records."

## Quick Recall ✅

-   **Purpose**: Standard SQL way to limit returned rows.
-   **Syntax**: `OFFSET ... ROWS FETCH ... ROWS ONLY`.
-   **Companion**: Always used with `ORDER BY`.
-   **Use Cases**: Pagination, top-N queries.
-   **Standard**: ANSI SQL standard.
-   **Special Feature**: `WITH TIES` includes rows that tie with the last row.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting `ORDER BY`**.
    -   Just like `LIMIT`, `FETCH` without `ORDER BY` gives you an arbitrary set of rows. The `ORDER BY` clause is mandatory in most databases when using `OFFSET`/`FETCH`.

2.  **Forgetting `ROWS ONLY`**.
    -   The syntax is verbose, and it's easy to forget parts of it. `FETCH FIRST 10 ROWS` is incomplete; it must be `FETCH FIRST 10 ROWS ONLY`.

3.  **Database Support**.
    -   `FETCH` is not supported in older database versions or in MySQL. Trying to use it in MySQL will result in a syntax error. Knowing which syntax to use for which database is important.

### Tricky Interview Scenarios

-   **"You need to select the top 10 products by sales. If there's a tie for 10th place, you need to include all tied products. How do you do it?"**
    -   This is a direct test of `WITH TIES` or window functions.
    -   **Answer using `FETCH`**: "I would use `ORDER BY sales DESC` and then `FETCH FIRST 10 ROWS WITH TIES;`. This is the most direct way to solve the problem."
    -   **Answer using Window Functions (if `FETCH` isn't supported or as an alternative)**: "I could use the `RANK()` or `DENSE_RANK()` window function. I'd calculate the rank of each product by sales, and then in an outer query, select all products where the rank is less than or equal to 10."
        ```sql
        WITH RankedProducts AS (
            SELECT
                product_name,
                sales,
                DENSE_RANK() OVER (ORDER BY sales DESC) as sales_rank
            FROM products
        )
        SELECT product_name, sales
        FROM RankedProducts
        WHERE sales_rank <= 10;
        ```

## Bonus

### Related Concepts
-   **`LIMIT` Clause**: The non-standard but very common equivalent in MySQL/PostgreSQL.
-   **`TOP` Clause**: The non-standard equivalent in SQL Server. `SELECT TOP 10 WITH TIES ...` is the SQL Server syntax for the `WITH TIES` feature.
-   **Window Functions (`RANK`, `DENSE_RANK`)**: These provide another, more powerful way to solve "top-N-with-ties" problems. `DENSE_RANK` is often preferred as it doesn't skip numbers after a tie (e.g., 1, 2, 2, 3), whereas `RANK` does (1, 2, 2, 4).
-   **Pagination**: `OFFSET`/`FETCH` is the standard building block for server-side pagination in applications.
