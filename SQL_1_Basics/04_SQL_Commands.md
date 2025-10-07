# SQL Commands (DDL, DQL, DML, DCL, and TCL)

## Overview
SQL commands are categorized into five main types based on their functionality. Understanding these categories is crucial for interviews as it demonstrates your grasp of database operations, from structure creation to data manipulation and access control.

## Core Concepts

### Command Categories Overview

```
┌─────────────────────────────────────────────────┐
│            SQL Command Categories               │
├─────────────────────────────────────────────────┤
│ DDL  → Define database structure (CREATE, ALTER)│
│ DQL  → Query data (SELECT)                      │
│ DML  → Manipulate data (INSERT, UPDATE, DELETE) │
│ DCL  → Control access (GRANT, REVOKE)           │
│ TCL  → Manage transactions (COMMIT, ROLLBACK)   │
└─────────────────────────────────────────────────┘
```

---

## 1. DDL - Data Definition Language

**Purpose**: Define and modify database structure/schema

**Key Commands**: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`

**Characteristics**:
- Auto-commit (changes are permanent immediately)
- Cannot be rolled back
- Affects structure, not data

### **CREATE** - Create Database Objects

```sql
-- Create Database
CREATE DATABASE ecommerce;

-- Create Table
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(15),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Create Index
CREATE INDEX idx_email ON customers(email);

-- Create View
CREATE VIEW active_customers AS
SELECT * FROM customers WHERE status = 'active';
```

### **ALTER** - Modify Existing Structure

```sql
-- Add Column
ALTER TABLE customers 
ADD COLUMN address VARCHAR(255);

-- Modify Column
ALTER TABLE customers 
MODIFY COLUMN phone VARCHAR(20);

-- Drop Column
ALTER TABLE customers 
DROP COLUMN address;

-- Add Constraint
ALTER TABLE orders 
ADD CONSTRAINT fk_customer 
FOREIGN KEY (customer_id) REFERENCES customers(customer_id);

-- Rename Column (MySQL 8.0+)
ALTER TABLE customers 
RENAME COLUMN phone TO phone_number;
```

### **DROP** - Delete Database Objects

```sql
-- Drop Table (deletes table and all data permanently)
DROP TABLE customers;

-- Drop Database
DROP DATABASE ecommerce;

-- Drop Index
DROP INDEX idx_email ON customers;

-- Drop with safety check
DROP TABLE IF EXISTS customers;  -- No error if table doesn't exist
```

### **TRUNCATE** - Remove All Rows (Keep Structure)

```sql
-- Faster than DELETE (no row-by-row logging)
TRUNCATE TABLE orders;

-- Differences from DELETE:
-- 1. Cannot use WHERE clause
-- 2. Resets AUTO_INCREMENT counter
-- 3. Faster (minimal logging)
-- 4. Cannot be rolled back (DDL command)
```

### **RENAME** - Rename Objects

```sql
-- Rename Table
RENAME TABLE old_customers TO customers;
ALTER TABLE old_name RENAME TO new_name;  -- Alternative syntax
```

---

## 2. DQL - Data Query Language

**Purpose**: Retrieve data from database

**Key Command**: `SELECT`

**Characteristics**:
- Most frequently used SQL command
- Read-only (doesn't modify data)
- Can be complex with joins, subqueries, aggregations

### **SELECT** - Query Data

```sql
-- Basic SELECT
SELECT * FROM customers;
SELECT name, email FROM customers;

-- With WHERE clause
SELECT * FROM customers WHERE country = 'USA';

-- With ORDER BY
SELECT * FROM products ORDER BY price DESC;

-- With LIMIT
SELECT * FROM products LIMIT 10;

-- With aggregate functions
SELECT COUNT(*) FROM orders;
SELECT AVG(price) FROM products;
SELECT SUM(total) FROM orders WHERE status = 'completed';

-- With GROUP BY
SELECT country, COUNT(*) AS customer_count
FROM customers
GROUP BY country
HAVING customer_count > 100;

-- With JOINs
SELECT c.name, o.order_id, o.total
FROM customers c
INNER JOIN orders o ON c.customer_id = o.customer_id;

-- With subqueries
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);

-- With DISTINCT
SELECT DISTINCT country FROM customers;

-- With CASE
SELECT 
    name,
    CASE 
        WHEN price < 100 THEN 'Cheap'
        WHEN price < 500 THEN 'Medium'
        ELSE 'Expensive'
    END AS price_category
FROM products;
```

---

## 3. DML - Data Manipulation Language

**Purpose**: Manipulate/modify data in tables

**Key Commands**: `INSERT`, `UPDATE`, `DELETE`

**Characteristics**:
- Can be rolled back (with transactions)
- Affects data, not structure
- Triggers DML triggers (if defined)

### **INSERT** - Add New Rows

```sql
-- Insert single row
INSERT INTO customers (name, email, phone)
VALUES ('John Doe', 'john@example.com', '1234567890');

-- Insert multiple rows
INSERT INTO customers (name, email, phone)
VALUES 
    ('Alice', 'alice@example.com', '1111111111'),
    ('Bob', 'bob@example.com', '2222222222');

-- Insert with SELECT
INSERT INTO archived_orders
SELECT * FROM orders WHERE order_date < '2020-01-01';

-- Insert with default values
INSERT INTO customers (name, email) 
VALUES ('Jane', 'jane@example.com');  -- Other columns get defaults
```

### **UPDATE** - Modify Existing Rows

```sql
-- Update single row
UPDATE customers 
SET phone = '9999999999'
WHERE customer_id = 1;

-- Update multiple rows
UPDATE products 
SET price = price * 1.10
WHERE category = 'Electronics';

-- Update with multiple columns
UPDATE customers
SET status = 'inactive', updated_at = NOW()
WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- ⚠️ DANGER: Update without WHERE updates ALL rows!
UPDATE products SET price = 0;  -- Sets all prices to 0!
```

### **DELETE** - Remove Rows

```sql
-- Delete specific rows
DELETE FROM orders 
WHERE status = 'cancelled';

-- Delete with subquery
DELETE FROM customers
WHERE customer_id NOT IN (SELECT DISTINCT customer_id FROM orders);

-- ⚠️ DANGER: Delete without WHERE deletes ALL rows!
DELETE FROM customers;  -- Deletes everything!

-- Difference from TRUNCATE:
-- DELETE: Row-by-row, can use WHERE, can rollback, slower
-- TRUNCATE: Bulk operation, no WHERE, faster, can't rollback
```

---

## 4. DCL - Data Control Language

**Purpose**: Control access to data (permissions and security)

**Key Commands**: `GRANT`, `REVOKE`

**Characteristics**:
- Manages user privileges
- Database administrator operations
- Important for security

### **GRANT** - Give Permissions

```sql
-- Grant SELECT permission
GRANT SELECT ON database.table TO 'username'@'host';

-- Grant multiple permissions
GRANT SELECT, INSERT, UPDATE ON ecommerce.* TO 'app_user'@'localhost';

-- Grant all permissions
GRANT ALL PRIVILEGES ON ecommerce.* TO 'admin_user'@'localhost';

-- Grant with grant option (user can grant to others)
GRANT SELECT ON customers TO 'user1'@'localhost' WITH GRANT OPTION;

-- Grant specific columns
GRANT SELECT (customer_id, name) ON customers TO 'readonly_user'@'localhost';
```

### **REVOKE** - Remove Permissions

```sql
-- Revoke specific permission
REVOKE INSERT ON ecommerce.customers FROM 'app_user'@'localhost';

-- Revoke all permissions
REVOKE ALL PRIVILEGES ON ecommerce.* FROM 'app_user'@'localhost';

-- Revoke grant option
REVOKE GRANT OPTION ON ecommerce.* FROM 'user1'@'localhost';
```

---

## 5. TCL - Transaction Control Language

**Purpose**: Manage database transactions (ACID compliance)

**Key Commands**: `COMMIT`, `ROLLBACK`, `SAVEPOINT`, `SET TRANSACTION`

**Characteristics**:
- Ensures data integrity
- Controls transaction boundaries
- Critical for multi-step operations

### **BEGIN/START TRANSACTION** - Start Transaction

```sql
START TRANSACTION;
-- or
BEGIN;
```

### **COMMIT** - Save Changes Permanently

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

COMMIT;  -- Both updates are now permanent
```

### **ROLLBACK** - Undo Changes

```sql
START TRANSACTION;

DELETE FROM customers WHERE customer_id = 100;

-- Oops, wrong ID!
ROLLBACK;  -- Deletion is undone

-- Customer 100 is still in the database
```

### **SAVEPOINT** - Create Checkpoint in Transaction

```sql
START TRANSACTION;

INSERT INTO orders VALUES (1, 'Order 1');
SAVEPOINT sp1;

INSERT INTO orders VALUES (2, 'Order 2');
SAVEPOINT sp2;

INSERT INTO orders VALUES (3, 'Order 3');

-- Undo to sp2 (removes Order 3, keeps 1 and 2)
ROLLBACK TO SAVEPOINT sp2;

COMMIT;  -- Orders 1 and 2 are saved
```

### **SET TRANSACTION** - Configure Transaction Properties

```sql
-- Set isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

START TRANSACTION;
-- Transaction code here
COMMIT;
```

---

## Interview-Focused Notes

### Common Interview Questions

1. **"Explain the difference between DDL and DML"**
   - DDL: Structure changes (CREATE, ALTER), auto-commits, can't rollback
   - DML: Data changes (INSERT, UPDATE), can rollback with transactions

2. **"What's the difference between DELETE and TRUNCATE?"**
   - DELETE: DML, row-by-row, can use WHERE, can rollback, keeps AUTO_INCREMENT
   - TRUNCATE: DDL, bulk operation, faster, can't rollback, resets AUTO_INCREMENT

3. **"What is a transaction? Give an example."**
   - A sequence of operations treated as a single unit (all succeed or all fail)
   - Example: Bank transfer (debit from A, credit to B—both must happen)

4. **"What does COMMIT do?"**
   - Saves all transaction changes permanently to the database

5. **"Can you rollback a DDL command?"**
   - No (in most databases). DDL commands auto-commit immediately.

### How to Explain in Interviews
"SQL commands are categorized by their purpose. DDL defines structure, DML manipulates data, DQL queries data, DCL manages permissions, and TCL controls transactions. Understanding these categories helps in designing robust applications—for instance, wrapping DML operations in transactions ensures ACID compliance for critical operations like financial transfers."

## Quick Recall ✅

### Command Categories
```
DDL (Definition):    CREATE, ALTER, DROP, TRUNCATE, RENAME
DQL (Query):         SELECT
DML (Manipulation):  INSERT, UPDATE, DELETE
DCL (Control):       GRANT, REVOKE
TCL (Transaction):   COMMIT, ROLLBACK, SAVEPOINT
```

### Key Differences
```
DELETE vs TRUNCATE:
DELETE → DML, can rollback, uses WHERE, slower, keeps AUTO_INCREMENT
TRUNCATE → DDL, can't rollback, no WHERE, faster, resets AUTO_INCREMENT

DROP vs TRUNCATE:
DROP → Deletes table structure + data
TRUNCATE → Deletes data, keeps structure
```

### Transaction Flow
```
START TRANSACTION
  ↓
Execute DML commands
  ↓
COMMIT (save) or ROLLBACK (undo)
```

### Auto-Commit Behavior
```
DDL Commands → Auto-commit (permanent immediately)
DML Commands → Can be part of transaction (rollback possible)
```

## Interview Traps & Confusions ⚠️

### Common Mistakes

1. **Forgetting WHERE in UPDATE/DELETE**
   ```sql
   ❌ DELETE FROM customers;  -- Deletes ALL customers!
   ✅ DELETE FROM customers WHERE customer_id = 10;
   
   -- Always test with SELECT first:
   SELECT * FROM customers WHERE customer_id = 10;
   -- Then replace SELECT with DELETE
   ```

2. **Confusing DROP and TRUNCATE**
   ```sql
   DROP TABLE orders;     -- Table gone forever
   TRUNCATE TABLE orders; -- Table exists, data gone
   ```

3. **Assuming TRUNCATE Can Be Rolled Back**
   ```sql
   START TRANSACTION;
   TRUNCATE TABLE test;  -- Can't rollback (DDL)!
   ROLLBACK;  -- ❌ Doesn't work
   ```

4. **Not Using Transactions for Critical Operations**
   ```sql
   ❌ No transaction:
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   -- App crashes here!
   UPDATE accounts SET balance = balance + 100 WHERE id = 2;
   -- Money lost!
   
   ✅ With transaction:
   START TRANSACTION;
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   UPDATE accounts SET balance = balance + 100 WHERE id = 2;
   COMMIT;  -- Both or neither
   ```

5. **Mixing DDL and DML in Transactions**
   ```sql
   START TRANSACTION;
   INSERT INTO orders VALUES (...);  -- Can rollback
   ALTER TABLE orders ADD COLUMN x;  -- Auto-commits! Previous INSERT committed too!
   ROLLBACK;  -- ❌ Doesn't undo INSERT
   ```

### Tricky Interview Scenarios

**Q: "What happens here?"**
```sql
START TRANSACTION;
INSERT INTO users VALUES (1, 'Alice');
TRUNCATE TABLE logs;  -- DDL command
ROLLBACK;

-- Answer: INSERT is committed (can't be rolled back)
-- Reason: TRUNCATE auto-commits, committing the entire transaction
```

**Q: "DELETE vs TRUNCATE performance?"**
```sql
DELETE FROM large_table;  -- Slow (logs each row deletion)
TRUNCATE TABLE large_table;  -- Fast (deallocates data pages)

-- TRUNCATE is 10-100x faster on large tables
```

**Q: "Can you rollback COMMIT?"**
```sql
START TRANSACTION;
DELETE FROM orders;
COMMIT;
-- ❌ Can't rollback after COMMIT (changes are permanent)
```

**Q: "What's the output?"**
```sql
SELECT * FROM products;  -- 10 rows

START TRANSACTION;
DELETE FROM products WHERE id = 1;
SELECT COUNT(*) FROM products;  -- Output: 9 (within transaction)
ROLLBACK;

SELECT COUNT(*) FROM products;  -- Output: 10 (rollback worked)
```

## Bonus

### Related Concepts
- **ACID Properties** → Transactions ensure Atomicity, Consistency, Isolation, Durability
- **Isolation Levels** → READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
- **Triggers** → Automatic actions on DML commands
- **Stored Procedures** → Combine multiple SQL commands
- **Constraints** → Defined with DDL, enforced during DML

### Real-World Use Cases

#### Bank Transfer (Transaction)
```sql
START TRANSACTION;

-- Debit from sender
UPDATE accounts 
SET balance = balance - 500 
WHERE account_id = 101;

-- Credit to receiver
UPDATE accounts 
SET balance = balance + 500 
WHERE account_id = 202;

-- Log transaction
INSERT INTO transaction_log (from_account, to_account, amount, timestamp)
VALUES (101, 202, 500, NOW());

COMMIT;  -- All or nothing!
```

#### User Registration (Multiple Operations)
```sql
START TRANSACTION;

-- Insert user
INSERT INTO users (username, email, password_hash)
VALUES ('john_doe', 'john@example.com', 'hashed_password');

-- Get the new user ID
SET @user_id = LAST_INSERT_ID();

-- Create user profile
INSERT INTO profiles (user_id, first_name, last_name)
VALUES (@user_id, 'John', 'Doe');

-- Assign default role
INSERT INTO user_roles (user_id, role_id)
VALUES (@user_id, 3);  -- Role 3 = 'customer'

COMMIT;
```

#### Audit Trail (Using Triggers with DML)
```sql
-- Create audit table
CREATE TABLE audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    action VARCHAR(10),
    user VARCHAR(50),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Trigger on DELETE
CREATE TRIGGER after_customer_delete
AFTER DELETE ON customers
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, action, user)
    VALUES ('customers', 'DELETE', USER());
END;
```

### Command Usage Frequency (Industry Perspective)
```
Frequency: Most Used → Least Used

DQL (SELECT)     ████████████ 80% of queries
DML (INSERT/UPDATE/DELETE) ████ 15%
DDL (CREATE/ALTER) ██ 4%
TCL (COMMIT/ROLLBACK) █ 0.8%
DCL (GRANT/REVOKE) ▌ 0.2%
```

### Best Practices by Category

**DDL**:
- Always backup before ALTER/DROP
- Use `IF EXISTS` for safety
- Plan schema changes carefully (production impact)

**DML**:
- Always test UPDATE/DELETE with SELECT first
- Use transactions for multi-step operations
- Add WHERE clause (avoid accidental mass updates)

**DQL**:
- Use indexes on frequently queried columns
- Limit result sets (avoid SELECT *)
- Use appropriate JOINs

**TCL**:
- Keep transactions short
- Handle errors and rollback appropriately
- Understand isolation levels for your use case

**DCL**:
- Follow principle of least privilege
- Regularly audit user permissions
- Revoke unused permissions

---

**Pro Tip**: In interviews, always mention transactions when discussing data integrity or multi-step operations!
