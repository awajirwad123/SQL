# 7. Table Lifecycle (Create / Alter / Drop)

Q: CREATE essentials?
A: Choose precise types, define PK early, add NOT NULL + constraints up front.

Q: ALTER add column?
A: ALTER TABLE t ADD COLUMN col TYPE [DEFAULT ...];

Q: Safely add NOT NULL column?
A: Add nullable → backfill → enforce NOT NULL → set DEFAULT.

Q: TRUNCATE vs DROP?
A: TRUNCATE removes all rows fast, keeps structure; DROP removes structure + data.

Q: Change column type?
A: ALTER TABLE t ALTER COLUMN col TYPE new_type; (may rewrite table).

Q: Migration pitfall?
A: Long blocking ALTER on huge table—use phased/online strategies.
