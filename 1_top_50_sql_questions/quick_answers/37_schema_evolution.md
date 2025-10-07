# 37. Schema Evolution

Q: Adding a column safely?
A: Add nullable or with constant default (fast metadata if supported), backfill in batches, then enforce NOT NULL.

Q: Dropping a column?
A: Deprecate usage in code first; verify no dependencies (views, triggers), then DROP in maintenance window.

Q: Renaming?
A: Add new column + copy + swap or use RENAME where atomic rename exists—provide compatibility view if needed.

Q: Data type change?
A: Create new column with target type, transform & copy, switch references, drop old (reduces lock duration).

Q: Track migrations?
A: Versioned scripts table (e.g., schema_migrations) + checksum.

Q: Pitfall?
A: Long exclusive locks on huge tables—use online DDL / partition swap or rolling deployment.
