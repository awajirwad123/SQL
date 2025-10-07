# RENAME TABLE

## Overview
Renaming a table is a common administrative task, often needed to reflect changes in business logic or to clean up legacy naming conventions. While the concept is simple, the exact SQL command can differ across database systems.

## Core Concepts

### Standard SQL Syntax (`ALTER TABLE`)
The most common and standard-compliant way to rename a table is using the `ALTER TABLE` command. This works in PostgreSQL, Oracle, and recent versions of MySQL.
```sql
ALTER TABLE old_table_name RENAME TO new_table_name;
```
**Example:**
```sql
ALTER TABLE customers RENAME TO clients;
```

### MySQL-Specific Syntax (`RENAME TABLE`)
MySQL also has a dedicated `RENAME TABLE` command, which can rename multiple tables at once.
```sql
RENAME TABLE old_table_name TO new_table_name;
```
**Example:**
```sql
-- Rename a single table
RENAME TABLE products TO items;

-- Rename multiple tables in one go
RENAME TABLE products TO items,
             suppliers TO vendors;
```
**Note**: While `RENAME TABLE` works, `ALTER TABLE ... RENAME TO` is now the recommended way in MySQL for better compatibility.

### SQL Server-Specific Syntax (`sp_rename`)
SQL Server uses a system stored procedure called `sp_rename` to rename objects, including tables.
```sql
EXEC sp_rename 'old_table_name', 'new_table_name';
```
**Example:**
```sql
EXEC sp_rename 'dbo.customers', 'dbo.clients';
```
Note the use of the schema name (`dbo.`). SQL Server will warn you that renaming an object can break scripts and stored procedures.

### What happens when you rename a table?
-   The table's data remains intact.
-   Indexes and primary key constraints associated with the table are automatically renamed/transferred.
-   **However, other objects are often NOT updated automatically**, which is the main risk.

**Objects that might break:**
-   Views that select from the old table name.
-   Foreign key constraints in other tables that reference the old table name.
-   Stored procedures and functions with hardcoded references to the old table name.
-   Triggers on the table (behavior can vary).

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you rename a table?"**
    -   "The standard SQL way is `ALTER TABLE old_name RENAME TO new_name;`. However, the exact syntax can vary. For example, SQL Server uses a system stored procedure called `sp_rename`."

2.  **"What are the risks associated with renaming a table?"**
    -   "The biggest risk is breaking dependencies. Renaming a table doesn't automatically update views, stored procedures, or foreign key constraints that refer to the old name. This can cause parts of the application to fail. It requires careful planning to identify and update all dependent objects."

3.  **"Does renaming a table also rename its indexes?"**
    -   "Yes, generally, indexes and primary key constraints that belong to the table are automatically handled. However, foreign key constraints in *other* tables that point to the renamed table are not."

### How to Explain in Interviews
"To rename a table, I typically use the `ALTER TABLE ... RENAME TO` command, which is widely supported. Before running it, however, my main focus is on identifying potential breaking changes. I would query the database's metadata (like `information_schema`) to find all views, foreign keys, and stored procedures that reference the table. Then, I'd create a migration script to rename the table and update all dependent objects in a single transaction to minimize downtime and risk."

## Quick Recall ✅

-   **Standard Command**: `ALTER TABLE old_name RENAME TO new_name;`
-   **MySQL Alternative**: `RENAME TABLE old_name TO new_name;`
-   **SQL Server Command**: `EXEC sp_rename 'old_name', 'new_name';`
-   **Main Risk**: Breaking dependent objects (views, foreign keys, stored procedures).
-   **What is preserved**: Data, indexes, and primary keys.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming the rename is atomic for all dependencies.**
    -   A junior developer might think renaming a table magically fixes everything. It doesn't. The core task isn't running the rename command; it's managing the fallout.

2.  **Forgetting about application code.**
    -   Even if you fix all database dependencies, the application code itself (e.g., Java, Python, C#) will have hardcoded queries with the old table name. These will break. Renaming a table requires a coordinated code and database change.

3.  **Permissions**
    -   You need `ALTER` and `DROP` permissions on the old table, and `CREATE` and `INSERT` permissions on the new table. Insufficient permissions can cause the operation to fail.

### Tricky Interview Scenarios

-   **"You've been asked to rename a critical table in a live production database with minimal downtime. What is your plan?"**
    -   This tests your understanding of production changes and risk management.
        1.  **Preparation (No Downtime)**:
            -   Identify all dependent database objects (views, foreign keys, etc.).
            -   Prepare a migration script that will:
                -   Rename the table.
                -   Update all dependent objects.
            -   Coordinate with the development team to prepare a new version of the application with the updated table name.
        2.  **Execution (Minimal Downtime Window)**:
            -   Schedule a short maintenance window.
            -   Put the application into maintenance mode.
            -   Take a quick backup of the database.
            -   Run the migration script within a single transaction.
            -   Deploy the new version of the application.
            -   Perform smoke tests to ensure everything is working.
            -   End the maintenance window.

-   **"Can you rename a table to a name that already exists?"**
    -   No. The new table name must be unique within the database/schema. The command will fail with an error like "Table 'new_name' already exists."

## Bonus

### Related Concepts
-   **Database Migrations**: Renaming a table is a type of schema migration. Tools like Flyway or Liquibase are used to manage these changes in a version-controlled, automated way.
-   **`information_schema`**: Your best friend for finding dependencies. You can query `information_schema.views`, `information_schema.key_column_usage`, etc., to find where your table is being used.
-   **Views as an Abstraction Layer**: One strategy to make renaming easier in the future is to have applications query views instead of tables directly. If you need to rename the underlying table, you can just update the view definition, and the application code doesn't need to change. This is known as using a "stable interface."

**Example: Using a view for abstraction**
```sql
-- Application queries this view
CREATE VIEW v_clients AS SELECT * FROM customers;

-- Later, you need to rename the table
RENAME TABLE customers TO customers_archive;
-- Now, just update the view. The application is unaffected.
ALTER VIEW v_clients AS SELECT * FROM customers_archive;
```
This is an advanced technique that shows a deep understanding of database design.
