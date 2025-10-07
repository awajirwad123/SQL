# Unique Constraint

## Overview
A `UNIQUE` constraint ensures that all values in a column or a set of columns are different from one another. It prevents duplicate values from being entered into the column(s) it is applied to. While similar to a `PRIMARY KEY` constraint in that it enforces uniqueness, there are key differences.

## Core Concepts

The `UNIQUE` constraint guarantees that no two rows of a table have the same value in the specified column(s).

**Key Properties:**
-   **Uniqueness**: Enforces that every value in the column is unique.
-   **`NULL` Handling**: This is a key differentiator. By default, most SQL databases (like PostgreSQL, SQL Server, Oracle) allow **multiple rows** to have `NULL` values in a unique column, because `NULL` is not considered equal to `NULL`. However, some, like MySQL, only allow **one `NULL`** value in a unique column.
-   **Multiple Constraints**: A single table can have multiple `UNIQUE` constraints.

**Syntax:**

1.  **During table creation (column level):**
    ```sql
    CREATE TABLE employees (
        employee_id INT PRIMARY KEY,
        email VARCHAR(100) UNIQUE,
        social_security_number VARCHAR(11) UNIQUE NOT NULL
    );
    ```

2.  **During table creation (table level):** This is required for a composite unique key.
    ```sql
    CREATE TABLE products (
        product_id INT PRIMARY KEY,
        product_code VARCHAR(20),
        manufacturer_id INT,
        UNIQUE (product_code, manufacturer_id)
    );
    ```

3.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE employees
    ADD CONSTRAINT uq_email UNIQUE (email);
    ```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a `UNIQUE` constraint?"**
    -   "A `UNIQUE` constraint ensures that all values in a column or a set of columns are distinct. It prevents duplicate entries, but unlike a primary key, a table can have multiple unique constraints, and they can, by default, allow `NULL` values."

2.  **"What is the difference between a `PRIMARY KEY` and a `UNIQUE` constraint?"**
    -   "This is a classic database question. There are two main differences:
        1.  A table can have only **one** `PRIMARY KEY`, but it can have **multiple** `UNIQUE` constraints.
        2.  A `PRIMARY KEY` is implicitly `NOT NULL`. A `UNIQUE` constraint allows `NULL` values (most databases allow multiple `NULL`s, as `NULL` is not equal to `NULL`)."

3.  **"Why would you use a `UNIQUE` constraint instead of a primary key?"**
    -   "You use a `UNIQUE` constraint for columns that must be unique but are not the primary identifier for the row. These are often called 'candidate keys'. For example, in an `employees` table, `employee_id` would be the primary key, but you would also want `email` and `social_security_number` to be unique. These would be perfect candidates for `UNIQUE` constraints."

### How to Explain in Interviews
"I use `UNIQUE` constraints to enforce business rules that require certain attributes to be unique, even if they aren't the primary identifier of the record. For example, a `users` table will have an `id` as its primary key, but I'll add a `UNIQUE` constraint on the `username` column and another on the `email` column to prevent duplicates. This maintains data integrity while keeping a stable, numeric primary key for foreign key relationships."

## Quick Recall ✅

-   **Purpose**: Enforces uniqueness on non-primary key columns.
-   **`NULL`s**: Allows `NULL` values (the exact number depends on the RDBMS).
-   **Multiplicity**: A table can have **many** `UNIQUE` constraints.
-   **Candidate Keys**: A `UNIQUE` constraint is used to enforce uniqueness on candidate keys.
-   **Index**: A `UNIQUE` constraint is enforced by the database creating a unique index on the column(s).

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming `UNIQUE` implies `NOT NULL`.**
    -   This is the most common confusion. A `UNIQUE` constraint does not enforce `NOT NULL` by itself. If you need a column to be both unique and mandatory, you must specify both constraints: `UNIQUE NOT NULL`.

2.  **Forgetting about the `NULL` behavior differences between databases.**
    -   In PostgreSQL/SQL Server/Oracle, you can have many rows with `NULL` in a unique column.
    -   In MySQL (with the InnoDB engine), you can only have one row with `NULL` in a unique column.
    -   This is a subtle but important difference in portability.

### Tricky Interview Scenarios

-   **"You have a table of users with a `UNIQUE` constraint on the `email` column. A user deactivates their account, and you want to free up their email address for a new user, but you need to keep the old user's record for historical purposes. How would you handle this?"**
    -   **Answer**: "Since the `email` column must be unique, I can't just clear it or set it to an empty string if another deactivated user has an empty string email. A common strategy is to 'retire' the old email address by appending a unique value to it, like the user's ID or a timestamp, before setting the user's status to inactive. For example, `email` becomes `user_123_deactivated@example.com`. This preserves the record while freeing up the original email address, satisfying the `UNIQUE` constraint."
    -   "Another option, if the `UNIQUE` constraint allows `NULL`s, is to set the deactivated user's email to `NULL`. This is simpler but results in data loss for that field."

-   **"Can a foreign key also be a unique key?"**
    -   **Answer**: "Yes, this is a common pattern for one-to-one relationships. For example, if you have an `employees` table and an `employee_details` table, the `employee_id` in `employee_details` would be both a primary key for its own table and a foreign key to the `employees` table. If you wanted to enforce the one-to-one relationship at the database level, you could make the `employee_id` in `employee_details` a `UNIQUE` key as well (in addition to being a foreign key)."

## Bonus

### Related Concepts
-   **Primary Key**: The main identifier for a table. It's implicitly `UNIQUE` and `NOT NULL`.
-   **Alternate Key**: A candidate key that was not chosen to be the primary key. `UNIQUE` constraints are used to enforce alternate keys.
-   **Candidate Key**: Any column or set of columns that could potentially serve as the primary key. All candidate keys must be unique.
-   **Index**: A `UNIQUE` constraint is implemented by creating a unique index. This has the side benefit of making lookups on that column very fast.
