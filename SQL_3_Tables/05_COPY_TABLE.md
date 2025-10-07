# COPY TABLE

## Overview
Copying a table is a common task used for backups, testing, or creating summary tables. Standard SQL does not have a single `COPY TABLE` command. Instead, copying is achieved using a combination of `CREATE TABLE` and `INSERT` statements, with different methods available depending on whether you need to copy just the structure, just the data, or both.

## Core Concepts

There are two main approaches to copying a table.

### Method 1: `CREATE TABLE ... AS SELECT` (CTAS)
This is the most common and powerful method. It creates a new table and copies data into it in a single command.

**Syntax:**
```sql
CREATE TABLE new_table_name AS
SELECT * FROM original_table_name
[WHERE condition];
```

**What it does:**
-   Creates a new table (`new_table_name`).
-   The new table's columns will match the columns from the `SELECT` statement (in name and inferred data type).
-   Copies the data from the `SELECT` query into the new table.

**Example 1: Copying an entire table**
```sql
-- Creates 'products_backup' with the same columns and all the data from 'products'
CREATE TABLE products_backup AS
SELECT * FROM products;
```

**Example 2: Copying a subset of data**
```sql
-- Creates 'usa_customers' with only customers from the USA
CREATE TABLE usa_customers AS
SELECT * FROM customers WHERE country = 'USA';
```

**Example 3: Creating a summary table**
```sql
-- Creates a table with total sales per category
CREATE TABLE category_sales AS
SELECT category, SUM(price) AS total_revenue
FROM products
GROUP BY category;
```

**Important Limitation of CTAS**:
-   CTAS copies the column names, data types, and data.
-   However, it **does not** typically copy constraints like `PRIMARY KEY`, `FOREIGN KEY`, `INDEXES`, or `DEFAULT` values. These must be added separately using `ALTER TABLE`.

### Method 2: `CREATE TABLE LIKE` + `INSERT`
This two-step method provides more control by first copying the exact structure (including indexes in some systems) and then populating the data.

**Step 1: Copy the structure**
This syntax is specific to some database systems like MySQL.
```sql
-- Creates 'products_backup' with the exact same structure as 'products',
-- including columns, data types, and indexes, but with no data.
CREATE TABLE products_backup LIKE products;
```
*   **PostgreSQL equivalent**: `CREATE TABLE products_backup (LIKE products INCLUDING ALL);`
*   **SQL Server workaround**: Often done through GUI tools or by scripting the `CREATE TABLE` statement.

**Step 2: Copy the data**
Use a standard `INSERT ... SELECT` statement.
```sql
INSERT INTO products_backup
SELECT * FROM products;
```

### Comparison of Methods

| Feature | `CREATE TABLE ... AS SELECT` (CTAS) | `CREATE TABLE LIKE` + `INSERT` |
| :--- | :--- | :--- |
| **Steps** | 1 | 2 |
| **Copies Structure** | Yes (basic) | Yes (exact, often including indexes) |
| **Copies Data** | Yes | Yes (in the `INSERT` step) |
| **Copies Constraints/Indexes**| **No** (generally) | **Yes** (generally) |
| **Use Case** | Quick backups, summary tables, data transformation. | Creating an exact structural duplicate for testing or as a new permanent table. |

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you copy a table in SQL?"**
    -   "There's no single `COPY TABLE` command. The most common way is using `CREATE TABLE new_table AS SELECT * FROM old_table;`. This copies the structure and data but usually not the constraints or indexes."

2.  **"What is the limitation of `CREATE TABLE ... AS SELECT`?"**
    -   "Its main limitation is that it doesn't copy constraints like primary keys, foreign keys, or indexes. You have to add those manually with `ALTER TABLE` after the table is created."

3.  **"You need to create an exact copy of a table, including all its indexes and constraints. How would you do it?"**
    -   "I would use a two-step process. First, I'd create an empty table with the exact same structure. In MySQL, I can use `CREATE TABLE new_table LIKE old_table;`. Then, I would populate it with data using `INSERT INTO new_table SELECT * FROM old_table;`."

### How to Explain in Interviews
"When I need to copy a table, my method depends on the goal. For quick, temporary backups or to create a transformed subset of data, I use `CREATE TABLE ... AS SELECT`. It's fast and efficient. If I need a perfect, persistent duplicate with all the original constraints and indexes, I prefer the two-step approach: first, replicate the schema using a command like `CREATE TABLE LIKE`, and then populate the data with an `INSERT ... SELECT` statement."

## Quick Recall ✅

-   **No `COPY TABLE` command** in standard SQL.
-   **Method 1 (Data + Basic Structure)**: `CREATE TABLE new_table AS SELECT * FROM old_table;`
-   **Limitation of Method 1**: Does **not** copy keys, indexes, or constraints.
-   **Method 2 (Exact Structure + Data)**:
    1.  `CREATE TABLE new_table LIKE old_table;` (MySQL)
    2.  `INSERT INTO new_table SELECT * FROM old_table;`
-   **Use CTAS for**: Backups, summaries, and transformed data.
-   **Use `LIKE` + `INSERT` for**: Perfect structural clones.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming constraints are copied with CTAS.**
    -   A developer might create a backup with CTAS and assume it's a perfect copy, only to find out later that the primary key and foreign key relationships are missing, leading to data integrity issues.

2.  **`SELECT *` in production code.**
    -   While `SELECT *` is convenient for copying tables, it's bad practice in production application code because it can break if the source table's column order or number changes. For copying, it's generally acceptable, but it's good to be aware of the risk.

3.  **Data Type Mismatches.**
    -   When using CTAS, the database infers the data types for the new table from the result of the `SELECT`. This can sometimes lead to unexpected type changes (e.g., a specific `DECIMAL` might become a more generic `NUMERIC`).

### Tricky Interview Scenarios

-   **"How would you copy just the structure of a table, without any data?"**
    -   **Method A**: Use `CREATE TABLE ... LIKE ...`.
        ```sql
        CREATE TABLE new_table LIKE old_table; -- (MySQL)
        ```
    -   **Method B (Universal)**: Use a CTAS query with a `WHERE` clause that is never true.
        ```sql
        CREATE TABLE new_table AS SELECT * FROM old_table WHERE 1=0;
        ```
        This is a clever trick. The `SELECT` statement returns no rows, so the new table is created with the same column structure but is completely empty. However, this still doesn't copy constraints.

-   **"You need to copy a table into a database on a different server. How would you do it?"**
    -   You can't use `CREATE TABLE AS SELECT` directly across servers. This is where backup tools come in.
    -   **Answer**: "I would use a database utility like `mysqldump` to export the table's structure and data from the source server into a `.sql` file. Then, I would transfer that file to the destination server and execute it using the `mysql` client to recreate the table and its data."

## Bonus

### Related Concepts
-   **Data Warehousing (ETL)**: The `CREATE TABLE ... AS SELECT` pattern is fundamental in ETL (Extract, Transform, Load) processes. It's often used to create staging tables or final, aggregated data marts.
-   **`information_schema`**: You can use the `information_schema` to programmatically generate the `CREATE TABLE` statement for any table, which is another way to get an exact structural copy. This is what many GUI tools do behind the scenes.
-   **Materialized Views**: A materialized view is essentially a query result that is stored as a physical table (`CREATE TABLE ... AS SELECT` is a manual way of creating one). The database can then refresh this table periodically. This is an advanced concept for performance optimization.
