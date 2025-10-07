# SQL Database Operations

## Overview
Beyond creating, dropping, or selecting a database, there are several other administrative operations used to manage and monitor databases. These commands help DBAs and developers understand the state, structure, and health of a database. This is a practical topic that interviewers use to gauge real-world experience.

## Core Concepts

### 1. Listing Databases
A fundamental operation is to see all the databases available on the server.

**Syntax:**
```sql
-- MySQL / MariaDB
SHOW DATABASES;

-- PostgreSQL (in psql tool)
\l
-- or as a SQL query:
SELECT datname FROM pg_database;

-- SQL Server
SELECT name FROM sys.databases;
-- or use the stored procedure:
EXEC sp_databases;
```

### 2. Viewing Database Creation Syntax
It's often useful to see the `CREATE` statement that was used to create a database, as it includes details like the character set and collation.

**Syntax (MySQL / MariaDB):**
```sql
SHOW CREATE DATABASE database_name;
```
**Example Output:**
```sql
+------------+-----------------------------------------------------------------+
| Database   | Create Database                                                 |
+------------+-----------------------------------------------------------------+
| my_app_db  | CREATE DATABASE `my_app_db` /*!40100 DEFAULT CHARACTER SET ... */ |
+------------+-----------------------------------------------------------------+
```

### 3. Backing Up a Database
Backing up a database is a critical operation for disaster recovery. This is typically done using command-line tools provided with the database system, not with a direct SQL command.

-   **Logical Backup**: Saves the data as SQL statements (`CREATE TABLE`, `INSERT`, etc.). It is human-readable and portable across versions but can be slow to restore.
-   **Physical Backup**: A file-level copy of the database. It is much faster for large databases but less flexible.

**Common Tools:**
-   **MySQL/MariaDB**: `mysqldump`
    ```bash
    # Logical backup of a single database
    mysqldump -u username -p database_name > backup_file.sql
    ```
-   **PostgreSQL**: `pg_dump`
    ```bash
    # Logical backup
    pg_dump -U username -d database_name -f backup_file.sql
    ```
-   **SQL Server**: Typically done through SQL Server Management Studio (SSMS) GUI or T-SQL `BACKUP DATABASE` command.
    ```sql
    BACKUP DATABASE database_name
    TO DISK = 'C:\path\to\backup.bak';
    ```

### 4. Restoring a Database
Restoring is the process of recreating a database from a backup file.

**Common Tools:**
-   **MySQL/MariaDB**: `mysql` client
    ```bash
    # First, create an empty database
    mysql -u username -p -e "CREATE DATABASE new_db;"
    # Then, import the data
    mysql -u username -p new_db < backup_file.sql
    ```
-   **PostgreSQL**: `psql` client
    ```bash
    # Create an empty database first
    createdb -U username new_db
    # Import the data
    psql -U username -d new_db -f backup_file.sql
    ```
-   **SQL Server**: `RESTORE DATABASE` command.
    ```sql
    RESTORE DATABASE new_db
    FROM DISK = 'C:\path\to\backup.bak';
    ```

### 5. Altering Database Characteristics
You can change certain characteristics of an existing database, like its character set.

**Syntax (MySQL / MariaDB):**
```sql
ALTER DATABASE database_name
CHARACTER SET new_character_set
COLLATE new_collation;
```
**Example:**
```sql
ALTER DATABASE my_app_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
**Note**: This only affects new tables created in the database. Existing tables must be altered individually.

## Interview-Focused Notes

### Common Interview Questions

1.  **"How would you list all databases on a server?"**
    -   "It depends on the system. In MySQL, I'd use `SHOW DATABASES;`. In PostgreSQL, I'd use `\l` in `psql` or query the `pg_database` catalog. In SQL Server, I'd query `sys.databases`."

2.  **"Describe the process of backing up and restoring a MySQL database."**
    -   "For a logical backup, I'd use the `mysqldump` utility to export the database to a `.sql` file. To restore, I would first create a new, empty database and then use the `mysql` client to import the `.sql` file into it."

3.  **"What is the difference between a logical and a physical backup?"**
    -   "A logical backup contains the SQL commands needed to recreate the data (like `INSERT` statements) and is portable but slower to restore. A physical backup is a direct copy of the data files, making it much faster for large databases but less flexible across different server versions or architectures."

4.  **"You need to move a database from one server to another. How would you do it?"**
    -   "The most common method is to perform a logical backup from the source server (e.g., using `mysqldump`) and then restore that backup on the destination server. This ensures compatibility even if the servers have different versions."

### How to Explain in Interviews
"Beyond simple `CREATE` and `DROP` commands, managing a database involves several key operations. I'm familiar with listing databases using `SHOW DATABASES` in MySQL, and I understand the critical importance of backups. I typically use command-line utilities like `mysqldump` or `pg_dump` to create logical backups, which are great for migration and recovery. For restoring, I'd import that backup into a newly created database."

## Quick Recall ✅

-   **List DBs (MySQL)**: `SHOW DATABASES;`
-   **List DBs (PostgreSQL)**: `\l` or `SELECT datname FROM pg_database;`
-   **View DB Create (MySQL)**: `SHOW CREATE DATABASE db_name;`
-   **Backup Tool (MySQL)**: `mysqldump`
-   **Backup Tool (PostgreSQL)**: `pg_dump`
-   **Restore Tool (MySQL)**: `mysql`
-   **Restore Tool (PostgreSQL)**: `psql`
-   **Alter DB**: `ALTER DATABASE db_name CHARACTER SET ...;`

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Confusing SQL Commands with CLI Tools**
    -   `mysqldump` is a command-line program, not an SQL command. You run it from your shell (like Bash or PowerShell), not from within an SQL client.

2.  **Thinking `BACKUP DATABASE` is Standard SQL**
    -   The `BACKUP DATABASE` command is specific to systems like SQL Server. It is not part of the SQL standard and does not exist in MySQL or PostgreSQL.

3.  **Restoring into a Non-Empty Database**
    -   Trying to import a full backup into a database that already contains tables can lead to errors (e.g., "Table already exists"). The standard practice is to restore into a fresh, empty database.

4.  **Ignoring Character Set Issues during Backup/Restore**
    -   If the source and destination servers have different default character sets, you can introduce data corruption during a restore. It's important to ensure consistency.

### Tricky Interview Scenarios

-   **"How would you copy a database without using `mysqldump` or any command-line tools?"**
    -   This tests creative, in-SQL solutions. It's not efficient but possible for small databases.
        1.  Run `SHOW CREATE TABLE table_name;` for every table in the old database.
        2.  Execute those `CREATE TABLE` statements in the new database.
        3.  For each table, run `INSERT INTO new_db.table_name SELECT * FROM old_db.table_name;`.
    -   You should immediately point out that this is slow, error-prone, and doesn't copy views, triggers, or stored procedures, making it a poor choice compared to proper backup tools.

-   **"Your `mysqldump` backup file is 50GB. The restore is taking too long. What could be the issue or potential solutions?"**
    -   This probes your knowledge of performance.
        -   **Issue**: Logical restores are inherently slow because the database has to execute every single `INSERT` statement, check constraints, and build indexes.
        -   **Solutions**:
            1.  **Disable Foreign Key Checks**: Wrap the import in commands to disable and re-enable foreign key checks to speed up data insertion.
            2.  **Use a Physical Backup**: For very large databases (VLDBs), physical backups are the industry standard for fast recovery.
            3.  **Parallel Tools**: Use more advanced tools like `mydumper` and `myloader` (for MySQL) which can perform backups and restores in parallel.

## Bonus

### Related Concepts
-   **DBA (Database Administrator)**: These operations are the core responsibilities of a DBA.
-   **Replication**: A more advanced way to create a copy of a database. Replication sets up a live "copy" (a replica) of a primary database that stays in sync automatically. This is used for high availability and read scaling, not just backups.
-   **`information_schema`**: A standards-compliant database that provides metadata about all other databases. You can write complex queries against it to script administrative tasks.
    ```sql
    -- A script to generate TRUNCATE statements for all tables in a DB
    SELECT CONCAT('TRUNCATE TABLE ', table_name, ';')
    FROM information_schema.tables
    WHERE table_schema = 'my_app_db';
    ```
This kind of query demonstrates advanced, practical knowledge.
