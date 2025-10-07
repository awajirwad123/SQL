# UNION ALL Operator

## Overview
The `UNION ALL` operator is a set operator used to combine the result sets of two or more `SELECT` statements into a single result set. Unlike the `UNION` operator, `UNION ALL` **includes all rows**, including any duplicates.

## Core Concepts

Like `UNION`, the `SELECT` statements being combined with `UNION ALL` must follow two rules:
1.  They must have the same number of columns.
2.  The data types of the corresponding columns must be compatible.

**Syntax:**
```sql
SELECT column1, column2, ... FROM table1
UNION ALL
SELECT column1, column2, ... FROM table2;
```

**Key Behavior: Includes Duplicates**
`UNION ALL` simply appends the result of the second `SELECT` statement to the end of the first one. It does not perform any check for duplicates.

**Example: Get a combined list of all cities where suppliers and customers are located, including duplicates.**
```sql
-- Table: suppliers
-- city
-- ------
-- London
-- Tokyo

-- Table: customers
-- city
-- ------
-- London
-- Paris

SELECT city FROM suppliers
UNION ALL
SELECT city FROM customers;

-- Result:
-- city
-- ------
-- London  (From suppliers)
-- Tokyo
-- London  (From customers)
-- Paris
```
The city 'London' appears twice because it was present in both tables.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the difference between `UNION` and `UNION ALL`?"**
    -   "This is a fundamental question. `UNION` combines two result sets and removes duplicate rows. `UNION ALL` combines the result sets but includes all rows, even if they are duplicates. Because `UNION ALL` doesn't have to perform the extra step of identifying and removing duplicates, it is much more performant."

2.  **"When should you use `UNION ALL` instead of `UNION`?"**
    -   "You should always default to using `UNION ALL` unless you have a specific reason to remove duplicates. If you know the data from the two queries is already mutually exclusive, or if duplicate rows are acceptable or even desired for the final result (e.g., for an aggregation), then `UNION ALL` is the better choice for performance."

3.  **"Is `UNION ALL` faster than `UNION`? Why?"**
    -   "Yes, `UNION ALL` is significantly faster. `UNION` requires the database to perform a sort or hash operation on the entire combined result set to find and eliminate duplicates. `UNION ALL` simply concatenates the results, avoiding this expensive operation entirely."

### How to Explain in Interviews
"I view `UNION ALL` as the default choice for combining result sets due to its performance advantage. I only use `UNION` when the business logic explicitly requires a de-duplicated list. For example, if I'm combining logs from two different tables for a time-series analysis, I'd use `UNION ALL` because every log entry is important, even if they look the same. If I'm creating a master list of unique customer emails from several tables, I'd use `UNION`."

## Quick Recall ✅

-   **Purpose**: Combines result sets of two or more `SELECT` statements.
-   **Key Feature**: **Includes all rows**, including duplicates.
-   **Rules**: Same number of columns, compatible data types.
-   **Performance**: **Much faster** than `UNION`.
-   **Default Choice**: Prefer `UNION ALL` unless you specifically need to remove duplicates.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Using `UNION` by default.**
    -   Many developers learn `UNION` first and use it by default, not realizing the performance penalty. Always think: "Do I need to remove duplicates?" If the answer is no, use `UNION ALL`.

2.  **Expecting results to be sorted.**
    -   Neither `UNION` nor `UNION ALL` guarantees the order of the final result set. If you need a specific order, you must add an `ORDER BY` clause at the very end of the entire statement.

### Tricky Interview Scenarios

-   **"You need to get a count of all employees and all customers. How would you do it in a single query?"**
    -   This is a good use case for `UNION ALL`.
    ```sql
    SELECT 'Employees' AS source, COUNT(*) AS total FROM employees
    UNION ALL
    SELECT 'Customers' AS source, COUNT(*) AS total FROM customers;
    ```
    This query returns two rows, one with the count of employees and one with the count of customers. Using `UNION` here would also work, but `UNION ALL` is more technically correct as the rows are guaranteed to be unique.

-   **"Can you mix `UNION` and `UNION ALL` in the same query?"**
    -   **Answer**: "Yes, you can. The operators will be evaluated in order, usually from top to bottom, unless parentheses are used to specify precedence. For example, `(query1 UNION ALL query2) UNION query3`. The part in parentheses is evaluated first, and then its result is unioned with `query3`, which will remove duplicates from the final combined set."

## Bonus

### Related Concepts
-   **`UNION`**: The de-duplicating counterpart.
-   **`INTERSECT`**: A set operator that returns only rows present in both result sets.
-   **`EXCEPT`**: A set operator that returns rows from the first set that are not in the second.
-   **Performance Tuning**: Choosing `UNION ALL` over `UNION` is a basic but important performance optimization technique.
-   **Data Warehousing (ETL)**: `UNION ALL` is used extensively in ETL (Extract, Transform, Load) processes to merge data from different sources or time periods into a single staging table before further processing.
