# 34. Building an ETL Using SQL

Q: Core phases of ETL?
A: Extract (ingest raw), Transform (clean, conform, enrich), Load (persist to target model).

Q: Typical SQL layer staging pattern?
A: Landing (raw append) -> Staging (typed/cleaned) -> Dim/Facts (modeled) -> Marts (aggregates).

Q: Key practices?
A:
- Idempotent loads (truncate+insert or MERGE on business keys).
- Use surrogate keys for slowly changing dimensions.
- Audit columns: created_at, updated_at, batch_id, source_file.
- Incremental loads using high-water mark (e.g., max updated_at tracked).

Q: Error handling?
A: Separate reject table for bad rows with reason; continue processing clean subset.

Q: Data quality checks?
A: Row counts, nullability, referential integrity, duplicate detection, numeric range checks.

Q: Performance tips?
A: Disable non-essential indexes during bulk load; use set-based operations (single INSERT...SELECT) not row-at-a-time; partition large fact tables.
