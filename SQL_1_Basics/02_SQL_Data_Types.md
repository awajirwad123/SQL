# SQL Data Types

## Overview
Data types define the kind of values a column can hold in SQL. Choosing the right data type is crucial for storage optimization, query performance, and data integrity. Interviewers test your understanding of when to use each type and the trade-offs involved.

## Core Concepts

### Main Categories of SQL Data Types

#### 1. **Numeric Data Types**

```sql
-- Integer Types
TINYINT      -- 1 byte  (-128 to 127 or 0 to 255)
SMALLINT     -- 2 bytes (-32,768 to 32,767)
MEDIUMINT    -- 3 bytes (-8,388,608 to 8,388,607)
INT/INTEGER  -- 4 bytes (-2.1B to 2.1B) [Most Common]
BIGINT       -- 8 bytes (-9.2 quintillion to 9.2 quintillion)

-- Decimal Types (Exact Values)
DECIMAL(p,s) -- Exact precision (p=precision, s=scale)
NUMERIC(p,s) -- Same as DECIMAL

-- Floating-Point (Approximate Values)
FLOAT        -- 4 bytes, ~7 decimal digits precision
DOUBLE       -- 8 bytes, ~15 decimal digits precision

-- Boolean
BOOLEAN/BOOL -- Stored as TINYINT(1): 0=FALSE, 1=TRUE
```

**Example:**
```sql
CREATE TABLE products (
    product_id INT,
    price DECIMAL(10,2),      -- Max: 99999999.99
    discount_rate FLOAT,      -- Approximate: 0.15
    in_stock BOOLEAN          -- TRUE/FALSE
);
```

#### 2. **String/Character Data Types**

```sql
-- Fixed Length
CHAR(n)      -- Fixed size (0-255), pads with spaces
             -- Fast for fixed-length data (country codes, zip codes)

-- Variable Length
VARCHAR(n)   -- Variable size (0-65,535 in MySQL)
             -- Most commonly used for text

-- Large Text
TEXT         -- Max 65,535 characters
MEDIUMTEXT   -- Max 16 MB
LONGTEXT     -- Max 4 GB

-- Binary Data
BINARY(n)    -- Fixed binary data
VARBINARY(n) -- Variable binary data
BLOB         -- Binary Large Object (images, files)
```

**Example:**
```sql
CREATE TABLE users (
    username VARCHAR(50),       -- Variable: "john" uses 4 chars
    country_code CHAR(2),       -- Fixed: "US" always 2 chars
    bio TEXT,                   -- Large text
    profile_pic BLOB            -- Binary data
);
```

#### 3. **Date and Time Data Types**

```sql
DATE         -- YYYY-MM-DD (1000-01-01 to 9999-12-31)
TIME         -- HH:MM:SS (-838:59:59 to 838:59:59)
DATETIME     -- YYYY-MM-DD HH:MM:SS (1000-01-01 to 9999-12-31)
TIMESTAMP    -- UTC timestamp (1970-01-01 to 2038-01-19)
             -- Auto-updates on row modification
YEAR         -- YYYY (1901 to 2155)
```

**Example:**
```sql
CREATE TABLE orders (
    order_id INT,
    order_date DATE,                    -- 2025-10-07
    order_time TIME,                    -- 14:30:00
    created_at DATETIME,                -- 2025-10-07 14:30:00
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP 
               ON UPDATE CURRENT_TIMESTAMP
);
```

#### 4. **Special Data Types**

```sql
-- JSON (MySQL 5.7+, PostgreSQL)
JSON         -- Stores JSON documents

-- XML
XML          -- Stores XML data (SQL Server, PostgreSQL)

-- ENUM (MySQL)
ENUM('val1', 'val2', ...)  -- Predefined values only

-- SET (MySQL)
SET('val1', 'val2', ...)   -- Multiple values from list

-- UUID/GUID
UUID         -- Universally Unique Identifier (PostgreSQL)
UNIQUEIDENTIFIER -- SQL Server
```

**Example:**
```sql
CREATE TABLE employees (
    emp_id INT,
    status ENUM('active', 'inactive', 'suspended'),
    skills SET('SQL', 'Python', 'Java', 'C++'),
    metadata JSON
);
```

### Data Type Selection Guidelines

| Data | Best Data Type | Why |
|------|---------------|-----|
| Age | TINYINT | Range: 0-255 sufficient |
| Price | DECIMAL(10,2) | Exact precision needed |
| Percentage | FLOAT or DECIMAL(5,2) | Depends on precision needs |
| Email | VARCHAR(255) | Variable length |
| Password Hash | CHAR(64) | Fixed length (SHA-256) |
| Large Text | TEXT/LONGTEXT | Articles, comments |
| Files/Images | BLOB | Binary data |
| Timestamps | TIMESTAMP | Auto-update feature |

## Interview-Focused Notes

### Common Interview Questions

1. **"What's the difference between CHAR and VARCHAR?"**
   - CHAR: Fixed-length, pads with spaces, faster for fixed-size data
   - VARCHAR: Variable-length, saves space, more flexible

2. **"When would you use DECIMAL over FLOAT?"**
   - DECIMAL: Financial data (exact precision required)
   - FLOAT: Scientific calculations (approximate is acceptable)

3. **"Difference between DATETIME and TIMESTAMP?"**
   - DATETIME: Stores literal date/time, no timezone
   - TIMESTAMP: Stores UTC, converts based on timezone, auto-updates

4. **"What happens if you exceed VARCHAR(50) limit?"**
   - Error or truncation (depends on SQL mode: STRICT vs non-STRICT)

5. **"How much storage does INT vs BIGINT use?"**
   - INT: 4 bytes, BIGINT: 8 bytes (choose based on range needs)

### How to Explain in Interviews
"Choosing the right data type is critical for performance and integrity. I always consider the range of values, precision requirements, and storage constraints. For example, I use DECIMAL for financial calculations to avoid floating-point errors, and VARCHAR instead of TEXT when I know the maximum length to optimize indexing."

## Quick Recall ✅

### Must-Remember Types
- **INT** = 4 bytes, most common integer type
- **VARCHAR(n)** = Variable-length string, most common text type
- **DECIMAL(p,s)** = Exact precision (use for money!)
- **DATETIME** = Date + Time (no timezone)
- **TIMESTAMP** = UTC timestamp (auto-updates)
- **CHAR vs VARCHAR** = Fixed vs Variable length
- **FLOAT vs DECIMAL** = Approximate vs Exact

### Size Reference
```
TINYINT   = 1 byte
SMALLINT  = 2 bytes
INT       = 4 bytes
BIGINT    = 8 bytes
FLOAT     = 4 bytes
DOUBLE    = 8 bytes
```

### Quick Decision Tree
```
Need exact precision? → DECIMAL
Need speed over precision? → FLOAT
Fixed-length data? → CHAR
Variable-length data? → VARCHAR
Very large text? → TEXT
Need timezone handling? → TIMESTAMP
Simple date only? → DATE
```

## Interview Traps & Confusions ⚠️

### Common Mistakes

1. **Using VARCHAR(255) for Everything**
   - ❌ Wastes space for short data like country codes
   - ✅ Use appropriate length: CHAR(2) for country codes

2. **FLOAT for Money**
   - ❌ `price FLOAT` → Rounding errors (0.1 + 0.2 ≠ 0.3)
   - ✅ `price DECIMAL(10,2)` → Exact precision

3. **TEXT vs VARCHAR Confusion**
   - ❌ Using TEXT for short strings
   - ✅ VARCHAR is better for indexing and performance on short strings

4. **NULL vs Empty String**
   - `NULL` ≠ `''` (empty string)
   - NULL means "no value", empty string means "value is empty"

5. **TIMESTAMP Range Limitation**
   - ❌ Using TIMESTAMP for dates before 1970 or after 2038
   - ✅ Use DATETIME for wider date ranges

### Tricky Interview Scenarios

**Q: "What's the output of this?"**
```sql
SELECT 0.1 + 0.2;  -- FLOAT arithmetic
-- Answer: 0.30000000000000004 (floating-point precision issue!)
```

**Q: "How much space does VARCHAR(100) always use?"**
```
❌ "100 bytes"
✅ "1-3 bytes overhead + actual string length"
   Example: "hi" in VARCHAR(100) = 2 bytes + overhead
```

**Q: "Can you index a TEXT column?"**
```
✅ Yes, but you must specify a prefix length:
   CREATE INDEX idx ON table(text_col(100));
   VARCHAR is better for full indexing.
```

## Bonus

### Related Concepts
- **Normalization** → Proper data types reduce redundancy
- **Indexing** → Data type affects index size and performance
- **Constraints** → Data types enforce basic validation
- **Storage Engines** → InnoDB vs MyISAM handle types differently

### Real-World Example: E-commerce Product Table
```sql
CREATE TABLE products (
    product_id BIGINT AUTO_INCREMENT PRIMARY KEY,  -- Large range
    sku VARCHAR(50) NOT NULL UNIQUE,               -- Variable product codes
    name VARCHAR(255) NOT NULL,                    -- Product names vary
    description TEXT,                              -- Long descriptions
    price DECIMAL(10,2) NOT NULL,                  -- Exact pricing
    discount_percent DECIMAL(5,2),                 -- e.g., 15.50%
    in_stock BOOLEAN DEFAULT TRUE,                 -- Simple flag
    category ENUM('Electronics', 'Clothing', 'Food'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP 
               ON UPDATE CURRENT_TIMESTAMP
);
```

### Performance Tips
- Smaller data types = less storage = better performance
- Use `INT` instead of `BIGINT` if range permits
- Use `VARCHAR(n)` with appropriate `n`, not always 255
- Avoid `TEXT/BLOB` in frequently queried columns
- Use `ENUM` for fixed, small sets of values

### Cross-Database Differences
```
Data Type     MySQL           PostgreSQL      SQL Server
---------------------------------------------------------
Auto-Increment AUTO_INCREMENT SERIAL          IDENTITY
Boolean        TINYINT(1)     BOOLEAN         BIT
Text           TEXT           TEXT            VARCHAR(MAX)
JSON           JSON           JSONB           NVARCHAR(MAX)
UUID           CHAR(36)       UUID            UNIQUEIDENTIFIER
```

---

**Pro Tip**: In interviews, always justify your data type choice with performance and precision reasoning!
