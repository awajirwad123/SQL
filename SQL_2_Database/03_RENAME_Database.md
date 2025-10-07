# RENAME Database

## Overview
Renaming a database is a surprisingly tricky operation because **standard SQL does not have a `RENAME DATABASE` command**. The method for renaming a database is specific to the database system (RDBMS) you are using, and it often involves workarounds. This topic is a great interview question to test deeper, practical knowledge beyond standard SQL.

## Core Concepts

### The "No Standard Command" Rule
First and foremost, remember this:
```
There is no `ALTER DATABASE old_name RENAME TO new_name;` in the SQL standard.
```
Attempting to run such a command will fail in many systems like MySQL and SQL Server.

### Common Workarounds and System-Specific Commands

#### 1. The Universal Workaround (All Systems)
This is the most common and reliable method, though it can be slow for large databases.
1.  **Backup**: Create a full backup of the old database (`old_db`).
    -   Example using `mysqldump`: `mysqldump -u root -p old_db > old_db_backup.sql`
2.  **Create New Database**: Create a new, empty database with the desired name (`new_db`).
    -   `CREATE DATABASE new_db;`
3.  **Restore**: Restore the backup into the new database.
    -   Example using `mysql`: `mysql -u root -p new_db < old_db_backup.sql`
4.  **Drop Old Database**: Once you have verified the new database is working correctly, drop the old one.
    -   `DROP DATABASE old_db;`

#### 2. PostgreSQL (Has a Direct Command)
PostgreSQL is one of the few systems that provides a direct command to rename a database.
```sql
ALTER DATABASE old_name RENAME TO new_name;
```
**Note**: You cannot be connected to the database you are trying to rename. You must be connected to another database (like `postgres`) to run this command.

#### 3. MySQL / MariaDB (No Direct Command)
There is no single command to rename a database. You must use a workaround.
-   **Workaround 1**: The backup-and-restore method described above (recommended).
-   **Workaround 2 (Risky)**: Create a new database, then rename each table individually. This is tedious and can break things like stored procedures and triggers.
    ```sql
    CREATE DATABASE new_db;
    RENAME TABLE old_db.table1 TO new_db.table1;
    RENAME TABLE old_db.table2 TO new_db.table2;
    -- ... for all tables
    DROP DATABASE old_db;
    ```
    This method is **not recommended** as it doesn't move other objects like views, triggers, or foreign key constraints correctly.

#### 4. SQL Server (System Procedure)
SQL Server uses a system stored procedure, but the database must be in single-user mode.
```sql
-- Set the database to single-user mode
ALTER DATABASE OldDB SET SINGLE_USER WITH ROLLBACK IMMEDIATE;

-- Rename the database
EXEC sp_renamedb 'OldDB', 'NewDB';

-- Set it back to multi-user mode
ALTER DATABASE NewDB SET MULTI_USER;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you rename a database in SQL?"**
    -   Start by stating that there is no standard SQL command. Then, explain that the method is vendor-specific. Mention the backup/restore workaround as a universal method and cite a specific command if you know one (like PostgreSQL's `ALTER DATABASE ... RENAME TO`).

2.  **"Why is renaming a database not a simple operation?"**
    -   Because a database is a top-level container. Renaming it can have wide-ranging implications, affecting connection strings in applications, user permissions, scheduled jobs, and internal object references. Database systems treat it as a major administrative task.

3.  **"What are the risks of renaming a database?"**
    -   **Application Downtime**: Applications connected to the old database name will fail.
    -   **Broken References**: Stored procedures, views, or jobs that explicitly reference the old database name will break.
    -   **Incomplete Migration**: Manual workarounds (like renaming tables one by one) can miss objects, leading to an inconsistent state.

### How to Explain in Interviews
"Renaming a database isn't supported by a standard SQL command, which often surprises people. The approach depends on the RDBMS. For PostgreSQL, there's a direct `ALTER DATABASE ... RENAME TO` command. However, for systems like MySQL, the safest and most common method is to perform a full backup of the old database, create a new database with the desired name, restore the backup into it, and then drop the old one after verifying everything works."

## Quick Recall ✅

-   **Standard SQL**: No `RENAME DATABASE` command exists.
-   **Universal Method**: Backup -> Create New -> Restore -> Drop Old.
-   **PostgreSQL**: `ALTER DATABASE old_name RENAME TO new_name;`
-   **MySQL**: No direct command. Use the backup/restore workaround.
-   **SQL Server**: Use the `sp_renamedb` stored procedure in single-user mode.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `RENAME DATABASE` Exists**
    -   The most common trap is confidently answering, "You use `RENAME DATABASE`." This immediately shows a lack of practical, system-specific experience.

2.  **Suggesting Risky Workarounds**
    -   Suggesting the table-by-table `RENAME TABLE` method for MySQL without mentioning its significant risks (broken foreign keys, triggers, etc.) is a red flag. Always recommend the backup/restore method as the safest.

3.  **Not Mentioning Downtime**
    -   Any method of renaming a database will require application downtime or a maintenance window. Connection strings in all applications must be updated. Forgetting this practical aspect is a common oversight.

### Tricky Interview Scenarios

-   **"Write the SQL to rename a database called `db1` to `db2`."**
    -   This is a trick question. The correct first step is to ask, **"Which database system are we using?"**
    -   If the interviewer says "PostgreSQL," you write: `ALTER DATABASE db1 RENAME TO db2;`
    -   If they say "MySQL," you explain the backup/restore process and state that there is no direct SQL command.
    -   If they say "standard SQL," you state that it's not possible.

-   **"Your company's policy forbids dropping databases. How do you rename a MySQL database?"**
    -   This tests creative problem-solving. The backup/restore/drop method is out. The next best (though still risky) approach is the table-by-table rename, but you must outline the steps to mitigate risk:
        1.  Put the application in maintenance mode.
        2.  Create the new database.
        3.  Script the renaming of all tables, views, and other objects.
        4.  Manually update all stored procedures and triggers that might reference the old database name.
        5.  Thoroughly test the new database.
        6.  Keep the old database for a while as a fallback before eventually archiving it.

## Bonus

### Related Concepts
-   **Database Migration**: Renaming a database is a form of migration. The principles of planning, testing, and execution are similar.
-   **Backup and Restore**: This is the most fundamental DBA skill and the safest way to perform major structural changes like a rename.
-   **System Catalogs**: Database systems store metadata about databases, tables, and other objects in system catalogs (e.g., `information_schema`). Advanced renaming scripts would query these catalogs to ensure all objects are moved.

### Why is there no standard command?
The SQL standard was designed in an era where databases were more static. Renaming a top-level object has complex implications for security, file systems, and transaction logs, which are handled very differently by each vendor. This made it difficult to standardize a single, reliable command.
