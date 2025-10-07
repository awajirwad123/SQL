# 36. Batch Inserts / Updates / Upserts

Q: Bulk insert pattern?
A: Single INSERT...SELECT or multi-row VALUES.
```sql
INSERT INTO target(col1, col2)
SELECT col1, col2 FROM staging;
```

Q: Upsert (Postgres)?
A: `INSERT ... ON CONFLICT (key) DO UPDATE SET ...`.
```sql
INSERT INTO users(id, email, name)
VALUES (1, 'a@x.com', 'Alice')
ON CONFLICT (id) DO UPDATE SET email = EXCLUDED.email;
```

Q: MySQL upsert?
A: `INSERT ... ON DUPLICATE KEY UPDATE col=VALUES(col);`

Q: MERGE (ANSI / SQL Server / Oracle)?
A: Single statement match + insert/update/delete.
```sql
MERGE INTO dim_product d
USING staging s ON d.sku = s.sku
WHEN MATCHED THEN UPDATE SET name = s.name
WHEN NOT MATCHED THEN INSERT (sku, name) VALUES (s.sku, s.name);
```

Q: Incremental load?
A: Filter source by last watermark (e.g., updated_at > last_max).

Q: Pitfall?
A: Race conditions causing lost updates—add WHERE with original hash/version for optimistic concurrency.
