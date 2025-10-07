# 9. JOIN Types

Q: INNER JOIN?
A: Returns rows with matching keys in both tables.
```sql
SELECT o.id, c.name FROM orders o INNER JOIN customers c ON o.customer_id = c.id;
```

Q: LEFT (OUTER) JOIN?
A: All left rows + matching right rows; non-matches get NULLs on right.
```sql
SELECT c.id, o.id FROM customers c LEFT JOIN orders o ON o.customer_id = c.id;
```

Q: RIGHT (OUTER) JOIN?
A: Mirror of LEFT; prefer rewriting as LEFT for readability.

Q: FULL OUTER JOIN?
A: All rows from both sides; unmatched sides filled with NULLs. (Not in MySQL pre-8.0; emulate with UNION of left + right anti parts.)
```sql
SELECT * FROM a FULL OUTER JOIN b ON a.id = b.id; -- Postgres / SQL Server / Oracle
```

Q: CROSS JOIN?
A: Cartesian product (every combination); implicit via comma (avoid unless intentional).

Pitfall: Forgetting join predicate causes accidental CROSS JOIN (explosion of rows).