# Parameters and Returns

## Overview
Parameters define the interface contract to a stored procedure or function. Proper parameter design improves reusability, safety, and plan quality. Return mechanisms vary: explicit RETURN values (functions), OUT / INOUT parameters, result sets, table returns.

## Core Concepts
- **Parameter Modes**:
  - IN: read-only input.
  - OUT: procedure assigns value for caller.
  - INOUT: both input seed and output result.
- **Defaults**: Simplify calls; must be at the end in some engines (SQL Server optional ordering via naming; MySQL requires order).
- **Data Type Fidelity**: Match table column types to avoid implicit conversions (hurts SARGability/plan reuse).
- **Table-Valued Parameters** (SQL Server) vs arrays/JSON for bulk sets (PG / MySQL) when passing lists.
- **Return Strategies**:
  - Scalar return (FUNCTION)
  - Set-returning function (Postgres)
  - OUT parameters (MySQL/Oracle)
  - Multiple result sets (SQL Server procedures)
  - Return codes (legacy integer status)

### Example Modes (MySQL)
```sql
CREATE PROCEDURE adjust_price(IN p_id INT, IN p_factor DECIMAL(5,2), OUT p_new DECIMAL(10,2))
BEGIN
  UPDATE products SET price = price * p_factor WHERE product_id = p_id;
  SELECT price INTO p_new FROM products WHERE product_id = p_id;
END;
```

### Table-Valued Parameter (SQL Server)
```sql
CREATE TYPE dbo.IdList AS TABLE(id INT PRIMARY KEY);
GO
CREATE OR ALTER PROCEDURE dbo.DeleteByIds @Ids dbo.IdList READONLY AS
BEGIN
  DELETE FROM items WHERE id IN (SELECT id FROM @Ids);
END;
```

## Interview-Focused Notes
1. Why prefer IN parameters + deterministic output vs INOUT? Clarity & testability.
2. OUT vs result set? OUT for scalar meta-info; result set for tabular data.
3. Benefits of table-valued parameter? Avoid splitting comma strings; preserves type & plan quality.
4. Danger of implicit conversion? Prevents index seek (e.g., NVARCHAR vs INT mismatch).
5. Passing large arrays? Consider temp table load or TVP vs huge IN lists (plan bloat risk).

## Quick Recall ✅
- Keep interface minimal & explicit.
- Use defaults for optional behavior.
- Match types exactly.
- OUT sparingly; consider single result set design.
- Bulk sets: TVP / temp table / staging table.

## Interview Traps & Confusions ⚠️
- Overusing INOUT causing side-effect ambiguity.
- Encoding collections in CSV strings (parsing cost + injection risk).
- Returning generic VARCHAR for typed numeric.
- Ignoring collation/case issues for text parameters.
- Using functions with side effects (bad composability expectations).

## Bonus
### JSON Parameter Pattern (Postgres)
Pass structured JSON, parse with `->>`; validate schema inside function.

### Defensive NULL Handling
Normalize NULL sentinel logic at boundary; avoid duplicating in body.

### Optional Filter Pattern
Apply parameter only if non-NULL: dynamic predicate building or `(col = @p OR @p IS NULL)` (beware of non-SARGability unless rewritten).

### Multi-Row Return (Postgres SRF)
```sql
CREATE OR REPLACE FUNCTION active_customers()
RETURNS SETOF customers AS $$
  SELECT * FROM customers WHERE active;
$$ LANGUAGE sql STABLE;
```