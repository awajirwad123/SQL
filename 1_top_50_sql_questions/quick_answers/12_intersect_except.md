# 12. INTERSECT and EXCEPT / MINUS

Q: INTERSECT?
A: Returns rows common to both queries (set intersection, distinct by default).

Q: EXCEPT (Postgres/SQL Server) / MINUS (Oracle)?
A: Returns rows in first query not in second (set difference, distinct by default).

Q: Example:
```sql
-- Customers who bought product A AND product B
SELECT customer_id FROM sales WHERE product_id = 'A'
INTERSECT
SELECT customer_id FROM sales WHERE product_id = 'B';

-- Customers who bought A but NOT B
SELECT customer_id FROM sales WHERE product_id = 'A'
EXCEPT
SELECT customer_id FROM sales WHERE product_id = 'B'; -- MINUS in Oracle
```

Q: All-rows version?
A: Use joins or EXISTS patterns; set operators inherently apply DISTINCT (unless vendor supports ALL variants).

Q: Pitfall?
A: Column counts/types/order must align exactly between both queries.
