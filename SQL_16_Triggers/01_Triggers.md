# Triggers (Overview)

## Overview
A trigger is a database callback that automatically executes in response to specified data definition (DDL) or data manipulation (DML) events (INSERT, UPDATE, DELETE) or occasionally system events (logon, schema change depending on engine). Triggers can enforce invariants, maintain derived data, audit changes, and integrate with external processes. They execute implicitly—powerful but potentially dangerous if misused.

## Core Concepts
- **Trigger Event**: Table or view operation (INSERT / UPDATE / DELETE); some engines support TRUNCATE, DDL (CREATE TABLE), or LOGON events (SQL Server, Oracle).
- **Timing**:
  - BEFORE / AFTER (Postgres, MySQL, Oracle)
  - INSTEAD OF (on views; intercepts operation and supplies custom logic)
- **Granularity**:
  - ROW-level: fires per affected row (access NEW/OLD record images)
  - STATEMENT-level: fires once per statement (SQL Server, Oracle; Postgres row triggers only; emulate statement via transition tables in PG v10+)
- **Transition Variables**: `NEW`, `OLD` (Postgres, MySQL), `inserted` / `deleted` pseudo tables (SQL Server), :NEW/:OLD (Oracle).
- **Mutation Constraints**: Some engines restrict reading the same table within trigger (Oracle mutating table error) to prevent inconsistent view.
- **Ordered Execution**: Multiple triggers—execution order may be unspecified (Postgres) or configurable (SQL Server `sp_settriggerorder`).
- **Determinism**: Hidden side-effects complicate debugging and performance prediction.

### Minimal Example (Postgres)
```sql
CREATE TABLE accounts (
  id BIGSERIAL PRIMARY KEY,
  balance NUMERIC(12,2) NOT NULL DEFAULT 0,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE OR REPLACE FUNCTION set_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at := now();
  RETURN NEW;
END;$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_accounts_set_updated
BEFORE UPDATE ON accounts
FOR EACH ROW
EXECUTE FUNCTION set_updated_at();
```

## Interview-Focused Notes
1. What is a trigger? Automatic code executed on defined DB event.
2. When to use? Auditing, denormalized column maintenance, enforcing cross-row constraints not expressible declaratively.
3. When NOT to use? Business logic better suited at application/service layer or logic causing performance unpredictability.
4. Row vs statement trigger difference? Row triggers fire per affected row; statement triggers run once per SQL statement.
5. View triggers (INSTEAD OF) usage? Allow updatable view semantics by custom logic mapping to base tables.

## Quick Recall ✅
- Hidden execution path—document thoroughly.
- Prefer declarative constraints over procedural enforcement.
- BEFORE triggers can modify NEW; AFTER cannot alter persisted row.
- INSTEAD OF triggers rewrite operations on views.
- Keep triggers short & set-based.

## Interview Traps & Confusions ⚠️
- Assuming trigger order if multiple exist (not guaranteed).
- Doing expensive remote calls inside high-frequency trigger.
- Recursive firing causing unintended loops.
- Using triggers for auditing when native CDC / temporal tables available.
- Complex branching logic hiding data issues.

## Bonus
### Transition Tables (Postgres 10+)
Use `REFERENCING NEW TABLE AS new_rows` to access set of affected rows statement-level style.

### Metadata Stamp Pattern
Maintain `updated_at`, `updated_by` automatically.

### Soft Delete Policy
Trigger intercepts DELETE → sets `deleted_at` while preventing physical removal.

### Denormalization Example
Maintain summary counters table inside AFTER trigger with UPSERT logic.
