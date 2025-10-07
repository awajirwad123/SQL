# 18. Running Totals & Moving Averages

Q: Running total pattern?
A: Use window SUM with ORDER BY and frame.
```sql
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date
                         ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total
FROM sales;
```

Q: Moving average (last 7 rows)?
```sql
SELECT order_date, amount,
       AVG(amount) OVER (ORDER BY order_date
                         ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS ma_7
FROM sales;
```

Q: Moving average by partition (per customer)?
```sql
AVG(amount) OVER (PARTITION BY customer_id ORDER BY order_date
                  ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)
```

Q: ROWS vs RANGE?
A: ROWS counts physical rows; RANGE groups peers with same ORDER BY value (can widen frame unexpectedly).

Q: Pitfall?
A: Omitting explicit frame leads to vendor default (often RANGE UNBOUNDED PRECEDING) altering expected running sums with duplicate ordering values.
