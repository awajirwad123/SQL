# CREATE TABLE

## Overview
The `CREATE TABLE` command is a foundational DDL (Data Definition Language) statement used to define the structure of a new table. This command specifies the table's name, its columns, the data type of each column, and the constraints that enforce data integrity. Mastering this is non-negotiable for any data-related role.

## Core Concepts

### Basic Syntax
The core structure of the command is as follows:
```sql
CREATE TABLE table_name (
    column1_name data_type [constraints],
    column2_name data_type [constraints],
    ...
    [table_level_constraints]
);
```

### Defining Columns and Data Types
Each column requires a name and a data type.
-   **Column Name**: A unique identifier for the column within the table.
-   **Data Type**: Defines what kind of data the column can hold (e.g., `INT`, `VARCHAR(255)`, `DECIMAL(10, 2)`, `TIMESTAMP`).

### Column and Table Constraints
Constraints are rules to ensure data accuracy and reliability.

-   **`NOT NULL`**: Ensures a column cannot have a `NULL` value.
-   **`UNIQUE`**: Ensures all values in a column are different. A table can have multiple `UNIQUE` constraints. `NULL` values are allowed (and multiple `NULL`s are typically permitted).
-   **`PRIMARY KEY`**: A combination of `NOT NULL` and `UNIQUE`. It uniquely identifies each record in a table. A table can have only **one** primary key.
-   **`FOREIGN KEY`**: Uniquely identifies a record in **another** table, creating a link between the two. This is the basis of relational databases.
-   **`CHECK`**: Ensures that all values in a column satisfy a specific condition.
-   **`DEFAULT`**: Provides a default value for a column when none is specified.
-   **`AUTO_INCREMENT` / `IDENTITY` / `SERIAL`**: (System-specific) Automatically generates a unique number when a new record is inserted. Commonly used for primary keys.

### Example: A `products` table
This example combines many of the concepts above.
```sql
CREATE TABLE products (
    -- Column-level constraints
    product_id INT PRIMARY KEY AUTO_INCREMENT, -- Uniquely identifies each product
    product_name VARCHAR(100) NOT NULL,        -- Product name cannot be empty
    sku VARCHAR(50) UNIQUE NOT NULL,           -- Stock Keeping Unit must be unique
    category VARCHAR(50) DEFAULT 'Uncategorized',
    price DECIMAL(10, 2) NOT NULL CHECK (price >= 0), -- Price must be non-negative
    supplier_id INT,

    -- Table-level constraint
    CONSTRAINT fk_supplier
        FOREIGN KEY (supplier_id) 
        REFERENCES suppliers(supplier_id)
        ON DELETE SET NULL -- If a supplier is deleted, set this field to NULL
);
```

### Composite Primary Key
A primary key can also be composed of multiple columns.
```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    
    -- A combination of order_id and product_id uniquely identifies the row
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you create a table in SQL?"**
    -   Start with the basic `CREATE TABLE` syntax and then add detail by mentioning columns, data types, and key constraints like `PRIMARY KEY` and `NOT NULL`.

2.  **"What is the difference between a `PRIMARY KEY` and a `UNIQUE` constraint?"**
    -   **Primary Key**: Can only be one per table. Cannot contain `NULL` values. It's the main identifier for the table.
    -   **Unique Key**: Can be many per table. Can allow `NULL` values (usually one, but this can vary). It ensures uniqueness for a column that isn't the primary identifier.

3.  **"What is a `FOREIGN KEY` and why is it important?"**
    -   It's a key used to link two tables together. It enforces referential integrity, meaning you can't insert a value in the foreign key column if that value doesn't exist in the primary key of the parent table.

4.  **"What does `AUTO_INCREMENT` do?"**
    -   It automatically generates a sequential unique number for a column, typically the primary key, every time a new row is inserted. This saves you from having to manually generate a new ID.

### How to Explain in Interviews
"To create a table, I use the `CREATE TABLE` statement. I define each column with an appropriate data type, like `INT` or `VARCHAR`. Most importantly, I apply constraints to enforce data integrity. I always define a `PRIMARY KEY` to uniquely identify each row, use `NOT NULL` for required fields, and set up `FOREIGN KEY` relationships to maintain referential integrity across the database schema."

## Quick Recall ✅

-   **Command**: `CREATE TABLE table_name (...)`
-   **Core Components**: Column names, data types, constraints.
-   **Primary Key**: One per table, `NOT NULL` + `UNIQUE`.
-   **Foreign Key**: Links to another table, enforces referential integrity.
-   **`UNIQUE` Constraint**: Allows `NULL`s (usually), can have multiple per table.
-   **`AUTO_INCREMENT`**: Generates IDs automatically.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting a Primary Key**
    -   While technically possible, a table without a primary key is usually a design flaw. It makes it difficult to uniquely identify and reference rows.

2.  **Using the wrong data type**
    -   Using `VARCHAR` for dates or `FLOAT` for money. This leads to data integrity issues and performance problems. Always choose the most specific data type.

3.  **Confusing `PRIMARY KEY` and `UNIQUE`**
    -   This is a very common point of confusion. Remember: A table can have many unique columns, but only one primary identifier.

4.  **`ON DELETE` and `ON UPDATE` clauses**
    -   Forgetting to specify the `ON DELETE` or `ON UPDATE` behavior for a foreign key can lead to unexpected errors later. Common options are `CASCADE`, `SET NULL`, `RESTRICT`, and `NO ACTION`.

### Tricky Interview Scenarios

-   **"Can a primary key be a foreign key at the same time?"**
    -   Yes. This is common in one-to-one relationships. For example, in a `user_profiles` table, the `user_id` can be both the primary key of that table and a foreign key referencing the `users` table.

-   **"You have a `UNIQUE` constraint on a column. How many `NULL` values can you insert into that column?"**
    -   The answer is RDBMS-dependent, which makes it a great trick question.
        -   **PostgreSQL, SQL Server**: You can insert many `NULL` values because `NULL` is not considered equal to another `NULL`.
        -   **MySQL**: You can insert many `NULL` values (if using the InnoDB engine).
        -   **Oracle**: Historically, only one `NULL` was allowed, but this can depend on the version.
    -   The safe answer is to say, "It depends on the database system, but most modern systems allow multiple NULLs as they don't consider NULL equal to NULL."

## Bonus

### Related Concepts
-   **Normalization**: The process of designing a database schema to minimize redundancy. `CREATE TABLE` is how you implement a normalized design.
-   **SQL Data Types**: Choosing the right data type is a critical part of table creation.
-   **Indexes**: When you create a `PRIMARY KEY` or `UNIQUE` constraint, the database automatically creates an index on that column to speed up lookups.

### `CREATE TABLE AS` (CTAS)
A powerful variation is `CREATE TABLE ... AS SELECT ...`, which creates a new table based on the result of a `SELECT` query.
```sql
-- Create a summary table of sales per category
CREATE TABLE sales_summary AS
SELECT
    category,
    SUM(amount) AS total_sales,
    COUNT(*) AS number_of_orders
FROM
    orders
GROUP BY
    category;
```
This is extremely useful for creating summary tables or backups of specific data.
