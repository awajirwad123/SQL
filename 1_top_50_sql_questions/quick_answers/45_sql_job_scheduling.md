# 45. Automating / Scheduling SQL Jobs

Q: How to schedule SQL jobs?
A: Use built-in schedulers (SQL Server Agent, Postgres cron extension, MySQL Event Scheduler) or external orchestrators (Airflow, cron, dbt, Prefect).

Q: Core components?
A: Task definition (query/script), schedule (cron-like), logging, alerting on failure, idempotence.

Q: Example (Postgres pg_cron):
```sql
SELECT cron.schedule('daily_sales', '0 2 * * *', $$CALL refresh_daily_sales()$$);
```

Q: Idempotence strategy?
A: Use MERGE/upsert patterns, truncate+reload staging, track high-water marks.

Q: Failure handling?
A: Retry with exponential backoff, escalate after N failures, persist error metadata.

Q: Pitfall?
A: Overlapping runs—implement a lock flag (advisory lock) or skip if previous still active.
