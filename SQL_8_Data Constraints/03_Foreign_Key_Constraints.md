# Foreign Key Constraint

## Overview
A `FOREIGN KEY` is a constraint used to link two tables together. It is a key in one table that refers to the `PRIMARY KEY` in another table. The table containing the foreign key is called the child table, and the table containing the primary key it references is called the parent table. This constraint is the cornerstone of relational databases, as it enforces referential integrity.

## Core Concepts

A `FOREIGN KEY` constraint ensures that a value in the child table's column must also exist in the parent table's primary key column. This prevents "orphaned" records—for example, an `orders` record with a `customer_id` that doesn't correspond to any customer in the `customers` table.

**Syntax:**

1.  **During table creation (`CREATE TABLE`):**
    ```sql
    CREATE TABLE orders (
        order_id INT PRIMARY KEY,
        order_date DATE,
        customer_id INT,
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
    );
    ```
    Here, `orders.customer_id` is the foreign key, and it references `customers.customer_id`.

2.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE orders
    ADD CONSTRAINT fk_customer
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id);
    ```

### Referential Integrity Actions (`ON DELETE` and `ON UPDATE`)
Foreign keys can also define what should happen to the child records when the parent record is deleted or its primary key is updated.

-   **`ON DELETE`**: Defines the action when a referenced parent row is deleted.
-   **`ON UPDATE`**: Defines the action when a referenced parent key value is changed (rare if using surrogate keys).

**Common Actions:**
-   **`RESTRICT` / `NO ACTION` (Default)**: Prevents the deletion or update of the parent row if there are any matching child rows. The operation will fail with an error.
-   **`CASCADE`**: If the parent row is deleted, all corresponding child rows are automatically deleted. If the parent key is updated, the foreign key values in the child rows are automatically updated to match.
-   **`SET NULL`**: If the parent row is deleted or updated, the foreign key column(s) in the corresponding child rows are set to `NULL`. This requires the foreign key column to be nullable.
-   **`SET DEFAULT`**: Similar to `SET NULL`, but sets the foreign key column(s) to their default value.

**Example with `ON DELETE CASCADE`:**
```sql
CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);
```
With this setup, if you delete an order from the `orders` table, all of its associated line items in `order_items` will be automatically deleted.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a foreign key?"**
    -   "A foreign key is a constraint that creates a link between two tables to enforce referential integrity. It's a column (or set of columns) in a child table that refers to the primary key of a parent table, ensuring that a row in the child table cannot have a foreign key value that doesn't exist in the parent table."

2.  **"What is referential integrity?"**
    -   "Referential integrity is the rule that ensures relationships between tables remain consistent. A foreign key is the mechanism for enforcing it. It means that if a foreign key contains a value, that value must refer to an existing, valid row in the parent table."

3.  **"Explain `ON DELETE CASCADE`."**
    -   "`ON DELETE CASCADE` is a referential action that automatically deletes all child records when the parent record they reference is deleted. It's useful for maintaining data consistency, for example, deleting all order items when an order is deleted. However, it must be used with caution as it can lead to mass deletions."

4.  **"Can a foreign key be `NULL`?"**
    -   "Yes, a foreign key column can contain `NULL` values. This represents an optional relationship. For example, an `employees` table might have a `manager_id` that is a foreign key to the same table. For the CEO, who has no manager, this value would be `NULL`."

### How to Explain in Interviews
"Foreign keys are what make a database 'relational'. They are the primary mechanism for enforcing rules and consistency between tables. When I design a schema, I use foreign keys to model the real-world relationships between entities, like customers and orders. I also carefully consider the `ON DELETE` behavior—I might use `CASCADE` for tightly-coupled data like order items, but `RESTRICT` or `SET NULL` for looser relationships to prevent accidental data loss."

## Quick Recall ✅

-   **Purpose**: Links two tables and enforces referential integrity.
-   **Structure**: A key in a child table that points to a primary key in a parent table.
-   **Main Rule**: Prevents "orphaned" records.
-   **`NULL`s**: Foreign key columns can be `NULL`, indicating an optional relationship.
-   **Referential Actions**: `ON DELETE` and `ON UPDATE` control behavior when the parent key changes (e.g., `CASCADE`, `SET NULL`, `RESTRICT`).

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting to create an index on the foreign key column.**
    -   While the database doesn't require it, not indexing foreign key columns is a major performance mistake. Joins will be much slower, and `DELETE` operations on the parent table can cause table-level locks on the child table. Most databases automatically index primary keys, but not foreign keys.

2.  **Using `ON DELETE CASCADE` carelessly.**
    -   While convenient, `CASCADE` can cause unexpected, widespread data loss. It should be used only when the child entity's lifecycle is completely dependent on the parent.

3.  **Data type mismatch.**
    -   The data type of the foreign key column must exactly match the data type of the primary key column it references.

### Tricky Interview Scenarios

-   **"What is a self-referencing foreign key?"**
    -   **Answer**: "This is when a foreign key in a table refers to the primary key of the *same table*. It's used to model hierarchical or recursive relationships. A classic example is an `employees` table where a `manager_id` column refers back to the `employee_id` column to represent the organizational chart."

-   **"Why might a database not have foreign key constraints defined, even if relationships exist?"**
    -   **Answer**: "This is sometimes seen in very large-scale systems, particularly in data warehousing or analytics databases. The reasons can include:
        1.  **Performance**: Enforcing foreign keys adds overhead to `INSERT`, `UPDATE`, and `DELETE` operations. For massive bulk data loads, these checks might be disabled.
        2.  **Sharding**: In a distributed or sharded database, it can be difficult or impossible to enforce foreign keys across different database nodes.
        3.  **Legacy Systems**: The application logic might have been made responsible for enforcing referential integrity instead of the database. This is generally considered bad practice for transactional systems but can be a practical choice in some analytics environments."

## Bonus

### Related Concepts
-   **Primary Key**: The constraint that a foreign key references.
-   **Relational Model**: The entire theoretical foundation of relational databases, built on these links.
-   **Entity-Relationship (ER) Diagram**: A visual tool used in database design to map out entities (tables) and their relationships (foreign keys).
-   **Indexing**: Foreign key columns should almost always be indexed to ensure good performance on `JOIN` and `DELETE` operations.
