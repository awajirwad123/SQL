# SQL Views Module (SQL_11_Views)

## Contents
1. `01_CREATE_VIEW.md` – Defining standard & materialized views, CHECK OPTION, security, schemabinding.
2. `02_UPDATE_VIEW.md` – Altering definitions & updating through updatable views.
3. `03_RENAME_VIEW.md` – Dialect-specific rename strategies & dependency safety.
4. `04_DROP_VIEW.md` – Safe removal, IF EXISTS, CASCADE vs RESTRICT, dependency audits.

## Purpose
Provide fast interview-ready coverage of view lifecycle: creation, evolution, renaming, and decommissioning—plus rules governing updatability & security usage.

## When to Use Views
- Encapsulate complex joins/filters.
- Limit exposed columns for least-privilege access.
- Provide stable interface while refactoring physical schema.
- Pre-stage logic for BI / reporting (possibly materialize).

## When NOT to Use
- As a substitute for proper schema normalization.
- Layering dozens of dependent views (plan bloat / performance).
- Performance caching (unless materialized / indexed view variant).

## Key Concepts Snapshot
| Topic | Key Point |
|-------|-----------|
| Updatable | Single base table + simple projection |
| CHECK OPTION | Enforce predicate for DML through view |
| Materialized View | Stores data; needs refresh strategy |
| Security | Grant on view; restrict base tables |
| Renaming | Dialect syntax differs widely |
| Dropping | Use IF EXISTS & dependency checks |

## Practice Prompts
1. Create a view limiting columns & rows; enforce predicate with CHECK OPTION.
2. Evolve a view to add derived column without breaking consumers.
3. Determine whether a view is updatable in two different RDBMS.
4. Safely rename a production view with dependency analysis.
5. Replace a slow ad hoc query with a materialized view + refresh cadence.

## Next Suggested Module
- Indexing Strategies
- Window Functions Advanced
- Query Optimization & Execution Plans

Use this module with earlier Joins/Constraints/Functions to round out schema + abstraction fundamentals.
