# Logical Operators

## Overview
In SQL, logical operators are used to combine or negate boolean expressions, primarily within the `WHERE` and `HAVING` clauses. They are the fundamental building blocks for creating complex, multi-part filters. The standard logical operators are `AND`, `OR`, and `NOT`.

## Core Concepts

A boolean expression is any expression that evaluates to `TRUE`, `FALSE`, or `UNKNOWN` (which is how SQL handles `NULL`). Logical operators take these boolean values as input and produce a new boolean value.

### 1. `AND`
-   **Purpose**: Combines two conditions and returns `TRUE` only if **both** are `TRUE`.
-   **Use Case**: To narrow a search and make a filter more specific.
-   **Example**: `WHERE price > 50 AND category = 'Electronics'`

### 2. `OR`
-   **Purpose**: Combines two conditions and returns `TRUE` if **either** one is `TRUE`.
-   **Use Case**: To widen a search and provide alternative criteria.
-   **Example**: `WHERE category = 'Books' OR category = 'Magazines'`

### 3. `NOT`
-   **Purpose**: Reverses the value of a boolean expression. `NOT TRUE` becomes `FALSE`, `NOT FALSE` becomes `TRUE`, and `NOT NULL` remains `NULL`.
-   **Use Case**: To exclude certain results.
-   **Example**: `WHERE NOT country = 'USA'`

### Truth Tables
Understanding how operators work, especially with `NULL`, is key.

**AND Truth Table**
| A | B | A AND B |
|:---|:---|:---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | FALSE |
| TRUE | NULL | NULL |
| FALSE| FALSE | FALSE |
| FALSE| NULL | FALSE |
| NULL | NULL | NULL |

**OR Truth Table**
| A | B | A OR B |
|:---|:---|:---|
| TRUE | TRUE | TRUE |
| TRUE | FALSE | TRUE |
| TRUE | NULL | TRUE |
| FALSE| FALSE | FALSE |
| FALSE| NULL | NULL |
| NULL | NULL | NULL |

### Operator Precedence
In SQL, the order of evaluation for logical operators is:
1.  `NOT` (highest precedence)
2.  `AND`
3.  `OR` (lowest precedence)

Because this can lead to confusion, it is a universal best practice to **use parentheses `()`** to explicitly control the order of operations.

**Example:**
`WHERE NOT on_sale AND (category = 'Books' OR price < 10)`
Here, the `OR` condition is evaluated first because of the parentheses, then the `AND`, and finally the `NOT`.

## Interview-Focused Notes

### Common Interview Questions

1.  **"What are the main logical operators in SQL?"**
    -   "The three main logical operators are `AND`, `OR`, and `NOT`. They are used to construct complex filtering logic in `WHERE` and `HAVING` clauses."

2.  **"Explain the order of precedence for logical operators."**
    -   "The default order is `NOT`, then `AND`, then `OR`. However, relying on this default precedence is risky and can lead to bugs. The best practice is to always use parentheses to group conditions and make the intended logic explicit and unambiguous."

3.  **"How does SQL handle `NULL` in logical operations?"**
    -   "SQL uses a three-valued logic system: `TRUE`, `FALSE`, and `UNKNOWN` (represented by `NULL`). Any comparison to `NULL` results in `NULL`. For a row to be included by a `WHERE` clause, the final expression must evaluate to `TRUE`. Both `FALSE` and `NULL` results are rejected. For example, `TRUE AND NULL` is `NULL`, so the row is rejected. `TRUE OR NULL` is `TRUE`, so the row is accepted."

### How to Explain in Interviews
"Logical operators are the glue for SQL filtering. `AND` narrows a search, `OR` widens it, and `NOT` inverts it. The most critical concept is operator precedence. I never rely on the default precedence of `AND` over `OR`. Instead, I use parentheses to group conditions, which makes the query's logic self-documenting and prevents common errors. Understanding three-valued logic with `NULL` is also key to debugging why certain rows might not be appearing in a result set."

## Quick Recall ✅

-   **Operators**: `AND`, `OR`, `NOT`.
-   **`AND`**: All conditions must be true.
-   **`OR`**: Any condition can be true.
-   **`NOT`**: Reverses the result of a condition.
-   **Precedence**: `NOT` > `AND` > `OR`.
-   **Best Practice**: **Always use parentheses `()`** to group mixed `AND`/`OR` conditions.
-   **Three-Valued Logic**: Conditions can evaluate to `TRUE`, `FALSE`, or `NULL`. Only `TRUE` passes the `WHERE` clause filter.

## Interview Traps & Confusions ⚠️

### Common Mistakes

1.  **Relying on Default Precedence:** The most common trap.
    `WHERE a = 1 AND b = 2 OR c = 3` is evaluated as `(a = 1 AND b = 2) OR c = 3`. This is often not what the developer intended.

2.  **Incorrectly Negating with `NOT`:**
    `WHERE NOT status = 'Active' OR status = 'Pending'`
    This is probably wrong. The user likely meant to exclude both.
    Correct way: `WHERE NOT (status = 'Active' OR status = 'Pending')`
    Or more simply: `WHERE status NOT IN ('Active', 'Pending')`

3.  **Thinking `col = NULL` works.**
    -   Any direct comparison to `NULL` yields `NULL`. You must use `IS NULL` or `IS NOT NULL`.

### Tricky Interview Scenarios

-   **"A query with `WHERE status <> 'Completed'` is not returning rows where the status is `NULL`. Why?"**
    -   **Answer**: "This is due to three-valued logic. When `status` is `NULL`, the expression `NULL <> 'Completed'` evaluates to `NULL`, not `TRUE`. The `WHERE` clause only includes rows that evaluate to `TRUE`, so the `NULL` rows are excluded. To include them, the condition would need to be `WHERE status <> 'Completed' OR status IS NULL`."

-   **"Simplify the following `WHERE` clause: `WHERE (a=1 AND b=2) OR (a=1 AND c=3)`"**
    -   This tests knowledge of logical simplification (distributive property).
    -   **Answer**: "You can factor out the common condition `a=1`. The simplified clause is `WHERE a = 1 AND (b = 2 OR c = 3)`."

## Bonus

### Other Logical Operators
While `AND`, `OR`, and `NOT` are the core, other SQL operators behave logically:
-   **`BETWEEN`**: `col BETWEEN 10 AND 20` is syntactic sugar for `col >= 10 AND col <= 20`.
-   **`IN`**: `col IN (1, 2, 3)` is syntactic sugar for `col = 1 OR col = 2 OR col = 3`.
-   **`EXISTS`**: A boolean operator that returns `TRUE` if a subquery returns any rows.
-   **`ALL` / `ANY` / `SOME`**: Quantifiers that compare a value to a list of values from a subquery.
