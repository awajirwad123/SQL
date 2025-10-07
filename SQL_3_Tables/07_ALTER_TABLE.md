# ALTER TABLE

## Overview
The `ALTER TABLE` command is a powerful DDL (Data Definition Language) statement used to modify the structure of an existing table. You can add, drop, or modify columns, as well as add or remove constraints. It's a fundamental command for evolving a database schema over time without having to drop and recreate tables.

## Core Concepts

`ALTER TABLE` is a versatile command with many sub-commands. Here are the most common operations:

### 1. Adding a Column
Use `ADD COLUMN` to add a new column to a table.
```sql
-- Add a single column
ALTER TABLE customers
ADD COLUMN last_login_date DATE;

-- Add a column with a default value
ALTER TABLE products
ADD COLUMN in_stock BOOLEAN DEFAULT TRUE;
```

### 2. Dropping a Column
Use `DROP COLUMN` to remove a column. Be careful, as this also deletes all the data in that column permanently.
```sql
ALTER TABLE customers
DROP COLUMN phone_number;
```

### 3. Modifying a Column
Use `MODIFY COLUMN` (MySQL/Oracle) or `ALTER COLUMN` (SQL Server/PostgreSQL) to change a column's data type, size, or `DEFAULT` value.

**Syntax (MySQL):**
```sql
-- Change data type
ALTER TABLE customers
MODIFY COLUMN customer_name VARCHAR(200);

-- Add a NOT NULL constraint
ALTER TABLE customers
MODIFY COLUMN email VARCHAR(255) NOT NULL;
```

**Syntax (PostgreSQL / SQL Server):**
```sql
-- Change data type
ALTER TABLE customers
ALTER COLUMN customer_name TYPE VARCHAR(200); -- PostgreSQL
ALTER TABLE customers
ALTER COLUMN customer_name VARCHAR(200); -- SQL Server

-- Add a default value
ALTER TABLE products
ALTER COLUMN price SET DEFAULT 9.99;
```

### 4. Renaming a Column
The syntax for renaming a column is highly vendor-specific.

**Syntax (MySQL):**
```sql
ALTER TABLE customers
CHANGE COLUMN old_column_name new_column_name VARCHAR(100);
-- Note: You must re-specify the data type in MySQL.
```

**Syntax (PostgreSQL / SQL Server):**
```sql
-- Rename column
ALTER TABLE customers
RENAME COLUMN old_column_name TO new_column_name; -- PostgreSQL

EXEC sp_rename 'customers.old_column_name', 'new_column_name', 'COLUMN'; -- SQL Server
```

### 5. Adding and Dropping Constraints
You can add or remove constraints like `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, and `CHECK`.

**Adding a Constraint:**
```sql
-- Add a UNIQUE constraint
ALTER TABLE users
ADD CONSTRAINT uq_email UNIQUE (email);

-- Add a FOREIGN KEY constraint
ALTER TABLE orders
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
```

**Dropping a Constraint:**
To drop a constraint, you usually need to know its name.
```sql
-- Drop a UNIQUE constraint
ALTER TABLE users
DROP CONSTRAINT uq_email;

-- Drop a FOREIGN KEY constraint
ALTER TABLE orders
DROP CONSTRAINT fk_customer; -- (or DROP FOREIGN KEY fk_customer in MySQL)
```

### Performing Multiple Operations
Some systems, like MySQL, allow you to combine multiple `ALTER` operations in a single statement for efficiency.
```sql
ALTER TABLE products
    ADD COLUMN manufacturer VARCHAR(100),
    MODIFY COLUMN product_name VARCHAR(250) NOT NULL,
    DROP COLUMN old_sku;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How would you add a new column to an existing table?"**
    -   "I would use the `ALTER TABLE table_name ADD COLUMN column_name datatype;` command. I'd also consider adding a `DEFAULT` value if it's a `NOT NULL` column being added to a non-empty table."

2.  **"What is the command to change a column's data type?"**
    -   "The command varies by system. In MySQL, it's `ALTER TABLE ... MODIFY COLUMN ...`. In PostgreSQL, it's `ALTER TABLE ... ALTER COLUMN ... TYPE ...`. It's an important distinction to be aware of."

3.  **"What are the risks of running `ALTER TABLE` on a large production table?"**
    -   "The main risk is locking. Many `ALTER` operations require a full table lock, which can block all reads and writes, causing significant application downtime. For large tables, this can take hours. It's crucial to use online schema change tools or perform these changes during a planned maintenance window."

4.  **"How do you add a foreign key to an existing table?"**
    -   "Using `ALTER TABLE table_name ADD CONSTRAINT constraint_name FOREIGN KEY (column_name) REFERENCES other_table(other_column);`. The data in the column must be valid (i.e., all values must exist in the parent table) for the command to succeed."

### How to Explain in Interviews
"`ALTER TABLE` is my go-to command for any structural modification to an existing table. I'm comfortable with adding, dropping, and modifying columns and constraints. However, I'm particularly cautious when using it on large production tables due to potential locking issues. In those scenarios, I would research the specific operation's locking behavior for the RDBMS in use and potentially use specialized online schema change tools like `pt-online-schema-change` or `gh-ost` to perform the migration without downtime."

## Quick Recall ✅

-   **Add Column**: `ALTER TABLE ... ADD COLUMN ...`
-   **Drop Column**: `ALTER TABLE ... DROP COLUMN ...`
-   **Modify Column (MySQL)**: `ALTER TABLE ... MODIFY COLUMN ...`
-   **Modify Column (PostgreSQL/SQL Server)**: `ALTER TABLE ... ALTER COLUMN ...`
-   **Add Constraint**: `ALTER TABLE ... ADD CONSTRAINT ...`
-   **Drop Constraint**: `ALTER TABLE ... DROP CONSTRAINT ...`
-   **Main Risk**: Table locking and downtime on large tables.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Adding a `NOT NULL` column to a table with existing data.**
    -   `ALTER TABLE users ADD COLUMN country VARCHAR(50) NOT NULL;` ❌
    -   This will fail on a non-empty table because the existing rows would have `NULL` for the new column, violating the `NOT NULL` constraint.
    -   **Solution**: Add the column as nullable first, update the existing rows to have a value, and then add the `NOT NULL` constraint.
        ```sql
        -- Step 1
        ALTER TABLE users ADD COLUMN country VARCHAR(50);
        -- Step 2
        UPDATE users SET country = 'USA' WHERE country IS NULL;
        -- Step 3
        ALTER TABLE users MODIFY COLUMN country VARCHAR(50) NOT NULL;
        ```
    -   Or, add the column with a `DEFAULT` value:
        ```sql
        ALTER TABLE users ADD COLUMN country VARCHAR(50) NOT NULL DEFAULT 'USA'; ✅
        ```

2.  **Changing a data type incompatibly.**
    -   Trying to change a `VARCHAR` column that contains text (e.g., "hello") to an `INT` will fail. The data must be convertible to the new type.

3.  **Underestimating the duration of the command.**
    -   An `ALTER TABLE` command on a table with millions of rows might take hours to complete, during which the application is down. This is a catastrophic mistake in a production environment.

### Tricky Interview Scenarios

-   **"You need to add a column to a 1-billion-row table with zero downtime. How do you do it?"**
    -   This is an advanced question designed to separate senior candidates from juniors.
    -   **Answer**: "A standard `ALTER TABLE` is not an option due to locking. The industry-standard solution is to use an online schema change tool. For MySQL, the most common tools are Percona's `pt-online-schema-change` or GitHub's `gh-ost`.
    -   **How they work (simplified)**:
        1.  They create a new, empty table with the desired new structure.
        2.  They create triggers on the original table to capture any ongoing changes (`INSERT`s, `UPDATE`s, `DELETE`s).
        3.  They copy data from the original table to the new table in small, non-locking chunks.
        4.  While the copy is happening, the triggers apply any new changes to the new table as well.
        5.  Once the copy is complete and the tables are in sync, they perform a quick, atomic `RENAME TABLE` to swap the original table with the new one.
    -   This process performs the migration online with minimal locking, achieving zero or near-zero downtime.

## Bonus

### Related Concepts
-   **Schema Migrations**: `ALTER TABLE` is the core command used in schema migration scripts. Tools like Flyway and Liquibase provide a version-controlled framework for applying these scripts.
-   **Locking**: Understanding row-level vs. table-level locks, and shared vs. exclusive locks, is critical for predicting the impact of an `ALTER TABLE` command.
-   **Online Schema Change Tools**: For large-scale systems, knowing about tools like `pt-online-schema-change` (Percona Toolkit) or `gh-ost` (GitHub) is a huge plus.
-   **`information_schema`**: Can be used to check a table's structure before and after an `ALTER` operation to verify the change.
