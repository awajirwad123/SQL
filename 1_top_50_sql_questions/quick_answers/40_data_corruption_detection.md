# 40. Data Corruption Detection & Handling

Q: What is data corruption?
A: Logical or physical data inconsistency (bad values, impossible states, unreadable pages) vs intended model.

Detection Techniques:
- Checksums/page verification (DB engine settings: e.g., DBCC CHECKDB, CHECKSUM_PAGE, ANALYZE warnings).
- Constraint failures (attempted inserts/updates rejected indicate upstream corruption source).
- Profiling queries: range/ domain violations (negative salary, future birthdate).
- Referential integrity mismatch counts vs expected cardinalities.
- Hash total comparisons between source and target batches.

Handling:
- Restore from last known good backup then replay logs.
- Use quarantine tables for suspect rows (INSERT ... WHERE validation fails).
- Rebuild indexes if structural only.
- Recompute derived columns from canonical sources.

Pitfall: Silent truncation on type change—always diff row counts + checksum before/after migrations.