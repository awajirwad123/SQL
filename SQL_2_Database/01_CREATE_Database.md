# CREATE Database

## Overview
The `CREATE DATABASE` command is the foundational DDL statement used to create a new, empty database in a SQL server. This is the first step before creating tables and adding data. In interviews, this tests your knowledge of basic setup and configuration options like character sets.

## Core Concepts

### Basic Syntax
The simplest way to create a database is:
```sql
CREATE DATABASE database_name;
```
**Example:**
```sql
CREATE DATABASE ecommerce_db;
```

### Using `IF NOT EXISTS`
To prevent an error if the database already exists, use the `IF NOT EXISTS` clause. This is a best practice in scripts.
```sql
CREATE DATABASE IF NOT EXISTS ecommerce_db;
```
This command will:
- Create `ecommerce_db` if it doesn't exist.
- Do nothing (and not throw an error) if it already exists.

### Specifying Options (Character Set & Collation)
You can specify the default character set and collation for the database. This determines how text is stored and sorted.

- **Character Set**: The set of characters that can be stored (e.g., `utf8mb4`).
- **Collation**: The rules for comparing and sorting characters (e.g., `utf8mb4_unicode_ci`).

```sql
CREATE DATABASE ecommerce_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;
```
- `utf8mb4`: A modern character set that supports emojis and most world languages.
- `utf8mb4_unicode_ci`: A collation that is case-insensitive (`_ci`).

### Viewing Databases
After creation, you can list all databases to confirm it exists.
```sql
-- MySQL / MariaDB
SHOW DATABASES;

-- PostgreSQL
\l

-- SQL Server
SELECT name FROM sys.databases;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you create a new database in SQL?"**
    -   Use the `CREATE DATABASE database_name;` command.

2.  **"What is the purpose of `IF NOT EXISTS`?"**
    -   It prevents the script from failing if the database already exists, making scripts more robust and re-runnable.

3.  **"Why is it important to specify a character set like `utf8mb4`?"**
    -   To ensure proper storage and handling of a wide range of characters, including international symbols and emojis. Using the right character set prevents data corruption or errors when storing multilingual text.

4.  **"What permissions are typically required to create a database?"**
    -   You usually need administrative privileges, such as the `CREATE` privilege on the server. A standard user cannot create databases.

### How to Explain in Interviews
"To create a database, I use the `CREATE DATABASE` command. I almost always include `IF NOT EXISTS` to make my setup scripts idempotent. I also specify the character set and collation, typically `utf8mb4` and `utf8mb4_unicode_ci`, to ensure modern text and emoji support, which is crucial for global applications."

## Quick Recall ✅

-   **Create DB**: `CREATE DATABASE db_name;`
-   **Safe Create**: `CREATE DATABASE IF NOT EXISTS db_name;`
-   **List DBs (MySQL)**: `SHOW DATABASES;`
-   **Recommended Character Set**: `utf8mb4` (supports emojis)
-   **Recommended Collation**: `utf8mb4_unicode_ci` (case-insensitive sorting)

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting the Semicolon**
    -   `CREATE DATABASE my_db` ❌ (Might work in some clients, but bad practice)
    -   `CREATE DATABASE my_db;` ✅ (Standard SQL syntax)

2.  **Permissions Error**
    -   Running `CREATE DATABASE` as a non-admin user will result in an "Access denied" or "Permission denied" error. This is a security feature, not a syntax error.

3.  **Database Already Exists Error**
    -   Running `CREATE DATABASE my_db;` twice will cause an error. This is why `IF NOT EXISTS` is so important in automated scripts.

4.  **Invalid Database Name**
    -   Database names cannot contain spaces or special characters (hyphens are often problematic). Use underscores instead (e.g., `my_database` instead of `my-database`).

### Tricky Interview Scenarios

-   **"Can you create a database within a transaction?"**
    -   No. `CREATE DATABASE` is a DDL command that causes an implicit commit. It cannot be part of a transaction and cannot be rolled back.

-   **"What's the difference between a database and a schema?"**
    -   This is a classic trick question as it depends on the SQL dialect:
        -   **MySQL**: A `SCHEMA` is a synonym for `DATABASE`. They are the same thing. `CREATE SCHEMA` is the same as `CREATE DATABASE`.
        -   **PostgreSQL / SQL Server**: A `DATABASE` is a container for multiple `SCHEMAS`. A schema is a namespace within a database that holds tables, views, etc. (e.g., `database.schema.table`).

## Bonus

### Related Concepts
-   **DDL (Data Definition Language)**: `CREATE DATABASE` is a core DDL command, alongside `CREATE TABLE`, `ALTER`, and `DROP`.
-   **Character Sets & Collations**: Understanding these is key to internationalization (i18n) and proper data handling.
-   **Database Users & Privileges (DCL)**: Creating a database is often followed by creating a dedicated user and granting it permissions on that database.

### Example: Full Setup Script
Here’s a typical sequence in a setup script:
```sql
-- 1. Create the database safely
CREATE DATABASE IF NOT EXISTS my_app_db
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

-- 2. Switch to the new database
USE my_app_db;

-- 3. Create a dedicated user for the application
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';

-- 4. Grant permissions to the user on the new database
GRANT ALL PRIVILEGES ON my_app_db.* TO 'app_user'@'localhost';

-- 5. Now, you can start creating tables...
CREATE TABLE users (
    id INT PRIMARY KEY,
    -- ...
);
```
This demonstrates a robust and secure way to initialize a new database environment.
