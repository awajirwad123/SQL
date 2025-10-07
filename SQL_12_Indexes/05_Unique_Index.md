# Unique Index

## Overview
A unique index enforces that the indexed key values are distinct across rows. It supports data integrity and can optimize lookups. Although uniqueness is often enforced via constraints (`PRIMARY KEY`, `UNIQUE` constraint), the underlying mechanism is typically a unique index.

## Core Concepts
- **Constraint vs Index**: A `UNIQUE` constraint is a logical rule; it *creates* (or uses) a unique index to enforce. A unique index alone enforces distinctness but lacks constraint semantics metadata (naming for documentation, dependency usage, information_schema clarity).
- **NULL Semantics**:
  - PostgreSQL / Oracle / SQL Server: Multiple NULLs allowed (NULL ≠ NULL).
  - MySQL (InnoDB): Allows one NULL per unique index historically; modern MySQL generally allows multiple NULLs (depends on version / configuration). Clarify in interviews.
- **Composite Unique**: Uniqueness across column combination: `UNIQUE (email, tenant_id)`.
- **Partial Unique (Postgres)**:
  ```sql
  CREATE UNIQUE INDEX uq_active_username ON users (username) WHERE active = TRUE;
  ```
  Enforces uniqueness only among active users.
- **Case-Insensitive Unique**:
  ```sql
  CREATE UNIQUE INDEX uq_lower_email ON users (LOWER(email));
  ```
- **Deferred (Postgres)**: Unique constraint can be DEFERRABLE; unique index alone cannot.

## Interview-Focused Notes
1. **"Why prefer UNIQUE constraint over unique index?"** – Expresses intent semantically, integrates with schema metadata and tooling.
2. **"Multiple NULLs allowed?"** – Yes in most systems (NULL not equal). MySQL nuance—discuss version awareness.
3. **"Why composite unique?"** – Multi-tenant or scoping uniqueness to natural key plus domain dimension.
4. **"Alternative for case-insensitive uniqueness?"** – Functional unique index on LOWER(...) or use case-insensitive collation / CITEXT.
5. **"Partial uniqueness use case?"** – Soft deletes or active-state enforcement without penalizing archived rows.

## Quick Recall ✅
- Unique index enforces distinct key values.
- Constraints add semantic clarity + deferrable option.
- NULL behavior differs by RDBMS.
- Functional & partial unique indexes solve nuanced business rules.
- Primary key = unique + not null automatically.

## Interview Traps & Confusions ⚠️
- Assuming unique index == unique constraint (conceptually close but constraints surface better in logical model).
- Forgetting to handle casing → duplicate logical emails.
- Believing uniqueness across individually unique columns ensures row uniqueness (needs composite if combined meaning).
- Relying on application checks instead of database enforcement (race conditions).
- Not considering performance cost (unique check adds lookup per insert/update).

## Bonus
### Enforce Multi-Tenant Email Uniqueness
```sql
CREATE UNIQUE INDEX uq_tenant_email ON users (tenant_id, LOWER(email));
```

### Soft Delete Pattern
```sql
CREATE UNIQUE INDEX uq_active_slug ON articles (LOWER(slug)) WHERE deleted_at IS NULL;
```

### Deferrable Unique Constraint (Postgres)
```sql
ALTER TABLE accounts
  ADD CONSTRAINT uq_accounts_name UNIQUE (account_name) DEFERRABLE INITIALLY DEFERRED;
```
(Useful for batch re-key operations.)

### Detect Near-Duplicates
Supplement unique with normalized fingerprint column (e.g., whitespace-collapsed).

### Handling Race Condition
Rely on unique violation exception; retry logic in application.
