# Temporary Tables

## Overview
A temporary table is a short-lived table that exists only for the duration of a database session or transaction. They are extremely useful for storing intermediate results in complex queries, breaking down complicated calculations, and improving code readability without cluttering the main database schema.

## Core Concepts

### Creating a Temporary Table
The syntax is very similar to `CREATE TABLE`, but with the `TEMPORARY` or `TEMP` keyword. In SQL Server, you use a `#` prefix.

**Syntax (MySQL / PostgreSQL):**
```sql
CREATE TEMPORARY TABLE temp_table_name (
    column1 datatype,
    ...
);
```

**Syntax (SQL Server):**
```sql
CREATE TABLE #temp_table_name (
    column1 datatype,
    ...
);
```

**Example: Storing a list of active user IDs**
```sql
-- MySQL / PostgreSQL
CREATE TEMPORARY TABLE active_users (
    user_id INT PRIMARY KEY
);

-- SQL Server
CREATE TABLE #active_users (
    user_id INT PRIMARY KEY
);
```

### Key Characteristics
-   **Visibility**: A temporary table is only visible to the session that created it. Another user or session cannot see or access it. This prevents naming conflicts.
-   **Lifetime**: It is automatically dropped when the database session ends. You can also drop it manually using `DROP TABLE temp_table_name;`.
-   **Storage**: They are created in a special database (`tempdb` in SQL Server) and are generally stored in memory or on disk, depending on their size.
-   **Performance**: They can be faster than permanent tables because they often have reduced logging.

### Populating a Temporary Table
You can populate them using `INSERT` statements or directly upon creation with `CREATE TABLE ... AS SELECT`.

**Example: Creating and populating at the same time**
```sql
-- MySQL / PostgreSQL
CREATE TEMPORARY TABLE top_customers AS
SELECT customer_id, SUM(order_amount) AS total_spent
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC
LIMIT 100;

-- SQL Server
SELECT customer_id, SUM(order_amount) AS total_spent
INTO #top_customers
FROM orders
GROUP BY customer_id
ORDER BY total_spent DESC;
-- Note: SQL Server uses 'SELECT ... INTO #' for this pattern.
```

### Use Cases
1.  **Storing Intermediate Results**: In a multi-step process, you can store the results of the first step in a temp table and then join against it in the second step.
2.  **Simplifying Complex Joins**: Instead of a massive 10-table join, you can pre-filter data from several tables into a few temp tables and then perform a simpler final join.
3.  **Improving Readability**: Breaking down a monolithic query into several logical steps using temp tables makes the code much easier to understand and debug.

**Example: Multi-step reporting query**
```sql
-- Step 1: Get all recent, high-value orders
CREATE TEMPORARY TABLE recent_orders AS
SELECT order_id, customer_id, order_value
FROM orders
WHERE order_date >= '2022-01-01' AND order_value > 1000;

-- Step 2: Get the names of the customers for those orders
CREATE TEMPORARY TABLE recent_customers AS
SELECT c.customer_id, c.customer_name, c.country
FROM customers c
JOIN recent_orders ro ON c.customer_id = ro.customer_id;

-- Step 3: Final report - aggregate by country
SELECT country, COUNT(DISTINCT customer_id) AS num_customers
FROM recent_customers
GROUP BY country;
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a temporary table and why would you use one?"**
    -   "A temporary table is a table that exists only for the current database session. I use them to store intermediate results in a complex query, which helps break down the problem into smaller, more manageable steps and can improve performance and readability."

2.  **"How is a temporary table different from a permanent table?"**
    -   "The main differences are visibility and lifetime. A temp table is only visible to the session that created it and is automatically deleted when the session ends. A permanent table is visible to all users (with permissions) and persists until explicitly dropped."

3.  **"How do you create a temporary table in SQL Server vs. MySQL?"**
    -   "In MySQL or PostgreSQL, you use the `CREATE TEMPORARY TABLE` syntax. In SQL Server, you prefix the table name with a hash, like `CREATE TABLE #my_temp_table`."

4.  **"Can you create indexes on a temporary table?"**
    -   "Yes, you can and you should. If you are going to join the temporary table with other tables or filter it with a `WHERE` clause, creating an index on the key columns can significantly improve query performance."

### How to Explain in Interviews
"I see temporary tables as a powerful tool for procedural logic within SQL. When a single `SELECT` statement becomes too complex or unreadable, I reach for a temp table. For example, I might pull a pre-filtered set of IDs into a temp table first, index it, and then join it against a larger table. This often results in a more efficient query plan and makes the code much easier to maintain."

## Quick Recall ✅

-   **Purpose**: Store intermediate results for a single session.
-   **Creation (MySQL/PostgreSQL)**: `CREATE TEMPORARY TABLE ...`
-   **Creation (SQL Server)**: `CREATE TABLE #...`
-   **Visibility**: Only visible to the current session.
-   **Lifetime**: Dropped automatically when the session ends.
-   **Indexes**: Can (and should) be created on them for performance.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Forgetting to index the temp table.**
    -   Creating a temp table and then joining it on a non-indexed column can be very slow. Always add indexes to the columns that will be used in `JOIN` or `WHERE` clauses.

2.  **Using them when a CTE would be better.**
    -   For simple, non-reusable, read-only intermediate results, a Common Table Expression (CTE) can be a cleaner solution. Temp tables are better when you need to reference the intermediate data multiple times or when you need to add indexes.

3.  **Name conflicts.**
    -   This is not a common mistake because temp tables are session-specific, but it's a point of confusion. You and another user can both create a temporary table with the exact same name (`#my_data`) and they will not conflict.

### Tricky Interview Scenarios

-   **"What is the difference between a temporary table and a Common Table Expression (CTE)?"**
    -   **Temp Table**: A physical table that stores data in the database (`tempdb`). You can create indexes on it and reference it multiple times in your script. It persists for the whole session.
    -   **CTE (`WITH` clause)**: A named, temporary result set that exists only for the duration of a single query. It's essentially a subquery with a name. It cannot be indexed and is re-evaluated every time it's referenced (in most systems).
    -   **Rule of thumb**: Use a CTE for simple, single-query logic. Use a temp table if you need to reference the data multiple times, need to create indexes, or are working with a very large intermediate data set.

-   **"What is a global temporary table?"**
    -   This is an advanced SQL Server concept. A global temp table is created with a double hash (`##my_global_table`). It is visible to *all* sessions and is dropped only when the last session referencing it has closed. They are much rarer and can be dangerous due to potential naming conflicts.

## Bonus

### Related Concepts
-   **Common Table Expressions (CTEs)**: The main alternative to temp tables for single-query readability.
-   **Table Variables (SQL Server)**: Another alternative in SQL Server (`DECLARE @my_table TABLE (...)`). They are similar to temp tables but have different use cases and performance characteristics, generally for smaller data sets.
-   **Query Optimization**: The database optimizer treats queries with temp tables differently than a single, large query. Sometimes, breaking a query up with a temp table can help the optimizer find a better execution plan.

### Example: CTE vs. Temp Table

**Using a CTE (cleaner for a single query):**
```sql
WITH TopCustomers AS (
    SELECT CustomerID, SUM(TotalAmount) as TotalSales
    FROM Orders
    GROUP BY CustomerID
    ORDER BY TotalSales DESC
    LIMIT 10
)
SELECT c.CustomerName, tc.TotalSales
FROM Customers c
JOIN TopCustomers tc ON c.CustomerID = tc.CustomerID;
```

**Using a Temp Table (better if you need to reuse `TopCustomers`):**
```sql
CREATE TEMPORARY TABLE TopCustomers (
    CustomerID INT,
    TotalSales DECIMAL(18,2)
);

INSERT INTO TopCustomers
SELECT CustomerID, SUM(TotalAmount)
FROM Orders
GROUP BY CustomerID
ORDER BY TotalSales DESC
LIMIT 10;

-- First report
SELECT c.CustomerName, tc.TotalSales
FROM Customers c
JOIN TopCustomers tc ON c.CustomerID = tc.CustomerID;

-- Second report (reusing the temp table)
SELECT tc.CustomerID, COUNT(o.OrderID) as OrderCount
FROM Orders o
JOIN TopCustomers tc ON o.CustomerID = tc.CustomerID
GROUP BY tc.CustomerID;
```
This clearly shows the reusability benefit of a temporary table.
