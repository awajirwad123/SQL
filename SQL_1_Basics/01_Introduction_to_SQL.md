# Introduction to SQL and Foundational Concepts

## Overview
SQL (Structured Query Language) is the standard language for managing and manipulating relational databases. It allows you to create, read, update, and delete data efficiently. In interviews, SQL tests your ability to think in sets, understand database relationships, and write optimized queries.

## Core Concepts

### What is SQL?
- **SQL** = Structured Query Language
- Declarative language (you specify *what* you want, not *how* to get it)
- Standard language for RDBMS (MySQL, PostgreSQL, SQL Server, Oracle, SQLite)
- Based on **relational algebra** and **set theory**

### Key Database Concepts

**Database**: Collection of organized data stored electronically

**Table (Relation)**: Structured data format with rows and columns
- **Row (Tuple/Record)**: Single entry in a table
- **Column (Attribute/Field)**: Property or characteristic of data

**Schema**: Logical structure/blueprint of the database

**Primary Key (PK)**: 
- Unique identifier for each record in a table
- Cannot be NULL
- Each table should have one PK

**Foreign Key (FK)**:
- Column that creates a relationship between two tables
- References a Primary Key in another table
- Maintains referential integrity

**Constraints**:
- Rules enforced on data columns
- Types: `NOT NULL`, `UNIQUE`, `PRIMARY KEY`, `FOREIGN KEY`, `CHECK`, `DEFAULT`

### RDBMS vs NoSQL (Quick Comparison)
```
RDBMS (SQL):
✓ Structured data with fixed schema
✓ ACID compliance (Atomicity, Consistency, Isolation, Durability)
✓ Vertical scaling
✓ Best for: Financial systems, ERP, complex queries

NoSQL:
✓ Flexible/dynamic schema
✓ Horizontal scaling
✓ BASE model (Basically Available, Soft state, Eventually consistent)
✓ Best for: Big data, real-time applications, unstructured data
```

### SQL Query Execution Order
```sql
-- Written Order:
SELECT column
FROM table
WHERE condition
GROUP BY column
HAVING condition
ORDER BY column
LIMIT count

-- Execution Order (Interview Favorite!):
1. FROM       -- Get the data source
2. WHERE      -- Filter rows
3. GROUP BY   -- Group rows
4. HAVING     -- Filter groups
5. SELECT     -- Choose columns
6. ORDER BY   -- Sort results
7. LIMIT      -- Limit results
```

## Interview-Focused Notes

### Common Interview Questions

1. **"What is SQL and why is it important?"**
   - Standard language for relational databases, enables data manipulation and retrieval efficiently

2. **"Explain the difference between SQL and MySQL"**
   - SQL = Language; MySQL = Database Management System that uses SQL

3. **"What is a relational database?"**
   - Database organized into tables with relationships through keys (PK/FK)

4. **"What are the main advantages of SQL?"**
   - Standardized, powerful querying, data integrity, supports large databases, ACID compliance

5. **"Explain normalization in one sentence"**
   - Process of organizing data to reduce redundancy and improve data integrity

### How to Explain in Interviews
"SQL is a declarative language used to interact with relational databases. It allows us to define what data we want using queries, while the database engine optimizes how to retrieve it. The relational model ensures data integrity through constraints and relationships between tables."

## Quick Recall ✅

- **SQL** = Structured Query Language (declarative, set-based)
- **RDBMS** = Relational Database Management System
- **ACID** = Atomicity, Consistency, Isolation, Durability
- **Primary Key** = Unique identifier (NOT NULL, UNIQUE)
- **Foreign Key** = References PK in another table (maintains integrity)
- **Schema** = Logical blueprint of database structure
- **Execution Order**: FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

## Interview Traps & Confusions ⚠️

### Common Mistakes

1. **SQL vs MySQL Confusion**
   - ❌ "I'm learning MySQL" (when you mean SQL)
   - ✅ "I'm learning SQL using MySQL database"

2. **Query Writing Order vs Execution Order**
   - ❌ Thinking SELECT runs first
   - ✅ Know that FROM runs first, SELECT runs after WHERE/GROUP BY

3. **NULL Handling**
   - ❌ `WHERE column = NULL` (Wrong!)
   - ✅ `WHERE column IS NULL` (Correct!)

4. **Primary Key Misconceptions**
   - ❌ Thinking PK can be NULL or duplicate
   - ✅ PK must be UNIQUE and NOT NULL

5. **RDBMS = SQL?**
   - ❌ Treating them as the same thing
   - ✅ RDBMS is the system; SQL is the language

### Tricky Interview Scenarios

- **"Can a table have multiple primary keys?"**
  - ❌ No, only ONE primary key per table
  - ✅ But a PK can be composite (multiple columns combined)

- **"What happens if you delete a row referenced by a foreign key?"**
  - Depends on `ON DELETE` action: CASCADE, SET NULL, RESTRICT, NO ACTION

## Bonus

### Related Concepts to Revise Together
- **Normalization** (1NF, 2NF, 3NF, BCNF) → reduces redundancy
- **Indexes** → speeds up data retrieval
- **Transactions** → ACID properties in action
- **Joins** → combining tables using relationships
- **Query Optimization** → understanding execution plans

### Real-World Connection
```
E-commerce System Example:
- Customers table (PK: customer_id)
- Orders table (PK: order_id, FK: customer_id)
- Products table (PK: product_id)
- Order_Items table (PK: composite of order_id + product_id)

This demonstrates relationships, constraints, and why SQL matters!
```

### Quick Win for Interviews
Always mention **ACID properties** when discussing RDBMS reliability:
- **A**tomicity: All or nothing (transactions)
- **C**onsistency: Data integrity maintained
- **I**solation: Concurrent transactions don't interfere
- **D**urability: Committed data persists even after system failure

---

**Next Steps**: Practice writing basic queries and understand the execution flow deeply!
