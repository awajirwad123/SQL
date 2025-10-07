# UPDATE Statement

## Overview
The `UPDATE` statement is a DML (Data Manipulation Language) command used to modify existing records in a table. It is one of the most powerful and dangerous commands in SQL because an `UPDATE` without a `WHERE` clause will modify **all rows** in the table.

## Core Concepts

### Basic Syntax
The `UPDATE` statement consists of three key parts:
1.  `UPDATE table_name`: The table to modify.
2.  `SET column1 = value1, column2 = value2, ...`: The columns to change and their new values.
3.  `WHERE condition`: **Crucially**, the condition that specifies *which* rows to update.

```sql
UPDATE table_name
SET column1 = new_value1,
    column2 = new_value2
WHERE condition;
```

**Example: Updating a single customer's email**
```sql
UPDATE customers
SET email = 'john.d.smith@example.com'
WHERE customer_id = 101;
```

**Example: Updating multiple rows**
```sql
-- Give a 10% discount to all products in the 'Clearance' category
UPDATE products
SET price = price * 0.9
WHERE category = 'Clearance';
```

### The Danger: `UPDATE` without `WHERE`
If you omit the `WHERE` clause, the `UPDATE` will be applied to **every single row** in the table. This is rarely what you want and can be a catastrophic mistake.

```sql
-- DANGEROUS: This will set the price of ALL products to 10.00
UPDATE products
SET price = 10.00;
```
Many database clients have a "safe update" mode that prevents `UPDATE` or `DELETE` statements without a `WHERE` clause or a `LIMIT`.

### Updating from another table (Advanced)
You can update a table based on values from another table. The syntax for this is highly vendor-specific.

**Syntax (SQL Server):**
```sql
UPDATE t1
SET t1.column = t2.column
FROM table1 AS t1
JOIN table2 AS t2 ON t1.join_col = t2.join_col
WHERE t1.condition;
```

**Syntax (MySQL):**
```sql
UPDATE table1 AS t1
JOIN table2 AS t2 ON t1.join_col = t2.join_col
SET t1.column = t2.column
WHERE t1.condition;
```

**Example: Updating product stock levels from a `daily_deliveries` table**
```sql
-- MySQL syntax
UPDATE products p
JOIN daily_deliveries d ON p.product_id = d.product_id
SET p.stock_quantity = p.stock_quantity + d.quantity_delivered;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"How do you modify data in a table?"**
    -   "I use the `UPDATE` statement. The syntax is `UPDATE table SET column = value WHERE condition;`. The `WHERE` clause is the most important part to ensure I only modify the intended rows."

2.  **"What is the most dangerous mistake you can make with an `UPDATE` statement?"**
    -   "Forgetting the `WHERE` clause. An `UPDATE` without a `WHERE` clause will update all rows in the table, which can lead to massive data corruption. I always double-check my `WHERE` clause, and often run it as a `SELECT` first to see what rows will be affected."

3.  **"How can you prevent accidental mass updates?"**
    -   "First, by being extremely careful and always writing the `WHERE` clause first. Second, by using a database client with a 'safe update' mode enabled. Third, by wrapping updates in a transaction, which allows me to `ROLLBACK` if I realize I've made a mistake before committing."

### How to Explain in Interviews
"The `UPDATE` statement is for modifying existing records. My personal workflow to ensure safety is to first write a `SELECT` statement with the exact `WHERE` clause I plan to use. This lets me preview exactly which rows will be changed. Once I've verified the selection is correct, I replace the `SELECT *` with my `UPDATE ... SET ...` syntax. This habit prevents accidental updates of the entire table."

## Quick Recall ✅

-   **Purpose**: Modify existing rows in a table.
-   **Syntax**: `UPDATE table SET col = val WHERE condition;`
-   **CRITICAL**: Always use a `WHERE` clause.
-   **Safety Check**: Write a `SELECT` with the same `WHERE` clause first to preview changes.
-   **DML Command**: `UPDATE` is a DML operation and can be rolled back if within a transaction.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting the `WHERE` clause.** This cannot be stressed enough.

2.  **Incorrect `WHERE` clause.**
    -   Using a `WHERE` clause that is too broad or incorrect can be just as damaging. For example, `UPDATE users SET active = false WHERE registration_date > '2022-01-01'` might deactivate far more users than intended.

3.  **Syntax for multi-table updates.**
    -   The syntax for updating a table using data from another is not standard and is a common point of confusion for developers who work with multiple database systems.

### Tricky Interview Scenarios

-   **"You just ran an `UPDATE` command on a production database and realized you forgot the `WHERE` clause. The command is still running. What do you do?"**
    -   This tests your crisis management skills.
    -   **Answer**: "My immediate action would be to kill the query. I would connect to the database, find the process ID of the running `UPDATE` statement using a command like `SHOW PROCESSLIST;` (in MySQL) or querying system views, and then execute the `KILL <process_id>;` command. This will stop the query before it affects all rows. If the command was wrapped in a transaction, I could then issue a `ROLLBACK`. If not, I'd have to immediately begin a point-in-time recovery from a backup."

-   **"Can an `UPDATE` statement change the value of a primary key?"**
    -   **Answer**: "Technically, yes, but it is extremely bad practice and often disabled or prevented by default. Changing a primary key can break all foreign key relationships that point to it, causing a cascading failure of data integrity. It's almost always better to `INSERT` a new record with the correct key and `DELETE` the old one, after migrating any child records."

## Bonus

### Related Concepts
-   **Transactions (TCL)**: `UPDATE` statements are a primary reason transactions exist. If you need to perform several related `UPDATE`s, wrapping them in a `START TRANSACTION ... COMMIT` block ensures that either all of them succeed or none of them do.
-   **Triggers**: You can define `BEFORE UPDATE` or `AFTER UPDATE` triggers that execute custom logic whenever a row is modified. This is often used for auditing (logging who changed what and when).
-   **Locking**: When you run an `UPDATE`, the database places locks on the rows being modified to prevent other sessions from changing them at the same time. A long-running `UPDATE` can cause significant locking contention in a busy system.

### Using `LIMIT` with `UPDATE` (MySQL-specific)
In MySQL, you can use a `LIMIT` clause to restrict the number of rows updated, which can be useful for batching large updates to avoid long-running transactions.
```sql
-- Update up to 1000 unprocessed records at a time
UPDATE orders
SET status = 'processing'
WHERE status = 'unprocessed'
LIMIT 1000;
```
This is a useful, non-standard feature for safely processing large tables in chunks.
