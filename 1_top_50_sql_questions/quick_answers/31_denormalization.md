# 31. Denormalization & Trade-offs

Q: What is denormalization?
A: Intentionally reintroducing redundancy to reduce join cost and improve read performance.

Q: Techniques?
A: Pre-joining (wide tables), duplicating dimension attributes, summary/aggregate tables, snapshotting.

Q: Benefits?
A: Faster queries, simpler reporting, fewer joins under high concurrency.

Q: Costs?
A: Update anomalies, larger storage, more complex ETL (must keep copies in sync), risk of inconsistency.

Q: When justified?
A: High read:write ratio, latency SLAs unmet with normalized model, heavy aggregation patterns.

Q: Mitigation?
A: Materialized views, triggers or batch jobs to refresh, checksums for drift detection.

Q: Pitfall?
A: Premature denormalization before measuring real bottlenecks.
