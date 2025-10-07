# DEFAULT Constraint

## Overview
The `DEFAULT` constraint is used to provide a default value for a column when an `INSERT` statement does not specify a value for it. If a row is inserted and no value is provided for a column with a `DEFAULT` constraint, the database automatically inserts the specified default value.

## Core Concepts

The `DEFAULT` constraint helps ensure data completeness and can simplify `INSERT` statements by providing sensible defaults for optional or standard fields.

**Syntax:**

1.  **During table creation (`CREATE TABLE`):**
    ```sql
    CREATE TABLE users (
        user_id INT PRIMARY KEY,
        username VARCHAR(50) NOT NULL,
        is_active BOOLEAN DEFAULT TRUE,
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        theme VARCHAR(20) DEFAULT 'light'
    );
    ```

2.  **Adding to an existing table (`ALTER TABLE`):**
    ```sql
    ALTER TABLE users
    ALTER COLUMN theme SET DEFAULT 'light';
    ```

**How it works:**
Consider the `users` table defined above.
If you run the following `INSERT` statement:
```sql
INSERT INTO users (user_id, username) VALUES (1, 'john.doe');
```
The database will actually insert the following row:
-   `user_id`: 1
-   `username`: 'john.doe'
-   `is_active`: `TRUE` (from the default)
-   `created_at`: The current timestamp (from the default)
-   `theme`: `'light'` (from the default)

The `DEFAULT` constraint is triggered only when a column is omitted from the `INSERT` statement's column list. If you explicitly provide a value, even `NULL` (if the column allows it), that value will be used instead of the default.

```sql
-- This will insert NULL for the theme, not 'light'
INSERT INTO users (user_id, username, theme) VALUES (2, 'jane.doe', NULL);
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a `DEFAULT` constraint?"**
    -   "A `DEFAULT` constraint specifies a default value for a column. If a new row is inserted without a value for that column, the database automatically provides the default value. It's useful for setting initial states, timestamps, or standard configuration values."

2.  **"Give an example of where you would use a `DEFAULT` constraint."**
    -   "A classic example is a `created_at` column, where I would set `DEFAULT CURRENT_TIMESTAMP` to automatically record the time a row is created. Another common use is for a status column, like `status VARCHAR(20) DEFAULT 'pending'`, to set the initial state of a new record."

3.  **"When is the `DEFAULT` value used?"**
    -   "The default value is used during an `INSERT` operation when the column is not included in the column list. It is **not** used if the column is explicitly assigned a `NULL` value in the `INSERT` statement."

4.  **"Can you use a function as a default value?"**
    -   "Yes, many databases allow functions to be used as default values. The most common example is using `CURRENT_TIMESTAMP` or `GETDATE()` to set a default for a timestamp column. Some databases also allow other functions, like `UUID()` for generating unique identifiers."

### How to Explain in Interviews
"`DEFAULT` constraints are a great way to simplify application logic and ensure data consistency. By setting a sensible default in the database, I can make columns `NOT NULL` without forcing the application to provide a value every single time. For example, for a user settings table, I can set default values for `theme` or `notifications_enabled`. This way, the `INSERT` statement only needs to provide the essential information, and the database handles the rest."

## Quick Recall ✅

-   **Purpose**: Provides a default value for a column on `INSERT`.
-   **Trigger**: Used when a column is omitted from an `INSERT` statement.
-   **`NULL`**: Does not override an explicit `NULL` insertion (if the column is nullable).
-   **Common Uses**: Timestamps (`CURRENT_TIMESTAMP`), initial status (`'pending'`), boolean flags (`TRUE`/`FALSE`).
-   **`NOT NULL`**: Often used together with `NOT NULL` to ensure a column *always* has a valid value.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Expecting `DEFAULT` to apply to `UPDATE`s.**
    -   The `DEFAULT` constraint only applies to `INSERT` statements. It does not automatically fill in a value if a column is set to `NULL` during an `UPDATE`.

2.  **Confusing `DEFAULT` with an empty string.**
    -   `DEFAULT ''` sets the default to an empty string. `DEFAULT NULL` sets the default to `NULL`. They are different.

3.  **Assuming `DEFAULT` overrides an explicit `NULL`.**
    -   If a column is nullable and you write `INSERT INTO my_table (my_column) VALUES (NULL);`, the column will contain `NULL`, not the default value.

### Tricky Interview Scenarios

-   **"You have a column `status VARCHAR(10) NOT NULL DEFAULT 'active'`. What happens if you run `INSERT INTO my_table (status) VALUES (NULL);`?"**
    -   **Answer**: "The statement will fail. The `NOT NULL` constraint is checked before the `DEFAULT` constraint could ever be a consideration. Since you are explicitly trying to insert a `NULL`, and the column does not allow `NULL`s, the database will raise a `NOT NULL` constraint violation error."

-   **"How can you reset a column to its default value in an `UPDATE` statement?"**
    -   **Answer**: "Most SQL dialects provide a `DEFAULT` keyword that can be used in an `UPDATE` statement to explicitly set a column back to its defined default value."
    ```sql
    UPDATE users
    SET theme = DEFAULT
    WHERE user_id = 123;
    ```
    "This will reset the theme for user 123 to whatever is defined in the `DEFAULT` constraint (e.g., 'light')."

## Bonus

### Related Concepts
-   **`NOT NULL` Constraint**: `DEFAULT` and `NOT NULL` are a powerful combination. A column can be mandatory (`NOT NULL`), but the application doesn't have to provide a value if a sensible `DEFAULT` is available.
-   **Data Integrity**: `DEFAULT` constraints contribute to domain integrity by ensuring that if data isn't provided, a valid, known default is used instead of `NULL`.
-   **`CURRENT_TIMESTAMP` / `GETDATE()` / `NOW()`**: Common database functions used as default values for recording creation or modification times.
-   **`UUID()` / `NEWID()`**: Functions used to generate unique identifiers, which can also be set as default values for primary key columns.
