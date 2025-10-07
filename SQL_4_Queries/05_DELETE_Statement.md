# DELETE Statement

## Overview
The `DELETE` statement is a DML (Data Manipulation Language) command used to remove one or more rows from a table. Just like the `UPDATE` statement, it is extremely powerful and potentially destructive, as a `DELETE` without a `WHERE` clause will remove **all rows** from the table.

## Core Concepts

### Basic Syntax
The `DELETE` statement has two main parts:
1.  `DELETE FROM table_name`: The table to remove rows from.
2.  `WHERE condition`: **Crucially**, the condition that specifies *which* rows to delete.

```sql
DELETE FROM table_name
WHERE condition;
```

**Example: Deleting a single user**
```sql
DELETE FROM users
WHERE user_id = 503;
```

**Example: Deleting multiple rows**
```sql
-- Delete all orders that were cancelled
DELETE FROM orders
WHERE status = 'cancelled';
```

### The Danger: `DELETE` without `WHERE`
If you omit the `WHERE` clause, **all rows in the table will be deleted**. This is a DML operation, so it can be rolled back if it's inside a transaction, but if `autocommit` is on, the data is gone.

```sql
-- DANGEROUS: This will delete ALL records from the 'logs' table
DELETE FROM logs;
```

### `DELETE` vs. `TRUNCATE`
This is a classic interview question.

| Feature | `DELETE` | `TRUNCATE` |
| :--- | :--- | :--- |
| **Type** | DML (Data Manipulation) | DDL (Data Definition) |
| **Purpose** | Removes specific rows (or all) | Removes all rows |
| **`WHERE` clause?** | **Yes** | No |
| **Rollback?** | **Yes** (if in a transaction) | No (in most systems) |
| **Speed** | Slower (logs each row deletion) | **Much Faster** |
| **Triggers** | Fires `DELETE` triggers | Does not fire triggers |
| **Identity Reset** | No | **Yes** |

Use `DELETE` when you need to remove specific rows. Use `TRUNCATE` when you need to quickly empty an entire table.

### Deleting from another table (Advanced)
You can delete rows from a table based on a join with another table. The syntax is vendor-specific.

**Syntax (SQL Server):**
```sql
DELETE t1
FROM table1 AS t1
JOIN table2 AS t2 ON t1.join_col = t2.join_col
WHERE t2.condition;
```

**Syntax (PostgreSQL):**
```sql
DELETE FROM table1
USING table2
WHERE table1.join_col = table2.join_col AND table2.condition;
```

**Syntax (MySQL):**
```sql
DELETE t1
FROM table1 AS t1
JOIN table2 AS t2 ON t1.join_col = t2.join_col
WHERE t2.condition;
```

**Example: Deleting users who have been inactive for over a year**
```sql
-- MySQL syntax
DELETE u
FROM users AS u
LEFT JOIN recent_logins AS rl ON u.user_id = rl.user_id
WHERE rl.user_id IS NULL; -- Assuming recent_logins only contains logins from the last year
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you remove records from a table?"**
    -   "I use the `DELETE FROM table_name WHERE condition;` statement. The `WHERE` clause is essential to ensure I only remove the specific rows I intend to."

2.  **"What is the difference between `DELETE` and `TRUNCATE`?"**
    -   "`DELETE` is a DML command that removes rows one by one, can be filtered with a `WHERE` clause, and can be rolled back. `TRUNCATE` is a DDL command that empties the entire table much more quickly by deallocating data pages, cannot be rolled back, and resets the identity column."

3.  **"What happens if you run `DELETE` without a `WHERE` clause?"**
    -   "It will delete all rows from the table. This is a dangerous operation, and I always make it a habit to run a `SELECT` with the same `WHERE` clause first to verify which rows will be affected before running the `DELETE`."

### How to Explain in Interviews
"The `DELETE` statement is the standard DML command for removing rows. My safety protocol for using it is to first write a `SELECT * FROM table` query with the `WHERE` clause I intend to use. This allows me to see exactly what data will be deleted. Once I'm confident, I replace `SELECT *` with `DELETE`. This simple habit has saved me from making critical mistakes. I also understand its transactional nature, meaning I can wrap it in a `BEGIN/COMMIT` block and `ROLLBACK` if necessary."

## Quick Recall ✅

-   **Purpose**: Remove existing rows from a table.
-   **Syntax**: `DELETE FROM table WHERE condition;`
-   **CRITICAL**: Always use a `WHERE` clause unless you intend to delete everything.
-   **Safety Check**: Write a `SELECT` with the same `WHERE` clause first.
-   **DML Command**: Can be rolled back.
-   **`DELETE` vs. `TRUNCATE`**: `DELETE` is for specific rows and is transactional; `TRUNCATE` is for emptying a whole table and is faster.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting the `WHERE` clause.** This is the most common and dangerous error.

2.  **Foreign Key Constraints**
    -   You cannot delete a row from a parent table if it is still referenced by a foreign key in a child table.
    -   `DELETE FROM customers WHERE customer_id = 101;` ❌
    -   This will fail if there are any orders in the `orders` table with `customer_id = 101`.
    -   **Solution**: You must either delete the child records first or have an `ON DELETE CASCADE` rule set up on the foreign key, which automatically deletes the child records.

3.  **Slow Deletes on Large Tables**
    -   Deleting millions of rows with `DELETE` can be extremely slow and generate huge transaction logs. For large-scale deletions, it's often better to use a batching approach (deleting in chunks) or a more advanced strategy like copying the data you want to *keep* to a new table and then dropping the old one.

### Tricky Interview Scenarios

-   **"You need to delete all rows from a table except for the 100 most recent ones. How would you do it?"**
    -   This tests your ability to combine `DELETE` with subqueries or more advanced logic.
    -   **Answer (using a subquery)**:
        ```sql
        DELETE FROM logs
        WHERE log_id NOT IN (
            SELECT log_id
            FROM logs
            ORDER BY created_at DESC
            LIMIT 100
        );
        ```
        This query finds the IDs of the 100 most recent logs and then deletes any log whose ID is not in that list.

-   **"You run `DELETE FROM my_table;` on a huge table. It's taking forever and locking the table. What should you have done instead?"**
    -   **Answer**: "If the goal was to delete all rows, I should have used `TRUNCATE TABLE my_table;`, which is much faster and uses fewer system resources. If I needed to delete most, but not all, rows, a better strategy would have been to `CREATE` a new table, `INSERT` the rows I want to keep into it, and then `DROP` the original table and `RENAME` the new one. This is often faster than a massive `DELETE`."

## Bonus

### Related Concepts
-   **Referential Integrity**: The database's mechanism for ensuring that relationships between tables remain valid. This is why you can't delete a parent row with child rows still attached.
-   **`ON DELETE CASCADE`**: A powerful but dangerous foreign key option. When you define a foreign key, you can specify `ON DELETE CASCADE`, which means that if you delete a parent row, all its corresponding child rows in the other table will be automatically deleted.
-   **Triggers**: `BEFORE DELETE` or `AFTER DELETE` triggers can be set up to run custom logic (like auditing) every time a row is deleted. `TRUNCATE` does not fire these triggers.
-   **Soft Deletes**: Instead of actually deleting a row, many applications use a "soft delete" pattern. This involves adding a column like `is_deleted` (a boolean) or `deleted_at` (a timestamp) to the table. To "delete" a row, you simply run an `UPDATE` statement to set `deleted_at = NOW()`. The data is never actually removed, which is great for auditing and recovery, but it requires all `SELECT` queries to include `WHERE deleted_at IS NULL`.
