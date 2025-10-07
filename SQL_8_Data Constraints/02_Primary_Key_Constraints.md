# Primary Key Constraint

## Overview
A `PRIMARY KEY` is a constraint that uniquely identifies each record in a table. It is one of the most important concepts in relational database design. A primary key must contain unique values and cannot contain `NULL` values. A table can have only one primary key.

## Core Concepts

The `PRIMARY KEY` constraint enforces two critical rules:
1.  **Uniqueness**: The value(s) in the primary key column(s) must be unique for each row. No two rows can have the same primary key value.
2.  **`NOT NULL`**: The primary key column(s) cannot contain `NULL` values. Every row must have a primary key value.

Together, these rules ensure that every row in a table can be reliably and uniquely identified.

**Syntax:**

1.  **During table creation (column level):**
    ```sql
    CREATE TABLE employees (
        employee_id INT PRIMARY KEY,
        first_name VARCHAR(50),
        last_name VARCHAR(50)
    );
    ```

2.  **During table creation (table level):** This syntax is required for composite primary keys.
    ```sql
    CREATE TABLE employees (
        employee_id INT,
        first_name VARCHAR(50),
        last_name VARCHAR(50),
        PRIMARY KEY (employee_id)
    );
    ```

3.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE employees
    ADD PRIMARY KEY (employee_id);
    ```
    *Note: This will fail if the column contains duplicate or `NULL` values.*

### Surrogate vs. Natural Keys
-   **Surrogate Key**: An artificial key with no business meaning, created solely to be the primary key. Typically, this is an auto-incrementing integer (e.g., `INT AUTO_INCREMENT` in MySQL, `SERIAL` in PostgreSQL, or `IDENTITY` in SQL Server). **This is the most common and recommended approach.**
    -   **Example**: `employee_id`, `customer_id`, `order_id`.
-   **Natural Key**: A key that is formed from one or more existing attributes that have a real-world, business meaning.
    -   **Example**: A `social_security_number` for a US citizen, a vehicle's `VIN`, or a book's `ISBN`.
    -   **Risks**: Natural keys can change (e.g., a country code is updated) or might not be as unique as assumed, making them brittle. Surrogate keys are generally preferred because they are stable and guaranteed to be unique.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a primary key?"**
    -   "A primary key is a constraint used to uniquely identify every row in a table. It must be unique and cannot be `NULL`. A table can only have one primary key."

2.  **"What are the properties of a primary key?"**
    -   "It has two main properties: it must be **unique**, meaning no two rows can have the same primary key value, and it must be **`NOT NULL`**, meaning every row must have a value for the primary key."

3.  **"What is the difference between a primary key and a unique key?"**
    -   "This is a classic question. Both enforce uniqueness. The key differences are:
        1.  A table can have only **one** primary key, but it can have **multiple** unique keys.
        2.  A primary key is implicitly `NOT NULL`. A unique key, by default, **can allow one `NULL` value** in most database systems."

4.  **"What is a surrogate key? Why is it often preferred over a natural key?"**
    -   "A surrogate key is an artificial key, usually an auto-incrementing integer, with no business meaning. It's preferred because it's stable and controlled by the database. Natural keys, which are based on real-world attributes like email addresses or social security numbers, can change or might not be truly unique, which can cause major issues in the database, especially with foreign key relationships."

### How to Explain in Interviews
"The primary key is the backbone of a table's integrity. It provides a reliable way to identify and reference any specific row. When designing tables, I almost always use a surrogate integer primary key, like `id`, because it's stable, efficient, and decouples the row's identity from any business data that might change. This key then becomes the target for foreign keys in other tables, creating the relational structure of the database."

## Quick Recall ✅

-   **Purpose**: Uniquely identifies each row in a table.
-   **Properties**: `UNIQUE` and `NOT NULL`.
-   **Limit**: Only **one** per table.
-   **Clustered Index**: In many databases (like SQL Server), the primary key automatically becomes the clustered index, which determines the physical storage order of the rows.
-   **Surrogate Key**: Artificial, auto-incrementing integer (preferred).
-   **Natural Key**: Business-related data (e.g., SSN, ISBN). Use with caution.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Choosing a poor natural key.** Using an email address as a primary key is a common mistake. What happens when a user changes their email address? All foreign keys referencing it would need to be updated, which is a cascading nightmare.

2.  **Confusing the primary key constraint with its underlying index.** The primary key is a logical concept (a constraint). To enforce uniqueness efficiently, the database creates a unique index on the primary key column(s). They are related but distinct concepts.

### Tricky Interview Scenarios

-   **"Can a primary key be made of more than one column?"**
    -   **Answer**: "Yes, this is called a **composite primary key**. It's used when a combination of two or more columns is needed to uniquely identify a row. For example, in a linking table like `order_items`, the primary key is often the combination of `order_id` and `product_id`."

-   **"You need to change a table's primary key. What are the steps and risks?"**
    -   **Answer**: "Changing a primary key is a high-risk operation. The steps would be:
        1.  Drop any foreign key constraints in other tables that reference the current primary key.
        2.  Drop the existing primary key constraint.
        3.  Add the new primary key constraint.
        4.  Re-create all the foreign key constraints to point to the new primary key.
        The main risk is data integrity; this process can be complex and could lead to orphaned records if not managed carefully. It's a strong argument for using stable surrogate keys from the beginning."

## Bonus

### Related Concepts
-   **Foreign Key**: A key in one table that refers to the `PRIMARY KEY` of another table, creating a link between them.
-   **Unique Key**: Enforces uniqueness but allows one `NULL` and a table can have many.
-   **Composite Key**: A key made up of two or more columns.
-   **Index**: A data structure used by the database to speed up data retrieval. A primary key always has a unique index associated with it.
-   **Entity Integrity**: This is the data integrity rule that states every table must have a primary key, and that key must be unique and not null. The `PRIMARY KEY` constraint is the direct implementation of this rule.
