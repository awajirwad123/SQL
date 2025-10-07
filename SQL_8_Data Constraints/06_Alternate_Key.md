# Alternate Key

## Overview
An **Alternate Key** is a column or set of columns in a table that is a candidate for a primary key but was not chosen to be the primary key. In practical terms, an alternate key is any key that enforces uniqueness and can be used to uniquely identify a row, other than the primary key. Alternate keys are enforced in the database using a `UNIQUE` constraint.

## Core Concepts

In database theory, a table can have several "candidate keys." A candidate key is any set of attributes that uniquely identifies a row. From this set of candidate keys, one is chosen to be the `PRIMARY KEY`. All other candidate keys are then referred to as **alternate keys**.

**Example:**
Consider an `employees` table:
-   `employee_id`: Unique, not null.
-   `social_security_number`: Unique, not null.
-   `email`: Unique, not null.

In this table, `(employee_id)`, `(social_security_number)`, and `(email)` are all **candidate keys** because any one of them could, in theory, serve to uniquely identify an employee.

-   We choose `employee_id` as the `PRIMARY KEY` (because it's a stable, simple surrogate key).
-   Therefore, `social_security_number` and `email` become **alternate keys**.
-   We enforce these alternate keys using `UNIQUE` constraints.

**Implementation:**
```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY, -- Primary Key
    social_security_number VARCHAR(11) UNIQUE NOT NULL, -- Alternate Key
    email VARCHAR(100) UNIQUE NOT NULL, -- Alternate Key
    first_name VARCHAR(50),
    last_name VARCHAR(50)
);
```
In this DDL:
-   The `PRIMARY KEY` constraint enforces the primary key.
-   The `UNIQUE` constraints enforce the alternate keys.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is an alternate key?"**
    -   "An alternate key is a column or group of columns that could have been chosen as the primary key but was not. It's a candidate key that is not the primary key. In practice, alternate keys are enforced using `UNIQUE` constraints to ensure they also uniquely identify each row."

2.  **"Give an example of a primary key and an alternate key."**
    -   "In a `users` table, the `user_id` (an auto-incrementing integer) would be the primary key. The `username` and `email` columns would be alternate keys, as they also must be unique for each user. I would enforce their uniqueness with `UNIQUE` constraints."

3.  **"Why do we distinguish between primary and alternate keys?"**
    -   "The distinction is important for clarity in database design and for defining relationships. The primary key is the main identifier used in `FOREIGN KEY` relationships. Alternate keys provide additional ways to look up a unique record and enforce business rules (like 'no two users can have the same email'), but they aren't the foundational key used for linking tables."

### How to Explain in Interviews
"An alternate key is essentially a 'runner-up' for being the primary key. When designing a table, I identify all the columns or combinations of columns that could uniquely identify a row—these are my candidate keys. I select the best one, usually a surrogate integer, as the primary key. All the other candidate keys become alternate keys, and I enforce them with `UNIQUE` constraints. This ensures high data integrity while maintaining a stable and simple primary key for joins."

## Quick Recall ✅

-   **Definition**: A candidate key that was not selected to be the primary key.
-   **Purpose**: Provides an alternative way to uniquely identify a row.
-   **Implementation**: Enforced using a `UNIQUE` constraint.
-   **Example**: `employee_id` is the primary key; `email` is an alternate key.
-   **Relationship to `UNIQUE`**: The `UNIQUE` constraint is the SQL mechanism to implement an alternate key.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Confusing alternate key with foreign key.**
    -   An alternate key enforces uniqueness within its *own table*.
    -   A foreign key creates a link to *another table*. They are completely different concepts.

2.  **Thinking it's a different type of constraint.**
    -   "Alternate Key" is a logical, database design term. There is no `ALTERNATE KEY` keyword in SQL. You implement it with the `UNIQUE` constraint.

### Tricky Interview Scenarios

-   **"If a `UNIQUE` constraint enforces an alternate key, why not just call it a unique key?"**
    -   **Answer**: "In casual conversation, they are often used interchangeably. However, in formal database design terminology, 'unique key' refers to the constraint itself, while 'alternate key' refers to the role of that key in the table's design. An alternate key is a *type* of candidate key, and it is *implemented* with a unique constraint. It's a subtle distinction between the logical model and the physical implementation."

-   **"You have a table with a primary key and two alternate keys. Which one should a foreign key from another table reference?"**
    -   **Answer**: "The foreign key should **always** reference the `PRIMARY KEY`. The primary key is guaranteed to be stable and is the designated identifier for the row. Referencing an alternate key (like an email address) is brittle; if the email address changes, the foreign key link breaks or needs complex updates. Foreign keys should always point to the most stable key, which is the primary key."

## Bonus

### Related Concepts
-   **Candidate Key**: Any column or set of columns that is a minimal superkey. It's a candidate to be the primary key.
-   **Primary Key**: The candidate key selected to be the main identifier for the table.
-   **Superkey**: Any set of columns that uniquely identifies a row. A candidate key is a *minimal* superkey (meaning you can't remove any column from it and still have it be unique).
-   **`UNIQUE` Constraint**: The SQL implementation of an alternate key.
-   **Database Normalization**: The process of organizing tables to minimize data redundancy. Identifying all candidate keys (and thus, alternate keys) is a key step in normalization.
