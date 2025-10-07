# NOT NULL Constraint

## Overview
The `NOT NULL` constraint is a column-level rule that ensures a column cannot have a `NULL` value. When a column is defined as `NOT NULL`, any `INSERT` or `UPDATE` operation that attempts to assign a `NULL` value to that column will fail. This is one of the most fundamental constraints for enforcing data integrity.

## Core Concepts

A `NULL` value in SQL represents missing or unknown data. By applying a `NOT NULL` constraint, you are declaring that a value for that column is mandatory for every row in the table.

**Syntax:**

1.  **During table creation (`CREATE TABLE`):**
    ```sql
    CREATE TABLE employees (
        id INT NOT NULL,
        first_name VARCHAR(50) NOT NULL,
        last_name VARCHAR(50) NOT NULL,
        middle_name VARCHAR(50) -- Can be NULL
    );
    ```

2.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE employees
    MODIFY COLUMN middle_name VARCHAR(50) NOT NULL;
    ```
    *Note: This operation will fail if the `middle_name` column already contains `NULL` values for any rows. You must first update the existing `NULL`s to a valid value.*

**Purpose:**
The primary purpose of `NOT NULL` is to guarantee that essential data is always present. For example:
-   An `employees` table should always have a `first_name` and `last_name`.
-   An `orders` table must have a `customer_id` and an `order_date`.
-   A `users` table requires a `username` and `password_hash`.

Without this constraint, you could have records with missing critical information, leading to application errors and incorrect analysis.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a `NOT NULL` constraint?"**
    -   "A `NOT NULL` constraint is a rule applied to a column to ensure that it cannot store `NULL` values. It forces every row in the table to have an actual value for that column, thereby guaranteeing data completeness for critical fields."

2.  **"What happens if you try to insert a `NULL` into a `NOT NULL` column?"**
    -   "The database will reject the `INSERT` or `UPDATE` statement and return an error. This is the core function of the constraint—to prevent missing data."

3.  **"Can you add a `NOT NULL` constraint to a column that already has `NULL` values?"**
    -   "No, the `ALTER TABLE` statement will fail. You must first update all existing rows to replace the `NULL` values with valid, non-null data before the constraint can be successfully applied."

### How to Explain in Interviews
"`NOT NULL` is the most basic and essential tool for data integrity. I use it on any column where a missing value would be meaningless or cause problems, like primary keys, required names, or status fields. It's the first line of defense against incomplete data entering the database."

## Quick Recall ✅

-   **Purpose**: Ensures a column cannot contain `NULL` values.
-   **Enforcement**: Rejects `INSERT` or `UPDATE` statements that attempt to set the value to `NULL`.
-   **Data Integrity**: Guarantees that mandatory data is always present.
-   **Application**: Can be defined at column creation or added later with `ALTER TABLE`.
-   **Prerequisite for `ALTER`**: Existing data in the column must not be `NULL`.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Confusing `NOT NULL` with an empty string (`''`).**
    -   A `NOT NULL` constraint **allows** empty strings. An empty string is a valid `VARCHAR` value of zero length, whereas `NULL` is the absence of a value.
    -   A column defined as `VARCHAR(50) NOT NULL` can happily accept `''`.

2.  **Assuming all key columns are automatically `NOT NULL`.**
    -   A `PRIMARY KEY` is, by definition, both `UNIQUE` and `NOT NULL`.
    -   However, a `UNIQUE` key column **can** be `NULL` by default in most database systems (though it can only contain one `NULL` value). If you want a unique key to also be mandatory, you must explicitly add the `NOT NULL` constraint.

### Tricky Interview Scenarios

-   **"You have a column `is_active BOOLEAN`. Should it be `NOT NULL`?"**
    -   **Answer**: "Yes, it almost certainly should. A `BOOLEAN` column is intended to represent two states: `TRUE` or `FALSE`. A `NULL` value introduces a third, ambiguous state ('unknown'), which can complicate application logic. For a field like `is_active`, you almost always want to know definitively if the user is active or not. I would define it as `BOOLEAN NOT NULL`, likely with a `DEFAULT` value."

-   **"How is `NOT NULL` different from a `CHECK` constraint?"**
    -   **Answer**: "`NOT NULL` is a specific type of constraint that only checks for the absence of a value (`NULL`). A `CHECK` constraint is more general and can enforce more complex rules on the *value itself*, for example, `CHECK (age >= 18)` or `CHECK (status IN ('Active', 'Inactive'))`. While you could theoretically write `CHECK (my_column IS NOT NULL)`, using the dedicated `NOT NULL` constraint is standard, clearer, and often more optimized."

## Bonus

### Related Concepts
-   **`DEFAULT` Constraint**: Often used together with `NOT NULL`. For example, you can define a column to be `NOT NULL` and provide a `DEFAULT` value, so if an `INSERT` statement omits the column, the default value is used instead of causing an error.
    ```sql
    CREATE TABLE settings (
        theme VARCHAR(20) NOT NULL DEFAULT 'light',
        font_size INT NOT NULL DEFAULT 14
    );
    ```
-   **Data Integrity**: `NOT NULL` is a fundamental type of data integrity, specifically *entity integrity* when applied to primary keys, and *domain integrity* by restricting the domain of a column to non-null values.
-   **Primary Key**: A primary key constraint is implicitly `NOT NULL`.
