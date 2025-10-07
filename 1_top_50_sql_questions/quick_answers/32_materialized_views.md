# 32. Materialized Views

Q: What is a materialized view?
A: A precomputed, stored result of a query (unlike a logical view which is just a saved SELECT).

Q: Why use?
A: Speed up expensive joins/aggregations and provide consistent snapshot for reporting.

Q: Refresh strategies?
A: On demand (REFRESH MATERIALIZED VIEW), scheduled, incremental (fast refresh if logs/change tracking available), or automatic (some systems).

Q: Use case example?
A: Daily sales per product aggregated from billions of fact rows.

Q: Trade-offs?
A: Storage cost, staleness window, refresh overhead/contention.

Q: Pattern for query rewrite?
A: Some optimizers auto-rewrite queries to use matching materialized view; others need explicit reference.

Q: Pitfall?
A: Choosing too granular MV—refresh cost rivals original query.
