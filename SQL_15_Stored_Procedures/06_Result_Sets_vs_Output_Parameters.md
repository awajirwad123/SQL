# Result Sets vs Output Parameters

## Overview
Procedures can return data via result sets, output (OUT) parameters, return codes, or combinations. Choosing the right mechanism affects client API simplicity, performance, and flexibility.

## Core Concepts
- **Result Set**: Tabular output consumed like a query; flexible column structure.
- **Output Parameter**: Scalar values returned through declared OUT/OUTPUT parameter placeholders.
- **Return Code**: Single integer (legacy status); limited expressiveness.
- **Multiple Result Sets**: Some drivers support sequential retrieval (SQL Server, MySQL). Adds complexity for ORMs.
- **Set vs Scalar**: Use result set for collections; output parameter for metadata (row count, status, generated id).
- **Streaming vs Buffering**: Large result sets benefit from streaming retrieval; ensure procedure avoids unnecessary temp staging.

### SQL Server Example
```sql
CREATE OR ALTER PROCEDURE dbo.FetchSummary
  @CustomerId INT,
  @TotalOrders INT OUTPUT
AS
BEGIN
  SET NOCOUNT ON;
  SELECT order_id, total_amount FROM orders WHERE customer_id=@CustomerId;
  SELECT @TotalOrders = COUNT(*) FROM orders WHERE customer_id=@CustomerId;
END;
```

### MySQL OUT Parameter Call
```sql
CALL FetchSummary(42, @total);
SELECT @total;
```

## Interview-Focused Notes
1. When prefer OUT param? Auxiliary scalar info alongside main result set.
2. Multiple result sets issues? ORM/tooling mismatch, harder caching, coupling order semantics.
3. Avoid return codes? Low expressiveness—prefer raising exceptions for error states.
4. Get generated keys? Use RETURNING clause (Postgres) or OUTPUT (SQL Server) vs separate OUT param if possible.
5. Single responsibility? Procedure should not mix unrelated result sets (confuses consumers).

## Quick Recall ✅
- Result set for primary data.
- OUT for supplemental scalar.
- Minimal multiple result sets.
- RETURN code deprecated style.
- Use RETURNING/OUTPUT for inserted keys.

## Interview Traps & Confusions ⚠️
- Overloading procedure with variable shape outputs.
- Relying on result set column order without naming.
- Using OUT params for lists (awkward; use set output).
- Returning huge unfiltered sets (lack of pagination/cursor logic).
- Hiding errors with status codes instead of exceptions.

## Bonus
### Encapsulate Metadata
Return JSON summary as second lightweight result (if client expects structured doc) instead of many OUT params.

### Single Result Normalization
Wrap scalar as one-row result set to simplify uniform client handling.

### Cursor-Based Streaming (If Needed)
Open and fetch controlled batches; but prefer set queries with pagination.

### Tooling Compatibility
Confirm ORM support for multiple result sets before designing API.
