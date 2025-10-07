# CHECK Constraint

## Overview
A `CHECK` constraint is a type of constraint that allows you to specify a condition that must be true for every row in a table. It is used to enforce domain integrity by limiting the range of valid values a column can hold, based on a boolean expression.

## Core Concepts

A `CHECK` constraint is defined with a logical expression that must evaluate to `TRUE` or `UNKNOWN` for a row to be valid. If the expression evaluates to `FALSE`, the `INSERT` or `UPDATE` operation is rejected.

**Syntax:**

1.  **During table creation (column level):**
    ```sql
    CREATE TABLE employees (
        employee_id INT PRIMARY KEY,
        first_name VARCHAR(50),
        age INT CHECK (age >= 18),
        status VARCHAR(10) CHECK (status IN ('Active', 'On Leave', 'Terminated'))
    );
    ```

2.  **During table creation (table level):** This allows for checks that involve multiple columns.
    ```sql
    CREATE TABLE products (
        product_id INT PRIMARY KEY,
        cost_price DECIMAL(10, 2),
        sale_price DECIMAL(10, 2),
        CHECK (sale_price >= cost_price)
    );
    ```

3.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE employees
    ADD CONSTRAINT chk_age CHECK (age >= 18);
    ```
    *Note: This will fail if any existing rows violate the new condition.*

**How it works with `NULL`:**
The condition in a `CHECK` constraint must evaluate to `TRUE` or `UNKNOWN`. A row is only rejected if the condition is `FALSE`.
-   `CHECK (age >= 18)`: If `age` is `25`, this is `TRUE` (allowed). If `age` is `16`, this is `FALSE` (rejected). If `age` is `NULL`, this is `UNKNOWN` (allowed).
-   If you want to ensure a value is both present and valid, you must combine `CHECK` with `NOT NULL`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a `CHECK` constraint?"**
    -   "A `CHECK` constraint is a rule that enforces domain integrity by limiting the values that are allowed in one or more columns. It uses a boolean expression, and any row that causes the expression to evaluate to `FALSE` is rejected."

2.  **"Give an example of where you would use a `CHECK` constraint."**
    -   "Sure. In a `products` table, I would use a `CHECK` constraint to ensure the `quantity_on_hand` is always zero or greater: `CHECK (quantity_on_hand >= 0)`. Another example is on a `users` table to ensure the `password` is a certain length: `CHECK (LENGTH(password) >= 8)`."

3.  **"How does a `CHECK` constraint handle `NULL` values?"**
    -   "A `CHECK` constraint allows `NULL` values because a comparison to `NULL` evaluates to `UNKNOWN`, not `FALSE`. If you want to prevent `NULL`s and also apply a check, you need to add a `NOT NULL` constraint as well."

4.  **"Can a `CHECK` constraint reference another column?"**
    -   "Yes, a `CHECK` constraint defined at the table level can reference multiple columns in the same row. A classic example is ensuring a `end_date` is after a `start_date`: `CHECK (end_date > start_date)`."

### How to Explain in Interviews
"`CHECK` constraints are a powerful way to enforce business rules directly in the database layer, which is much more reliable than relying on application-level validation alone. I use them to ensure data values fall within a valid domain, for example, ensuring a `discount_percentage` is between 0 and 1, or that a `sale_price` is always greater than the `cost_price`. This maintains data integrity regardless of which application is interacting with the database."

## Quick Recall ✅

-   **Purpose**: Enforces custom business rules on column values.
-   **Condition**: A boolean expression that must not evaluate to `FALSE`.
-   **`NULL` Handling**: Allows `NULL`s by default (evaluates to `UNKNOWN`).
-   **Scope**: Can apply to a single column or multiple columns in the same row.
-   **Data Integrity**: Enforces *domain integrity*.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Trying to reference other rows or tables.**
    -   A `CHECK` constraint can only reference columns within the *same row*. It cannot perform a subquery to check against values in other rows or other tables. To enforce that kind of complex rule, you would need to use triggers.

2.  **Assuming it's supported everywhere.**
    -   While `CHECK` is in the SQL standard, MySQL only started enforcing `CHECK` constraints in version 8.0.16. In older versions of MySQL, you could write the syntax, but it would be ignored. This is a major "gotcha" for those working with older MySQL databases.

### Tricky Interview Scenarios

-   **"You want to ensure a `gender` column only accepts 'Male', 'Female', or 'Other'. How would you do this?"**
    -   **Answer**: "I would use a `CHECK` constraint with an `IN` clause."
    ```sql
    CREATE TABLE users (
        user_id INT PRIMARY KEY,
        gender VARCHAR(10) CHECK (gender IN ('Male', 'Female', 'Other'))
    );
    ```
    "This is much cleaner than writing `CHECK (gender = 'Male' OR gender = 'Female' OR gender = 'Other')`."

-   **"Is it better to enforce a business rule in the application or with a `CHECK` constraint in the database?"**
    -   **Answer**: "It's best to do both, but the database is the ultimate authority. Application-level validation is great for providing immediate feedback to the user. However, data can enter the database from multiple sources (different apps, direct updates, data imports). A `CHECK` constraint in the database guarantees that no invalid data can ever enter the table, regardless of the source. It's the most robust way to protect data integrity."

## Bonus

### Related Concepts
-   **Domain Integrity**: One of the core types of data integrity in the relational model. It ensures that all values in a column are from a valid, defined set (its "domain"). `CHECK` constraints, `NOT NULL`, and data types are all mechanisms for enforcing domain integrity.
-   **Triggers**: For more complex rules that `CHECK` constraints can't handle (like referencing other tables), you need to use triggers. A trigger is a stored procedure that automatically runs when an event (`INSERT`, `UPDATE`, `DELETE`) occurs on a table.
-   **`ENUM` Data Type**: Some databases, like MySQL and PostgreSQL, offer an `ENUM` data type, which is another way to restrict a column to a specific list of string values. `CREATE TYPE gender_enum AS ENUM ('Male', 'Female', 'Other');` can be more efficient than a `CHECK` constraint for this purpose.
