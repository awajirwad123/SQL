# LIKE Operator

## Overview
The `LIKE` operator is used in a `WHERE` clause to search for a specified pattern in a string column. It is an essential tool for flexible string matching when you don't know the exact value you're looking for.

## Core Concepts

The `LIKE` operator uses two special wildcard characters to create search patterns.

-   **`%` (Percent sign)**: Represents zero, one, or multiple characters.
-   **`_` (Underscore)**: Represents a single character.

**Syntax:**
```sql
SELECT column_list
FROM table_name
WHERE column_name LIKE 'pattern';
```

**Common Patterns:**

| Pattern | Description |
| :--- | :--- |
| `LIKE 'a%'` | Finds any values that start with "a". |
| `LIKE '%a'` | Finds any values that end with "a". |
| `LIKE '%abc%'` | Finds any values that have "abc" in any position. |
| `LIKE '_r%'` | Finds any values that have "r" in the second position. |
| `LIKE 'a_%'` | Finds any values that start with "a" and are at least 2 characters long. |
| `LIKE 'a__%'` | Finds any values that start with "a" and are at least 3 characters long. |
| `LIKE 'c%t'` | Finds any values that start with "c" and end with "t". |

**Example: Find all customers whose name starts with 'J'**
```sql
SELECT customer_name
FROM customers
WHERE customer_name LIKE 'J%';
```
This would match 'John', 'Jane', and 'Jennifer'.

**Example: Find all products containing the word 'shirt'**
```sql
SELECT product_name
FROM products
WHERE product_name LIKE '%shirt%';
```
This would match 'T-shirt', 'Polo shirt', and 'shirt'.

### Case Sensitivity
The case sensitivity of `LIKE` depends on the database system and its collation settings.
-   **MySQL, SQL Server (default)**: Case-insensitive. `LIKE 'j%'` will match 'John'.
-   **PostgreSQL, Oracle**: Case-sensitive by default. `LIKE 'j%'` will not match 'John'.
-   In case-sensitive systems, you can use the `ILIKE` operator (PostgreSQL) or functions like `LOWER()` or `UPPER()` to perform a case-insensitive search: `WHERE LOWER(customer_name) LIKE 'j%'`.

### Escaping Wildcards
What if you need to search for a literal percent sign or underscore? You use the `ESCAPE` keyword.

**Example: Find products with '50%' in their name**
```sql
SELECT product_name
FROM products
WHERE product_name LIKE '%50!%%' ESCAPE '!';
```
Here, `!` is defined as the escape character. So, `!%` is interpreted as a literal `%` sign.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is the `LIKE` operator used for?"**
    -   "`LIKE` is used for pattern matching in string data. It allows you to search for substrings or specific patterns using wildcards, primarily the percent sign (`%`) for multiple characters and the underscore (`_`) for a single character."

2.  **"What is the difference between the `%` and `_` wildcards?"**
    -   "The percent sign (`%`) matches any sequence of zero or more characters. The underscore (`_`) matches exactly one character."

3.  **"Is the `LIKE` operator case-sensitive?"**
    -   "It depends on the database's collation. In MySQL and SQL Server, it's typically case-insensitive by default. In PostgreSQL and Oracle, it's case-sensitive. To ensure a case-insensitive search on a sensitive system, you can use functions like `LOWER()` on both the column and the pattern, or use the `ILIKE` operator in PostgreSQL."

### How to Explain in Interviews
"`LIKE` is my go-to operator for any kind of string pattern matching. I use `%` for wildcard searches, like finding a word within a text field, and `_` for more structured searches where I know the position of characters. I'm also aware of its performance implications, especially with leading wildcards, and I know how to handle case sensitivity and escape literal wildcard characters depending on the database system."

## Quick Recall ✅

-   **Purpose**: String pattern matching.
-   **Wildcards**:
    -   `%`: Zero or more characters.
    -   `_`: Exactly one character.
-   **Case Sensitivity**: Varies by database (MySQL/SQL Server are case-insensitive; PostgreSQL is sensitive).
-   **`ILIKE`**: Case-insensitive `LIKE` in PostgreSQL.
-   **Performance**: A leading wildcard (`LIKE '%...'`) is often slow as it can't use a standard B-Tree index.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Performance of Leading Wildcards:**
    -   A query like `WHERE column LIKE '%text'` is notoriously slow on large tables because it cannot use a standard index. The database must perform a full table scan.
    -   A trailing wildcard (`LIKE 'text%'`) is much faster because it *can* use an index on that column.

2.  **Confusing `%` and `*`:**
    -   Some developers new to SQL might mistakenly use the asterisk (`*`), which is a common wildcard in command-line interfaces, instead of the percent sign (`%`).

3.  **Forgetting to escape special characters.**
    -   Searching for a product code like `PROD_A` using `WHERE code LIKE 'PROD_A'` will also match `PROD1A`, `PROD2A`, etc. The correct way is `WHERE code LIKE 'PROD!_A' ESCAPE '!'`.

### Tricky Interview Scenarios

-   **"Your query `WHERE description LIKE '%error%'` is too slow. How can you optimize it?"**
    -   This is a classic performance tuning question.
    -   **Answer**: "A leading wildcard prevents the use of a standard B-Tree index. The best solution is to use a **Full-Text Index**. Database systems like MySQL, PostgreSQL, and SQL Server provide full-text search capabilities. I would create a full-text index on the `description` column and then use the dedicated full-text search function (like `MATCH ... AGAINST` in MySQL or `CONTAINS` in SQL Server) instead of `LIKE`. This is much more efficient for searching for words within large text blocks."

-   **"How would you find all names that start with 'A' and end with 'n'?"**
    -   This tests the combination of wildcards.
    ```sql
    SELECT name FROM my_table WHERE name LIKE 'A%n';
    ```

## Bonus

### Related Concepts
-   **Regular Expressions**: For more complex pattern matching, many databases support regular expression operators (e.g., `REGEXP` or `RLIKE` in MySQL, `~` in PostgreSQL). `LIKE` is for simple patterns; regular expressions are for complex ones (e.g., finding valid email formats).
-   **Full-Text Search**: A specialized set of functions and index types designed for efficiently searching for words and phrases within large blocks of text. It's the proper solution for the performance problems caused by `LIKE '%...%'`.
-   **Collation**: The set of rules that determines how data is sorted and compared. It's the underlying reason for `LIKE`'s case sensitivity behavior.
