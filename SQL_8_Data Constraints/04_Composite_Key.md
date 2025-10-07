# Composite Key

## Overview
A **composite key** is a key made up of two or more columns in a table that are used together to uniquely identify each row. Like any key, its primary purpose is to enforce uniqueness. A composite key can be a primary key or a unique key.

## Core Concepts

A composite key is necessary when a single column is not sufficient to uniquely identify a record. The uniqueness is guaranteed only when the values of all columns in the key are considered together.

**When to use a composite key:**
The most common use case is in a **linking table** (also known as a junction, join, or associative table) that resolves a many-to-many relationship.

**Example: Many-to-Many Relationship**
Consider a `students` table and a `courses` table. A student can enroll in many courses, and a course can have many students. To model this, we create a linking table called `enrollments`.

-   `students` (<u>student_id</u>, student_name)
-   `courses` (<u>course_id</u>, course_name)
-   `enrollments` (<u>student_id</u>, <u>course_id</u>, enrollment_date)

In the `enrollments` table:
-   `student_id` by itself is not unique (a student can enroll in many courses).
-   `course_id` by itself is not unique (a course can have many students).
-   However, the **combination** of `(student_id, course_id)` **is** unique. A student can only enroll in the same course once.
-   Therefore, `(student_id, course_id)` is the composite primary key for the `enrollments` table.

**Syntax (Composite Primary Key):**
You must use the table-level syntax to define a composite key.
```sql
CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**Syntax (Composite Unique Key):**
```sql
CREATE TABLE appointments (
    appointment_id INT PRIMARY KEY,
    doctor_id INT,
    patient_id INT,
    appointment_time DATETIME,
    -- A doctor can only have one appointment at a specific time
    UNIQUE (doctor_id, appointment_time)
);
```

## Interview-Focused Notes

### Common Interview Questions

1.  **"What is a composite key?"**
    -   "A composite key is a key that consists of two or more columns, where the combination of the column values must be unique for every row in the table. It's used when a single column isn't sufficient to uniquely identify a record."

2.  **"Where would you typically use a composite primary key?"**
    -   "The most common scenario is in a linking table that models a many-to-many relationship. For example, in an `order_items` table linking `orders` and `products`, the composite primary key would be `(order_id, product_id)` to ensure a product can only be added to an order once."

3.  **"Can the individual columns of a composite primary key contain duplicate values?"**
    -   "Yes, absolutely. That's the whole point of a composite key. For example, in an `enrollments` table with a primary key of `(student_id, course_id)`, `student_id` can appear multiple times and `course_id` can appear multiple times. It's only the combined pair that must be unique."

4.  **"Can any part of a composite primary key be `NULL`?"**
    -   "No. By definition, a primary key (whether single or composite) cannot contain any `NULL` values. All columns that are part of the composite primary key must be defined as `NOT NULL`."

### How to Explain in Interviews
"A composite key is a practical solution for when a single attribute can't uniquely identify a record. I use them most often for linking tables in many-to-many relationships. For instance, to connect users and roles, a `user_roles` table would have a composite primary key of `(user_id, role_id)`. This elegantly enforces the business rule that a user can have a specific role assigned only once."

## Quick Recall ✅

-   **Definition**: A key formed by combining two or more columns.
-   **Purpose**: To uniquely identify a row when one column is not enough.
-   **Main Use Case**: Linking tables in many-to-many relationships.
-   **Uniqueness**: The combination of values across the columns must be unique.
-   **`NULL`s**: If it's a composite **primary** key, no part can be `NULL`. If it's a composite **unique** key, the rules for `NULL`s can vary by database system (some allow multiple rows as long as at least one part of the key is `NULL`).

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Overusing composite keys as primary keys.**
    -   While necessary for linking tables, using large composite keys (3+ columns) as the primary key can become cumbersome. Foreign keys referencing them also have to include all the columns, making joins more complex and using more storage.
    -   **Alternative**: In such cases, it's often better to have a simple surrogate `id` column as the primary key and create a composite `UNIQUE` constraint on the columns that define natural uniqueness.
    ```sql
    CREATE TABLE complex_linking (
        id INT PRIMARY KEY AUTO_INCREMENT, -- Surrogate PK
        col_a INT,
        col_b INT,
        col_c INT,
        UNIQUE (col_a, col_b, col_c) -- Composite UNIQUE key
    );
    ```
    This gives you the best of both worlds: a simple, stable primary key for foreign key relationships and the enforcement of the composite business rule.

2.  **Order of columns in the key.**
    -   The order of columns in a composite key matters for the underlying index. For performance, you should generally place the column with the highest cardinality (most unique values) first in the index definition. For example, if you query more often by `course_id` than `student_id`, defining the key as `(course_id, student_id)` might be more performant.

### Tricky Interview Scenarios

-   **"You have a linking table with a composite primary key. Is it a good idea to add a separate surrogate primary key as well?"**
    -   **Answer**: "It depends. For a simple linking table like `(order_id, product_id)`, a composite primary key is standard and efficient. However, if other tables need to reference a specific row *in the linking table itself* (e.g., an `order_item_notes` table needs to point to a specific `order_item`), then having a single-column surrogate primary key on the linking table (`order_item_id`) makes those subsequent foreign key relationships much simpler. So, if the linking table is just a link, a composite key is fine. If it's an entity in its own right, a surrogate key is better."

## Bonus

### Related Concepts
-   **Primary Key**: A composite key can serve as a table's primary key.
-   **Unique Key**: A composite key can also be defined as a unique key, enforcing uniqueness while allowing for a separate primary key.
-   **Many-to-Many Relationship**: The primary modeling scenario that requires composite keys.
-   **Index**: A composite key is always backed by a composite index. The performance of this index is a key database tuning consideration.
-   **Candidate Key**: In database theory, any key (simple or composite) that is a candidate to be the primary key is a candidate key.
