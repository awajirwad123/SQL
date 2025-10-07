# Create Procedure

## Overview
Creating a stored procedure defines a reusable execution unit in the database catalog. Syntax and capabilities vary by RDBMS, but all provide parameter declaration, body definition, and permission assignment.

## Core Concepts
- **Basic Syntax (SQL Server)**:
```sql
CREATE OR ALTER PROCEDURE dbo.GetRecentOrders
  @CustomerId INT,
  @SinceDate  DATETIME2 = DATEADD(DAY,-30,SYSDATETIME())
AS
BEGIN
  SET NOCOUNT ON;
  SELECT order_id, total_amount, created_at
  FROM orders
  WHERE customer_id = @CustomerId AND created_at >= @SinceDate
  ORDER BY created_at DESC;
END;
```
- **MySQL Syntax**:
```sql
DELIMITER $$
CREATE PROCEDURE GetRecentOrders(IN p_customer_id INT, IN p_since DATETIME)
BEGIN
  SELECT order_id, total_amount, created_at
  FROM orders
  WHERE customer_id = p_customer_id AND created_at >= p_since
  ORDER BY created_at DESC;
END$$
DELIMITER ;
```
- **PostgreSQL (FUNCTION vs PROCEDURE)**:
```sql
CREATE OR REPLACE FUNCTION get_recent_orders(p_customer_id INT, p_since TIMESTAMPTZ)
RETURNS SETOF orders AS $$
BEGIN
  RETURN QUERY
  SELECT * FROM orders
  WHERE customer_id = p_customer_id AND created_at >= p_since
  ORDER BY created_at DESC;
END;$$ LANGUAGE plpgsql STABLE;
```
- **Oracle (PL/SQL)**:
```sql
CREATE OR REPLACE PROCEDURE get_recent_orders (p_customer_id IN NUMBER, p_since IN DATE) AS
BEGIN
  OPEN :result_cursor FOR
    SELECT order_id, total_amount, created_at
    FROM orders
    WHERE customer_id = p_customer_id AND created_at >= p_since
    ORDER BY created_at DESC;
END;
```

### Options & Modifiers
- Idempotent create (`CREATE OR ALTER` SQL Server, `CREATE OR REPLACE` PG/Oracle).
- Schema qualification prevents name collision.
- SET options (e.g. `SET NOCOUNT ON;`).
- Language spec (PostgreSQL PL/pgSQL, SQL, PL/Python, etc.).
- Determinism / volatility classification (Postgres: IMMUTABLE/STABLE/VOLATILE).

## Interview-Focused Notes
1. Why use default parameter values? Simplify API; reduce overload proliferation.
2. Importance of schema qualification? Avoid implicit resolution issues & security spoofing.
3. When choose function over procedure? Need scalar/inline table result, composability in SQL expressions.
4. Why include SET NOCOUNT ON (SQL Server)? Reduce network traffic from rowcount messages.
5. Why declare volatility (Postgres)? Enables optimizer rewrites/caching decisions.

## Quick Recall ✅
- Use OR ALTER / OR REPLACE for idempotent deployments.
- Provide sensible defaults.
- Qualify schema names.
- Classify volatility where supported.
- Keep procedure focused (single responsibility).

## Interview Traps & Confusions ⚠️
- Hardcoding environment-specific values.
- Omitting schema prefix leading to search path surprises.
- Copy-paste duplication instead of parameterization.
- Using procedures for simple one-line queries (adds complexity).
- Forgetting result ordering if consumer expects determinism.

## Bonus
### Deployment Friendly Wrapper
Wrap DDL in transaction with exception handling (where supported) for safe migrations.

### Template Skeleton (Postgres)
```sql
CREATE OR REPLACE FUNCTION fn_template(param1 INT)
RETURNS VOID LANGUAGE plpgsql AS $$
BEGIN
  -- logic here
END;$$;
```

### Code Comment Standard
Header: purpose, parameters, expected side effects, author, change history.

### Version Tagging
Embed semantic version in comment block for auditing.
