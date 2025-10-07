# 38. OLTP vs OLAP

Q: OLTP purpose?
A: Support high-volume transactional operations (insert/update-focused, low latency per row).

Q: OLAP purpose?
A: Analytical queries scanning large data sets for aggregates and trends.

Q: Schema design?
A: OLTP: Highly normalized for integrity. OLAP: Denormalized/star for scan efficiency.

Q: Workload pattern?
A: OLTP: Many small reads/writes. OLAP: Fewer long-running aggregations.

Q: Indexing/storage?
A: OLTP: B-Tree row store. OLAP: Columnar storage, compression, bitmap indexes.

Q: Concurrency?
A: OLTP: Strict ACID + fine-grained locking/MVCC. OLAP: Batch loads, snapshot isolation sufficient.

Q: Pitfall?
A: Using OLTP schema directly for BI reporting → slow, complex queries.
