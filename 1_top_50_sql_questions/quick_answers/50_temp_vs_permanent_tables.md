# 50. Temporary vs Permanent Tables

Q: Temporary table?
A: Session- or transaction-scoped table (e.g., TEMP, #temp) automatically dropped at end of scope.

Q: Permanent table?
A: Persisting object until explicitly dropped.

Use Cases (Temp):
- Break complex transformations into stages.
- Hold intermediate subsets for reuse in a single job.
- Avoid recomputation of expensive subqueries.

Performance Considerations:
- Temp tables may reside in memory (small) then spill to disk; fewer logging considerations in some engines.
- Overuse can thrash tempdb/system catalogs—keep minimal.

Alternatives:
- CTEs for one-off readability (no reuse across statements).
- Inline views / subqueries for single-use logic.

Pitfall: Relying on temp tables for every step—may increase I/O vs a single well-structured set-based query.
