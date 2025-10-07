# 13. Correlated Subquery

Q: What is it?
A: A subquery that references columns of the outer query; re-evaluated per outer row.

Q: Example (latest order date per customer):
```sql
SELECT c.id, c.name,
       (SELECT MAX(o.order_date)
        FROM orders o
        WHERE o.customer_id = c.id) AS last_order
FROM customers c;
```

Q: Performance note?
A: May become N+1 lookups; optimizer can sometimes decorrelate to a join.

Q: Alternative?
A: JOIN + GROUP BY or window functions.

Q: Pitfall?
A: Forgetting correlation predicate → subquery returns multi-row error (scalar context) or incorrect global value.
