# TRUNCATE TABLE

## Overview
The `TRUNCATE TABLE` command is a DDL (Data Definition Language) statement used to remove all rows from a table quickly. While it seems similar to `DELETE`, it operates very differently and is much faster for clearing out large tables. It's a common topic in interviews to test a candidate's understanding of the nuances between DDL and DML commands.

## Core Concepts

### Basic Syntax
The command is very simple.
```sql
TRUNCATE TABLE table_name;
```
**Example:**
```sql
TRUNCATE TABLE event_logs;
```
When this command is executed:
-   All rows in the `event_logs` table are removed.
-   The table structure itself (columns, constraints, indexes) remains intact.
-   In most database systems (like MySQL and SQL Server), the `AUTO_INCREMENT` or `IDENTITY` counter for the primary key is reset to its starting value.

### Key Characteristics
-   **DDL, not DML**: It is a data *definition* command. This is why it cannot be easily rolled back and doesn't fire DML triggers.
-   **Performance**: It is extremely fast. Instead of deleting rows one by one, it deallocates the data pages used by the table, which is a minimal logging operation.
-   **No `WHERE` clause**: You cannot specify conditions. It's an "all or nothing" operation for the data in a table.
-   **Resets Identity**: The counter for auto-incrementing keys is reset. The next `INSERT` will start from 1 again.
-   **Locking**: It typically requires a higher-level lock on the table, which can block other operations.

### `TRUNCATE` vs. `DELETE` vs. `DROP`
This comparison is a classic interview question.

| Feature | `TRUNCATE` | `DELETE` | `DROP` |
| :--- | :--- | :--- | :--- |
| **Purpose** | Empties a table | Removes rows from a table | Removes a table |
| **Type** | DDL | DML | DDL |
| **Rollback?** | No (in most systems) | Yes (if in a transaction) | No |
| **`WHERE` clause?** | No | Yes | No |
| **Speed** | **Fastest** | Slowest | Fast |
| **Triggers** | No `DELETE` triggers | Fires `DELETE` triggers | No triggers |
| **Identity Reset** | **Yes** | No | N/A |

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the difference between `TRUNCATE` and `DELETE`?"**
    -   "`TRUNCATE` is a DDL command that quickly removes all rows by deallocating data pages and resets the table's identity counter. It can't be rolled back. `DELETE` is a DML command that removes rows one by one, can use a `WHERE` clause, fires triggers, and can be rolled back."

2.  **"When would you use `TRUNCATE` instead of `DELETE`?"**
    -   "I would use `TRUNCATE` when I need to completely clear out a large table, for example, a staging or logging table, and performance is critical. Since it's a minimally logged operation, it's much faster than deleting millions of rows individually."

3.  **"Can you roll back a `TRUNCATE` command?"**
    -   "Generally, no. Because `TRUNCATE` is a DDL command, it often involves an implicit commit. While some systems like SQL Server can technically roll it back if it's wrapped in an explicit transaction, the standard answer and safe assumption is that it's not rollback-able, unlike `DELETE`."

4.  **"Does `TRUNCATE` fire triggers?"**
    -   "No, it does not. It bypasses the individual row-level operations that would fire DML triggers. If you have logic in a `DELETE` trigger that needs to run, you must use the `DELETE` command."

### How to Explain in Interviews
"I use `TRUNCATE TABLE` for high-performance, bulk deletion of all data within a table. I think of it as a 'fast reset' for a table's data. The key things I remember are that it's a DDL operation, it resets `AUTO_INCREMENT` counters, and it doesn't fire `DELETE` triggers. This makes it fundamentally different from `DELETE`, which is a more granular, row-level DML operation."

## Quick Recall ✅

-   **Command**: `TRUNCATE TABLE table_name;`
-   **Action**: Removes all rows, keeps the table structure.
-   **Type**: DDL.
-   **Speed**: Very fast.
-   **Rollback**: No (generally).
-   **`WHERE` Clause**: Not allowed.
-   **Identity Reset**: Yes.
-   **Triggers**: Not fired.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Assuming it's just a faster `DELETE`**
    -   The side effects are different (identity reset, no triggers). Understanding these nuances is key.

2.  **Trying to use a `WHERE` clause**
    -   `TRUNCATE TABLE logs WHERE event_date < '2022-01-01';` ❌ This is a syntax error. `TRUNCATE` is all or nothing.

3.  **Ignoring Foreign Key Constraints**
    -   You cannot truncate a table that is referenced by a `FOREIGN KEY` constraint from another table. The command will fail. You must first disable or drop the foreign key.
    ```sql
    -- This will fail if another table references 'products'
    TRUNCATE TABLE products;
    ```

### Tricky Interview Scenarios

-   **"You run `TRUNCATE` inside a `BEGIN TRANSACTION ... COMMIT` block. What happens if you issue a `ROLLBACK` before the `COMMIT`?"**
    -   This is a trick question that depends on the RDBMS.
        -   **MySQL/Oracle**: `TRUNCATE` causes an implicit commit, so the transaction is committed at the point of the `TRUNCATE` command. The `ROLLBACK` will do nothing.
        -   **SQL Server/PostgreSQL**: `TRUNCATE` can be rolled back if it is part of an explicit transaction.
    -   **Best Interview Answer**: "The behavior depends on the database system. In MySQL, a `TRUNCATE` operation causes an implicit commit, so it cannot be rolled back. However, in SQL Server, it can be rolled back if it's inside an explicit transaction."

-   **"You need to empty a table with 1 billion rows. What command do you use and why?"**
    -   The answer is unequivocally `TRUNCATE TABLE`.
    -   **Why**: `DELETE FROM my_billion_row_table;` would be disastrously slow. It would generate a massive amount of transaction log data (for rollback capability) and could take hours. `TRUNCATE` would likely finish in seconds.

## Bonus

### Related Concepts
-   **DDL vs. DML**: `TRUNCATE` is the perfect example to illustrate the practical differences between these two categories of SQL commands.
-   **Transaction Logs**: `TRUNCATE` is minimally logged, whereas `DELETE` logs every single row deletion. This is the primary reason for the performance difference.
-   **Identity Columns**: The fact that `TRUNCATE` resets the identity (`AUTO_INCREMENT`) is a key feature, useful for resetting test environments.

### A Note on Permissions
To run `TRUNCATE TABLE`, a user typically needs `ALTER` permission on the table, not just `DELETE` permission. This is another reflection of it being a DDL-like operation that modifies the table's storage characteristics, not just the data in it.
_