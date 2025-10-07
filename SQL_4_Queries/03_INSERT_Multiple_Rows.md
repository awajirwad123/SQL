# INSERT Multiple Rows

## Overview
Inserting multiple rows of data in a single `INSERT` statement is far more efficient than inserting rows one by one. It reduces network overhead, transaction log writes, and parsing costs, making it a critical best practice for bulk data loading.

## Core Concepts

### The `VALUES` List Syntax
The standard way to insert multiple rows is to extend the `VALUES` clause with multiple comma-separated tuples (sets of values).

**Syntax:**
```sql
INSERT INTO table_name (column1, column2, ...)
VALUES
    (value_a1, value_a2, ...),
    (value_b1, value_b2, ...),
    (value_c1, value_c2, ...);
```

**Example:**
```sql
INSERT INTO products (product_name, category, price)
VALUES
    ('Laptop', 'Electronics', 1200.00),
    ('Keyboard', 'Electronics', 75.50),
    ('Coffee Mug', 'Kitchenware', 15.00);
```
This single statement inserts three rows into the `products` table.

### Performance Benefits
Why is this so much better than three separate `INSERT` statements?

| Feature | Single `INSERT` with Multiple `VALUES` | Multiple `INSERT` Statements |
| :--- | :--- | :--- |
| **Network Round Trips** | 1 | 3 |
| **Query Parsing** | 1 | 3 |
| **Transaction Overhead** | 1 (atomic) | 3 (if auto-commit is on) |
| **Locking** | More efficient table/index locking | Repeatedly acquires and releases locks |
| **Overall Speed** | **Much Faster** | Much Slower |

For loading hundreds or thousands of rows, the performance difference is dramatic.

### `INSERT ... SELECT`
This is another powerful way to insert multiple rows, where the data comes from the result of a `SELECT` query. This is covered in detail in the `INSERT INTO` topic but is fundamentally a multi-row insert operation.

**Example: Populating a `subscribers` table from the `users` table**
```sql
INSERT INTO subscribers (email, subscription_date)
SELECT email, NOW()
FROM users
WHERE has_opted_in = TRUE;
```
This could insert thousands of rows in a single, efficient operation.

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you insert multiple rows into a table in a single command?"**
    -   "You can provide a comma-separated list of value tuples in the `VALUES` clause. For example: `INSERT INTO my_table (col1, col2) VALUES (1, 'a'), (2, 'b'), (3, 'c');`."

2.  **"Why is it better to insert multiple rows at once instead of one by one in a loop?"**
    -   "It's significantly more performant. A single multi-row `INSERT` reduces network latency, database parsing overhead, and transaction log writes. It's a crucial optimization for any kind of bulk data loading."

3.  **"What are the two main ways to perform a bulk insert?"**
    -   "The first is using the `INSERT ... VALUES (), (), ...` syntax for inserting explicit data. The second is using the `INSERT ... SELECT` statement to insert data from another table or query result."

### How to Explain in Interviews
"For inserting multiple rows, I always use the `INSERT` statement with a multi-tuple `VALUES` clause. This is a fundamental performance optimization. Instead of my application sending, say, 100 separate `INSERT` commands to the database, it sends just one. This minimizes network round trips and allows the database to process the batch of rows much more efficiently. For copying data between tables, I use `INSERT ... SELECT`, which is the standard for bulk data migration."

## Quick Recall ✅

-   **Syntax**: `INSERT INTO table (cols) VALUES (row1), (row2), (row3);`
-   **Performance**: Significantly faster than single-row inserts.
-   **Key Benefit**: Reduces network overhead and database workload.
-   **Alternative Method**: `INSERT INTO ... SELECT ...` for inserting from a query.
-   **Atomicity**: A single multi-row `INSERT` statement is atomic. If one row fails (e.g., due to a constraint violation), the entire statement is rolled back, and no rows are inserted.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Exceeding Packet Size Limits**
    -   While a single `INSERT` is efficient, trying to insert too many rows at once (e.g., hundreds of thousands) can create a query string that is too large for the database's configured maximum packet size (`max_allowed_packet` in MySQL).
    -   **Solution**: Batch your inserts. Instead of inserting 100,000 rows at once, insert them in batches of 1,000.

2.  **Forgetting that the whole statement is atomic.**
    -   If you are inserting 1,000 rows and row #999 has an error (e.g., violates a `UNIQUE` key), **none** of the 1,000 rows will be inserted. The entire command fails. This is a feature (atomicity), but it can be surprising if you're not expecting it.

3.  **Syntax Errors**
    -   A missing comma between tuples or an extra comma at the end of the list are common syntax errors.
    -   `... VALUES (1, 'a'), (2, 'b')` ✅
    -   `... VALUES (1, 'a') (2, 'b')` ❌ (missing comma)
    -   `... VALUES (1, 'a'), (2, 'b'),` ❌ (trailing comma)

### Tricky Interview Scenarios

-   **"You need to insert 1 million rows from a file into a database table as quickly as possible. An `INSERT` statement is too slow. What do you do?"**
    -   This question pushes beyond basic SQL to test knowledge of database-specific bulk-loading tools, which are even faster than multi-row `INSERT`s.
    -   **MySQL**: "I would use the `LOAD DATA INFILE` command. It's a highly optimized utility that reads data directly from a file and loads it into a table, bypassing much of the overhead of SQL parsing."
    -   **PostgreSQL**: "I would use the `COPY` command. Similar to MySQL's `LOAD DATA`, it's designed for extremely fast bulk loading from a file."
    -   **SQL Server**: "I would use the `BULK INSERT` command or the `bcp` utility."
    -   Mentioning these tools shows senior-level, practical knowledge.

-   **"You are inserting multiple rows, but you want the database to ignore any rows that would cause a duplicate key error. How would you do that?"**
    -   **MySQL**: Use `INSERT IGNORE`.
        ```sql
        INSERT IGNORE INTO products (product_code, product_name)
        VALUES
            ('A1', 'Product A'), -- Inserts if A1 is new
            ('B2', 'Product B'); -- Is ignored if B2 already exists
        ```
    -   **PostgreSQL**: Use `ON CONFLICT ... DO NOTHING`.
        ```sql
g        INSERT INTO products (product_code, product_name)
        VALUES
            ('A1', 'Product A'),
            ('B2', 'Product B')
        ON CONFLICT (product_code) DO NOTHING;
        ```
    -   This demonstrates knowledge of error handling within DML statements.

## Bonus

### Related Concepts
-   **ETL (Extract, Transform, Load)**: Multi-row inserts are the heart of the "Load" phase in ETL processes, where transformed data is loaded into a data warehouse.
-   **Database Drivers and ORMs**: Modern database drivers and Object-Relational Mappers (ORMs) often have built-in support for "batching," where they automatically group multiple `INSERT` or `UPDATE` commands together to gain this performance benefit.
-   **Transaction Control (TCL)**: For maximum performance when using multiple `INSERT` statements (if you can't use a single one), you can wrap them in a transaction to reduce commit overhead.
    ```sql
    START TRANSACTION;
    INSERT INTO my_table (col) VALUES (1);
    INSERT INTO my_table (col) VALUES (2);
    INSERT INTO my_table (col) VALUES (3);
    COMMIT;
    ```
    This is still slower than a single multi-row `INSERT` but much faster than three separate auto-committed inserts.
