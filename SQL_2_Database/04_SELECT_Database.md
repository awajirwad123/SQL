# SELECT Database (USE Command)

## Overview
In SQL, you often work with multiple databases on the same server. The `USE` command (or its equivalent) allows you to select a database to act as the default or "current" context for all subsequent commands. This means you don't have to specify the database name for every table you query.

## Core Concepts

### The `USE` Command
The most common command for selecting a database is `USE`. It is supported by popular systems like MySQL, MariaDB, and SQL Server.

**Syntax:**
```sql
USE database_name;
```

**Example:**
```sql
-- List all available databases
SHOW DATABASES;

-- Select the 'ecommerce_db' to work with
USE ecommerce_db;

-- Now, all subsequent queries run against 'ecommerce_db'
SELECT * FROM products; -- This queries ecommerce_db.products
SELECT * FROM users;    -- This queries ecommerce_db.users
```

### Why is it important?
-   **Convenience**: It saves you from typing the database name repeatedly.
-   **Readability**: Queries are cleaner and easier to read.
-   **Portability**: Scripts are easier to move between databases if they don't have hardcoded database names.

### Fully Qualified Table Names
Even after selecting a database with `USE`, you can still query tables in other databases by using a **fully qualified name**.

**Syntax**: `database_name.table_name` (in MySQL) or `database_name.schema_name.table_name` (in SQL Server/PostgreSQL).

**Example (MySQL):**
```sql
USE ecommerce_db;

-- This query joins a table from 'ecommerce_db' with a table from 'inventory_db'
SELECT
    p.product_name,
    s.stock_level
FROM
    products AS p -- 'products' is in the current db (ecommerce_db)
JOIN
    inventory_db.stock AS s ON p.sku = s.sku; -- 'stock' is in another db
```

### System-Specific Equivalents

#### PostgreSQL
PostgreSQL does not have a `USE` command. Instead, you use the `\c` (connect) meta-command in its command-line tool, `psql`.
```sql
-- In psql
\c database_name;
```
When connecting to PostgreSQL from an application, the database is specified in the connection string, so there is often no need to switch databases within a session.

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you switch to a different database in your SQL session?"**
    -   "In MySQL or SQL Server, I use the `USE database_name;` command. This sets the default database for all following queries in that session."

2.  **"What happens if you try to query a table without selecting a database first?"**
    -   "You will get an error, typically 'No database selected'. The server doesn't know which database's table you are referring to."

3.  **"Can you query a table from a database that you haven't `USE`d?"**
    -   "Yes, by using its fully qualified name. For example, in MySQL, you can write `SELECT * FROM other_db.my_table;` even if your current database is `main_db`."

4.  **"How does PostgreSQL handle switching databases?"**
    -   "PostgreSQL's command-line tool, `psql`, uses the `\c` meta-command to connect to a different database. There is no `USE` statement in PostgreSQL's SQL dialect."

### How to Explain in Interviews
"The `USE` command is a simple but essential part of my workflow in MySQL or SQL Server. It sets the session's context to a specific database, which simplifies queries by allowing me to refer to tables directly. For cross-database queries, I use fully qualified names like `database.table` to avoid ambiguity and join data from different sources."

## Quick Recall ✅

-   **Select DB Command**: `USE database_name;`
-   **Applies To**: MySQL, MariaDB, SQL Server.
-   **PostgreSQL Equivalent**: `\c database_name` (in `psql` tool).
-   **Purpose**: Sets the default database for the current session.
-   **Error if not used**: "No database selected".
-   **Override**: Use fully qualified names (e.g., `db_name.table_name`).

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `USE` is Standard SQL**
    -   `USE` is not part of the official SQL standard. It's an extension implemented by several major RDBMSs. Knowing this shows a deeper understanding.

2.  **Forgetting which database is active**
    -   In a long script or interactive session, it's easy to forget which database is currently active. This can lead to running queries against the wrong database, which is especially dangerous for `UPDATE` or `DELETE` statements.
    -   **Good Practice**: Some developers explicitly add a `USE` statement at the top of every script to be certain.

3.  **Using `USE` in PostgreSQL**
    -   Running `USE db_name;` in PostgreSQL will result in a syntax error. This is a common trip-up for developers who switch between MySQL and PostgreSQL.

### Tricky Interview Scenarios

-   **"Your script needs to run commands on two different databases. How would you write it?"**
    -   This tests your knowledge of both `USE` and fully qualified names.
    ```sql
    -- Method 1: Switch context
    USE db1;
    UPDATE table1 SET status = 'processed';

    USE db2;
    INSERT INTO log_table (message) VALUES ('Table1 in db1 was processed');

    -- Method 2: Use fully qualified names (often cleaner)
    UPDATE db1.table1 SET status = 'processed';
    INSERT INTO db2.log_table (message) VALUES ('Table1 in db1 was processed');
    ```
    Method 2 is often preferred as it's more explicit and less prone to errors from forgetting the current context.

-   **"Does `USE` commit a transaction?"**
    -   No. Unlike DDL commands, `USE` does not commit an open transaction. It simply changes the default database for the session.
    ```sql
    START TRANSACTION;
    UPDATE table_a SET col = 1;
    USE other_db; -- Transaction is still active
    -- You can continue the transaction on the new database if needed
    UPDATE table_b SET col = 2;
    COMMIT; -- Commits changes to both table_a and table_b
    ```

## Bonus

### Related Concepts
-   **Connection Strings**: In application code, the database is typically specified in the connection string. The `USE` command is more for interactive sessions or administrative scripts.
-   **Schemas**: In systems like PostgreSQL and SQL Server, the search path can be more complex. You might need to specify `database.schema.table`. The `USE` command sets the database, but you might also need to set the default schema.
-   **`information_schema`**: This is a special database/schema that exists in most SQL systems. You can query it to get metadata about other databases without having to `USE` them.
    ```sql
    -- Find all tables in a specific database without using USE
    SELECT table_name
    FROM information_schema.tables
    WHERE table_schema = 'ecommerce_db';
    ```

### Checking the Current Database
How do you check which database is currently selected?
```sql
-- MySQL / MariaDB
SELECT DATABASE();

-- SQL Server
SELECT DB_NAME();

-- PostgreSQL
SELECT current_database();
```
This is very useful for debugging or in complex scripts to confirm the context before running a critical command.
